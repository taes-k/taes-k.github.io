---
layout: default
comments: true
title: ExceptionHandler 예외처리
parent: spring
date: 2019.07.15
---

<h1>{{ page.title }}</h1>  
<div style="text-align:right; font-size:11px; color:#aaa">{{ page.date }} </div>
---

### ExceptionHandler
스프링에서는 예외처리를 위한 어노테이션인 Exception Handler를 제공해주고 있습니다.   
간단한 예제로써 알아보도록 하겠습니다.  

```java
// SampleController.java

@RestController
@RequestMapping("/")
public class SampleController.java {

    private static Logger logger = LogManager.getLogger(SentenceModularController.class);
    private SentenceModularService sentenceModularService;

    @Autowired
    public SentenceModularController(SentenceModularServiceImpl sentenceModularService) {
        this.sentenceModularService = sentenceModularService;
    }

    @RequestMapping(value = "/url", method = RequestMethod.GET)
    public ResponseEntity sampleMethod() throws SampleException, IOException{
        sampleService.setSample();
        return new ResponseEntity(result, HttpStatus.OK);
    }
    
    @ExceptionHandler(value = SampleException.class)
    public ResponseEntity SampleExceptionHandler(SampleException e) {
        logger.error("SampleException error : " + e);
        return new ResponseEntity(HttpStatus.BAD_REQUEST);
    }
    
}
```

위와같이 `@ExceptionHandler` 어노테이션을 이용해 controller내부 exception들을 통합하여 관리해 줄 수 있습니다. 하지만 Controller가 여러개이고, Exception의 종류도 여러개라면 Controller마다 Exception handler를 정의해주는것은 여간 귀찮은일이 아닐수 있습니다.  
  
스프링에서는 또 위의 상황을 커버해주기위해 Global Exception 처리를 위해 `@ControllerAdvice`를 제공해주고 있습니다.  


### Controller advice
```java
// GlobalExceptionHandler.java

@ControllerAdvice
@RestController
public class GlobalExceptionHandler {

    private static Logger logger = LogManager.getLogger(SentenceModularController.class);

    @ExceptionHandler(value = SampleException.class)
    public ResponseEntity SampleExceptionHandler(SampleException e) {
        logger.error("SampleException error : " + e);
        return new ResponseEntity(HttpStatus.BAD_REQUEST);
    }
    @ExceptionHandler(value = IOException.class)
    public ResponseEntity IOExceptionHandler(IOException e) {
        logger.error("IOException error : " + e);
        return new ResponseEntity(HttpStatus.BAD_REQUEST);
    }
    @ExceptionHandler(value = Exception.class)
    public ResponseEntity ExceptionHandler(Exception e) {
        logger.error("Exception error : " + e);
        return new ResponseEntity(HttpStatus.INTERNAL_SERVER_ERROR);
    }

}
```

위와같이 `@ControllerAdvice`선언을 통해 global한 Exception handler를 만들어 줄 수 있습니다.  
