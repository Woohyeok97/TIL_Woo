# 목차
- [인터셉션옵저버](#intersectionobserver)


<br/>

# Intersectionobserver
web에서 제공하는 DOM요소 관찰 API

- DOM요소의 뷰포트교차를 관찰함
- 동기적으로 실행되는 일반 이벤트와 달리, **비동기적**으로 실행되기 때문에 성능최적화

## 활용예시
- 스크롤 이벤트 : DOM요소를 관찰해서 데이터를 fetching, 애니메이션 등등
- Lazy Loading : 페이지 방문시, 동영상같은 무거운 컨텐츠를 특정DOM요소에 도달했을때 로딩하여 성능최적화
- **제품분석 :** 인터셉션옵저버는 DOM요소를 관찰할수있기 때문에 광고노출 & 사용자 행동분석 등등에도 활용할수있음

<br/>

## 사용예시(리액트)
1. useIntersectionObserver 이라는 커스텀 훅을 생성
2. 훅의 인자로 관찰할 DOM요소(usrRef)와 인터셉션옵저버 매개변수를 받음
3. 인터셉션옵저버가 관찰하는 DOM요소를 담을 state를 생성(ts 사용시, 타입은 **IntersectionObserverEntry**)
4. useEffect를 사용해서 인터셉션옵저버 객체를 생성하고 DOM요소를 관찰함(아래는 예시)
```js
useEffect(() => {
    const node = elementRef?.current; // 파라미터로받은 DOM요소(useRef)
    const hasIOSupport = !!window.IntersectionObserver; 

    // 관찰하는DOM or 브라우저가 인터셉션옵저버를 지원안하면 리턴해버림
    if(!node || !hasIOSupport) return


    const observerParams = { threshold, root, rootMargin }
    // 인터셉션 옵저버 객체생성(인자로 콜백함수, 옵션을 넣어줌)
    const observer = new IntersectionObserver(콜백함수, observerParams)

    // 생성한 인터셉션옵저버 객체를 이용해 DOM관찰시작
    // 옵션(observerParams)에 따라 node가 뷰포트와의 교차상태가 변화할때마다, 콜백함수(state변경함수)가 실행됨
    observer.observe(node)

    // 컴포넌트 언마운트때 DOM관찰중지
    return () => observer.disconnect()

    // 의존성 배열(커스텀 훅 파라미터)
}, [elementRef?.current, JSON.stringify(threshold), root, rootMargin])
```
5. 이렇게 useEffect로 인터셉션옵저버를 실행하고 state(관찰한 DOM요소)를 반환해서 컴포넌트에게 제공 

<br/>

## 내생각
웹어플리케이션에서 사용되는 무한스크롤이나, Lazy Loading를 비동기적으로 구현할수있다는 점이 장점같음.<br/>
하지만, 더 큰 장점은 사용자의 행동패턴, 광고노출을 추적하여 제품을 분석할수 있다는점 같음..<br/>
<br/>
인터셉션옵저버를 이용하면 제품을 성능적으로도 발전시킬수 있고,<br/>
**비지니스적**으로도 발전시킬수 있지않을까? 하는 생각이듬

<br/>
