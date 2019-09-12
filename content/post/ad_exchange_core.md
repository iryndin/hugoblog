+++
Tags = ["misc"]
date = "2019-09-11T02:55:26Z"
title = "Ad Exchange core implementation"
+++

Here is the task: implement the core of ad aggregator, abstracting away from network details. 
Just core functionality: receive requests, send them to partners, receive responses, 
select response with max price and notify partner that its response is selected as a winner. 
And do it all in a thread-safe non-blocking manner.

<!--more-->

## Scenario

1. Exchange receive a request from a user
2. Request is sent to the partners
3. Partners send back responses with their prices
4. Max price is selected, and partner is notified that he is a winner. 

There is a timeout we set to receive partner responses. 

## Key interfaces

### DemandPartner

Here is contract for `DemandPartner`:

```java
/**
 * This is a bidder with the networking abstracted away.
 **/
public interface DemandPartner {
	/**
	 * Called from the exchange to inform the demand partner of an auction
	 *
	 * @param bidRequest
	 * @return
	 */
	BidResponse processRequest(BidRequest bidRequest);

	/**
	 * Called when the partner wins the auction
	 *
	 * @param notify
	 */
	void processWin(Notify notify);
}
```

And here is implementation of `RandomDemandPartner` that just generates random response and random sleep time 
to simulate real world behaviour:


```java
public class RandomDemandPartner implements DemandPartner {

    private final static double ALMOST_ZERO_RATIO = 1e-4;
    private final static double ALMOST_INFINITY_RATIO = 1e4;
    private final static int MIN_BIDPRICE = 1;
    private final static int MAX_BIDPRICE = 100;

    private final Random random = new SecureRandom();

    private final int minBidPrice;
    private final int maxBidPrice;
    private final int minSleepMillis;
    private final int maxSleepMillis;

    public RandomDemandPartner(double intimeToTimeoutRatio, long timeoutMillis) {
        this(MIN_BIDPRICE, MAX_BIDPRICE, intimeToTimeoutRatio, timeoutMillis);
    }

    public RandomDemandPartner(int minBidPrice, int maxBidPrice, double intimeToTimeoutRatio, long timeoutMillis) {
        this.minBidPrice = minBidPrice;
        this.maxBidPrice = maxBidPrice;
        int[] sleepTimes = calculateMinMaxSleepMillis(intimeToTimeoutRatio, timeoutMillis);
        this.minSleepMillis = sleepTimes[0];
        this.maxSleepMillis = sleepTimes[1];
    }

    // ...

    @Override
    public BidResponse processRequest(BidRequest bidRequest) {
        final long bidPrice = minBidPrice + random.nextInt(maxBidPrice-minBidPrice);
        final long sleepMillis = minSleepMillis + random.nextInt(maxSleepMillis - minSleepMillis);

        try {
            Thread.sleep(sleepMillis);
        } catch (InterruptedException e) {
            //e.printStackTrace();
        }
        return new BidResponse(bidPrice);
    }

    @Override
    public void processWin(Notify notify) {
        // do nothing here
    }
}
```   
 
### Ad Exchange

Here is an interface for `Exchange`:

```java

/**
 * A simple exchange server. 
 * Publishers will call processBidRequest when they have inventory they want to auction on the exchange
 */
public interface Exchange extends AutoCloseable {
	/**
	 *
	 * @param bidRequest
	 *            a bid request to send to all the demand partners
	 */
	void processBidRequest(BidRequest bidRequest);
}
```

## Exchange implementation 

### Forward request to partners with timeout

We need to send request to each partner in parallel, then wait for responses not more than a predefined timeout, 
then select and return response with maximum price. We are going to use `invokeAll` method of `ExecutorService`, 
its timeout version. 

Here is the implementation of method:

```java
private Optional<ImmutablePair<DemandPartner, BidResponse>> processBidRequestBlockingWay(BidRequest bidRequest) {
    // 1. Create callables for each partner
    List<DemandPartnerCallable> demandPartnerResults = partners.stream()
            .map(dp -> new DemandPartnerCallable(dp, bidRequest))
            .collect(Collectors.toList());
    try {
        // 2. Send requests to each partner
        List<Future<ImmutablePair<DemandPartner, BidResponse>>> futures =
                demandPartnerCallsExecutorService.invokeAll(
                        demandPartnerResults, 
                        WAIT_TIMEOUT_MILLIS, 
                        TimeUnit.MILLISECONDS);
        // 3. Get results from responses and finally get maximum result
        return selectBestResponse(futures);
    } catch (InterruptedException e) {
        //e.printStackTrace();
        return Optional.empty();
    }
}

private Optional<ImmutablePair<DemandPartner, BidResponse>> selectBestResponse(List<Future<ImmutablePair<DemandPartner, BidResponse>>> futures) {
    return futures.stream()
        .map(f -> futureToResultOrNull(f))
        .filter(pair -> {
            if (pair == null) {
                exchangeStats.timeoutIncr();
                return false;
            } else {
                exchangeStats.intimeIncr();
                return true;
            }
        })
        .max(Comparator.comparingLong(p -> p.getSecond().getBidPrice()));
}
```

Some important points about this implementation:
 * call of `invokeAll` is a blocking call
 * we are within timeout of `WAIT_TIMEOUT_MILLIS`

Now let's see how do we process each incoming `BidRequest` in a non-blockng way. 

### Process incoming BidRequests in a non-blocking way

We simply do this using another executor:

```java
@Override
public void processBidRequest(BidRequest bidRequest) {
    bidRequestExecutorService.submit(() -> {
        Optional<ImmutablePair<DemandPartner, BidResponse>> result = processBidRequestBlockingWay(bidRequest);
        if (result.isPresent()) {
            ImmutablePair<DemandPartner, BidResponse> pair = result.get();
            pair.getFirst().processWin(new Notify(bidRequest.getRequestId(), pair.getSecond().getBidPrice()));
        }
    });
}
```

What if we do all this processing using only one executor? 
That wouldn't work because bid processing runnables would saturate request broadcasting runnables, 
hence we need here 2 executors.

## Number of threads in 2 executors

Most interesting question here is to consider what number of threads should contain `demandPartnerCallsExecutorService`
and `bidRequestExecutorService`. Let's find out this.

Since we have 20 `DemandPartners` in this test, it is logical that we should have 20 times more threads in 
`demandPartnerCallsExecutorService` then in `bidRequestExecutorService`. But the situation is a little bit more complex.
We should also take care about success-to-fail ratio, as well as intime-to-outtime ratio. 

Success case is the case when we were able to get at least one response from `DemandPartners` in time. 
I.e. if out of 20 `DemandPartner` responses 19 were timed out, and one was in time, then this case is considered to be success.
Obviously, failed case is the case when _all_ responses were timed out. 
Of course, the higher is `success-to-fail` ratio the better.

Next, what is `intime-to-outtime` ratio? This is the ratio of in-time `DemandPartner` responses to timed out ones. 
If we look again at `RandomDemandPartner` we can notice that we set this ratio in its constructor. 
It load test stats the closer is intime-to-outtime ratio to the value set in `RandomDemandPartner` the better. 

With this in mind, let's look at what we have for our load tests. They were done with following values:
 - 20 demand partners (i.e this means that for every incoming `BidRequest` we do 20 outcoming requests to `DemandPartners`) 
 - intime-to-outtime ratio = 1.0 (i.e. the same number of in-time and timed out responses from `DemandPartners`)
 - timeout = 50 millis
  
I used following ratios of bidrequest to demandpartner threads: 10,15,20.

Stats is following:

|\. | 32 | 64 | 96 | 128 | 160 | 192 |
|---|----|----|----|-----|-----|-----|
|10 |rps=592,<br/> success/fail=506,<br/> intime/timeout=0.41|rps=1123,<br/> success/fail=506,<br/> intime/timeout=0.43|rps=1063,<br/> success/fail=3124,<br/> intime/timeout=1.87|rps=1075,<br/> success/fail=37,<br/> intime/timeout=2.42|rps=1190,<br/> success/fail=7,<br/> intime/timeout=1.89|rps=1408,<br/> success/fail=2.5,<br/> intime/timeout=1.05|
|15 |rps=595,<br/> success/fail=Inf,<br/> intime/timeout=0.97|rps=1052,<br/> success/fail=9999,<br/> intime/timeout=1.17|rps=1041,<br/> success/fail=2499,<br/> intime/timeout=2.67|rps=1041,<br/> success/fail=32,<br/> intime/timeout=2.24|rps=1234,<br/> success/fail=4.3,<br/> intime/timeout=1.34|rps=1587,<br/> success/fail=1.6,<br/> intime/timeout=0.64|
|20 |rps=598,<br/> success/fail=5881,<br/> intime/timeout=1.03|rps=1075,<br/> success/fail=7141,<br/> intime/timeout=1.13|rps=1010,<br/> success/fail=4346,<br/> intime/timeout=2.53|rps=1052,<br/> success/fail=15,<br/> intime/timeout=1.79|rps=1333,<br/> success/fail=3,<br/> intime/timeout=0.90|rps=1754,<br/> success/fail=1.3,<br/> intime/timeout=0.45|

So, looking at this table we can select the best thread ratio. 
It is 64 threads in `bidRequestExecutorService`, and 64*15=960 threads in `demandPartnerCallsExecutorService`. 
With this configuration we have highest success-to-fail ratio (9999), 
fair intime-to-outtime ratio (1.17, close to 1.00) and good rps (1052).

It is also interesting to look at how decreases `success-to-fail` ratio while number of threads in 
`bidRequestExecutorService` grows. Indeed, we can't afford to have e.g. every 7-th or every 37-th request to fail 
in real-world system. Hence, even with considerably greater rps we do not like configs where `success-to-fail` ratio is low.

Summary is, that even in this case of pretty simple system that contains only 2 executors, its behaviour is non-linear at all. 
Only multiple experiments and tests can show you the best available configuration that gives this system 
performance (rps) together with acceptable other parameters like `success-to-fail` and `intime-to-outtime` ratios.

Code is here: https://github.com/iryndin/misc/tree/master/adexchange
