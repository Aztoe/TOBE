# Singletone 
  - 싱글톤이란 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴

    ```java
    public class SingletonService {
    //1 static 영역에 객체를 딱 한 개만 생성
    private static  final SingletonService instance = new SingletonService();
    // public 으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회한다
    public static SingletonService getInstance() {
        return instance;
    }
    //3. 생성자를 private 로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다
    private SingletonService() {
        
    }
    ```
    이 코드를 통해서 생성자를 private 처리 해 외부에서 new 키워드로 객체가 생성되는 것을 막았다.

    싱글톤 패턴을 테스트해보자
  ```java
@Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest(){

        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();

        System.out.println("singletonService1 = "+singletonService1);
        System.out.println("singletonService2 = "+ singletonService2);

        assertThat(singletonService1).isSameAs(singletonService2);
    }
```



```
singletonService1 = com.example.hello.testservlet.singleton.SingletonService@4009e306
singletonService2 = com.example.hello.testservlet.singleton.SingletonService@4009e306
```



같은 객체 인스튼서를 반환하는 것을 볼 수 있다.
싱글톤 패턴을 사용하는 이유 ?
- 객체가 요청될때 마다 새로운 객체가 생성되면 메모리 낭비가 심하기 때문에

## 싱글톤 패턴 문제점
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 결론적으로 유연성이 떨어진다.
- 안티패턴으로 불리기도 한다.

그럼 스프링에서는 싱글톤 패턴을 어떤식으로 관리할까 ?
- 스프링에서는 스프링 컨테이너를 통해 싱글턴 패턴을 적용하지 않아도 객체 인스턴스를 싱글톤으로 관리한다.



 즉 스프링 컨테이너를 만들면 스프링 컨테이너 안의 객체는 싱글톤으로 유지되는 것이다.

 # @Configuration 과 바이트 코드 조작
```java

    @Test
    void configurationTest() {
         //스프링 컨테이너 생성
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);


        MemberRepository memberRepository1 = memberService.getMemberRepository();

        MemberRepository memberRepository2 = orderService.getMemberRepository();
        // 모두 같은 인스턴스를 참조하고 있다
        System.out.println("memberService -> memberRepository = "+memberRepository1);
        System.out.println("orderService -> memberRepository = "+ memberRepository2);
        System.out.println("memberRepository =" +memberRepository);
        Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
}
```
코드를 보면 2번의 객체를 호출하고 있고 2개의 다른 인스턴스가 생성되어야 하지만 코드를 확인해보면
```
memberService -> memberRepository = com.example.hello.testservlet.member.MemoryMemberRepository@681a8b4e
orderService -> memberRepository = com.example.hello.testservlet.member.MemoryMemberRepository@681a8b4e
```
같은 인스턴스를 공유하는 것을 확인할 수 있다.
각 Service 로직에서 memeberRepository() 를 sysout으로 찍어봐도 1 번만 호출되는데 그 비밀은 AppConfig에서 적용한 ```@Configuration``` 에 있다.
@Configuration을 적용해서 Appconfig 를 호출하면
  ```java
 @Test
    void configurationDeep() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean="+bean.getClass() );
    }
}
```

```
bean=class com.example.hello.testservlet.AppConfig$$EnhancerBySpringCGLIB$$bc8748fd
```
Appconfig 뒤에 $$EnhancerBySpringCGLIB$$bc8748fd 이 붙어서 나온다. 그 이유는 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서
Appconfig 를 상속받은 다른 클래스가 스프링 빈으로 등록되어 싱글톤이 보장되도록 해주는 것이다.
```@Configuration``` 적용하지 않으면 다 다른 인스턴스로 만들어지고 싱긍톤을 적용하지 않는다.
## 정리
- @Bean만 사용해도 스프링 빈으로 등록되지만 싱글톤 보장x.
- 스프링 설정 정보는 항상 ```@Configuration```을 사용하자



> ##### Reference
> [스프링 핵심 원리](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8) 
