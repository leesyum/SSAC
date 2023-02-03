# 3. java.time패키지

Date와 Calendar 의 단점을 해소하기 위해, JDK1.8부터 ‘java.time’ 패키지(+4개의 하위 패키지)가 추가됨

| 패키지 | 설명 |
| --- | --- |
| java.time | 날짜와 시간을 다루는 데 필요한 핵심 클래스들을 제공 |
| java.time.chrono | 표준(ISO)이 아닌 달력 시스템을 위한 클래스들을 제공 |
| java.time.format | 날짜와 시간을 파싱하고, 형식화하기 위한 클래스들을 제공 |
| java.time.temporal | 날짜와 시간의 필드(field)와 단위(unit)를 위한 클래스들을 제공 |
| java.time.zone | 시간대(time-zone)와 관련된 클래스들을 제공 |

위의 패키지들에 속한 클래스들의 가장 큰 특징은 **불변(immutable)** 이라는 것. (String클래스처럼!)

그래서 날짜나 시간을 변경하는 메서드들은 기존의 객체를 변경하는 대신 항상 변경된 새로운 객체를 반환.

→ **thread-safe** (스레드에 안전) 하다!

→ 기존의 Calendar 클래스는 변경가능하므로, 스레드에 안전하지 X
    (+ 멀티스레드 환경에서는 동시에 여러 스레드가 같은 객체에 접근할 수 있기 때문에, 변경가능한 객체는 데이터가 잘못될 가능성 있음)

## 3.1 java.time패키지의 핵심 클래스

날짜와 시간을 별도의 클래스로 분리하였기에,

**LocalDate**(날짜) + **LocalTime**(시간) → **LocalDateTime**

**LocalDateTime** + 시간대(time-zone) → **ZonedDateTime**

이 외에도 날짜를 더 세부적으로 다룰 수 있는 Year, YearMonth, MonthDay와 같은 클래스도 존재

### Period와 Duration

날짜와 시간의 간격을 표현하기 위한 클래스

**Period**는 **두 날짜간의 차이**, **Duration**은 **시간의 차이**를 표현하기 위함

### 객체 생성하기 - now(), of()

java.time 패키지에 속한 클래스의 객체를 생성하는 가장 기본적인 방법

```java
LocalDate date = LocalDate.now();                   // 2023-01-19
LocalTime time = LocalTime.now();                   // 21:54:01.875
LocalDateTime dateTime = LocalDateTime.now();       // 2023-01-19T21:54:01.875
ZonedDateTime zonedTimeInKr = ZonedDateTime.now();  // 2023-01-19T21:54:01.875+09:00[Asia/Seoul]
```

now()는 현재 날짜와 시간을 저장하는 객체 생성

```java
LocalDate date = LocalDate.of(2023, 01, 19);   // 2023-01-19
LocalTime time = LocalTime.of(23, 59, 59);     // 23시 59분 59초

LocalDateTime dateTime = LocalDateTime.of(date, time);
ZonedDateTime zDateTime = ZonedDateTime.of(dateTime, ZoneId.of("Asia/Seoul");
```

of()는 단순히 해당 필드의 값을 순서대로 지정해주면 됨. 각 클래스마다 다양한 종류의 of() 정의됨

### Temporal과 TemporalAmount

Temporal, TemporalAccessor, TemporalAdjuster 인터페이스를 구현한 클래스

→ LocalDate, LocalTime, LocalDateTime, ZonedDateTime, Instant 등

TemporalAmount 인터페이스를 구현한 클래스

→ Period, Duration

### TemporalUnit과 TemporalField

날짜와 시간의 **단위**를 정해놓은 TemporalUnit 인터페이스

→ 얘를 구현한 것이 열거형 ChronoUnit

년, 월, 일 등 날짜와 시간의 **필드**를 정해놓은 TemporalField 인터페이스

→ 얘를 구현한 건, 열거형 ChronoField

```java
LocalTime now = LocalTime.now();     // 현재 시간
int minute = now.getMinute();        // 현재 시간에서 분(minute)만 뽑아냄
int minute2 = now.get(ChronoField.MINUTE_OF_HOUR);    // 위의 문장과 동일
```

특정 필드의 값만을 얻을 때는 get()이나 get으로 시작하는 이름의 메서드 이용

```java
LocalDate today = LocalDate.now();                    // 오늘
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);  // 오늘에 1일을 더함
LocalDate tomorrow = today.plusDays(1);               // 위의 문장과 동일
```

특정 날짜와 시간에서 지정된 단위의 값을 더하거나 뺄때는 plus() 또는 minus()와 함께 열거형 ChronoUnit 을 사용

```java
// (참고) get()과 plus()의 정의. 
int get(TemporalField field)
LocalDate plus(long amountToAdd, TemporalUnit unit)
```

특정 TemporalField나 TemporalUnit을 사용할 수 있는지 확인하는 메서드

```java
boolean isSupported(TemporalUnit unit)    // Temporal에 정의
boolean isSupported(TemporalField field)  // TemporalAccessor에 정의
```

## 3.2 LocalDate와 LocalTime

java.time 패키지의 가장 기본이 되는 클래스

객체 생성방법은 now()와 of() - 3.1 참조

일 단위나 초 단위로도 지정 가능함

```java
LocalDate birthDate = LocalDate.ofYearDay(1999, 365);  // 1999년 12월 31일
LocalTime birthTime = LocalTime.ofSecondDay(86399);    // 23시 59분 59초
```

또는 parse()를 이용하여 문자열 → 날짜/시간 변환 가능

```java
LocalDate birthDate = LocalDate.parse("1999-12-31");  // 1999년 12월 31일
LocalTime birthTime = LocalTime.parse("23:59:59");    // 23시 59분 59초
```

### 특정 필드의 값 가져오기 -  get(), getXXX()

주의할 점은 Calendar와 달리 월(month)의 범위가 1~12이고, 요일은 월요일이 1, 화요일이 2, …, 일요일은 7이라는 점

![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8f5fcda4-7b9e-4372-b1a9-a6ccbf5aaba6/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230202%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230202T134331Z&X-Amz-Expires=86400&X-Amz-Signature=5e99b53a52251d90097a10798e88f168a576662c52af6a87f6ca563d30ff160b&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

이 외에도 get()과 getLong()이 있는데, 원하는 필드를 직접 지정 가능함

이 메서드들의 매개변수로 사용할 수 있는 필드의 목록은

![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a1a07082-a573-4af1-b2f3-0d2d4278e573/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230202%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230202T134530Z&X-Amz-Expires=86400&X-Amz-Signature=2f7a390cdfa2b62a5cfbf5fcb43d64d1c962fb11433902cd4385d9bd5680aafb&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

이름 옆에 * 표시가 있는 메서드는 getLong() 사용해야 함

### 필드의 값 변경하기 - with(), plus(), minus()

날짜와 시간에서 특정 필드의 값을 변경 - with로 시작하는 메서드 사용

```java
LocalDate withYear(int year)
LocalDate withMonth(int month)
LocalDate withDayOfMonth(int dayOfMonth)
LocalDate withDayOfYear(int dayOfYear)

LocalTime withHour(int hour)
LocalTime withMinute(int minute)
LocalTime withSecond(int second)
LocalTime withNano(int nanoOfSecond)
```

필드를 변경하는 메서드들은 항상 새로운 객체를 생성하여 반환함으로, 대입연산자 함께 사용해야 함

특정 필드에 값을 더하거나 빼는 plus()와 minus()

그리고 LocalTime의 truncatedTo()는 지정된 것보다 작은 단위의 필드를 0으로 초기화

```java
LocalTime time = LocalTime.of(12, 34, 56);  // 12시 34분 56초
time = time.truncatedTo(ChronoUnit.HOURS);  // 시(hour)보다 작은 단위를 0으로
System.out.println(time);                   // 12:00
```

LocalTime과 달리 LocalDate에는 truncatedTo가 없음 ← 년, 월, 일은 0이 될 수 없기 때문

### 날짜와 시간의 비교 - isAfter(), isBefore(), isEqual()

compareTo()가 적절히 오버라이딩 되어 있음

```java
int result = date1.compareTo(date2);       // 같으면 0, date1이 이전이면 -1, 이후면 1
```

보다 편리하게 비교할 수 있는 메서드들이 추가로 제공됨

```java
boolean isAfter(ChronoLocalDate other)
boolean isBefore(ChronoLocalDate other)
boolean isEqual(ChronoLocalDate other)      // LocalDate에만 있음
```

equals()가 있는데도 isEqual()을 쓰는 이유는, 연표(chronology)가 다른 두 날짜를 비교하기 위함

equals()는 모든 필드가 일치해야 하지만, isEqual()은 오직 날짜만 비교함

## 3.3 Instant

에포크 타임(EPOCH TIME, 1970-01-01 00:00:00 UTC)부터 경과된 시간을 나노초 단위로 표현

사람에겐 불편하지만, 단일 진법으로만 다루기 때문에 계산이 용이함

```java
Instant now = Instant.now();
Instant now2 = Instant.ofEpochSecond(now.getEpochSecond());
Instant now3 = Instant.ofEpochSecond(now.getEpochSecond(), now.getNano());
```

Instant 생성 시에는 now()와 ofEpochSecond()를 사용함

그리고 필드에 저장된 값을 가져올 때는

```java
long epochSec = now.getEpochSecond();
int nano = now.getNano();
```

Instant는 시간을 초 단위와 나노초 단위로 나누어 저장함

UTC(+00:00)을 기준으로 하기에 LocalTime과 차이가 있을 수 있음

→ 시간대를 고려해야하는 경우 OffsetDateTime을 사용하는 것이 나을 수 있음

Instant와 Date간의 변환

Instant는 기존의 java.util.Date를 대체하기 위한 것으로 JDK1.8부터 Date에 Instant로 변환할 수 있는 새로운 메서드가 추가 됨

```java
static Date    from (Instant instant)   // Instant -> Date
       Instant toInstant()              // Date -> Instant
```

## 3.4 LocalDateTime과 ZonedDateTime

### LocalDate와 LocalTime으로 LocalDateTime 만들기

```java
LocalDate date = LocalDate.of(2023, 12, 31);
LocalTime time = LocalTime.of(12, 34, 56);

LocalDateTime dt = LocalDateTime.of(date, time);
LocalDateTime dt2 = date.atTime(time);
LocalDateTime dt3 = time.atDate(date);
LocalDateTime dt4 = date.atTime(12, 34, 56);
LocalDateTime dt5 = time.atDate(LocalDate.of(2023, 12, 31));
LocalDateTime dt6 = date.atStartOfDay();      // dt6 = date.atTime(0, 0, 0);
```

날짜와 시간을 직접 지정할 수 있는 다양한 버전의 of()와 now()도 정의되어 있음

### LocalDateTime의 변환

LocalDateTime → LocalDate 또는 LocalTime 변환 가능

```java
LocalDateTime dt = LocalDateTime.of(2023, 12, 31, 12, 34, 56);
LocalDate date = dt.toLocalDate();
LocalTime time = dt.toLocalTime();
```

### LocalDateTime으로 ZonedDateTime만들기

```java
ZoneId = zid = ZoneId.of("Asia/Seoul");
ZonedDateTime zdt = dateTime.atZone(zid);
System.out.println(zdt);                  //2023-01-19T19:03:50.451+09:00[Asia/Seoul]
```

LocalDate에 atStartOfDay()메서드 이용한 버전도 가능

```java
ZonedDateTime zdt = LocalDate.now().atStartOfDay(zid);
System.out.println(zdt);                  //2023-01-19T00:00+09:00[Asia/Seoul]
```

### ZoneOffset

UTC로부터 얼마만큼 떨어져 있는지를 ZoneOffSet으로 표현

(서울은 +9)

```java
ZoneOffset krOffset = ZonedDateTime.now().getOffset();
int krOffsetInSec = krOffset.get(ChronoField.OFFSET_SECONDS);   //32400초
```

### OffsetDateTime

ZonedDateTime은 ZoneId로 구역을 표현함. ZoneId가 아닌 ZoneOffset을 사용하는 것이 OffsetDateTime

단지 시간대를 시간의 차이로만 구분함 → 서로 다른 시간대에 존재하는 컴퓨터 간의 통신에는 OffsetDateTime이 필요

### ZonedDateTime의 변환

그레고리력과 가장 유사한 것이 ZonedDateTime

```java
// ZonedDateTime -> GregorianCalendar
GregorianCalendar from(ZonedDateTime zdt)

// GregorianCalendar -> ZonedDateTime 
ZonedDateTime toZonedDateTime()
```

## 3.5 TemporalAdjusters

자주 쓰일만한 날짜 계산들을 대신 해주는 메서드를 정의해놓은 것이 TemporalAdjuster클래스

```java
LocalDate today = LocalDate.now();
LocalDate nextMonday = today.with(TemporalAdjusters.next(DaysOfWeek.MONDAY));
```

![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9ab6f817-8627-4d52-a7a0-feb09110a0f9/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230202%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230202T134646Z&X-Amz-Expires=86400&X-Amz-Signature=2ba6f8613e1246aaa43620dd13559b482765690267b32af0f02bd054a0f1d90d&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

### TemporalAdjuster 직접 구현하기

필요한 경우 자주 사용되는 날짜 계산 메서드를 직접 구현 가능

```java
LocalDate with(TemporalAdjuster adjuster)

// 이 추상메서드 하나만 구현하면 됨
@FunctionalInterface
public interface TemporalAdjuster {
	Temporal adjustInto(Temporal temporal);
}
```

만약 특정 날짜로부터 2일 후의 날짜를 계산하는 DayAfterTomorrow는 다음과 같이 작성

```java
class DayAfterTomorrow implements TemporalAdjuster {
	@Overrride
	public Temporal adjustInto(Temporal temporal) {
		return temporal.plus(2, ChronoUnit.DAYS);  // 2일을 더함
	}
}
```

## 3.6 Period와 Duration

### between()

두 날짜의 차이를 나타내는 Period는 between으로 얻을 수 있음

```java
LocalDate date1 LocalDate.of(2022, 1, 1);
LocalDate date2 LocalDate.of(2022, 12, 31);

Period pe = Period.between(date1, date2);
```

date1이 date2보다 날짜 상으로 이전이면 양수로, 이후면, 음수로 Period에 저장됨

시간 차를 구할때는 Period가 아니라 Duration인 것을 제외하면 Period와 동일

특정 필드의 값을 얻을 때에는 get()을 사용

```java
long year = pe.get(ChronoUnit.YEARS);
// MONTHS, DAYS, SECONDS, NANOS 전부 가능
```

but. Duration에는 getHours(), getMinutes() 같은 메서드가 없기 때문에 Duration → LocalTime으로 변환한 다음 가져오는 것이 간단함

```java
Duration du = Duration.between(time1, time2);  // time1과 time2는 LocalTime
LocalTime tmpTime = LocalTime.of(0, 0).plusSeconds(du.getSeconds());

int hour = tmpTime.getHour();
int min = tmpTime.getMinute();
int sec = tmpTime.getSecond();
int nano = tmpTime.getNano();
```

### between()과 until()

거의 유사함. between()은 static 메서드이고, until()은 인스턴스 메서드라는 차이가 있음

Period는 년월일을 분리하여 저장하기 때문에 D-day를 구할땐 두 개의 매개변수를 받는 until()을 사용하는 것이 낫다

```java
Period pe = today.until(myBirthDay);
long dday = today.until(myBirthDay, ChronoUnit.DAYS);
```

### of(), with()

Period → of(), ofYears(), ofMonths(), ofWeeks(), ofDays()

Duration → of(), ofDays(), ofHours(), ofMinutes(), ofSeconds() 등

특정 필드의 값을 변경하는 with()도 있음

### 사칙연산, 비교연산, 기타 메서드

곱셈과 나눗셈을 위한 메서드도 존재

```java
pe = pe.minusYears(1).multipliedBy(2);  // 1년을 빼고, 2배를 곱함
du = du.plusHours(1).dividedBy(60);     // 1시간을 더하고 60으로 나눔
```

Period는 날짜의 기간을 표현하기 위함이기에 나눗셈을 위한 메서드가 없음(별로 유용하지 않음)

음수인지 확인하는 isNegative()  /  0인지 확인하는 isZero()

```java
// 두 날짜 또는 시간을 비교할 때, 어느 쪽이 앞인지 또는 같은지 알아낼 수 있음
boolean sameDate = Period.between(date1, date2).isZero();
boolean isBefore = Duration.between(time1, time2).isNegative();
```

부호를 반대로 변경하는 negated()와 부호를 없애는 abs()가 있음

Period는 abs()가 없기 때문에 isNegative()이면 negated() 하는 식으로 사용해야 함

Period에 normalized() 메서드는 월(Month)의 값이 12를 넘지 않게 변경해줌

```java
pe = Period.of(1, 13, 32).normalized();  // 1년 13개월 32일 -> 2년 1개월 32일
```

### 다른 단위로 변환 - toTotalMonths(), toDays(), toHours(), toMinutes()

Period와 Duration을 다른 단위의 값으로 변환하는데 사용

get()은 특정 필드의 값을 그대로 가져오는 것이지만, 특정 단위로 반환한 결과를 반환한다는 차이가 있음

![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4118a217-3f5a-4411-9068-9e3dd3d02beb/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230202%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230202T134820Z&X-Amz-Expires=86400&X-Amz-Signature=08621ea47bf668dfefaf59673308526110132ac35049e4052b1d195bdda0e3dc&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

## 3.7 파싱과 포맷

날짜와 시간을 원하는 형식으로 출력하고 해석(파싱, parsing)

형식화와 관련된 클래스들은 java.time.format패키지에 들어있는데 이 중에서 **DateTimeFormatter**가 핵심

```java
LocalDate date = LocalDate.of(2023, 1, 2);
String yyyymmdd = DateTimeFormatter.ISO_LOCAL_DATE.format(date);  // "2023-01-02"
String yyyymmdd = date.format(DateTimeFormatter.ISO_LOCAL_DATE);  // "2023-01-02"
```

ISO_LOCAL_DATE 이외에도 다양한 형식이 정의되어 있음

### 로케일에 종속된 형식화

DateTimeFormatter의 static 메서드 ofLocalizedDate(), ofLocalizedTime(), ofLocalizedDateTime()은

로케일(locale)에 종속적인 포맷터를 생성

```java
DateTimeFormatter fomatter = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);
String shortFormat = formatter.format(LocalDate.now());
```

FormatStyle의 종류에 따른 출력 형태

![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/04ba0716-cda5-4ec8-96b3-5fbff95a9360/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230202%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230202T135054Z&X-Amz-Expires=86400&X-Amz-Signature=eae5fd5e94fd27db53dbf5bb5b1e7f51ee88ae41a956a6aadee4dddcf639c1ae&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)  

### 출력형식 직접 정의하기

`ofPattern()`으로 원하는 출력형식을 직접 작성 가능

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
```

패턴에 사용되는 기호는 다음과 같음

![img](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f3c93ca8-3e22-4975-ad94-fd88afa50641/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230202%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20230202T135125Z&X-Amz-Expires=86400&X-Amz-Signature=9105ba90a7ca46d74837aa7e0002ad04d577d95a33eb056ecb5ff145d37009a7&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22Untitled.png%22&x-id=GetObject)

### 문자열을 날짜와 시간으로 파싱하기

static 메서드 **parse()** 사용

오버로딩된 메서드 중 주로 사용되는 것은

```java
static LocalDateTime parse(CharSequence text)
static LocalDateTime parse(CharSequence text, DateTimeFormatter formatter)

// 예를 들어
LocalDate date = LocalDate.parse("2023-01-02", DateTimeFormatter.ISO_LOCAL_DATE);
```

ofPattern()을 이용해서도 파싱 가능

```java
DateTimeFormatter pattern = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime endOfYear = LocalDateTime.parse("2023-12-31 23:59:59", pattern);
```
