## 목차
- [Infinite Queries](#infinite-queries)

<br/>

# React Query

### React Query
리액트에서 사용할수있는 **비동기 데이터 상태관리** 라이브러리<br/>
React Query를 사용하면 Axios같은 API요청 라이브러리를 보다 효율적으로 사용할수있음<br/>
<br/>

### React Query 기능
- **데이터 캐싱**
    - 동일한 요청에 대해서 데이터를 자동으로 캐싱해줌
    - 이로인해 네트워크 비용을 줄일수있고 사용자 경험도 향상시킴
    - 예를들어 이전페이지로 복귀했을때 캐시된 데이터를 사용해서 추가 API 요청이 필요없음

- **데이터 동기화**
    - React Query는 데이터를 자동으로 캐싱하긴 하지만, 설정에 따라 데이터를 자동으로 동기화 시킬수있음
    - 이러한 데이터 동기화를 통해서 서버 & 클라이언트의 데이터를 최신으로 일치시킬수있음

    - 아래처럼 useQuery()에 캐싱을 설정할수도 있고,
    ```js
    const { data, isLoading } = useQuery('쿼리key', API요청 함수, {
        // staleTime은 '오랜된 정도'를 설정함(설정한 시간동안 새로고침x)
        staleTime: 1000 * 60 * 5, 
        // refetchOnWindowFocus는 윈도우 포커스시, 데이터를 재요청함
        refetchOnWindowFocus: true, 
        // refetchInterval은 설정한 시간마다 백그라운드에서 데이터를 새로고침함
        refetchInterval: 1000 * 60 * 2, 
    }
    ```
    <br/>
    
    - 데이터 업데이트 이후, queryClient.invalidateQueries()를 사용해서 데이터를 동기화 시킬수도있음
    ```js
    // invalidateQueries()는 쿼리키를 '무효화' 하고 데이터를 재요청함 
    const mutation = useMutaion(API요청 함수, {
        onSuccess : () => queryClient.invalidateQueries('재요청할 쿼리key')
    })
    ```
    <br/>

- **요청 상태관리**
    - 요청상태, 에러여부 같은 서버 상태관리를 제공함
    - 때문에 별도의 state 없이, React Query가 제공하는 상태관리를 사용할수있음 

<br/>

### React Query가 매력적인 이유
단순히 Axios같은 API요청 라이브러리만 단독으로 사용하면 데이터상태도 개별적으로 관리해야하고<br/>
에러처리, 데이터 동기화 등등 개별적으로 관리를 해야함<br/>
또한, 별도의 캐싱없이는 매번 페이지 방문시 API요청이 발생해서 비용도 많이 발생하고, 사용자 경험도 떨어질수있음.<br/>
<br/>
React Query는 이러한 점들을 효율적으로 관리할수있음<br/>
특히 **'성능최적화'**, **'사용자경험 향상'** 이부분이 가장 큰 매력이라 생각함<br/> 

<br/>

# Infinite Queries

### Infinite Queries
리액트 쿼리의 useInfiniteQuery()를 사용하면 **배열형태**의 데이터 요청을 쉽게 관리할수있음.<br/>

예를들어 무한스크롤, 페이지네이션같이 추가적인 데이터를 요청하고 병합하는 **데이터페이징** 기능을 구현하는데 효과적임.<br/>

<br/>

useInfiniteQuery()는 아래와 같은 장점을 가짐
- **요청데이터 자동병합**
    - 요청한 배열데이터를 자동으로 병합해줌
    - 따로 상태관리 할필요 없음

- **요청상태, 요청메소드 제공**
    - 요청상태, 에러여부, 다음데이터 여부 등의 상태관리를 제공함
    - 또한 다음, 이전 데이터요청 메소드도 제공

- **데이터 캐싱을 통한 성능최적화**
    - 요청했던 데이터들을 자동으로 캐싱해줌
    - 데이터를 요청하고 다른 페이지 이동후 복귀해도, 별다른 재요청없이 캐싱된 데이터를 사용함
    - 이로인해 성능최적화도 가능하고 사용자 경험도 향상시킬수있음
    - 개인적으로 useInfiniteQuery()를 사용하는 가장 큰 이유라고 생각함

- **데이터 자동 업데이트**
    - 설정에 따라 백그라운드에서 데이터를 자동 업데이트 할수있음
    - 평소에는 캐시된 데이터를 사용하다가, 필요시 업데이트된 최신 데이터 요청가능
    - 사용자에게 최신의 데이터를 보장할수있다는 큰 장점이라 생각함

<br/>


### useInfiniteQuery()

다음은 예시로 가게데이터를 페이징하는 코드<br/>

우선 useInfiniteQuery()의 인자로들어갈 API요청 함수를 선언해줌.<br/>
이 함수는 불러올 페이지번호를 인자로 받고(여기선 초기값 1), 추가적인 데이터요청마다 실행됨.<br/>

```js
  // API 요청 함수
    const fetchStores = async ({ pageParam = 1 }) => {
        const { data } = await axios.get(`/api/stores?page=${pageParam}`)
        return data
    }
```
<br/>

useInfiniteQuery()의 인자로 **쿼리키**, **API 요청함수**와 **getNextPageParam()** 등을 넣어줌<br/>
그러면 데이터페이징에서 사용할수있는 다양한 값들을 반환함

```js
const { data : storeList, isFetching, fetchNextPage, isFetchingNextPage, hasNextPage, isLoading, isError } = useInfiniteQuery(
    'stores', 
    fetchStores,
    {
        getNextPageParam : (lastPage) => lastPage?.data?.length > 0 ? lastPage.page + 1 : undefined
    })
```

- **data** : 데이터결과 객체
    - data.pages : 현재까지 요청한 데이터배열(여기서 **자동병합**이 되는거임)
    - data.pageParams : 현재까지 사용된 페이지번호(fetchStores의 파라미터)배열

- **요청상태** : 따로 상태관리할 필요없이 useInfiniteQuery()가 요청상태를 제공함
    - isFetching : 데이터요청상태(불리언)
    - isFetchingNextPage : 다음페이지 데이터요청 상태(불리언)
    - hasNextPage : 다음페이지 데이터가 있는지(불리언)
    - isLoading : 로딩상태(불리언)
    - isError : 에러여부(불리언)

- **요청메소드** : 간편하게 추가요청을 실행할수있음
    - fetchNextPage : 다음페이지 데이터요청 메소드
    - fetchPreviousPage : 이전페이지 데이터요청 메소드

<br/>

또한, getNextPageParam를 사용해서 다음 데이터요청의 페이지번호(fetchStores의 파라미터)를 설정해줄수있음.<br/>
여기서 인자(lastPage)는 API요첨 함수(여기선 fetchStores)의 반환값 의미함(즉, 가장 최근 data.pages)<br/>

```js
    getNextPageParam : (lastPage) => lastPage?.data?.length > 0 ? lastPage.page + 1 : undefined
```

이후로 fetchNextPage, fetchPreviousPage 메소드로 다음, 이전 데이터를 요청할수있음.<br/>

<br/>

