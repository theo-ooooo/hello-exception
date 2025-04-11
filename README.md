# 🛠️ 예외 처리와 오류 페이지, API 예외 처리

## 1. 서블릿 예외 처리 흐름

- 컨트롤러에서 예외 발생 → 서블릿 컨테이너가 예외 처리
- 스프링 부트는 기본 `/error` 를 통해 HTML 오류 응답 제공

```java
@GetMapping("/error-ex")
public void errorEx() {
    throw new RuntimeException("예외 발생!");
}
```

---

## 2. 서블릿 오류 페이지 등록

### WebServerCustomizer

```java
@Bean
public WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> webServerFactoryCustomizer() {
    return factory -> {
        factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404"));
        factory.addErrorPages(new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500"));
        factory.addErrorPages(new ErrorPage(RuntimeException.class, "/error-page/500"));
    };
}
```

---

## 3. 오류 페이지 컨트롤러

```java
@Slf4j
@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String error404(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 404");
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String error500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage 500");
        log.info("EXCEPTION = {}", request.getAttribute(RequestDispatcher.ERROR_EXCEPTION));
        return "error-page/500";
    }
}
```

---

## 4. API 예외 처리 (@RestControllerAdvice)

### `ErrorResult` DTO

```java
@Data
@AllArgsConstructor
public class ErrorResult {
    private String code;
    private String message;
}
```

### 전역 예외 처리

```java
@RestControllerAdvice
public class ExControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandler(IllegalArgumentException e) {
        return new ErrorResult("BAD", e.getMessage());
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandler(Exception e) {
        return new ErrorResult("EX", "내부 오류");
    }
}
```

---

## 5. 예외 처리 우선순위

- `@ExceptionHandler` > `@ControllerAdvice` > 서블릿 컨테이너 오류 페이지

---

## 6. HTTP 상태 코드 주의

- HTML 응답은 상태코드 의미가 중요
- API 응답은 `ResponseEntity` 또는 `@ResponseStatus` 명시 필수

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {}
```

