---
title: Next.js 15 변경된 cache 설정에 대해 
date: 2024-12-12 14:10:38 +0800
categories: [nextjs, cache]
tags: [nextjs]     # TAG names should always be lowercase
---

## Next.js v15의 변경점

[Next.js v15 업데이트 문서](https://nextjs.org/blog/next-15)를 보면 크게 바뀐 점은 두 가지라고 생각함.

### 1. **[Async Request APIs](https://nextjs.org/blog/next-15#async-request-apis-breaking-change)**

`headers`, `cookies`, `params`, `searchParams`와 같은 API들이 비동기 방식으로 전환됨.  
이런 전환의 이유는 불필요한 **Blocking**을 줄여 렌더링을 더 빠르게 하기 위함임.

기존에는 모든 컴포넌트 렌더링이 요청 데이터를 기다리며 지연되었지만, 실제로 일부 컴포넌트는 요청 데이터에 의존하지 않음. 이런 컴포넌트들을 **즉시 렌더링**할 수 있도록 개선된 것임.

> 이 변경사항은 큰 변화이기 때문에 **v15에서는 기존 API도 병행해서 사용 가능함**. 단, 다음 메이저 버전까지 지원된다는 경고가 나옴.


### 2. **[기본 cache 규칙이 `cached`에서 `uncached`로 전환됨](https://nextjs.org/blog/next-15#caching-semantics)**

이 부분은 상당히 큰 영향을 끼칠 수 있음. 버전 업그레이드 후에도 정상적으로 페이지가 표시되기 때문에 이 변경점을 놓치기 쉬움.

이 점을 제대로 이해하지 못하고 캐시 설정을 변경하지 않으면 **과도한 fetch 요청**이 서버에 부하를 일으킬 수 있음. 심할 경우 **서버 다운**이나 **socket 최대 갯수 초과** 문제가 발생할 수 있음.

---

## Next.js App Router에서 fetch 동작 확인

Next.js App Router에서 Server Component 렌더링 시 fetch 동작을 확인해봤음.  
공식 문서에 따르면 여러 컴포넌트가 각각 fetch를 호출해도 **중복된 요청은 Next.js가 자동 병합**해서 처리해줌.

![deduplicated fetch requests](/assets/2024-12-17-nextjs-15-cache-changes-tested/deduplicated-fetch-requests.png)

이 동작은 React의 컴포넌트 지향적인 방식과 잘 어울리며, **컴포넌트 간 의존성을 줄이는 장점**이 있음.


### v15로 업데이트 시 `uncached` 기본값 문제

v15에서 캐시의 기본값이 `uncached`로 바뀌면서 다음과 같은 문제가 발생할 수 있음:

1. **모든 요청이 API 서버로 전송됨**.
2. 페이지 하나에 **수십~수백 개의 API 요청**이 발생, API 서버에 과부하가 걸릴 수 있음.

기존 동작을 기대한다면 **fetch 옵션에서 `cache: "force-cache"`를 설정해야 함**.

```typescript
async function getPost(id: string) {
  const res = await fetch(`https://api.vercel.app/blog/${id}`, {
    cache: "force-cache",
  });
  const post: Post = await res.json();
  if (!post) notFound();
  return post;
}
```

---

## fetch 캐시 유효성 측정

`cache: "force-cache"`를 설정하면 문제가 해결된 것 같지만, **정말 캐시가 유효하게 동작하는지 확인**이 필요함.  
설정이 잘못되어도 페이지는 정상 표시되기 때문에, 서버 과부하나 요금 폭탄을 맞기 전에 징후를 확인해야 함.


### 측정 방법

클라이언트 측 **패킷 캡처**를 통해 캐시 상태를 확인함. 사용된 환경 및 툴:

- **MacOS**
- **tcpdump**
- **Wireshark**

1. **API 서버 주소 확인**

```bash
$ nslookup www.sample.com
```

2. **패킷 캡처 시작**

```shell
$ tcpdump -s 0 -w ~/capture.log dst host 5.22.145.121 or dst host 5.22.145.16
```

3. **Wireshark 분석**  
   tcpdump로 저장된 파일을 열고, **`Statistics → Protocol Hierarchy`** 메뉴에서 트래픽 확인.

---

### 측정 결과

1. **v14 (캐시 설정 없음)** → 정상 캐싱  
   ![v14 정상 캐싱](/assets/2024-12-17-nextjs-15-cache-changes-tested/v14-cached.png)

2. **v14 (캐시: `no-store`)** → 캐시 비활성화  
   ![v14 uncached](/assets/2024-12-17-nextjs-15-cache-changes-tested/v14-uncached.png)

3. **v15 (캐시 설정 없음)** → 캐시 비활성화 (트래픽 증가 확인)  
   ![v15 uncached](/assets/2024-12-17-nextjs-15-cache-changes-tested/v15-uncahed.png)

4. **v15 (캐시: `force-cache`)** → 기존 v14와 동일 (캐싱 활성화)  
   ![v15 cached](/assets/2024-12-17-nextjs-15-cache-changes-tested/v15-cached.png)


## 결론

측정 결과 **v15에서는 `cache` 옵션을 지정하지 않으면 캐시가 비활성화**된다는 점이 확인됨.  
기존 동작을 유지하려면 반드시 `cache: "force-cache"` 옵션을 설정해야 함.

---

## 마무리

"추측하지 말고 **측정하라**"는 말을 다시 한번 떠올리게 됨.  
설정이 기대대로 동작하는지 검증하는 습관이 중요하다고 생각함.

