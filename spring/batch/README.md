# Spring Batch 분석 & 정리하기
> https://jojoldu.tistory.com/324 [원본]글을 참고하여 개인용으로 정리하였습니다.
> https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/index.html
>https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/en/Ch02_SpringBatchArchitecture.html#Ch02_SpringBatchArch_Overview_SpringBatch

## 1. What is batch application ? 

웹 어플리케이션을 많이 개발하다 보니 배치, 배치잡, 크론탭, 스케줄링 등 다양한 용어를 혼합하여 배치라고 부르고 있습니다. 저부터도...        
[위키피디아](https://en.wikipedia.org/wiki/Batch_processing) 의 정의를 보면 **배치 프로세싱 이란 일괄 처리**라고 적혀있습니다.    
주로 웹에서 배치라 함은 **정해진 주기(스케줄링)를 가지고 정해진 작업(Job)을 처리** 하는 것들을 말하죠.    
가령 예를 들면 정산/마감, 통계, 메일발송 등을 이런 작업들을 모아서 일괄 처리 하도록 스케줄링을 합니다.  
여기서 중요한 포인트는 스케줄링과 배치 프로세싱은 전혀 다르다는것 입니다. (저는 공부하면서 다시금 깨달았습니다.)    
Spring scheduler(linux cron 같은놈)를 통해 **관리자 페이지 내부에 주로 사용자가 없는 시간에 정해진 주기를 두고,**  
**위의 일괄처리가 필요한 작업들을 코드로 구현해 놓고 사용 했습니다.**

### 정리 하자면 배치라고 부르는 작업과 스케줄링 전혀 다른 개념 이라는거를 꼭 이해 하시고 넘어가면 되겠습니다. (아시면 Pass) 🥳
**스케줄러는 원하는 시간에 원하는 주기를 가지고 특정 작업을 할 수 있도록 Event trigger 같은 느낌으로다가 접근을 하시면 되고
배치,배치잡,배치 프로세싱은 원하는 작업을 구현해 놓은 비지니스 로직 구간 입니다. 즉 위의 스케줄러가 호출하게 될 실제 구현체 부분입니다.**    


### 1-1 배치 애플리케이션의 조건
> 배치 애플리케이션은 아래의 조건을 만족해야 합니다.

- **대용량 데이터 처리**: 대량의 데이터를 가져오고, 전달, 계산하는 등의 처리를 할 수 있어야 합니다.
- **자동화**: 사용자의 개입 없이 실행 되어야 합니다.
- **견고성**: 잘못된 데이터가 있더라도 충돌/중단 없이 처리할 수 있어야 합니다.
- **신뢰성**: 무엇이 잘못되었는지를 추적할 수 있어야 합니다.(로깅, 알림)
- **성능**: 지정한 시간 안에 처리를 완료하거나, 동시에 실행되는 다른 애플리케이션에 영향 없이 수행되어야 합니다.

## 2. Why should we use Spring Batch ?
Spring Batch는 엔터프라이즈 시스템의 일상적인 운영에 필수적인 강력한 배치 애플리케이션을 개발할 수 있도록 설계된 가볍고 포괄적 인 배치 프레임 워크입니다.  
Spring Batch는 사람들이 기대하는 Spring Framework의 특성 (생산성, POJO 기반 개발 접근 방식 및 일반적인 사용 용이성)을 기반으로 구축되며, 개발자가 필요할 때 더 진보 된 엔터프라이즈 서비스에 쉽게 액세스하고 활용할 수 있도록 합니다.  
**Spring Batch는 스케줄링 프레임 워크가 아닙니다.**  
상용 및 오픈 소스 공간 모두에서 사용할 수있는 좋은 엔터프라이즈 스케줄러 (예 : Quartz, Tivoli, Control-M 등)가 많이 있습니다.  
**스케줄러를 대체하는 것이 아니라 스케줄러와 함께 작동하기위한 것입니다.**  

Spring Batch는 로깅 / 추적, 트랜잭션 관리, 작업 처리 통계, 작업 재시작, 건너 뛰기, 리소스 관리 등 대량의 레코드 처리에 필수적인 재사용 가능한 기능을 제공합니다.  
또한 최적화 및 파티셔닝 기술을 통해 대용량 및 고성능 배치 작업을 가능하게하는 고급 기술 서비스 및 기능을 제공합니다.  
Spring Batch는 단순한 사용 사례 (예 : 데이터베이스로 파일 읽기 또는 저장 프로 시저 실행)뿐만 아니라 복잡한 대용량 사용 사례 (예 : 데이터베이스간에 대량의 데이터 이동, 변환 등에서 사용할 수 있습니다.) 대용량 배치 작업은 확장 성이 뛰어난 방식으로 프레임 워크를 활용하여 많은 양의 정보를 처리 할 수 ​​있습니다.

스프링 배치의 더 세부적인 내용을 확인하시려면 [Spring batch architecture](https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/spring-batch-intro.html#springBatchArchitecture) 확인 하실 수 있습니다.   
아래 설명드릴 내용도 다 나와 있습니다. 뭐라는건지... 어려운 소리만 자아안뜩 해놨어요.😤 

## 3. Components of Spring Batch
Spring Batch의 다양한 구성요소 그리고 연결 방식
![spring-batch-components](./images/spring-batch-components.jpg)

위의 링크에서 매커니즘을 100% 이해하지 못해 Job Launcher, Job Repository는 우선 제외하겠습니다.  
찾아봐도 아직 명쾌하게 설명드리기는 어렵네요... 😱

### 3-1 Job
배치 작업 그 자체 이며, 혹은 Flow 라고도 불립니다.    
최소 하나의 Step을 가져야 하며 엄청나게 복잡한 Job이 아닌 이상 2-10개의 Step을 권장한다 (솔직히 10개도 너무 많다).
아래 내용까지 보시면 아시겠지만 조졸두님 께서는 순차적인 작업들을 나눌때 Step을 통해 구분 하기 보다는,   
Step들도 개별적인 동작을 할 수 있도록 구성하는 것이 좋다고 합니다.(유지보수의 편의성 및 추상화)
 


### 2-1 !!!추가 기재필요 !!! Job, Step, Tasklet or (Reader, Processor, Writer)

### 2- Chunk Oriented Tasklet
**Chunk 지향 처리**는 트랜잭션 경계 내에서 Chunk 단위로 데이터를 읽고 생성하는 기법입니다.  
Chunk란 아이템이 트랜잭션에서 커밋되는 수를 말합니다. 읽어온 데이터를 지정된 Chunk 수에 맞게 트랜잭션 커밋을 진행합니다.

예를 들어 1천 개의 데이터를 업데이트해야 한다고 해보겠습니다.  
이럴 경우 위에서 말한 **Tasklet 방식 이나 청크로 데이터를 지정하지 않은 경우 천 개의 데이터에 변경을 한 번에 해야 합니다.**   
이렇게 되면 천 개의 데이터 중 한 개의 데이터라도 업데이트를 실패하여 Exception 발생할 경우 **천 개의 데이터를 롤백 해야 합니다.**   
또한 트랜잭션 시간을 길게 가져감으로써 데이터베이스로부터 읽어온 데이터들이 lock이 걸리게 됩니다.  
이러한 문제를 해결하기 위해서는 **Chunk 지향 처리를 이용**합니다.  
**Chunk 단위로 transaction을 수행하기** 때문에 실패할 경우 해당 Chunk 만큼만 Rollback이 되고, 이전에 커밋된 transaction 범위는 반영이 됩니다.  

![chunk-oriented-tasklet](./images/chunk-oriented-tasklet.png)

- Reader에서 아이템을 하나 읽어옵니다.
- 읽어온 아이템을 Processor에서 가공합니다.
- 가공된 데이터들을 별도의 공간에 모은 뒤, Chunk 단위만큼 쌓이게 되면 Writer에 전달 하여 저장합니다.
- 자바 코드로 표현 하면 다음과 같습니다.

``` java
// chunkSize 단위로 묶어서 처리한다.
for(int i = 0; i < totalSize; i += chunkSize) {
    List items = new ArrayList();
    for(int j=0; j < chunkSize; j++) {
        Object item = itemReader.read();
        Object processedItem = itemProcessor.process(item);
        items.add(processedItem);
    }
    itemWriter.write(items);
}

```


### 2- Metadata table
> [schema for spring batch docs](https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/schema-appendix.html#metaDataSchema)

배치 하나가 하나의 Job 이며, 여러개의 Step 이 모여 Job 을 이루게 된다. 다만, Step 은 Reader , Processor, Writer 가 Tasklet 로 대체 될 수 있는데 이것은 하나의 큰 덩어리로 나의 경우 굳이 3분류로 나누지 않고 각 스텝별로 Tasklet 단위로 작성을 하는게 기능 별로 정의하는 느낌적인 느낌.


### 3-1 Spring Batch 관리 도구들 (Tools)
>**호출방식 2가지** CLI 통한 jar 실행, Java 내부 호출 통한 실행

**1. Cron** - Linux cron scheduling execute jar
**2. Spring MVC + API Call** - 웹 MVC 내부에서 Rest API 내부에서 호출
**3. Spring Batch Admin (Deprecated)** - Spring Cloud Data Flow 사용하라고 하나 분석 필요.
**4. Quartz + Admin** - quartz로 Admin페이지를 만들어서 scheduling 하여 호출
**5. CI Tools (Jenkins/Teamcity/...)** - CI 툴을 이용해서 호출

### 3-2 @jojuldu 갓 형님은 jenkins 추천
- integration(Slack, Email ...): event에 대한 callback 처리가 장점
- 실행이력 / 로그관리 / Dashboard
- 다양한 실행 방법 (Rest API / 스케줄링 / 수동 실행): job별로 rest api를 지원해서 admin에 추가해서 호출 가능
- 계정 별 권한 관리
- pipe line: 
- Web UI + Script 둘다 사용 가능
- Plugin (ansible, Github, Logentries ...)

3,4번 같은 경우는 별도의 Admin UI 구성하여서 처리 하기 때문에 공수가 더 추가된다.

### 3-3 Jenkins & Batch Jar 실행 명령

``` bash
java -jar \
-Dspring.profiles.active={Spring 프로파일} \
${BATCH_JAR_HOME}/setter_batch.jar \
--job.name={job 이름} \
baseDate=${baseDate} \
version=${VERSION} \
```

``` bash
java -jar \
-XX:+UseG1GC \
-Dspring.profiles.active={Spring 프로파일} \
${BATCH_JAR_HOME}/setter_batch.jar \
--job.name={job 이름} \
baseDate=${baseDate} \
version=${VERSION} \
```

- 실행시 공통으로 사용하는 환경변수를 Jenkins config를 사용하자.  
Global properties 사용해서 환경변수를 선언해 놓고 사용하면 shell script에서 공통 사용

### 3-4 무중단 배포
> linux link, readlink 확인 !
배포를 하기 위해서 모든 배치잡이 실행 완료를 기다려야 하는 상황이 생긴다.
배포, 배치 젠킨스를 분리하여서 사용하고, 배포시 실행중인 jar파일과 충돌이 나지 않도록 해야한다.

### 3-5 Tips
**여러 작업이 순차적으로** 실행이 필요할때 Step으로 나누기 보다는 **파이프라인을 우선** 고려한다.
A -> B -> C 의 순서로 Job이 구성되어 있고 Step으로 구분지을 경우 A,B,C의 개별적인 스탭을 실행할 수가 없다.  
구성할 당시에는 개별 실행의 이슈가 없다고 하더라도 절대적...일까??? 때문에 파이프라인을 우선 고려 하는것이 좋다.

### 3-6 Test
1. @ConditionalOnProperty() / @TestPropertySource(properties = "job.name=")
위의 어노테이션을 이용한 테스트 작성
2. 테스트 케이스가 100개 이상 부터 속도 저하 발생.
    - Environment가 변경될떄 Spring Context를 재시작
    - @ConditionalOnProperty()가 환경을 변경하면서 테스트 메소드 실행을 하기 위해 매번 스프링 리로드.
3. Environment가 변경되는 조건
    - 테스트 코드에서 @MockBean / @SpyBean 사용시
    - 테스트 코드에서 @TestPropertySource로 환경변수 변경시
    - @ConditionalOnProperty로 테스트마다 Config가 다를때

### 3-7 JPA + Batch
1. N+1 Issue
    - Join Fetch
    - default_batch_fetch_Size 설정 사용
    - JpaPagingItemReader 에서는 작동 X / HibernateCursorItemReader는 가능
