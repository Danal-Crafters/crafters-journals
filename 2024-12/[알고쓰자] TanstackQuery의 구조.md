[ 크래프터스 : \[알고쓰자\] TanstackQuery의 구조 ]
===============================================================

![image](https://github.com/user-attachments/assets/71e34ec6-9653-4a7c-8264-06129773d7f8)

Intro
=====

프론트엔드 개발을 하며 여러 상태관리 툴들을 사용하지만, 아마 정말 많이
쓰시는 게 `Tanstack Query` 일 것입니다.

상태(State) 라는 개념은 해당 값의 변화로 인해 프론트엔드의 컴포넌트에
변화를 줄 수 있는 데이터를 의미합니다. 이 데이터는 클라이언트 사이드의
데이터가 될 수도 있고 서버사이드의 데이터가 될 수도 있죠.
`Tanstack Query` 는 서버사이드의 상태를 관리합니다.

서버사이드의 상태라는 개념을 좀 쉽게 정리해보면, 서버에서 패칭 해서 받아
온 데이터를 의미합니다. 서버에게 HTTP 요청을 보내서 받은 데이터를
컴포넌트를 랜더링하는 데 사용한다면 그 값은 상태가 될 수 있습니다.
`Tanstack Query` 는 이런 데이터들을 관리하는 데 도움을 줍니다.

우리가 해당 값들을 매번 서버에 패칭하지 않아도 `staleTime`, `gcTime`
설정 값에 따라 인메모리의 캐싱 값을 참조할 수도 있습니다. `REST API` 가
멱등성을 지니거나 굳이 데이터를 자주 패칭 할 필요가 없다면 이런 캐싱
값을 참조하는 게 프론트엔드 애플리케이션의 성능에 큰 영향을 끼칩니다.

> 예를 들면 쇼핑몰의 이벤트 배너 정보를 굳이 새로고침 하는 게 아닌 이상
> 매번 메인화면이 랜더링 될 때마다 패칭하는 것 보다는 컴포넌트 마운트
> 시점에서 패칭 한 값을 쓰는 게 성능에 도움이 될 것입니다.

덕분에 편리하게 프론트엔드 애플리케이션을 개발하지만, 어쨌든 써드파티
라이브러리고 `Tanstack Query` 에 의존하는 코드를 작성하게 되는 이슈가
발생합니다.

과거에 `React Query` 에서 `Tanstack Query` 로 변경되는 과정에서, 그리고
버전 업 되는 과정에서 상당히 많은 코드들을 수정 해야 했던 경험이 있는
저는 이 라이브러리를 스스로 제대로 알고 쓰고 있는 지 의문이 들었습니다.

그래서 이번 포스팅에서는 이 `Tanstack Query` 의 구조를
살펴보고 동작 원리를 알아보려고 합니다.

Tanstack Query 라이브러리 구조
==============================

> tanstack query 5.51.24 버전 기준으로 작성 되었습니다.

```bash
yarn add @tanstack/react-query
```

```bash
@tanstack
  ├── query-core
  │     ├── notifyManager.ts
  │     ├── query.ts
  │     ├── queryCache.ts
  │     ├── queryClient.ts
  │     └── queryObserver.ts
  │
  └── react-query
        ├── useBaseQuery.ts
        └── useQuery.ts
```

`Tanstack Query` 를 설치하면 `query-core` 모듈이 설치 됩니다.
`react-query` 를 설치했다면 위와 같을 것이고 `vue-query` 를 설치했다면
코어모듈은 그대로일테고 `vue-query` 디렉토리가 생성된다는 게 다른
점입니다. 해당 모듈은 `vue` 나 `react` 에서 코어모듈을 사용하도록
도와줍니다.

Query Core
----------

그럼 먼저 `Query Core` 먼저 살펴보아야겠죠. `Query Core` 에서 중요한
요소 네가지는 아래와 같습니다.

-   QueryClient

-   QueryCache

-   QueryObserver

-   Query

이 객체들의 관계성을 이해해 보는 것 먼저 시작 해 보겠습니다.

### QueryClient 

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

// QueryClient 인스턴스 생성
const queryClient = new QueryClient()

function App() {
  return (
    // 리액트 앱에 적용시키기
    <QueryClientProvider client={queryClient}>
      <RestOfYourApp />
    </QueryClientProvider>
  )
}
```

`QueryClient` 는 `React context api` 를 사용하여 애플리케이션 전역적으로
사용할 수 있습니다. `QueryClientProvider` 의 `client` 로 넘겨주면 해당
인스턴스를 전역적으로 사용할 수 있습니다. 따로 여러개의 Provider를 만들
지 않는다면 1개만 존재합니다.


``` typescript
export class QueryClient {
  #queryCache: QueryCache
  #mutationCache : MutationCache
  #defaultOptions : DefaultOptions
  #queryDefaults : Map<string, QueryDefaults>
  #mutationDefaults : Map<string, MutationDefaults>
  #mountCount : number
  #unsubscribeFocus?: () => void
  #unsubscribeOnline? : () => void

  constructor(config: QueryClientConfig = {}) {
    this.#queryCache = config.queryCache || new QueryCache()
    //이하 생략
  }
```

`QueryClient` 인스턴스가 생성될 때 `queryCache` 값을 가지게 됩니다.

### QueryCache

``` typescript
interface QueryCacheconfig {
    onError?: (
        error: DefaultError,
        query: Query<unknown, unknown, unknown>,
    ) => void
    onSuccess?: (data: unknown, query: QueryQuery<unknown, unknown, unknown>) => void
    onSettled?: (
        data: unknown | undefined,
        error: DefaultError | null,
        query: Query<unknown, unknown, unknown>,
    ) => void
}

export interface QueryStore {
    has: (queryHash: string) => boolean
    set: (queryHash: string, query : Query) => void
    get: (queryHash: string) => Query | undefined
    delete: (queryHash: string) => void 
}

// CLASS

export class QueryCache extends Subscribable<QueryCacheListener> {
  #queries: QueryStore

  constructor(public config: QueryCacheConfig = {}) {
    super()
    this.queries = new Map<string, Query>()
  }
  ...
  add(query: Query<any, any, any, any>): void {
    if (!this.#queries.has(query.queryHash)) {
      this.#queries.set(query.queryHash, query)
      this.notify({
        type: 'added',
        query,
      })
    }
  }
  ...
}
```

`QueryCache` 는 `queries` 인스턴스에 `Query` 인스턴스들을 저장합니다.
코드 상에서는 생략 되었지만, `useQuery()` 의 인자로 넘겨 준 `queryKey`
를 사용해 `queryHash` 를 만들고 `queries` 인스턴스의 key로 사용합니다.
그리고 value로 `Query` 가 저장됩니다.

`queries` 는 Map 형태로 구성 되어있습니다. Key 값을 알고 있다면 전달받은
요청이 이전에 들어온 요청과 동일한 요청인 지 판단할 수 있습니다.

### Query 

```typescript
export class Query<

  TQueryFnData = unknown,
  TError = DefaultError,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey,
> extends Removable {
  queryKey: TQueryKey
  queryHash: string
  options!: QueryOptions<TQueryFnData, TError, TData, TQueryKey>
  state: QueryState<TData, TError>
  isFetchingOptimistic?: boolean
  
  #initialState: QueryState<TData, TError>
  #revertState?: QueryState<TData, TError>
  #cache: QueryCache
  #retryer?: Retryer<TData>
  observers: Array<QueryObserver<any, any, any, any, any>>
  #defaultOptions?: QueryOptions<TQueryFnData, TError, TData, TQueryKey>
  #abortSignalConsumed: boolean

  constructor(config: QueryConfig<TQueryFnData, TError, TData, TQueryKey>) {

    super()
    this.observers = []
    this.#cache = config.cache
    this.queryKey = config.queryKey
    this.queryHash = config.queryHash
    this.#initialState = getDefaultState(this.options)
    this.state = config.state ?? this.#initialState
    ...
}
```

`Query` 는 자신을 가지고있는 `QueryCache` 와 자신의 상태가 변경되었을 때
호출할 옵저버를 가집니다. 만약 구독 요청이 들어왔을 때 `this.observers`
에 옵저버가 추가됩니다. 상태가 변경된다면 등록 된 옵저버들을 호출합니다.

`this.state` 에는 `data`, `isLoading`, `isFetching` 등 `useQuery()` 를
통해 얻을 수 있는 정보들이 담겨있습니다.

### QueryObserver

```typescript
export class QueryObserver<
  TQueryFnData = unknown,
  TError = DefaultError,
  TData = TQueryFnData,
  TQueryData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey,
> extends Subscribable<QueryObserverListener<TData, TError>> {
    super()

    this.#client = client
    this.#selectError = null
    this.bindMethods()
    this.setOptions(options)
   ...
    #trackedProps = new Set<keyof QueryObserverResult>()

    protected onSubscribe(): void {
      if (this.listeners.size === 1) {
        this.#currentQuery.addObserver(this)

        if (shouldFetchOnMount(this.#currentQuery, this.options)) {
          this.#executeFetch()
        } else {
          this.updateResult()
        }
        this.#updateTimers()
      }
    }

  ...

    #updateQuery(): void {
      const query = this.#client.getQueryCache().build(this.#client, this.options)
  
      if (query === this.#currentQuery) {
        return
      }

      const prevQuery = this.#currentQuery as
        | Query<TQueryFnData, TError, TQueryData, TQueryKey>
        | undefined
      this.#currentQuery = query
      this.#currentQueryInitialState = query.state

      if (this.hasListeners()) {
        prevQuery?.removeObserver(this)
        query.addObserver(this)
      }
    }
  ...

}
```

`QueryObserver` 는 `QueryClient` 와 `Query` 를 가지게 됩니다.
`trackedProps` 는 우리가 어떤 상태들을 요청했는 지를 기록하는 역할을
합니다.

구독 요청이 들어오면 `Query` 에서 옵저버로 `QueryObserver` 가
등록됩니다.

### 구조도

![image](https://github.com/user-attachments/assets/60aeb7a9-27b6-413d-abfb-0850bb6a0b9a)


`QueryClient` 내에 `QueryCache` 가 생성되고, `QueryCache` 에 존재하는
`QueryStore` 에 `Query` 정보들을 저장합니다.

물론 `Query` 들 또한 생성될 때 본인들이 어떤 `Store` 에 저장되어있는
지를 기록합니다.\
그리고 `QueryObserver.onSubscribe()` 가 호출되면 `Query` 의 `observers`
에 해당 `QueryObserver` 가 등록됩니다.

`QueryObserver` 가 생성될 때 `QueryClient` 에 등록되고, 이전에 가지고
있던 `Query` 가 없거나 새로 업데이트 되면 `this.query` 를 최신화 합니다.

`Query` 와 `QueryObserver` 간에 관계를 살펴보면, `useQuery` 에서 같은
Key를 사용한다면 동일한 `Query` 를 여러개의 `QueryObserver` 가 공유하게
됩니다. `Query` 를 공유하기 때문에 과거에 해당 Key 값에 대한 패칭 비동기
함수를 실행했을 경우 추가적으로 요청하지 않고 `Query` 에 등록되어 있던
옵저버들을 호출하여 리랜더링을 진행합니다.

### NotifyManager

```typescript
export function createNotifyManager() {

  let queue: Array<NotifyCallback> = []
  let transactions = 0
  let notifyFn: NotifyFunction = (callback) => {
    callback()
  }
  let batchNotifyFn: BatchNotifyFunction = (callback: () => void) => {
    callback()
  }
  let scheduleFn: ScheduleFunction = (cb) => setTimeout(cb, 0)
  const setScheduler = (fn: ScheduleFunction) => {
    scheduleFn = fn
  }

  const batch = <T>(callback: () => T): T => {
  ...
  }

  const schedule = (callback: NotifyCallback): void => {
  ...
  }
  
  const batchCalls = <T extends Array<unknown>>(
    callback: BatchCallsCallback<T>,
  ): BatchCallsCallback<T> => {
  ...
  }

  const flush = (): void => {
  ...
  }

  return {
    batch,
    batchCalls,
    schedule,
    ...
  } as const

} 

// SINGLETON

export const notifyManager = createNotifyManager()
```
:::
:::

`notifyManager` 는 싱글톤으로 관리되며 클로저를 사용합니다. `Query`,
`QueryCache`, `QueryObserver` 는 `notifyManager` 를 이용해 상태변경,
옵저버 추가 등의 이벤트를 서로에게 알려줍니다. 또한 `notifyManager`
내부에서 컴포넌트 리렌더링이 최대한 동시에 일어날 수 있도록 `batch` 기능
또한 제공합니다.

클로저에 대해서는 추후 다른 포스팅에서 자세히 설명하겠지만, 조금만
설명하고 넘어가자면 1급 객체입니다. 함수를 객체로 선언하는 방법입니다.
특정한 함수를 객체로 지정하고 이름을 붙힘으로서 `어휘적 범위 지정`, 즉
의미를 가지는 인스턴스로써 선언하는 방법입니다.

useQuery Hook 의 실행 흐름
==========================

지금까지 Tanstack Query core 구조에 대해 알아보았으니 이제 유저가
useQuery Hook 을 실행시켰을 때 실행 흐름과 React 컴포넌트의 리랜더링
과정까지를 한번 살펴 보겠습니다.


```typescript
import axios from 'axios';
import { useQuery } from '@tanstack/react-query';

const DanalComponent = () => {
  const { isLoading, error, data, isFetching } = useQuery({
    queryKey: ['danalData'],
    queryFn: () =>
      axios
        .get('<https://danal.api.com/api/v1/test>')
        .then((res) => {
          return res.data;
        }),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>An error has occurred: + {error.message}</div>;
  return (<div>{JSON.stringify(data)}</div>)
};

export default DanalComponent;
```

```typescript
export function useQuery<
  TQueryFnData,
  TError = DefaultError,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey,
>(
  options: UseQueryOptions<
    TQueryFnData,
    TError,
    TData,
    TQueryFnData,
    TQueryKey
  >,
  queryClient?: QueryClient,
):
  | UseQueryReturnType<TData, TError>
  | UseQueryDefinedReturnType<TData, TError> {
  return useBaseQuery(QueryObserver, options, queryClient)
}
```

`useQuery` 를 통해 TODO 데이터를 가져오는 예시입니다. `useQuery` 를
호출하면 전달받은 인자들을 통해 `useBaseQuery` Hook 을 호출합니다. 이때
전달 된 옵션과 정의 된 `QueryObserver` 클래스를 전달합니다.


```typescript
export function useBaseQuery(options, Observer) {
  const queryClient = useQueryClient({ context: options.context })
  const defaultedOptions = queryClient.defaultQueryOptions(options)

  const [observer] = React.useState(() => new Observer(queryClient, defaultedOptions))
  const result = observer.getOptimisticResult(defaultedOptions)
  
  useSyncExternalStore(
      React.useCallback(      
          (onStoreChange) => {        
              const unsubscribe = observer.subscribe(notifyManager.batchCalls(onStoreChange))        
              // Update result to make sure we did not miss any query updates        
              // between creating the observer and subscribing to it.        
              observer.updateResult()        
              return unsubscribe      
              },      
              [observer],    
        ),    
        () => observer.getCurrentResult(),    
        () => observer.getCurrentResult(),  
    )
  // Handle result property usage tracking
  return !defaultedOptions.notifyOnChangeProps ? observer.trackResult(result) : result
}
```

`useBaseQuery` 에서 Provider 를 통해 전달 받은 `queryClient` 를 가져오고
`QueryObserver` 를 `useState` 를 통해 생성합니다. `Query` 의 변화를
관찰한 `QueryObserver` 는 리액트 컴포넌트를 리랜더링 시키기 위해서
`useSyncExternalStore` 훅을 사용하게 되는데 콜백 파라미터로
`onStoreChange`, 즉 훅을 호출하는 리액트 컴포넌트에 대해 리랜더링을
지시하는 함수를 전달 받습니다.


``` typescript
export class Subscribable<TListener extends Function> {
  protected listeners = new Set<TListener>()

  constructor() {
    this.subscribe = this.subscribe.bind(this)
  }

  subscribe(listener: TListener): () => void {
    this.listeners.add(listener)
    this.onSubscribe()

    return () => {
      this.listeners.delete(listener)
      this.onUnsubscribe()
    }
  }

  hasListeners(): boolean {
    return this.listeners.size > 0
  }

  protected onSubscribe(): void {
    // Do nothing
  }

  protected onUnsubscribe(): void {
    // Do nothing
  }
}
```

`onStoreChange` 는 `observer.subscribe(onStoreChange)` 를 따라
이동합니다. `onStoreChange` 는 `listeners` 에 등록됩니다.


``` typescript
export class QueriesObserver<
  TCombinedResult = Array<QueryObserverResult>,
> extends Subscribable<QueriesObserverListener> {

  constructor() {}
  ...
  #notify(): void {
    notifyManager.batch(() => {
      this.listeners.forEach((listener) => {
        listener(this.#result)
      })
    })
  }
}
```

`notify` 메소드는 등록 된 `listeners` 를 순회하며 `listener` 함수를 실행
시킵니다.

이 과정에서 `QueryObserver` 인스턴스와 연결 된 컴포넌트들에 대해
리랜더링을 지시합니다. 그 결과 컴포넌트가 리렌더링 되며 `Query` 의 변경
된 데이터를 표시합니다.

정리 해 보면, `useQuery` 훅을 호출하게 되면 `Query` 의 변화를 관찰하는
`QueryObserver` 가 생성되어 컴포넌트와 연결됩니다. `Query` 의 변화에
따라 컴포넌트의 리렌더링이 발생하며 변경 된 데이터가 반영 됩니다.

Outro
=====

지금까지 `Tanstack Query` 라이브러리의 구조, 그리고 `useQuery` Hook API
를 사용했을 때 컴포넌트에 데이터가 반영되는 과정에 대해 알아 보았습니다.

처음 라이브러리를 도입했을 때 동작 원리를 잘 몰라서 사용하는 데 애를
먹었던 기억이 났습니다. 라이브러리를 도입하는 것도 중요하지만, 내부
구조를 파악하고 원리를 이해한다면 결과물의 유지보수성에 큰 도움이 되지
않을까요? 😊

Reference
=========

<https://fe-developers.kakaoent.com/2023/230720-react-query/>

<https://www.timegambit.com/blog/digging/react-query/01>\
<https://www.timegambit.com/blog/digging/react-query/02>\
<https://tkdodo.eu/blog/inside-react-query>\
<https://react.dev/reference/react/useSyncExternalStore>

------------
<div align="center">
    
|Profile|Link|Bio|
|--|--|--|
|<img src="https://github.com/user-attachments/assets/24d3f306-8014-41ae-98a3-a2f94bf6bdb9" width="100" height="100"/> | [github](https://github.com/swj9707) <br/> [LinkedIn](https://www.linkedin.com/in/woojin-son-18989b202/) |**"천천히, 하지만 꾸준히!"**|

</div>
