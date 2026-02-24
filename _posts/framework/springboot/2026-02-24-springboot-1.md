---
title: "[Spring Boot] 전역 예외 처리"
excerpt: "전역 예외 처리 템플릿 정리"

categories:
  - Spring Boot
tags:
  - [springboot, exception]

permalink: /springboot/post-1/

toc: true
toc_sticky: true

date: 2026-02-24
last_modified_at: 2026-02-24
---

## 예외 패키지 구조
> exception\
> |__BusinessLogicException\
> |__ErrorResponse\
> |__ExceptionCode\
> |__GlobalExceptionHandler

## 1. Custom Exception Code Enum Class
- 비즈니스 로직에 대한 예외 코드 열거 클래스
- 상태코드와 메세지 정의
- `ExceptionCode.USER_NOT_FOUND`

```java
@Getter
public enum ExceptionCode {
    //AUTH
    INVALID_CREDENTIALS(400, "INVALID_CREDENTIALS","잘못된 아이디 또는 비밀번호"),
    //USER
    USER_NOT_FOUND(404,"USER_NOT_FOUND", "존재하지 않는 사용자"),
    EMAIL_ALREADY_EXIST(409, "EMAIL_ALREADY_EXIST","이미 존재하는 이메일"),
    USER_NAME_ALREADY_EXIST(409, "USER_NAME_ALREADY_EXIST", "이미 존재하는 사용자이름")
    // 그 외 도메인
    ;

    private final int code;
    private final String status;
    private final String message;

    ExceptionCode(int code, String status, String message) {
        this.code = code;  
        this.status = status;
        this.message = message;
    }
}

```
## 2. Business Logic Exception Class
- 비즈니스 로직에 대한 예외 클래스
  - `RuntimeException`을 상속
    - 언체크 예외(Unchecked Exception)로서 트랜잭션 롤백을 발생시키기 위함 
    - `super(message)`생성자 호출
- 1에서 정의한 예외 코드를 사용하여 생성
- 비즈니스 로직에서 커스텀 예외를 던짐
  - `throw new BusinessLogicException(ExceptionCode.USER_NOT_FOUND)`

```java
public class BusinessLogicException extends RuntimeException {
    @Getter
    private final ExceptionCode exceptionCode;
    
    public BusinessLogicException(ExceptionCode exceptionCode) {
        super(exceptionCode.getMessage());
        this.exceptionCode = exceptionCode;
    }
}
```

## 3. Error Response Class
- 예외 응답 DTO 클래스
  - JSON의 구조(Schema)가 항상 일정
- 상태코드와 메세지 전달
- 예외 종류에 따른 정적 팩토리 메서드(`of`)
- 파라미터 유효성 검증과 경로 유효성 예외 응답을 위한 내부 정적 클래스
  - `FieldError`, `ViolationError`

```java
@Getter
public class ErrorResponse {

  private List<FieldError> fieldErrors;
  private List<ConstraintViolationError> violationErrors;
  private Integer code;
  private String message;

  private ErrorResponse(List<FieldError> fieldErrors, 
          List<ConstraintViolationError> violationErrors) {
    this.fieldErrors = fieldErrors;
    this.violationErrors = violationErrors;
  }

  private ErrorResponse(Integer code, String message) {
    this.code = code;
    this.message = message;
  }

  public static ErrorResponse of(BindingResult bindingResult) {
    return new ErrorResponse(FieldError.of(bindingResult), null);
  }

  public static ErrorResponse of(Set<ConstraintViolation<?>> violations) {
    return new ErrorResponse(null, ConstraintViolationError.of(violations));
  }

  public static ErrorResponse of(Integer code, String message) {
    return new ErrorResponse(code, message);
  }
  // 내부 정적 클래스 위치
}
```

### 1) FieldError Inner Class
- API 요청에서 DTO 유효성 검증 실패 시 발생하는 예외 응답 DTO 클래스
  - `@Request Body`+`@Valid`가 붙은 파라미터에 발생
  - 필드 오류(타입 불일치, 제약조건) 처리
  - `BindingResult`객체를 인자로 받아 에러 정보를 DTO로 반환
- 예외 발생 필드, 예외 발생 객체, 예외 메세지 전달
- <mark>여러 필드 위반 발생 가능 </mark> → 항상 `List`형태
  - `getFieldErrors()`로 컬렉션 형태로 에러 객체 반환
  - 각 에러 객체에서 필요한 필드 추출하여 DTO형태로 변환
- `org.springframework.validation.FieldError`와 혼동 주의

```java
@Getter
public static class FieldError{
    private String field;
    private Object rejectedValue;
    private String message;

    private FieldError(String field, Object rejectedValue, String message){
        this.field = field;
        this.rejectedValue = rejectedValue;
        this.message = message;
    }
    private static List<FieldError> of(BindingResult bindingResult){
        return bindingResult.getFieldErrors().stream()
                .map(e -> new FieldError(
                            e.getField(),
                        e.getRejectedValue() == null ? null:e.getRejectedValue(),
                        e.getDefaultMessage()
                )).collect(Collectors.toList());
    }
}
```

### 2) ConstraintViolationError Inner Class
- API 요청에서 경로 파라미터 검증 실패 시 발생하는 예외 응답 DTO 클래스
  - `@PathVariable`이나 `@RequestParam`으로 들어온 값 자체를 검증
    - `@Validated`+ 제약조건 어노테이션(`@Min()`, `@Email` 등) 필수
    - 타입이 틀린 경우는 `MethodArgumentTypeMismatchException`이 발생
  - `Set<ConstraintViolation>`객체를 인자로 받아 에러 정보를 DTO로 반환
- 예외 발생 필드, 예외 발생 객체, 예외 메세지 전달
- <mark>여러 필드 위반 발생 가능 </mark> → 항상 `List`형태
  - set의 각 에러 객체에서 필요한 필드 추출하여 DTO형태로 변환

```java
@Getter
public static class ConstraintViolationError{
    private String propertyPath;
    private Object rejectedValue;
    private String message;
    private ConstraintViolationError(String propertyPath, Object rejectedValue, String message){
        this.propertyPath = propertyPath;
        this.rejectedValue = rejectedValue;
        this.message = message;
    }
    private static List<ConstraintViolationError> of(Set<ConstraintViolation<?>> violations){
        return violations.stream()
                .map(v->new ConstraintViolationError(
                        v.getPropertyPath().toString(),
                        v.getInvalidValue(),
                        v.getMessage()
                )).collect(Collectors.toList());
    }
}
```

## 4. GlobalExceptionHandler Class
- 전역 예외처리를 위한 핸들러 클래스
- `@RestControllerAdvice`를 클래스 레벨에 붙임
  - 스프링이 클라이언트에 예외를 던지기 전에 예외를 가로채서 처리
- 핸들러 메서드를 정의하여 각 예외에 대한 응답 생성
  - 개별 예외 핸들러로 처리 되지 않은 예외를 처리하기 위해 최상위 예외(`Exception.class`) 핸들러 필수
    - `e.getMessage()`는 서버의 예기치 않은 오류를 클라이언트에 전달할 수 있기 때문에 <mark>"Internal Server Error"</mark> 메세지 전달

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // 기본 에러
    @ExceptionHandler(value = Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e){
        ErrorResponse response = ErrorResponse.of(500, "Internal Server Error");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
    //커스텀
    @ExceptionHandler(value = BusinessLogicException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessLogicException e){
        ErrorResponse response = ErrorResponse.of(e.getExceptionCode().getCode(), e.getMessage());
        return ResponseEntity.status(HttpStatus.valueOf(e.getExceptionCode().getCode())).body(response);
    }
    //파라미터
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    //@ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValidException(MethodArgumentNotValidException e){
        return ResponseEntity.badRequest().body(ErrorResponse.of(e.getBindingResult()));
    }
    //경로
    @ExceptionHandler(value = ConstraintViolationException.class)
    public ResponseEntity<ErrorResponse> handleConstraintViolationException(ConstraintViolationException e){
        return ResponseEntity.badRequest().body(ErrorResponse.of(e.getConstraintViolations()));
    }
}
```

## 5. 응답 예시

```json
{
  "fieldErrors": [
    {
      "field": "email",
      "rejectedValue": "user",
      "message": "이메일 형식이 올바르지 않습니다."
    },
    {
      "field": "username",
      "rejectedValue": "",
      "message": "사용자 이름을 입력해주세요."
    }
  ],
  "violationErrors": null,
  "code": null,
  "message": null
}
```