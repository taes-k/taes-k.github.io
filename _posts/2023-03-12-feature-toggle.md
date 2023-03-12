---
layout: post
comments: true
title: 피쳐토글 스위치를 활용한 신규서비스 개발 전략
tags: [featuretoggle]
---

### 기능 전환 롤백

개발을 하면서 신규 기능이 추가되는 경우 배포후 이슈가 확인되었을때 빠르게 롤백하는것은 장애영향을 줄이기위해 아주 중요한 작업입니다. 특히나 최근 제가 작업중인 대규모 시스템 리팩토링에서는 배포 -> 이슈확인 -> 롤백이 자주 일어날 수 있는 작업입니다. 일반적으로 롤백은 이전 이미지 배포 형상으로 되돌리거나, 혹은 이전 배포 코드베이스를 다시 배포하는방식으로 처리하게되는데 이때 이방식은 몇가지 단점이 있습니다.

1. 롤백에 시간이 걸립니다.
  아무래도 이전 형상으로 배포를 해야하는 프로세스가 있기때문에 다시 배포되기전까지는 어느정도 시간이 소요될수 있고 이 시간동안 장애 영향이 지속적으로 발생 할 가능성이 있습니다. 
2. 전체 코드가 롤백됩니다.
  어쩌면 이케이스가 조금더 문제가 될 수 있습니다. 일부 기능 이슈로인해 전체 코드가 롤백된다면 빠르게 해당기능이 수정되지 않는다면 다른 기능들 배포가 함께 딜레이가 될 수도 있습니다. 또는 수정후 재배포를 했을때 다른 이슈가 또 발생해서 전체 롤백해야 될 가능성도 있습니다.

이러한 이슈로 인해 신규기능 개발/전환시에는 가능하면 기능별로 롤백을 시켜줄수 있는 장치를 준비해두는방식이 운영적인 측면에서 큰 도움이 될 수 있을것입니다.

---

### 피쳐토글 스위치 (feature toggle switch)

검색해보면 많이 나오지만 소개하려는 피쳐토글 스위치는 동적으로 기능을 on/off 할 수 있는 소프트웨어 전환 전략으로 볼수 있겠습니다. 이를 통해 어플리케이션의 특정 기능을 켜고 끔으로서 개발 및 유지보수를 더욱 유연하게 만들어주어 새로운 기능을 안정적으로 배포할 수 있습니다. 뿐만 아니라 특정 사용자 및 특정 요청에 대해서만 기능 on/off를 결정해줌으로서 신규 기능을 제한적으로 제공하거나 테스트 할 수 있는 기능 또한 적용 해 줄수 있습니다.

코드 예시를 통해 조금더 자세히 알아보도록 하겠습니다.

```java
public String doSomething(Param param){
    if (featureToggle.isEnabled(param.userNo)) {
        // 새로운 기능 실행
        newService.doSomething(param);
    } else {
        // 이전 기능 실행
        oldService.doSomething(param)
    }
}
```
```java
public class FeatureToggle {
    private static final boolean FEATURE_TOGGLE = true;
    private static final Long TOGGLE_WEIGHT = 10; // N/100
    private static final List<Long> USER_FILTER = List.of(100000L, 100001L); // N/100

    public boolean isEnabled(Long userNo) {
        return TFEATURE_TOGGLE && filterUser(memberId);
    }

    private boolen filterUser(Long userNo){
        return USER_FILTER.contains(userNo) || ((userNo % 100) < TOGGLE_WEIGH);
    }
}
```

정말 간단한 코드 예시를 통해 알아보았는데, 위와 같이 스위치 조건에따라 코드 분기를 나누어 실행시켜줄수 있습니다. 신규 개발한 기능의 변경부분을 대상으로 스위치를 추가한다면 조금더 간편하게 특정 기능에 대해서만 컨트롤 할수도 있고 비율 혹은 필터를 조정하여 특정 사용자 혹은 조건에서만 수행하도록해 A/B 테스트를 수행시킬수도 있습니다.

다만 위 코드방식으로는 스위치 조건들이 코드상에 고정되어있기 때문에 이슈상황시 컨트롤하기 어렵습니다. 조금더 개선된 방법으로는 환경변수 기반으로 설정하여 개발/운영등 환경별로 세팅을 다르게해서 코드분기를 적용 할 수도 있습니다.

```java
public class FeatureToggle {
    @Value("${feature-toggle.switch.enable}")
    private static final boolean featureToggle;

    @Value("${feature-toggle.switch.weight}")
    private static final Long toggleWeight; // N/100

    @Value("${feature-toggle.switch.user-filter}")
    private static final List<Long> userFilter;

    public boolean isEnabled(Long userNo) {
        return featureToggle && filterUser(memberId);
    }

    private boolen filterUser(Long userNo){
        return userFilter.contains(userNo) || ((userNo % 100) < toggleWeight);
    }
}
```

위방식으로 수행한다면 개발환경에서 switch를 켜고 테스트 진행 한 후에 운영환경에서 켜는 방식으로 수행 할 수도 있을것입니다. 다만, 이 방법으로도 급한 상황에서 대응은 어려울수 있습니다. 다른방법으로는 아래와같이 스위치 설정 데이터를 외부에서 가지고 오는 방법이 있습니다.

```java
@RequiredArgumentConstructor
public class FeatureToggle {
    private static boolean featureToggle = false;

    private static Long toggleWeight = 0L; // N/100

    private static List<Long> userFilter = List.of();

    private final SwitchRepository switchRepository;
    private final SwitchApiClient switchApiClient;

    @Scheduled(fixedDelay = 1000)
    public void settingToggle(){
        // SwitchSettings settings = switchRepository.getSettings();
        SwitchSettings settings = switchApiClient.getSettings();

        featureToggle = settings.getFeatureToggle();
        toggleWeight = settings.getToggleWeight();
        userFilter = settings.getUserFilter();
    }

    public boolean isEnabled(Long userNo) {
        return featureToggle && filterUser(memberId);
    }

    private boolen filterUser(Long userNo){
        return userFilter.contains(userNo) || ((userNo % 100) < toggleWeight);
    }
}
```

repository를 활용해 설정값들을 데이터저장소 (DB, Redis, ...)에 저장해두고 스위치 설정값을 동적으로 할당시켜주는 방식입니다. 여기서 `isEnabled()` 호출에서 매번 데이터를 읽어들여 동적으로 호출해주는 방법도 있겠지만 호출량이 많을경우 부하가 발생 할 수 있기때문에 내부적으로 데이터를 캐싱해두고 사용하는 방식이 외부 호출에 부하를 절감시키고 또한 에러 포인트를 줄일수 있는 방법입니다. 에러가 발생했을때 특정 DB 데이터를 직접 변경함으로써 빠르게 동적으로 스위치 on/off를 조절 할 수 있을것입니다.

별도 api를 호출하는 방법도 있을텐데 별도의 switch 관리 API를 두거나 좀더 가볍게 사용할수 있는 람다 서비스를 활용한다면 간단한 수정으로 스위치 수정사항을 반영 할 수도 있을것입니다.

---

### 마무리

대규모 시스템 리팩토링 작업을 진행하면서 유용하게 사용했던 피쳐토글 스위치에 대한 설명을 공유드렸습니다. 꼭 이방식이 아니더라도 점진적 전환 방식에서 기능단위 on/off 전략은 반드시 필요한 장치입니다.