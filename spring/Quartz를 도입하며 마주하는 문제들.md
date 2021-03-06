안녕하세요. 이번에 회사에서 다소 네이티브하고 복잡하게 구현된 스케줄러 구조를 개선하는 업무를 맡았습니다. 저는 Quartz라는 기술을 선택해서 리팩토링을 진행했습니다. 하지만 처음 접해보는 기술이라 도입 과정에서 헷갈리거나 어려운 부분들이 많았습니다.

그래서 오늘은 저와 같이 Quartz를 처음 사용하시는 분들에게 작은 도움이 될 수 있도록 아래 2가지 내용을 적어보고자 합니다.
- Quartz를 선택한 이유
- Quartz를 도입하며 마주했던, 헷갈렸던 문제

Quartz를 처음 사용하시는 분들은 읽어보시면 꼭 도움이 되실 겁니다.

## Quartz를 선택한 이유 

Quartz를 도입한 이유는 `확장성`과 `비즈니스 로직 집중`이라는 장점이 있기 때문입니다.

#### 확장성

클러스터링을 통한 분산처리가 가능할 뿐만 아니라, 다중 인스턴스에서도 중단 없이 배포가 가능합니다.

Job에 대한 추가 및 삭제가 쉽습니다. 필요한 Job을 정의하고, 트리거와 함께 스케줄러에 등록만 하면 됩니다.
> 기존에는 적절하게 추상화되어 있지 않아 대부분의 로직들이 특정 Job 하나에 의존되는 형태를 갖고 있었습니다. 다른 Job을 추가할 때의 변경에 대한 파급력은 무시할 수 없는 수준이었습니다. 이런 측면에서 생각해볼 때, Quartz가 제공해주는 API는 정말 매력적입니다.

#### 비즈니스 로직 집중

몇가지 설정을 제외하곤, 클러스터링을 위한 코드를 직접 작성할 필요가 없습니다.

Job과 Trigger를 정의할 때 무엇을 언제 해야할지만 정의하면 됩니다. Job의 등록과 삭제는 스케줄러에게 간단히 요청하는 것만으로도 쉽게 이뤄집니다. 

## Quartz를 도입하며 마주했던, 헷갈렸던 문제들

공식 문서에서 제공해주는 Tutorial과 CookBook을 읽으면서 헷갈렸던 부분과 문서에서 알려주지 않는 부분들에 대한 문제를 나름대로 해결했던 경험을 공유하고자 합니다.

### Recovery 사용시 주의점

우선 Recovery 매커니즘이 언제 작동하는지 알아야 합니다. Recovery는 시스템이 알 수 없는 이유로 셧다운 되는 등, 작업이 실행 도중에 끊겼을 때 재시도하는 속성을 말합니다. 다시 말해, Job이 논리적인 이유로 실패했을 때는 Recovery 속성이 true라고 하더라도, Retry되지 않습니다. 논리적인 이유로 실패할 경우 Retry가 필요하다면, 직접 별도의 로직을 작성해야 합니다.

Recovery는 언뜻 보면 좋아보이지만, 통제할 수 없는 영역과 함께 Job을 처리하는 경우 주의가 필요합니다.

쇼핑몰에서 어떤 물건을 구매하고 구매확정을 눌렀을 때, 1일 뒤 구매후기를 독려하는 알림톡을 사용자에게 발신하는 Job이 있다고 가정해보겠습니다. 해당 Job이 알림톡을 발송하는 것까진 실행됐는데, 그 후 처리 작업이 실행되지 않는 경우 이러한 상황은 Recovery의 대상으로 적합한 것일까요? 만일 Recovery 된다면, 알림톡이 1번 이상 발송될 수 있습니다.

따라서 Job이 수행될 로직을 적절하게 작성하는 것이 먼저지만, 혹여나 그렇지 못한 상황이라면 이 땐 Recovery 속성을 끄는 것이 맞습니다. 

### @DisallowConcurrentExecution 

Job이 동시에 실행되지 않도록 제한하는 어노테이션입니다.

1분 주기로 Trigger되는 Job이 있고, 해당 Job이 작업을 처리하는 시간이 2분이라고 가정해보겠습니다. 한 개의 Job이 1분동안 처리되면, 동일한 Job이 또 다시 실행되게 됩니다. 이러한 상황을 Quartz에서 ConcurrentExecution이라고 말합니다.

따라서 동시에 실행되어도 괜찮은 Job인지 생각해보고, 어노테이션을 사용해야 합니다.

### JobDetail 중복 코드 문제

<div>
<img src="https://raw.githubusercontent.com/momentjin/blog-repository/master/resource/image/quartz1.png" >
</div>

<br>

JobDetail을 정의할 때, 항상 위와 같이 Job에 대한 데이터를 삽입하고, 꺼내쓰는 과정에서 중복 코드가 발생합니다. 이 부분은 2가지 이유로 유지보수적인면에서 불안정하다고 생각합니다. 

1. Job에서 필요로 하는 데이터를 명시적으로 선언할 수 없기 때문에, 컴파일타임에 오류를 체크할 수 없다.
2. Job에 필요한 데이터가 무엇인지 불명확하다.

이 문제를 해결할 수 있는 구조에 대해 고민한 결과, 하나의 Job을 구현할 때 하나의 JobDetailImpl을 상속하는 구현체를 만들고, JobDetail 생성시 해당 객체를 사용하는 룰을 만드는 것입니다. (아래 코드 참고 부탁드립니다)

<div>
<img src="https://raw.githubusercontent.com/momentjin/blog-repository/master/resource/image/quartz2.png"  >
</div>

<br> 

위와 같이 만들면, 다음과 같이 역할을 구분할 수 있게 됩니다.
- JobDetailImpl 구현 객체(MyJobDetail) : Job이 수행될 때 필요한 데이터를 정의한다.
- Job 구현 객체(MyJob) : Job이 실질적으로 수행되어야할 내용을 담는다.

물론 Job 객체에서 데이터를 꺼내쓰는 부분까지 추상화하진 못했습니다만, 적어도 해당 객체를 사용하는 **클라이언트가 JobDetail을 정의할 때 일관성있게 사용하도록 유도할 수 있다는 장점**이 있습니다. 또한 Job의 스펙을 명시적으로 선언할 수 있어서 클라이언트 입장에서 사용하기 편리합니다. 

### 불발된 Job에 대한 처리 전략은 필수

Quartz에서 불발(misfire)이란, 실행되었어야할 Job이 실행되지 않은 경우를 말합니다.
- 예시) 2020-08-03 15:00에 1번만 실행되어야 하는 Job A가 데이터베이스에 저장되어 있다고 가정을 해보겠습니다. 무중단배포가 아닌 상황에서, 14:59에 인스턴스를 중단하고 배포를 했더니 15:02가 되었습니다. 따라서 실행되지 못한 Job A는 Misfired Job으로 취급됩니다.

만일 특정 시간에 **정확히** 수행되어야 할 Job이 불발되었다고 가정해보겠습니다. 이 Job이 다시 트리거된다면 이는 비즈니스적인 오류가 될 것 입니다. 따라서 이런 상황에서는 불발된 Job이 다시 실행되지 않도록 처리하도록 옵션을 설정해야 합니다.

### JobDataMap에 넣지 말아야할 타입의 값?

공식 문서를 보면 아래와 같은 글이 있습니다.

> The "org.quartz.jobStore.useProperties" config parameter can be set to "true" (defaults to false) in order to instruct JDBCJobStore that all values in JobDataMaps will be Strings, and therefore can be stored as name-value pairs, rather than storing more complex objects in their serialized form in the BLOB column. This is much safer in the long term, as you avoid the class versioning issues that there are with serializing your non-String classes into a BLOB.

장기적인 관점에서 클래스 직렬화 이슈를 피하기 위해 `원시타입`과 `String`만 사용하라고 합니다. 변경에 유연하게 대처하기 위함인 것 같습니다.

하지만 이 부분을 반드시 따라야 하는지는 의문입니다. 객체를 사용할지, 원시타입만 사용할지는 목적에 따라 다르다고 생각합니다. 강한 타입 체크를 원한다면 클래스 직렬화 이슈를 피할 이유가 없습니다. 하지만 이에 따라 발생되는 사이드 이펙트를 감당하는 것은 개인의 몫이겠죠. 선택은 자유입니다.

## 끝으로

클러스터링 부분은 다루지 못했습니다. 아직은 필요가 없어서 공부를 안했네요..

문서를 읽는 것은 기본이지만, 문서만으로는 **'잘 쓰는 것'** 이 어려운 일임을 깨달았습니다. 운영하면서 많은 시행착오를 거쳐야할 듯 합니다. 꿀팁이 있으시다면, 공유 부탁드립니다! 이견도 언제나 환영입니다. 읽어주셔서 감사합니다.