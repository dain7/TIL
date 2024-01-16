# Github action 

### workflow
- 여러 잡으로 구성되고, 이벤트에 의해 트리거될 수 있는 자동화된 프로세스
- 최상위개념
- workflow 파일은 yaml로 작성되고, github repo의 ./githun/workflows 폴더 아래에 저장됨

### events
- workflow를 trigger하는 특정 활동이나 규칙
- 특정 브랜치로 push하거나 pr하거나 특정 시간대에 반복

### jobs
- job은 여러 step으로 구성되고, 가상 환경의 인스턴스에서 실행됨
- 다른 job에 의존 관계를 가질 수 있고, 독립적으로 병렬 실행도 가능함

### steps
- task들의 집합으로, 커맨드를 날리거나 action을 실행할 수 있음

### actions
- workflow의 가장 작은 블럭
- job을 만들기 위해 step들을 연결할 수 있음
- 재사용 가능

### runners
- github action runner 어플리케이션이 설치된 머신으로, workflow가 실행될 인스턴스 