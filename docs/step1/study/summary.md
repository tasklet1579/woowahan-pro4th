### ✏️ AssertJ Exception Assertions

AssertJ
- assertion을 제공하는 자바 라이브러리
- 다양한 검증 메서드를 제공하고 있어서 테스트 코드를 더욱 쉽게 작성할 수 있다.
- 메서드 체이닝을 지원하여 직관적인 테스트 흐름을 나타낼 수 있다.

***assertThatThrownBy***
```
import static org.assertj.core.api.Assertions.assertThatThrownBy;

@DisplayName("위치 값을 벗어나면 Exception이 발생한다.")
@Test
void charAt_throwException_givenIndexGreaterThanLength() {
    assertThatThrownBy(() -> {
        // ...
    }).isInstanceOf(IndexOutOfBoundsException.class)
      .hasMessageContaining("Index: 2, Size: 2");
}
```

***assertThatExceptionOfType***
```
import static org.assertj.core.api.Assertions.assertThatExceptionOfType;

@DisplayName("위치 값을 벗어나면 Exception이 발생한다.")
@Test
void charAt_throwException_givenIndexGreaterThanLength() {
    assertThatExceptionOfType(IndexOutOfBoundsException.class)
        .isThrownBy(() -> {
            // ...
    }).withMessageMatching("Index: \\d+, Size: \\d+");
}
```

자주 발생하는 Exception의 경우 아래의 메서드를 제공하고 있다.
- assertThatIllegalArgumentException()
- assertThatIllegalStateException()
- assertThatIOException()
- assertThatNullPointerException()

***Behaviour-Driven Development***
```
import static org.assertj.core.api.Assertions.catchThrowable;

@DisplayName("위치 값을 벗어나면 Exception이 발생한다.")
@Test
void charAt_throwException_givenIndexGreaterThanLength() {
    Throwable thrown = catchThrowable(() -> { throw new ...  });
    
    assertThat(thrown).isInstanceOf(IndexOutOfBoundsException.class)
                      .hasMessageContaining("...");
}
```

### ✏️ JUnit 5 Parameterized Tests

다른 값을 사용하여 하나의 테스트 코드를 반복해서 실행할 수 있다.

@ParameterizedTest를 추가하는 것을 제외하고는 다른 테스트와 동일하다.

***Simple Values***
- @ValueSource
  - 테스트 메서드에 값을 순서대로 하나씩 전달할 때 사용할 수 있다.
  - 지원하는 자료형은 다음과 같다.
    - short, byte, int, long, float, double, char, java.lang.String, java.lang.Class
```
public class Strings {
    public static boolean isBlank(String input) {
        return input == null || input.trim().isEmpty();
    }
}
```
```
@ParameterizedTest
@ValueSource(strings = {"", "  "})
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

***Null and Empty Values***
- @NullSource
  - 원시 자료형은 null 값을 허용할 수 없으므로 사용할 수 없다.
- @EmptySource
  - String뿐만 아니라 Collection과 arrays에도 empty 값을 전달할 수 있다.
- @NullAndEmptySource
  - null 값과 empty 값을 모두 전달하기 위해 사용할 수 있다.
  - @ValueSource와 결합하여 문자열 값을 다양하게 전달할 수 있다.
```
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {"  ", "\t", "\n"})
void isBlank_ShouldReturnTrueForAllTypesOfBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

***Enum***
- @EnumSource
  - 열거형 값을 테스트 메서드에 전달할 때 사용할 수 있다.
  - names 속성
    - 리터럴 문자열과 일치하는 열거형 값을 가진다. 
    - 리터럴 문자열 외에도 정규 표현식을 속성에 전달할 수 있다.
```
@ParameterizedTest
@EnumSource(Month.class) // passing all 12 months
void getValueForAMonth_IsAlwaysBetweenOneAndTwelve(Month month) {
    int monthNumber = month.getValue();
    assertTrue(monthNumber >= 1 && monthNumber <= 12);
}
```

```
@ParameterizedTest
@EnumSource(
    value = Month.class,
    names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER", "FEBRUARY"},
    mode = EnumSource.Mode.EXCLUDE)
void exceptFourMonths_OthersAre31DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(31, month.length(isALeapYear));
}
```

```
@ParameterizedTest
@EnumSource(value = Month.class, names = ".+BER", mode = EnumSource.Mode.MATCH_ANY)
void fourMonths_AreEndingWithBer(Month month) {
    EnumSet<Month> months = 
        EnumSet.of(Month.SEPTEMBER, Month.OCTOBER, Month.NOVEMBER, Month.DECEMBER);
    assertTrue(months.contains(month));
}
```

***CSV Literals***
- @CsvSource
  - 입력값과 예상 값을 전달하여 테스트가 필요할 경우 사용할 수 있다.
  - 배열의 원소를 콤마로 나누고 각각의 값을 테스트 메서드의 파라미터로 전달한다.
  - 콤마 이외에도 속성을 통해 다른 구분자로 커스터마이즈 할 수 있다.
```
@ParameterizedTest
@CsvSource({"test,TEST", "tEst,TEST", "Java,JAVA"})
void toUpperCase_ShouldGenerateTheExpectedUppercaseValue(String input, String expected) {
    String actualValue = input.toUpperCase();
    assertEquals(expected, actualValue);
}
```

```
@ParameterizedTest
@CsvSource(value = {"test:test", "tEst:test", "Java:java"}, delimiter = ':')
void toLowerCase_ShouldGenerateTheExpectedLowercaseValue(String input, String expected) {
    String actualValue = input.toLowerCase();
    assertEquals(expected, actualValue);
}
```

***Method***
- @MethodSource
  - 복잡한 객체를 사용하여 전달하기 위해 Argument Source로 메서드를 사용한다.
    - 하나의 단일 값을 사용하는 경우는 Arguments abstraction이 필요하지 않다.
  - 이름을 지정하지 않으면 JUnit은 테스트 메서드명과 같은 메서드를 소스 코드에서 찾는다.
  - 메서드를 이용하면 다른 테스트 클래스 간에 공유할 수 있기 때문에 재활용 측면에서 유용하다.
```
class StringsUnitTest {
    @ParameterizedTest
    @MethodSource("com.baeldung.parameterized.StringParams#blankStrings")
    void isBlank_ShouldReturnTrueForNullOrBlankStringsExternalSource(String input) {
        assertTrue(Strings.isBlank(input));
    }
}
```

```
public class StringParams {
    static Stream<String> blankStrings() {
        return Stream.of(null, "", "  ");
    }
}
```

***Customizing Display Names***
- @ParameterizedTest의 name 속성을 이용해서 Display Names를 커스터마이즈 할 수 있다.
  - {displayName} is display name of the method
  - {index} will be replaced with the invocation index. Simply put, the invocation index for the first execution is 1, for the second is 2, and so on.
  - {arguments} is a placeholder for the complete, comma-separated list of arguments.
  - {0}, {1}, ... are placeholders for individual arguments.
```
@ParameterizedTest(name = "{index} {0} is 30 days long")
@EnumSource(value = Month.class, names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"})
void someMonths_Are30DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

```
someMonths_Are30DaysLong(Month)
- 1 APRIL is 30 days long
- 2 JUNE is 30 days long
- 3 SEPTEMBER is 30 days long
- 4 NOVEMBER is 30 days long
```
