# Istio

Envoy를 이용해서 서비스 매쉬를 구현하기 위해서는 Envoy로 구성된 데이타 플레인을 컨트롤할 솔루션이 필요하다. -> Istio

### 서비스 매쉬
- 애플리케이션의 다양한 부분들이 서로 데이터를 공유하는 방식을 제어하는 방법, 애플리케이션에 구축된 전용 인프라 계층

- 예를 들어, 큰 식당이 있다고 생각해봐요. 이 식당에는 여러 요리사, 서빙 스태프, 청소부 등이 각각 자신만의 역할을 하고 있죠. 그들이 서로 소통하며 일을 잘 하려면 어떻게 해야 할까요? 누군가 그들 사이에서 "소통 관리자" 역할을 해서, 각자 무슨 일을 해야 하고 어떻게 협력해야 하는지를 안내해 주는 거예요.

이게 서비스 메쉬가 하는 일입니다. 각 서비스(요리사, 서빙 스태프 등)가 서로 정보를 주고받는 과정에서 문제가 없도록 돕고, 보안도 신경 쓰며, 누가 얼마나 바쁜지도 모니터링하는 역할을 합니다.

### 데이타 플레인
- 데이타 플레인은 envoy를 서비스 옆에 붙여서 사이드카(주 서비스와 나란히 실행되는 보조 프로세스) 형식으로 배포를 해서, 서비스로 들어오고 나가는 트래픽을 envoy를 통해 통제하게 된다.

- 서비스들 사이에서 오가는 요청과 응답을 제어하고, 라우팅을 한다. 

### 컨트롤 플레인
- 데이타 플레인에 배포된 envoy를 컨트롤 하는 부분으로, 파일럿/믹서/시타델의 3개의 모듈로 구성되어 있음 

##### 파일럿
- envoy에 대한 설정 관리
- 엔드포인트들의 주소를 얻을 수 있는 서비스 디스커버리 기능 제공

##### 믹서
- 액세스 컨트롤, 정책 통제, 모니터링 지표 수집 

##### 시타델
- 보안에 관련된 기능 담당, 서비스를 사용하기 위한 사용자 인증, 인가


### Envoy
- 서비스 옆에 사이드카가 Envoy
- Envoy를 통해 트래픽을 통제한다.

![서비스 매쉬 drawio](https://github.com/user-attachments/assets/35289b8e-2c24-4b70-90fc-109fba66f6e2)



