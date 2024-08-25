# Optional을 사용한 null 처리

모든 원시 타입은 기본값을 가지고 있다. 예를 들어 숫자 타입은 0, boolean 타입은 false를 기본값으로 가진다.

반면 클래스나 인터페이스, 배열과 같은 비원시 타입은 값이 할당되지 않을경우 기본값으로 null을 갖게 된다.

## null 참조의 문제점

먼저 null 참조는 변수, 인수, 반환값에 대한 유효한 값이다. 그렇다고 해서 Null이 각 상황에서 예상되거나, 적합하거나 허용되는 값이라는 의미는 아니다.

더욱이 이러한 값이 적절하게 처리되지 않으면 나중에 문제가 발생할 수 있다.

null 참조와 관련된 두 번째 문제는 바로 타입 모호성이다. null은 구체적인 타입이 없어도 모든 타입을 대표할 수 있다.

이러한 독특한 성질은 코드 전반에 걸쳐 값의 부재라는 개념을 나타내기 위해 다양한 객체 유형마다 다른 타입이나 키워드를 사용하지 않기 위해 필요하다.

## 자바에서 null을 다루는 방법 (Optional 도입 전)

### 변수를 null로 초기화하지 않도록 주의하기

변수는 항상 null이 아닌 값을 가져야 한다.

```java
// 비권장
String value = null;
if (condition) {
    value = "Condition is true";
} else {
    value = "Condition is false";	
}

// 권장
String asTernary = condition ? "Condition is true" : "Condition is false";
```

### null 값을 전달하거나 수용 또는 반환하지 않아야 한다.

변수가 null이 아니어야 하는 것처럼 모든 인수나 반환값 역시 null을 피해야 한다.

필수가 아닌 값에 대한 특정 메서드와 생성자를 제공한 후 원본 메서드에서 null을 허용하지 않아야 한다.

```java
public record User(long id, String firstname, String lastname) {
    public User{
        Objects.requireNonNull(firstname);
        Objects.requireNonNull(lastname);
    }
}
```

### 도구를 활용한 null 검사

자바의 null 참조 경우 어노테이션을 사용하여 변수, 인수 및 메서드 반환 타입을 @Nullable 또는 @NonNull로 표시할 수 있다.

```java
@NonNull
List<@Nullable String> getListOfNullableStrings();

@Nullalbe
List<@NonNull String> getNullableListOfNonNullStrings();

void doWork(@Nullable String identifier);
```

## Optional 알아보기

자바 8의 새로운 Optional은 null을 일관성 있게 처리하기 위한 전문 타입일 뿐만 아니라, JDK에서 사용할 수 있는 모든 함수적 기능의 혜택을 받는 유사 함수형 파이프라인 이기도 하다.

Optioanl은 스트림처럼 부가적인 래퍼는 메서드 호출과 그 결과로 생기는 스택 프레임에 대한 오버헤드를 발생시킨다.

그러나 null 값을 가진 일반적인 작업 흐름을 더욱 간결하고 직관적으로 표현할 수 있는 추가적인 기능을 제공한다.

### Optional 없이 내용 불러오기

```java
public Content get(String contentId) {
    if (contentId == null) {
        return null;
    }
	
    if (contentId.isBlank()) {
        return null;
    }
    
    var cacheKey = contentId.toLowerCase();
	
    var content = this.cache.get(cacheKey);
    if (content == null) {
        content = loadFromDB(contentId);
    }
    
    if (content != null) {
        return null;
    }
	
    if (!content.isPublished()) {
        return null;
    }
    
    return null;
}
```


### Optional 호출 체인을 사용하여 내용 불러오기

```java
public Optional<Content> get(String contentId) {
    return Optional.ofNullable(contentId)
        .filter(Predicate.not(String::isBlank))
        .map(String::toLowerCase)
        .map(this.cache::get)
        .or(() -> loadFromDB(contentId))
        .filter(Content::isPublished);
}
```

이 방식은 모든 상황에 Null 검사를 하는 전통적인 방식과 대비되는 방식이다.

### Optional 파이프라인 구축하기

Optional 파이프라인은 스트림과는 다르게 파이프라인에 최종 연산이 추가되기 전까지는 느긋하게 연결되지 않고 모든 연산은 호출되는 즉시 수행된다.

#### Optional 생성

- Optional.ofNullable(T value) : 값이 null일 가능성이 있거나 비어 있을 수 도 있다고 생각되면 ofNullable를 통해 Null 값이 허용되는 새 인스턴스를 만든다.
- Optional.of(T value) : 값이 null이 아님을 확신할 때 사용한다. 값이 null이면 NullPointerException이 발생한다.
- Optional.empty() : 비어 있는 Optional 인스턴스를 만든다.

## Optional과 스트림

### Optional을 스트림 요소로 활용하기











