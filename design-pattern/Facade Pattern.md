# Facade Pattern 

Facade Pattern에 대해 정리한 글 입니다.

최근 기술 서적을 읽으면서, 가끔 눈에 띄는 모르는 용어가 있었습니다. Facade Pattern이었는데, 이에 대해 확실히 알고 넘어가고자 이 글을 작성하게 되었습니다.

# 개요

Facade pattern이란, 다수의 복잡한 서브시스템의 인터페이스에 대한 통합된 인터페이스를 제공하는 디자인 패턴입니다. 이 패턴을 사용하는 이유는 단순합니다. 복잡한 인터페이스를 단순하게 사용하기 위함입니다.

한 마디로 Facade Pattern은 서브시스템을 쉽게 사용하기 위한 Wrapper 클래스입니다. 굳이 설명하지 않아도 모두가 이해할 수 있는 아주 일반적이고 쉬운 패턴이지만, 학부생 때의 경험을 한 번 공유해보겠습니다.

# 간단한 사례

해커톤에서 Microsoft의 Custom Vision Service API를 이용한 태그 기반의 이미지 분류기를 만든 적이 있습니다. 이 때 태그로 이미지를 분류하려면 약 3~4개의 API를 혼합해서 사용해야 했습니다. 근데 만약 Custom Vision Service에서 태그로 이미지를 분류하는 기능을 만들어줬더라면, 내부적으로 3~4개의 API를 Wrapping해서 제공해줬더라면 사용자 입장에서 정말 편리했을 것 입니다.

이 때 사용할법한 Pattern이 바로 Facade Pattern입니다. 각각의 API의 집합을 서브시스템이라고 한다면, 이를 Wrapping한 것이 Facade Pattern을 적용한 Facade Class가 되겠죠.

# 끝으로

개발하면서 가장 알게 모르게, 직관적으로 사용하는 패턴인 것 같습니다. 그래도 알고 모르는 것엔 하늘과 땅 차이가 있으니, 간단한 패턴이지만 하나 배워서 기쁘네요 :)