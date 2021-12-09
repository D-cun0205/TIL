### 모델 바인딩과 검증

    기본적으로 스프링에 준비되어 있는 기본 프로퍼티 에디터를 이용해서 HTTP 파라미터 값을
    모델의 프로퍼티 타입에 맞게 전환하고 전환이 불가능한 경우 BindingResult 오브젝트 안에
    바인딩 오류를 저장해서 컨트롤러로 넘겨주거나 예외를 발생한다

커스텀 프로퍼티 에디터

    스프링의 디폴트 에디터의 확인 가능한 타입은 자바의 기본 타입 20여 가지 정도
    개발자가 작성한 enum 타입의 경우는 에디터에서 변환할 수 없다
    이런 경우 PropertyEditorSupport 를 상속받아서 적용하여 해결할 수 있다
    
```java
//Level = enum
public class LevelPropertyEditor extends PropertyEditorSupport {
    @Override
    public String getAsText() {
        return String.valueOf(((Level)this.getValue()).intValue());
    }

    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        this.setValue(Level.valueOf(Integer.parseInt(text.trim())));
    }
}

@Configuration
public class Configuration {
    @InitBinder
    public void initBinder(WebDataBinder dataBinder) {
        dataBinder.registerCustomEditor(Level.class, new LevelPropertyEditor());
    }
}
```

    위 예제와 같이 바인더를 커스텀해서 어노테이션으로 등록만 해주면 개발자는 Level 타입을 받는
    파라미터에 대한 타입검사를 직접하지 않고 스프링 에디터에 등록되어 자동으로 확인해준다
    커스텀 프로퍼티 에디터를 전역에서 사용하고 싶은 경우 WebBindingInitializer 를 구현하고
    빈으로 등록하여 사용할 수 있다