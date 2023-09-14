# Annotation

- java에서는 @interface라고 선언했다면, kotlin에서는 annotation 클래스라고 선언한다.


## Meta Annotation
- annotation을 위한 annotation
### @Target
- 어노테이션을 할 수 있는 요소를 지정한다
### @Retention
- 어노테이션을 컴파일 된 클래스로 저장할지 여부와 런타임에 리플렉션을 이용해 접근할 수 있게 할지 여부를 지정한다.
### @Repeatable 
- 같은 어노테이션을 한 요소에 여러번 적용할 수 있는지를 지정한다.
### @MustBeDocumented
- 어노테이션이 공개 API에 속하는지와  생성한 API 문서의 클래스나 메서드 시그니처에 포함되어야 하는지를 지정한다.