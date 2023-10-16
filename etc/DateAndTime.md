# UTC
- 시계에서 공통적으로 쓰이는 시간 표준.
- KST, GMT와 같이 현지 시간을 반영한 Time Zone과 달리 UTC는 현지 시간이 반영되지 않은게 특징이다. 

# LocalTime / LocalDate / LocalDatetime

- 시간대에 대한 정보가 전혀 없는 API이다.
- 따라서 한국에서 2018-09-07T08:00:04 -> 미국에서도 똑같.
- 생일과 같은 개념에 사용 가넝

# ZoneOffset

- UTC 기준으로 시간(Time Offset)을 나타낸 것
- 우리나라는 <b>KST</b>ㄹ를 사용하는데 KST는 UTC보다 9시간이 빨라 <b>UTC +09:00</b>로 표기한다.

### 다른 타임존 / 시차 적용
```java
ZonedDateTime seoul = Year.of(2002).atMonth(6).atDay(18).atTime(20, 30).atZone(ZoneId.of("Asia/Seoul"));
System.out.println("Seoul =     " + seoul);

ZonedDateTime utc = seoul.withZoneSameInstant(ZoneOffset.UTC);
System.out.println("UTC =       " + utc);
System.out.println("UTC =       " + utc);

ZonedDateTime london = seoul.withZoneSameInstant(ZoneId.of("Europe/London"));
System.out.println("London =    " + london);

ZonedDateTime newYork = seoul.withZoneSameInstant(ZoneOffset.of("-0400"));
System.out.println("NewYork =   " + newYork);

ZonedDateTime vancouver = seoul.withZoneSameInstant(ZoneId.of("America/Vancouver"));
System.out.println("Vancouver = " + vancouver);
```