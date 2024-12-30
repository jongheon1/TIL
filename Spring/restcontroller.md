# @RestController도 MVC라 할 수 있을까

### 핵심 요약

Spring의 @RestController는 MVC 프레임워크의 기반 위에 구축되었지만, 전통적인 MVC 패턴과는 다른 방식으로 동작한다. View 계층이 명시적으로 존재하지 않고, 데이터를 직접 클라이언트에게 전달하는 방식을 취한다. 이는 엄밀히 말하면 전통적인 MVC 패턴과는 차이가 있지만, 현대적인 웹 애플리케이션의 요구사항에 맞춰 진화한 형태로 볼 수 있다.


**처리 흐름의 차이**

Spring MVC의 핵심인 DispatcherServlet은 모든 HTTP 요청의 시작점이다. Front Controller로서 요청을 받아 적절한 Controller를 찾고, 그에 맞는 HandlerAdapter를 통해 요청을 처리한다. 여기까지는 @Controller와 @RestController가 동일하다.

하지만 실제 처리 과정에서 큰 차이를 보인다:

1. @Controller는 "View 이름 → ModelAndView → View 렌더링 → HTML 응답"의 과정을 거친다. 전형적인 MVC 패턴이다.
2. @RestController는 "객체 → MessageConverter 변환 → 응답 직접 작성"의 과정을 거친다. View라는 개념이 존재하지 않는다.

이제 상세한 처리 흐름을 살펴보자.

**1. 공통 시작점: DispatcherServlet**

모든 HTTP 요청은 Front Controller 패턴의 구현체인 DispatcherServlet에서 시작된다. DispatcherServlet은 다음 핵심 작업을 수행한다:

1. 요청에 맞는 핸들러(Controller) 조회
2. 해당 핸들러를 실행할 수 있는 HandlerAdapter 조회
3. HandlerAdapter를 통한 핸들러 실행
4. 실행 결과 처리

**2. @Controller의 처리 흐름**

1. 컨트롤러 메서드가 View의 논리적 이름(String)을 반환
2. HandlerAdapter가 View 이름으로 ModelAndView 객체 생성
3. ViewResolver를 통해 실제 View 객체 찾기
4. View의 render() 메서드가 Model 데이터로 HTML 생성
5. 생성된 HTML을 HTTP 응답에 작성


**3. @RestController의 처리 흐름**

1. 컨트롤러 메서드가 도메인 객체를 직접 반환
2. MessageConverter가 반환된 객체를 JSON 등으로 변환
3. 변환된 데이터를 HTTP 응답에 직접 작성
4. View 처리 과정 생략

**4. 주요 차이점**

반환값 처리
- @Controller: View 이름 → ModelAndView → View 렌더링 → HTML 응답
- @RestController: 객체 → MessageConverter 변환 → 응답 직접 작성

응답 생성 위치
- @Controller: View 템플릿에서 Model 데이터로 HTML 생성
- @RestController: MessageConverter가 객체를 JSON으로 변환

처리 결과
- @Controller: HTML 형태의 View를 응답
- @RestController: JSON/XML 등의 데이터를 직접 응답


이제 이러한 차이가 실제 코드에서 어떻게 구현되어 있는지 살펴보겠다. 핵심 로직을 따라가보면서, 두 컨트롤러의 처리 흐름이 어떤 지점에서 나뉘고 각각 어떻게 처리되는지 확인해보겠다.

코드는 가독성을 위해 핵심적인 부분만 남기고 정리했다.

### Spring MVC의 내부 동작 코드로 살펴보기

> 참고: IntelliJ를 사용한다면 아무 스프링 프로젝트에서 External Libraries / org.springframework:spring-webmvc / spring-webmvc.jar / org.springframework.web.sevlet / DispatcherServelt.class에서 실제 코드를 따라가면서 볼 수 있다.

```java
// 1. DispatcherServlet - 모든 요청의 시작점
public class DispatcherServlet extends FrameworkServlet {

    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {  
        try {  
            ModelAndView mv = null;  
            try {  
                // 1.1 http request 를 처리할 핸들러(컨트롤러) 조회
                mappedHandler = this.getHandler(processedRequest);  
                
                // 1.2 해당 핸들러를 처리할 수 있는 어댑터 조회
                HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());  
                
                // 1.3 핸들러 어댑터를 통해 실제 요청 처리 시작
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  
                
                // 1.4 얻어진 ModelAndView로 결과 처리
                this.processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);  
            } catch (Exception ex) {  
                dispatchException = ex;
            }  
        } finally {
            // 리소스 정리
        }  
    }
}

// 2. RequestMappingHandlerAdapter - 실제 핸들러 처리
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter {

	// 1.3 에서 호출. 핸들러 어댑터를 통한 요청 처리 (정확히는 AbstractHandlerMethodAdapter 에 handle 이 있고 거기서 handleInternal 호출)
    protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, 
            HandlerMethod handlerMethod) throws Exception {  
        ModelAndView mav;
        // 2.1 실제 핸들러 메서드 호출 준비
        mav = this.invokeHandlerMethod(request, response, handlerMethod);  
        return mav;
    }

		// 2.1 에서 호출
    protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, 
            HandlerMethod handlerMethod) throws Exception {  
            // ...
            // 핸들러 메서드 호출 준비
        ServletWebRequest webRequest = new ServletWebRequest(request, response);  
        try {  
            // 2.2 실제 핸들러 메서드 실행 및 반환값 처리
            invocableMethod.invokeAndHandle(webRequest, mavContainer, new Object[0]);  
            
            // 2.3 ModelAndView 생성
            ModelAndView mv = this.getModelAndView(mavContainer, modelFactory, webRequest);  
            return mv;
        } finally {  
            webRequest.requestCompleted();  
        }  
    }
}

// 3. ServletInvocableHandlerMethod - 핸들러(컨트롤러) 메서드 실행
public class ServletInvocableHandlerMethod extends InvocableHandlerMethod {

	// 2.2 에서 호출. 
    public void invokeAndHandle(ServletWebRequest webRequest, ModelAndViewContainer mavContainer, 
            Object... providedArgs) throws Exception {  
            
        // 3.1 핸들러(컨트롤러) 메서드 실행하여 반환값 얻기
        // @Controller 라면 String, Model 등, @Restcontroller 라면 json 등
        Object returnValue = this.invokeForRequest(webRequest, mavContainer, providedArgs);  


        // 3.2 위에서 얻은 반환값을 처리할 핸들러(컨트롤러랑 다름)를 찾고 처리
        this.returnValueHandlers.handleReturnValue(returnValue, this.getReturnValueType(returnValue), 
                mavContainer, webRequest);  
    }
}

// 4. HandlerMethodReturnValueHandlerComposite - 반환값 처리자 결정
public class HandlerMethodReturnValueHandlerComposite {

	// 3.2 에서 호출. returnValue, returnTyre (반환값) 정보를 통해 이것을 처리할 수 있는 핸들러를 조회하고 처리한다.
	// 여기서 @Controller, @Restcontroller 등 컨트롤러 마다 다른 핸들러가 선택되어 다르게 처리된다.
    public void handleReturnValue(Object returnValue, MethodParameter returnType,
            ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {  
            
        // 4.1 반환값을 처리할 수 있는 핸들러 조회
        HandlerMethodReturnValueHandler handler = this.selectHandler(returnValue, returnType);  
        
        // 4.2 해당 핸들러로 반환값 처리
        handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);  
    }

	// 4.1 에서 호출. 
    private HandlerMethodReturnValueHandler selectHandler(Object value, MethodParameter returnType) {  
        Iterator<HandlerMethodReturnValueHandler> var4 = this.returnValueHandlers.iterator();  
        HandlerMethodReturnValueHandler handler;
        do {
            handler = var4.next();
            // 4.3 등록된 핸들러들을 순회하면서 현재 반환 타입을 처리할 수 있는 핸들러를 찾음
        } while(!handler.supportsReturnType(returnType));  
        return handler;  
    }
}


// 4에서 컨트롤러 마다 그것을 처리할 핸들러가 다르기 때문에 @Controller, @Restcontroller 의 진행이 나누어진다.

// 5(@Controller). ViewNameMethodReturnValueHandler - 여러가지 핸들러 구현체 중 하나이자 @Controller 용 String 반환값 처리 담당
public class ViewNameMethodReturnValueHandler {

	// 4.3 에서 호출
    public boolean supportsReturnType(MethodParameter returnType) {  
        Class<?> paramType = returnType.getParameterType();  
		// 5.1 CharSequence(String) 반환 타입을 지원하는지 확인한다
        return Void.TYPE == paramType || CharSequence.class.isAssignableFrom(paramType);  
    }
}

// 5(@Restcontroller). RequestResponseBodyMethodProcessor - 여러가지 핸들러 구현체 중 하나이자 @RestController용 반환값 처리 담당
public class RequestResponseBodyMethodProcessor {

    // 4.3 에서 호출
    public boolean supportsReturnType(MethodParameter returnType) {  
        // 5.1 컨트롤러나 메서드에 @ResponseBody 어노테이션이 붙어 있는지 확인한다
        return AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ResponseBody.class) 
            || returnType.hasMethodAnnotation(ResponseBody.class);  
    }
}


// 위 과정을 거쳐서 4 로 돌아감.
public void handleReturnValue(Object returnValue, MethodParameter returnType,
		ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {  
		
	// 4.1 반환값을 처리할 수 있는 핸들러 조회
	// 금방 여기서 핸들러를 찾은 것
	HandlerMethodReturnValueHandler handler = this.selectHandler(returnValue, returnType);  
	
	// 4.2 해당 핸들러로 반환값 처리
	// 이제 진짜 최종 반환값을 처리하러 감. 이것도 위에서 다른 핸들러 구현체를 얻었으니 다른 로직이 돌아감
	handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);  
}


// 5(@Controller) RequestResponseBodyMethodProcessor - 여러가지 핸들러 구현체 중 하나이자 @RestController용 반환값 처리 담당
public class ViewNameMethodReturnValueHandler {

    // 4.2 에서 호출
    public void handleReturnValue(Object returnValue, MethodParameter returnType,
            ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {  
		// CharSequence(String) 타입인 경우
        if (returnValue instanceof CharSequence) {  
            String viewName = returnValue.toString();  
            // 5.2 View 이름 설정 - 나중에 ViewResolver가 실제 View를 찾을 때 사용
            mavContainer.setViewName(viewName);  
        }
    }
}

// 5(@Restcontroller). RequestResponseBodyMethodProcessor -  여러가지 핸들러 구현체 중 하나이자 @RestController용 반환값 처리
public class RequestResponseBodyMethodProcessor {

    // 4.2 에서 호출
    public void handleReturnValue(Object returnValue, MethodParameter returnType,
            ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {  
        // View 처리 건너뛰기 설정(이미 handle 되었으니 나중에 렌더링 안 하겠다는 뜻)
        mavContainer.setRequestHandled(true);  
        // 5.2 MessageConverter를 통해 http response 본문에 직접 결과값 쓰기
        writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);  
    }
}


// 여기까지 왔으면, @Controller 는 뷰 이름을 설정했으니 뷰를 만들 준비가 됨.
// @Restcontroller 는 이미 response 에다가 결과값을 모두 작성함.

// 위 과정을 거쳐서 2 로 돌아감..
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter {

		// 2.1(handleInternal) 에서 호출
    protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, 
            HandlerMethod handlerMethod) throws Exception {  
            // ...
            // 핸들러 메서드 호출 준비
        ServletWebRequest webRequest = new ServletWebRequest(request, response);  
        try {  
            // 2.2 실제 핸들러 메서드 실행 및 반환값 처리
            // 금방까지 보고 온 부분.. 여기서 뷰 이름 결정이나 응답 작석이 이루어짐..
            invocableMethod.invokeAndHandle(webRequest, mavContainer, new Object[0]);  
            
            // 2.3 ModelAndView 생성
            // 이제야 뷰 이름과 모델 둘 다 있다..! 그러니 모델뷰를 만들 수 있음음
            ModelAndView mv = this.getModelAndView(mavContainer, modelFactory, webRequest);  
            return mv; // 그리고 모델뷰 반환
        } finally {  
            webRequest.requestCompleted();  
        }  
    }
}

// 6. RequestMappingHandlerAdapter - ModelAndView 생성 
public class RequestMappingHandlerAdapter {

	// 2.3 에서 호출.
    private ModelAndView getModelAndView(ModelAndViewContainer mavContainer, 
            ModelFactory modelFactory, NativeWebRequest webRequest) throws Exception {  
        // 7.1 @RestController의 경우
	    // 이전에 @RestController 용 핸들러에서 mavContainer.setRequestHandled(true); 라고.. 렌더링 안 하겠다는 처리를 함..
	    // 그래서 여기서 이미 처리되었으면 모델뷰로 null 반환
        if (mavContainer.isRequestHandled()) {  
            return null;  
        } else {  
            // 7.2 @Controller의 경우 정상적으로 모델뷰 생성
	        // 이전에 @Controller 용 핸들러에서 mavContainer.setViewName(viewName);  라고.. 뷰 이름 설정한 것을 여기서 사용해 모델뷰를 만듬..
            ModelAndView mav = new ModelAndView(mavContainer.getViewName(), 
                mavContainer.getModel(), mavContainer.getStatus());  
            return mav;   // 결과로 모델뷰 반환..
        }  
    }
}


// 다시 2 번으로 돌아감..
public class RequestMappingHandlerAdapter extends AbstractHandlerMethodAdapter {

		// 2.1(아래 handleInternal) 에서 호출
    protected ModelAndView invokeHandlerMethod(HttpServletRequest request, HttpServletResponse response, 
            HandlerMethod handlerMethod) throws Exception {  

            // 바로 위에서 모델뷰 만든 부분..
            ModelAndView mv = this.getModelAndView(mavContainer, modelFactory, webRequest);  
            return mv; // 그렇게 얻은 모델뷰를 반환
        } finally {  
            webRequest.requestCompleted();  
        }  
    }
    
	// 1.3(doDispatch) 에서 호출.
    protected ModelAndView handleInternal(HttpServletRequest request, HttpServletResponse response, 
            HandlerMethod handlerMethod) throws Exception {  
        ModelAndView mav;
        // 바로 위 함수..
        // 긴긴 과정을 거쳐서 모델뷰를 얻어냄
        mav = this.invokeHandlerMethod(request, response, handlerMethod);  
        return mav; // 그걸 또 반환..
    }
}

// 그리고 젤 처음 DispatcherServlet 의 doDispatch 로 돌아와서..
public class DispatcherServlet extends FrameworkServlet {

    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {  
        try {  
            ModelAndView mv = null;  
            try {  
                // 1.1 http request 를 처리할 핸들러(컨트롤러) 조회
                mappedHandler = this.getHandler(processedRequest);  
                
                // 1.2 해당 핸들러를 처리할 수 있는 어댑터 조회
                HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());  
                
                // 1.3 핸들러 어댑터를 통해 실제 요청 처리 시작
                // 지금까지 이 부분을 본 것... 힘겹게 모델뷰를 얻어옴..
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  
                
                // 1.4 그렇게 얻어진 ModelAndView로 진짜 최종 결과 처리
                this.processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);  
            } catch (Exception ex) {  
                dispatchException = ex;
            }  
        } finally {
            // 리소스 정리
        }  
    }
}



// 8. DispatcherServlet - 최종 결과 처리
public class DispatcherServlet {
    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
            HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {  
		// 이전에 @Restcontroller 인 경우 모델뷰로 null 을 반환했었다..! 그래서 여기서 rest 는 렌더링 하지 않고 그냥 지나가는 것이다.. (이미 response 에 결과값이 작성 됨..)
        if (mv != null && !mv.wasCleared()) {  
        
			// 하지만 mvc 는 모델뷰가 존재하고 그 모델뷰로 렌더링 진행..
		    // 렌더링에선 뷰리졸버로 뷰를 찾고 그 뷰에 모델을 전달해 진짜 렌더링을 한다.. 
		    // 그리고는 html 코드를 만들어내 http response 에 작성함..
            render(mv, request, response);  
        }  
	    // @Controller, @Restcontroller 모두 결과 작성이 마무리 되었다.. 나머지는 이후 부가적인 처리
    }
}
```



**정리**

**DispatcherServlet의 doDispatch 진입**

```java
// 1-1. HTTP 요청을 처리할 핸들러(Controller) 찾기
mappedHandler = this.getHandler(request);

// 1-2. 해당 핸들러를 처리할 수 있는 어댑터 찾기
HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());

// 1-3. 핸들러 어댑터를 통해 실제 요청 처리 시작
mv = ha.handle(request, response, handler);
```

**HandlerAdapter(RequestMappingHandlerAdapter)의 처리**

```java
// 2-1. handleInternal에서 실제 메서드 호출 준비
mav = invokeHandlerMethod(request, response, handlerMethod);

// 2-2. ServletInvocableHandlerMethod를 통해 실제 컨트롤러 메서드 실행
invocableMethod.invokeAndHandle(webRequest, mavContainer);
```

**반환값 처리 분기**

```java
// 3-1. HandlerMethodReturnValueHandlerComposite에서 적절한 핸들러 선택
HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);

// 3-2. @Controller와 @RestController 분기
if (@Controller) {
    // ViewNameMethodReturnValueHandler
    mavContainer.setViewName(viewName);  // 뷰 이름 설정
} else if (@RestController) {
    // RequestResponseBodyMethodProcessor
    mavContainer.setRequestHandled(true);  // 뷰 처리 건너뛰기
    writeWithMessageConverters(returnValue, response);  // 직접 응답 작성
}
```

**ModelAndView 생성**

```java
// 4-1. getModelAndView 호출
if (mavContainer.isRequestHandled()) {
    return null;  // @RestController인 경우
} else {
    return new ModelAndView(viewName, model);  // @Controller인 경우
}
```

**최종 결과 처리**

```java
// 5-1. DispatcherServlet의 processDispatchResult
if (mv != null && !mv.wasCleared()) {
    render(mv, request, response);  // @Controller: 뷰 렌더링
} else {
    // @RestController: 이미 응답이 작성되어 있으므로 추가 처리 없음
}
```

**핵심 차이점**

**@Controller**: 뷰 이름 반환 → ModelAndView 생성 → 뷰 렌더링 → HTML 응답
**@RestController**: 객체 반환 → MessageConverter로 변환 → 바로 응답에 작성 → 뷰 처리 스킵


### 결론

**MVC 관점에서의 @RestController**

@RestController는 Spring MVC 프레임워크의 인프라를 기반으로 동작한다. DispatcherServlet을 시작점으로 하여 핸들러 매핑, 어댑터 호출 등 기본적인 요청 처리 흐름은 @Controller와 동일하다. 하지만 실제 처리 방식에서 중요한 차이를 보인다.

**View의 존재 여부**

엄격한 의미에서 @RestController는 View 계층이 존재하지 않는다:
1. ModelAndView 객체를 생성하지 않는다
2. ViewResolver를 통한 뷰 검색 과정이 없다
3. 뷰 렌더링 단계를 완전히 건너뛴다

**Model의 처리**

전통적인 MVC에서 Model은 View가 렌더링할 데이터를 담는 역할을 했지만, @RestController에서는 Model 객체를 별도로 사용하지 않는다. 대신 반환되는 도메인 객체 자체가 데이터의 역할을 한다.

**응답 생성 방식의 재해석**

보다 유연한 관점에서 보면, MessageConverter가 도메인 객체를 JSON/XML로 변환하여 HTTP 응답에 작성하는 과정을 View의 역할로 해석할 수 있다:
1. Controller: REST 엔드포인트를 처리하는 @RestController
2. Model: 반환되는 도메인 객체
3. View: MessageConverter의 변환 과정


**결론**

@RestController는 전통적인 MVC 패턴을 현대 웹 서비스의 요구사항에 맞게 재해석한 것으로 볼 수 있다. 비록 명시적인 View 계층은 존재하지 않지만, HTTP 응답 생성 과정을 View의 역할로 간주한다면 넓은 의미에서 MVC 패턴의 한 변형이라고 할 수 있다. 이는 프레임워크가 진화하면서 디자인 패턴도 함께 현대화되는 좋은 예시를 보여준다.

결국 @RestController가 MVC인가 아닌가의 답은 관점에 따라 달라질 수 있다. 전통적이고 엄격한 MVC 정의를 따른다면 아니라고 할 수 있지만, 현대적 웹 서비스의 맥락에서 패턴을 재해석한다면 MVC의 한 형태로 볼 수 있다. 중요한 것은 이러한 패턴의 진화가 실제 개발자들의 필요와 현대 웹 서비스의 요구사항을 더 잘 충족시킨다는 점이다.
