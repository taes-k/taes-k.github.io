---
layout: post
comments: true
title: 네트워크 timeout의 종류들
tags: [network, timeout]
---

### 네트워크 타임아웃

어느정도 규모가 있는 서비스라면 보통 외부와의 통신없이 사용하는 stand-alone 으로 사용하는 경우는 드물다고 할 수 있습니다. 서비스 특성에 따라 일반적으로 사내 API 혹은 외부 API 들을 연계해서 사용하는것이 대부분일텐데 이때 각 서비스들간에 네트워크 통신을 하게 될 것 입니다. 그리고 그 과정속에서 `TimeoutException` 에러를 흔하게 받아보셨을거라 생각합니다. 이번 포스팅에서는 네트워크 통신에서 발생하는 여러가지 `Timeout`에 대해서 정리해보려고 합니다.

---

### RestTemplate

먼저 `Timeout`의 종류를 알아보기위해 Spring에서 HTTP Request를 위해 사용하는 `RestTemplate`를 살펴보도록 하겠습니다. RestTemplate의 생성자에는 `HttpComponentsClientHttpRequestFactory`라는 파라미터를 통해 다양한 연결정보 설정을 할 수 있도록 되어 있습니다. 

```java
// package org.springframework.http.client.HttpComponentsClientHttpRequestFactory

public class HttpComponentsClientHttpRequestFactory implements ClientHttpRequestFactory, DisposableBean {

    ...

	public void setConnectTimeout(int timeout) {
        ...
	}

	public void setConnectionRequestTimeout(int connectionRequestTimeout) {
		...
	}

    public void setReadTimeout(int timeout) {
        ...
	}

    // httpClient(socketTimeout)
    public void setHttpClient(HttpClient httpClient) {
		...
	}
}
```

정리해보자면 다음과 같은 HTTP 통신을 위해 다음과 같은 `Timeout`들을 정의하고 있음을 확인 하실수 있습니다.

- connectionTimeout
- connectionRequestTimeout
- readTimeout
- socketTimeout

---

### Timeout-s

각각의 타임아웃들이 무엇을 의미하는지 아래 그림으로 알아보도록 하겠습니다.

![1]({{ site.images | relative_url }}/posts/2022-01-22-read-timeout/1.png)   

가장먼저 `Connection timeout`은 실제 request를 하기전, 말그대로 연결을 맺는데 걸리는 유효시간 입니다. 3-handshaking 과정을 통해 클라이언트와 서버간의 `Connection`을 생성하는데 걸리는 유효시간을 의미합니다. 일반적으로 네트워크 상태가 불량할경우 timeout을 초과할때가 많습니다.

일반적으로 가장 많이 custom 수정되는 `Read timeout`은 요청을 보내고 응답을 받는데까지 걸리는 유효시간을 의미합니다. 일반적으로 서버측에서 처리하는 시간이 길어지거나 (`long-transaction`) 요청/응답에 보내지는 네트워크 데이터가 큰 경우에 timeout을 초과할때가 많습니다.

다음으로 `Socket timeout`은 네트워크상에서 데이터는 패킷단위로 전송되어지는데, 연속된 다음 패킷을 받는데까지 걸리는 유효시간을 의미합니다. 일반적으로 패킷사이즈가 크거나 네트워크 상태가 불량할경우 timeout을 초과할때가 많습니다.

마지막으로 그림에는 `connectionRequestTimeout`이 존재하지 않는데, 이는 `httpClient` 구현상에서 `connection-pool`을 정의해두고 `Connection`을 재사용하는데 connection-pool에서 connection을 가지고 올때까지 걸리는 유효시간을 의미합니다.

---

### timeout의 발생 상황

이제 위에서 timeout의 종류와 개념에 대해 알아보았으니 각각의 timeout이 발생하는 상황들을 유추해 볼 수 있을것같습니다.

#### connectionTimeout
- 해당 네트워크 경로(호스트, 포트)에 제공되는 서비스가 정상정이지 않은경우
- 서버가 connection을 받을수 없는 상태인 경우
- 네트워크지연으로 인한 connection 수행이 지연되는 경우

#### connectionRequestTimeout
- connection-pool의 모든 connection이 사용중인 경우
- 새로운 connection을 만드는 과정에서 네트워크지연이 발생한경우

#### readTimeout
- connection-pool의 모든 connection이 사용중인 경우
- 새로운 connection을 만드는 과정에서 네트워크지연이 발생한경우

#### socketTimeout
- connection-pool의 모든 connection이 사용중인 경우
- 새로운 connection을 만드는 과정에서 네트워크지연이 발생한경우
