---
layout: post
comments : true
tags : spring, java
title: Jackson Object Mapper를 이용한 Date String 파싱 방법
---

Spring에서 json을 파싱하게 될 경우, json to object 로 매핑 하기 위해 Jackson을 사용한다.
String, int, long 등 기본적인 타입은 바로 매핑이 되지만,
지원하지 않는 클래스(Date, Enum등)는 Deserializer (반대의 경우에는 Serializer) 를 구현해 주어야 한다.

## 1. Jackson Version이 v2.0이상 일 경우

- 단순하게 annotation으로 처리가 가능하다.
{% highlight java %}
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm a z")
private Date date;
{% endhighlight %}
- Jackson 2.0부터는 package도 변경되었다.
{% highlight xml %}
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
<dependency>
<groupId>com.fasterxml.jackson.core</groupId>
<artifactId>jackson-core</artifactId>
<version>2.8.5</version>
</dependency>
{% endhighlight %}


## 2. Jackson Version이 v2.0이하 일 경우

- JsonDeserializer를 상속 받아 String to Date Deserializer를 만들어 준다.
{% highlight java %}
public class DateDeserializer extends JsonDeserializer<Date> {
    @Override
    public Date deserialize(JsonParser jsonparser, DeserializationContext context) throws IOException, JsonProcessingException {
        String dateAsString = jsonparser.getText();
        try {
	    return DateUtils.parseDate(dateAsString,DATE_FORMAT);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}
{% endhighlight %}
- 구현한 Deserializer를 이용해 mapping할 Date property에 Annotation을 추가해준다.
{% highlight java %}
@JsonDeserialize(using = DateDeserializer.class)
private Date date;
{% endhighlight %}
- 1.9.13 버전의 Jackson
{% highlight xml %}
<!-- https://mvnrepository.com/artifact/org.codehaus.jackson/jackson-mapper-asl -->
<dependency>
<groupId>org.codehaus.jackson</groupId>
<artifactId>jackson-mapper-asl</artifactId>
<version>1.9.13</version>
</dependency>
{% endhighlight %}
