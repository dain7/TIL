# Spring Batch Reader Enhancement

- Reader
  - 데이터를 read하는 것

- Chunk Processing
  - 데이터를 어떤 청크 단위로 나누어 처리하는 것 
  - 대량 배치 처리는 반드시 청크로 동작해야 함 

- 스프링 배치에서 일반적으로 많이 사용하는 Paging Reader
  - RepositoryItemReader
  - JpaPagingItemReader
  - But offset 방식은 태생적인 한계가 존재

- ZeroOffsetItemReader
  - offset을 0으로 유지
  - where 절을 이용해 조건절을 건다

- Cursor 기반 
  - ExposedCursorItemReader 사용
