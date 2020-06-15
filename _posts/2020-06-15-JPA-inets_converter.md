---
layout: post
title: JPA Converter for mysql inet_aton
tags: [JAVA, Spring, JPA]
categories:
    - Java
    - Spring
    - JPA
    - Hibernate
permalink: jpa-inets-converter.html
createat: 2020-06-15
thumbnail: 
---
mysql inet_aton 을 이용하면 문자열로 된 ip 주소를 정수로 컬럼에 저장할 수 있습니다.  
그리고 이를 통해 저장용량이나 검색속도에서 이득을 볼 수 있습니다.  

처음에 mybatis 를 이용한 프로젝트에서는 쿼리를 일일히 작성했기 떄문에 mysql 의 inet_aton과 inet_ntoa 함수를 바로 적용할 수 있지만,  
jpa 의 경우에는 쿼리를 바로 입력하지 않기 때문에 Entity 클래스에 type converter 를 어노테이션을 통해 등록해 주어야 합니다.
이렇게 Entity 에 converter 를 등록하면 별도로 작업할 필요 없이 DB 컬럼에는 정수로, Entity 에서는 String 으로 관리할 수 있어 좋습니다.  

### Converter example

```java
import javax.persistence.AttributeConverter;
import javax.persistence.Converter;

@Converter
public class InetsConverter implements AttributeConverter<String, Long> {

    public long ipToLong(String ipAddress) {

        String[] ipAddressInArray = ipAddress.split("\\.");

        long result = 0;
        for (int i = 0; i < ipAddressInArray.length; i++) {

            long power = 3 - i;
            long ip = Long.parseLong(ipAddressInArray[i]);
            result += ip * Math.pow(256, power);

        }

        return result;
    }

    public String longToIp(long ip) {

        return ((ip >> 24) & 0xFF) + "."
                + ((ip >> 16) & 0xFF) + "."
                + ((ip >> 8) & 0xFF) + "."
                + (ip & 0xFF);

    }

    @Override
    public Long convertToDatabaseColumn(String s) {
        return ipToLong(s);
    }

    @Override
    public String convertToEntityAttribute(Long aLong) {
        return longToIp(aLong);
    }
}
```
우선 @Converter 어노테이션을 통해 Converter 임을 알려 줍니다.  
그리고 Entity 이용할 컨버터들은 AttributeConverter 를 구현 (implements) 하여 사용 하는데, 
제내릭으로는 각각 Entity 에서 사용하는 타입과 Database 에서 사용할 타입을 순서 대로 넣어 줍니다.  

AttributeConverter 을 구현하기 위해서는  
convertToDatabaseColumn 메서드 즉, Entity 에서 Database 로 바꿔주는 함수와  
convertToEntityAttribute 메서드, Database에서 Entity로 나올 때 변환 하는 함수를 각각 정의해 주면 됩니다.
여기서는 ip를 정수로 변환하는 ipToLong함수와 반대로 정수를 ip로 변환하는 longToIp 두가지 메서드를 각각 리턴 했습니다.  

### Entity 예제

```java
@Entity
public class EntityExample {
    @Id
    private Long id;

    @Convert(converter=InetsConverter.class)
    private String ipAddress;
    // 생략
}
```
위에서 만든 converter 를 Entity에 등록합니다.  
@Convert 어노테이션에 converter 를 해당 클래스로 지정해 주면 됩니다.  