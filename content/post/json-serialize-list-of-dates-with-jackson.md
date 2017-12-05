+++
date = "2017-10-02T10:50:35+03:00"
draft = false
title = "Serialize list of dates into JSON with Jackson"

+++

There are a lot of examples about how to serialize `Date` objects into JSON. 
But I was not able to find examples that explain how to serialize a collection of dates: `Collection<Date>`. 
I had to dive into code and find out it myself. In this post I share my findings with you. 

<!--more-->

## Problem

Let's imagine that we need to return JSON form of the following Java object: 

```
import java.util.Date;

public class A {
    private String name;
    private List<Date> dates;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Date> getDates() {
        return dates;
    }

    public void setDates(List<Date> dates) {
        this.dates = dates;
    }
}
```

If we return this class 'as is', without any helping directives to JSON serializer, we could get it in the following form:

```
{
  "name": "aaa",
  "dates": [1506556800000,1506643200000,1506729600000,1506816000000]
}
```

Rather than that we would like to have it in the following form (format dates as 'yyyy-MM-dd' rather than take is as epoch timestamp):

```
{
  "name": "aaa",
  "dates": ["2017-09-27","2017-09-28","2017-09-29","2017-09-30"]
}
```

## Solution

To achieve this formatted response, we need to use 
[com.fasterxml.jackson.databind.ser.std.CollectionSerializer](https://github.com/FasterXML/jackson-databind/blob/master/src/main/java/com/fasterxml/jackson/databind/ser/std/CollectionSerializer.java) serializer:

```
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.ser.std.CollectionSerializer;
import com.fasterxml.jackson.databind.type.SimpleType;

import java.util.Date;

public class JsonListOfDatesSerializer extends CollectionSerializer {

    public JsonListOfDatesSerializer() {
        super(SimpleType.constructUnsafe(Date.class), true, null, (JsonSerializer)new JsonDateSerializer());
    }
}
```

This serializer defines how to serilialize a list of dates, applying *JsonDateSerializer* to each date. 
*JsonDateSerializer* has following definition: 

```
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

import java.text.SimpleDateFormat;
import java.io.IOException;
import java.util.Date;

public class JsonDateSerializer extends JsonSerializer<Date> {

    @Override
    public void serialize(Date value, JsonGenerator jgen,
                          SerializerProvider provider) throws IOException {
        String formattedDate = new SimpleDateFormat("yyyy-MM-dd").format(value);
        jgen.writeString(formattedDate);
    }
}
```

## Another serializers suitable for Collections 

Jackson also has another serializaers that can be extended to conveniently serialize collections of objects:

* com.fasterxml.jackson.databind.ser.std.IterableSerializer
* com.fasterxml.jackson.databind.ser.impl.IndexedListSerializer
* com.fasterxml.jackson.databind.ser.std.MapSerializer

