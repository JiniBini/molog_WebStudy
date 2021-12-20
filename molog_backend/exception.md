# 예외 처리
기능을 구현하고 테스트하다 보면 500 에러가 나는 경우를 흔하게 볼 수 있습니다. 그 이유로는 다음과 같습니다.
<ul>
  <li>서버 통신의 Timeout 시간 지연 오류</li>
  <li>서버 트래픽 과부하</li>
  <li>서버 언어의 구문 에러(스크립트 문법 오류)</li>
</ul>
이러한 500 error들을 처리해야 합니다.

## BaseResponseCode Enum 생성
처리하고 싶은 error를 처리하고, 응답으로 돌려줄 때 그 내용을 담은 코드를 Enum 타입으로 생성합니다.
HttpStatus와 String 타입의 message가 있습니다.
```
enum class BaseResponseCode(status: HttpStatus, message: String) {
    BAD_REQUEST(HttpStatus.BAD_REQUEST, "잘못된 요청입니다."),
    INVALID_PASSWORD(HttpStatus.BAD_REQUEST, "잘못된 비밀번호입니다. 다시 입력해주세요."),
    DUPLICATE_EMAIL(HttpStatus.BAD_REQUEST, "중복된 이메일입니다. 다시 입력해주세요."),

    USER_NOT_FOUND(HttpStatus.NOT_FOUND, "사용자를 찾을 수 없습니다."),
    FAILED_TO_SAVE_USER(HttpStatus.NOT_FOUND, "사용자를 등록에 실패했습니다."),
    GUEST_BOOK_NOT_FOUND(HttpStatus.NOT_FOUND, "방명록을 찾을 수 없습니다."),
    OK(HttpStatus.OK, "요청 성공");

    public val status: HttpStatus = status
    public val message: String = message
}
```

## BaseException 생성
error를 처리하고 해당 error를 의도한 내용으로 응답하기 위해서 BaseException을 생성합니다.
```
class BaseException(baseResponseCode: BaseResponseCode): RuntimeException() {
    public val baseResponseCode: BaseResponseCode = baseResponseCode
}
```
RuntimeException를 상속받은 클래스로, 그 내용으로는 baseResponseCode를 담고 있습니다.
예외를 던져줄 때 BaseException을 사용해 처리된 형태로 응답을 보낼 수 있습니다.
```
throw BaseException(BaseResponseCode.BAD_REQUEST)
```
```
{
  "status": "BAD_REQUEST",
  "message": "잘못된 요청입니다."
}
```

## ExceptionHandler 구현
위에서 throw 한 예외를 받아서 처리해줄 수 있는 controller를 구현해 줍니다.
```
@RestControllerAdvice
class ExceptionHandler {
    @ExceptionHandler(BaseException::class)
    protected fun handleBaseException(e: BaseException): ResponseEntity<BaseRes> {
        return ResponseEntity.status(e.baseResponseCode.status)
            .body(BaseRes(e.baseResponseCode.status, e.baseResponseCode.message))
    }
}
```
@ResControllerAdvice를 클래스에 선언해주면, 모든 @Controller에 대한 전역으로 발생하는 예외를 잡아서 처리할 수 있습니다. <br />
또한, 예외 처리를 위해 선언된 함수 위에 @ExceptionHandler(BaseException::class)를 선언해주면, 발생한 BaseException을 처리해줄 수 있습니다.
