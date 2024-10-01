# ArchTest

아키텍처 테스트 란?
프로젝트가 정해놓은 아키텍처를 제대로 따르고 있는지 확인하는 것

### ArchUnit

다양한 아키텍처 체크를 할 수 있는 메소드 제공

### 테스트 방안 

1. 클래스에 @AnalyzeClasses(packages = "테스트할 패키지"), 변수엔 @ArchTest 어노테이션을 붙혀준다.

2. 검증할 패키지 조건 정의

##### 패키지 의존성 체크 

```java
@AnalyzeClasses(packages = "com.effortguy")
public class PackageDependency {
    // source에 있는 패키지에 있는 클래스는 foo 패키지를 의존하고 있으면 안된다.
    @ArchTest
    ArchRule archRule = noClasses().that().resideInAPackage("..source..")
            .should().dependOnClassesThat().resideInAPackage("..foo..");

   // target 패키지에 있는 클래스는 source 패키지만 의존하고 있어야 한다. 
    @ArchTest
    ArchRule archRule2 = classes().that().resideInAPackage("..target..")
            .should().onlyHaveDependentClassesThat().resideInAPackage("..source..");
}
```

##### 클래스 의존성 체크, 패키지 네이밍 룰 체크

```java
@AnalyzeClasses(packages = "com.effortguy.perftest.archunit.classdependency")
public class ClassDependency {

    @ArchTest 
    ArchRule archRule = classes().that().haveNameMatching(".*Repository")
            .should().onlyHaveDependentClassesThat().haveNameMatching(".*Service");

    @ArchTest // ②
    ArchRule archRule2 = classes().that().haveSimpleNameEndingWith("Controller")
            .should().resideInAPackage("com.effortguy.perftest.archunit.classdependency.controller");
}
```
