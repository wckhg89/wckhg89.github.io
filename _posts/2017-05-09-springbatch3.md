---
layout: post
title:  "스프링 배치(스프링 Boot 기반)삽질기 3탄 - Spring Batch Meta-data Schema 커스터마이징"
date:   2017-05-09 08:40:00 -0500
categories: SPRING
fb_title: springbatch3
---

회사에서 스프링 배치를 이용한 배치성 프로그램을 제작을 하게 되었습니다.
오늘은 마지막으로 StepExecution을 사용함으로 인해서 Batch  Meta-data를 저장하면서 성능이슈(Meta-data를 저장하며 Serialize 및 DB IO로 인한 성능 저하 이슈)가 발생하는 현상과 StepExecution을 이용하지 않고 Step간의 데이터를 공유하는 방법을 포스팅하려 합니다.

# 시작하며

스탭들간의 데이터를 공유할 때 ExecutionContext를 이용하게 되면 metaDataSchema에 서로 공유되는 데이터를 저장해주게 됩니다.

지난 포스팅에서 해당 내용에 대해 간단하게 작성했습니다.
그런데 이때 저장하는 데이터를 json string 형태로 Serialize 해주면서 성능 이슈가 발생하게 됩니다.

아시다시피 ObjectMapper를 통한 Serialize는 비용이 굉장히 큰 작업입니다. 회사에서 프로젝트를 진행하며 이러한 성능이슈로 배치 작업이 더디게 일어나는 이슈가 있어 우회하는 방법으로 step 간에 데이터를 공유한 방법에 대해서 포스팅 해보겠습니다.

# Step간 데이터 공유 (싱글톤 빈을 사용하면?)

1편 2편에서 말씀드린것 처럼 일단 metaDataSchema에 데이터는 저장하도록 어찌어찌 수정은 했는데 배치 잡이 한번 도는데 약 10분이상이 소요되었습니다.

도저히 실 서비스에서 사용할 수 없는 성능이었습니다.
사수님께 바로 달려가 현재 배치의 성능이슈를 말씀드렸고 어떤식으로 해결하면 좋을지 조언을 구했습니다.

`` "굳이 StepExecution을 이용해서 데이터를 공유할 필요가 있나요?" ``

명쾌한 조언이었습니다..;

왜 굳이 필요하지도 않은 meta-data를 만들면서 까지 StepExecution을 이용할 필요는 없었습니다. 그래서 생각한 방법은 ``@Component`` 싱글톤 빈을 하나 만들어 멤버변수에 Map을 두어 데이터를 공유하면 되겠다 생각했습니다. (어차피 배치성 프로그램이라 프로그램이 한번 실행하고 종료되면 heap 메모리 영역은 초기화 되기 때문에 배치 실행시 heap 메모리만 넉넉히 준다면 gc 이슈는 없을것이라 생각했습니다.)

데이터를 공유하기 위한 클래스를 정의한 아래와 같습니다.

``` java

@Component
public class DataShareBean <T> {

    private static Logger logger = LoggerFactory.getLogger(DataShareBean.class);
    private Map<String, T> shareDataMap;


    public DataShareBean () {
        this.shareDataMap = Maps.newConcurrentMap();
    }

    public void putData(String key, T data) {
        if (shareDataMap ==  null) {
            logger.error("Map is not initialize");
            return;
        }

        shareDataMap.put(key, data);
    }

    public T getData (String key) {

        if (shareDataMap == null) {
            return null;
        }

        return shareDataMap.get(key);
    }

    public int getSize () {
        if (this.shareDataMap == null) {
            logger.error("Map is not initialize");
            return 0;
        }

        return shareDataMap.size();
    }

}

```

멤버변수에 스레드 세이프한 ConcurrentMap을 선언하고 여기에 데이터를 저장해두고 step들 간에 데이터를 공유하도록 하였습니다.

> step1 processor에서 생성한 데이터를 step2Reader에서 사용하는 코드입니다. (주석은 기존에 execution context를 이용했던 코드입니다.)

``` java
// step1 processor
@Component
@StepScope
public class Step1Processor extends SuperStepExecution<Member>
        implements ItemProcessor<List<Member>, Member> {

    @Autowired
    private DataShareBean<Member> dataShareBean;

    private static Logger logger = LoggerFactory.getLogger(Step1Processor.class);

    private final int SPECIFIC_MEMBER_IDX = 0;

    @Override
    public Member process(List<Member> item) throws Exception {
        logger.info("Step1 Processor 시작");
        Member specificMember = item.get(SPECIFIC_MEMBER_IDX);

        logger.info("Step1 Processor 첫번째 회원 정보 : {}", specificMember);
        // super.putData("SPECIFIC_MEMBER", specificMember);
        dataShareBean.putData("SPECIFIC_MEMBER", specificMember);

        return specificMember;
    }

//    @BeforeStep
//    public void saveStepExecution(StepExecution stepExecution) {
//        super.setStepExecution(stepExecution);
//    }
}

```

step1에서는 공유를 위해 설정한 빈을 @Autowired 해주어서 step1에서 가공한 데이터를 해당 빈에 데이터를 넣어주었습니다.


``` java
// step2Reader
@Component
@StepScope
public class Step2Reader extends SuperStepExecution<Member> implements ItemReader<List<Content>> {

    @Autowired
    private DataShareBean<Member> dataShareBean;

    private static Logger logger = LoggerFactory.getLogger(Step2Reader.class);
    private boolean isRead;
    private Member specificMember;

    @PostConstruct
    public void init () {
        isRead = false;
    }

    @Override
    public List<Content> read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {

        logger.info("Step2 Reader 시작");
        if (!isRead) {
            isRead = true;
            List<Content> contentList = this.specificMember.getContents();

            logger.info("Step2 Reader 첫번째 회원의 게시글 수 : {}", contentList.size());

            return contentList;
        }

        return null;
    }

    @BeforeStep
    public void retrieveInterstepData(StepExecution stepExecution) {
        // super.setStepExecution(stepExecution);
        // this.specificMember = (Member) super.getData("SPECIFIC_MEMBER");
        this.specificMember = dataShareBean.getData("SPECIFIC_MEMBER");
    }
}

```

step2에서 역시 공유를 위한 빈을 의존성 주입 받아 데이터를 꺼내오고 있습니다.

# 마치며

이런식으로 스텝간 데이터를 공유를 해주니 metaDataSchema에 데이터를 넣지 않으니 Serialize로 인한 비용이 전혀 없어졌고 10분이상 걸리던 배치 잡이 단 2분만에 완료되었습니다.

사내에서 처음으로 배치를 처음부터 만들면서 여러 삽질(?)을 하였습니다. 스프링 배치는 배치 프로그램을 작성하기에 좋은 구조를 제공하는 것 같습니다. 그러나 저 처럼 metaDataSchema같은 부분을 잘 모르고 사용하면 여러 문제에 부딪힐 수 있을것 같다고 생각하여 후에 같은 실수를 반복하지 않고자 블로그를 작성했습니다.

후에 배치를 또 만들일이 있으면 더 잘 짜보고싶습니다.. 
