---
layout: post
title: '스프링부트 - @ResponseBody 커스터마이징 Jascskon Converter'
date: '2016-12-14 20:30:01 -0500'
categories: SPRING
fb_title: messageconverter
---

스프링에서 @ResponseBody로 JSON 응답을 줄 때, Jackson Converter를 커스터마이징 하는 방법에 대해서 포스팅 합니다.

# 시작하며

API 서버를 만들때, 특정 객체를 JSON 객체로 바꾸어 응답을 줘야합니다.
스프링을 사용하고 있다면 일반적으로 Jascskon Converter를 이용해서 자바 객체를 JSON 객체로 만들어 주게됩니다.

API 서버를 만들던 중 JodaTime의 DateTime 필드를 JSON 객체로 만들어 주는데 DateTime객체 자체가 JSON 필드로 바뀌는 문제가 발생했습니다.

제가 원한것은 timestamp 값만 JSON 객체로 만들어 주기를 원했는데... 원하던 타입이 아니었습니다.

이번 포스팅에서는 이와 같이 특정 필드를 본인이 원하는 JSON 필드 값으로 세팅해주는 방법에 대해서 포스팅하고자 합니다.



# 여러가지 방법이 있더군요...

``` java
import org.joda.time.DateTime;

...

@Convert(converter = LocalDateTimeConverter.class)
@JsonProperty
private DateTime createDate = DateTime.now();


```

위의 코드의 DateTime 필드를 포함한 객체를 API로 호출해보니 아래와 같은 결과가 나왔습니다.

![jacson_problem](/images/jackson_problem.jpg)

> 내가 원하는건 오직 millies 필드일 뿐! (이렇게 많은 필드는 필요없어요...)

이유를 찾아보니 DateTime 객체 자체가 Jascskon Conveter에 의해서 JSON 객체로 바뀌게 되어 DateTime 객체가 JSON 객체로 바뀌어 결과값을 보여주고 있었습니다.

저는 API 호출시 단지 millies 필드만을 필요로 했기 때문에 단지 시간을 알기 위해 이렇게
까지 많은 필드는 필요하지 않았습니다.

그래서 첫번째로 생각한 방법은 timestamp만을 가지는 필드를 만드는 것이 었습니다.

## 방법 1) timestamp 필드를 만들고 DateTime 필드를 ignore 하자!

``` java

@Convert(converter = LocalDateTimeConverter.class)
@JsonIgnore
private DateTime createDate = DateTime.now();

...

public java.sql.Timestamp getTimestampCreateDate () {
    if (createDate == null) {
        return null;
    }

    return new java.sql.Timestamp(createDate.getMillis());
}
```

``` JSON

// 20161218225301
// http://localhost:9000//api/v1/questions

[
  {
    "id": 1,
    "writer": {

    },
    "title": "국내에서 Ruby on Rails와 Play가 ",
    "contents": "Ruby on Rails(이하 RoR)는",
    "timestampCreateDate": 1482069127028
  },
  {
    "id": 2,
    "writer": {

    },
    "title": "산지기가 쓴 글",
    "contents": "산지기는 군생활 때 나의 별명. 자바지기의 유래는 산지기",
    "timestampCreateDate": 1482069127029
  }
]

```

위의 코드에서 보는것과 같이 기본의 DateTime 필드에는 ``@JsonIgnore`` 어노테이션을 붙여서 더이상 JSON 객체로 바꾸지 않도록 해주었습니다. 그리고 필요로하는 필드인 timestamp 값을 getter로 만들어 주었습니다. (Jascskon Converter는 변형되는 자바 객체의 getter를 호출하는 방식으로 동작합니다.)

이렇게 했더니 제가 원하는 timestamp 필드를 얻을 수 있었습니다.

그러나 이 방법에는 문제점이 한가지 있었습니다. JSON으로 변형하고자 하는 객체에 DateTime 필드가 존재한다면 매 객체마다 위와 같이 getter를 만들어 주어야 한다는 번거로움이 존재하더군요...

따라서 한군데서만 설정을 해두면 해당필드에는 지속적으로 적용되는 방법을 찾아보았습니다.

## 방법 2) JacksonConverter를 커스터마이징 하자!

방법을 찾아보던 중 Jasckson Conveter를 커스터마이징 하는 방법이 있다는 것을 알게 되었습니다. 스프링의 `` @ResponseBody `` 어노테이션은 ``MappingJackson2HttpMessageConverter`` 에 의해서 JSON 객체로 컨버팅 되고 있는데 자바 config를 통해서 해당 Converter를 커스터마이징이 가능했습니다.

먼저 커스터마이징을 하기 전에 ``MappingJackson2HttpMessageConverter`` 가 어떤 방식으로 동작하는지 간단하게 말씀드려야 할 것 같습니다.

``MappingJackson2HttpMessageConverter``는 ObjectMapper 에 의해서 Javs 객체를 JSON 객체로 바꾸어 줍니다. 이 때 ObjectMapper 에는 SimpleModule을 등록해주어 어떤 식으로 JAVA객체를 JSON객체로 serialize 할지 또는 JSON객체를 JAVA 객체로 deserialize 할지를 등록 해 줄 수 있습니다.

``` java

public class DateTimeSerializer extends JsonSerializer<DateTime> {
    @Override
    public void serialize(DateTime value, JsonGenerator gen, SerializerProvider serializerProvider) throws IOException {
        // 원하는 timestamp 필드만을 serialize 한다.
        gen.writeNumber(value.getMillis());
    }
}

```

먼저 Json 객체로 Serialize 하기 위해서 JsonSerializer를 상속받은 DateTimeSerializer 객체를 만들어 주었습니다. serialize 할 때, timestamp 필드만 serialize 하도록 구현을 해두었습니다.

``` java

public class DateTimeModule extends SimpleModule {
    public DateTimeModule() {
        super();
        // 모듈에 Serializer를 등록해 준다.
        addSerializer(DateTime.class, new DateTimeSerializer());
    }
}

```

ObjectMapper에 등록할 DateTimeModule 을 생성했습니다. (SimpleModule을 상속 받았습니다.) 이 모듈에 위에서 생성한 DateTimeSerializer를 등록해주었습니다.


``` java

@Configuration
public class MessageConverterConfig {
    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
        MappingJackson2HttpMessageConverter jsonConverter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = new ObjectMapper();
        // 생성한 모듈을 등록해 준다.
        objectMapper.registerModule(new DateTimeModule());
        jsonConverter.setObjectMapper(objectMapper);

        return jsonConverter;
    }
}

```

마지막으로 사용되는 converter의 objectMapper에 생성한 모듈을 등록해 주었습니다.
이렇게 등록을 해두면 DateTime 필드에 대해서는 제가 커스터마이징한 모듈이 동작하여 모든 객체에 동일하게 DateTime 필드를 Timestamp 필드로 Serialize 해주게 될 것입니다.

``` JSON

// 20161218231029
// http://localhost:9000//api/v1/questions

[
  {
    "id": 1,
    "writer": {

    },
    "title": "국내에서 Ruby on Rails와 Play가 ",
    "contents": "Ruby on Rails(이하 RoR)는",
    "createDate": 1482070227056,
    "timestampCreateDate": 1482070227056
  },
  {
    "id": 2,
    "writer": {

    },
    "title": "산지기가 쓴 글",
    "contents": "산지기는 군생활 때 나의 별명. 자바지기의 유래는 산지기",
    "createDate": 1482070227056,
    "timestampCreateDate": 1482070227056
  }
]

```

다시 호출해보니 createDate 필드가 제가 원하는데로 timestamp 필드만을 json 객체로 바꾸어 준 것을 보실 수 있습니다.

# 마치며

사실 위 문제는 신입분이 프로젝트를 진행하면서 저에게 질문을 했던 문제입니다...
그 당시 저도 원인을 몰라서 명확한 해결 방법을 말씀드리지 못했습니다.

그래서 주말에 API 서버를 만들어 보며 비슷한 상황을 연출해 보았고 그 원인을 알게 되어 이렇게 포스팅을 하게 되었습니다.

신입분들이 들어오시고 알려드리고 싶은건 많은데 아직 부족한게 너무 많아 제대로 도움을 못드리는것 같아 창피하기도하고 미안하기도 합니다.

앞으로 이러한 상황이 있으면 계속적으로 포스팅을 하여 잊어버리지 않도록 기록으로 남겨두어야겠다는 생각을 했습니다. (빨리 성장해서 좋은 선배가 되었으면 좋겠습니다...)
