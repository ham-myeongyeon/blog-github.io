---
title: swr 내부분석
date: '2015-05-01T22:12:03.284Z'
description: 'swr라이브러리 코드를 구현해보면서 학습한 내용을 글로 정리했습니다. 이 글은 swr라이브러리의 사용법 보다는 내부를 파악한 내용으로 구성되어 있습니다. swr내부에는 크게 어떤 요소들로 이루어져 있고, 또 어떤 흐름으로 활용되는지 설명하겠습니다.'
---

swr라이브러리 코드를 구현해보면서 학습한 내용을 글로 정리했습니다. 이 글은 swr라이브러리의 사용법 보다는 내부를 파악한 내용으로 구성되어 있습니다. swr내부에는 크게 어떤 요소들로 이루어져 있고, 또 어떤 흐름으로 활용되는지 설명하겠습니다.

## SWR 주요 요소들

앞으로 설명드릴 주요 요소들의 코드 디렉토리는 다음과 같이 있습니다.

```
├── _internal
│   └── utils
│       ├── config-context.ts
│       ├── config.ts
│       ├── global-state.ts
│       ├── resolve-args.ts
│       └── use-swr-config.ts
│           ...
└── core
    └── use-swr.ts
				...
```

전체적으로 크게 `SWRGlobalState`와 `config`, 그리고 `useSwr`로 구성되어 있습니다.

![기본구조.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c0fc1a43-1127-4384-85d1-5fff37e02f46/0eb3b4cf-22e6-4f2d-b07f-993e1ef86039/%E1%84%80%E1%85%B5%E1%84%87%E1%85%A9%E1%86%AB%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9.png)

### SWRGlobalState

![SwrGlobalState.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c0fc1a43-1127-4384-85d1-5fff37e02f46/f8401433-2491-4395-bb80-029f592ac4f8/SwrGlobalState.png)

`SWRGlobalState`는 `swr`에서 관리되는 `cache`를 키로 하고, 전역에서 상태를 관리하는 요소들로 이루어진 `GlobalState`를 값으로 맵핑한 `WeakMap` 자료구조로 되어 있습니다.

<aside>
💡 `WeakMap`?
`WeakMap`은 `Map`과 같은 객체 자료구조로이지만 차이점은 키로 원시값이 대신 객체만 받게 됩니다. 여기서는 키로 `Cache`라는 객체를 받게 됩니다. 
또한 `Map`과의 차이점은 프로그램 내에 객체에 대한 참조가 `WeakMap`을 제외하고 존재하지 않으면 가비지로 수거되는 특징이 있습니다.
그래서 메모리 누수에서 이점이 있고 `swr`에서는 이러한 이점을 이용하기 위해 `WeakMap`을 이용한것으로 보입니다.

</aside>

여기서 `cache`는 데이터를 저장하고 활용하는 메커니즘이며 형태는 `useSwr`을 구분하는 고유한 식별자 `key`와, 그에 해당하는 데이터 `state`를 맵핑하는 인스턴스입니다.

`GlobalState`는 `swr`에서 사용되는 상태를 관리하는 요소들의 집합이며 `Revalidators`, `Mutations`, `Fetch`, `Preload`, `Mutator`, `Setter`, `Subscriber`등을 배열로 저장합니다.

```tsx
// global-state.ts

export const SWRGlobalState = new WeakMap<Cache, GlobalState>()

...

export type GlobalState = [
  Record<string, RevalidateCallback[]>, // Revalidators
  Record<string, [number, number]>, // Mutations: [ts, end_ts]
  Record<string, [any, number]>, // Fetch: [data, ts]
  Record<string, FetcherResponse<any>>, // Preload
  ScopedMutator, // Mutator
  (key: string, value: any, prev: any) => void, // Setter
  (key: string, callback: (current: any, prev: any) => void) => () => void // Subscriber
]

...

export interface Cache<Data = any> {
  keys(): IterableIterator<string>
  get(key: string): State<Data> | undefined
  set(key: string, value: State<Data>): void
  delete(key: string): void
}

```

### Config

![Config.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c0fc1a43-1127-4384-85d1-5fff37e02f46/35949df9-a5d5-4927-8f94-e8ad9f423187/Config.png)

`config`는 `swr`에 사용되는 옵션 값들과 `cache`를 담고 있으며 `useSwr`에 제공합니다.

우선 `config`의 구성은 이미 선언된 `defaultConfig`와 선택적으로 지정하여 사용할 수 있는 상위 `SWRConfigContext`에서 상속받은 값을 병합하여 최종적으로 형성됩니다.

```tsx
// config.ts

const [cache, mutate] = initCache(new Map()) as [Cache<any>, ScopedMutator]

export const defaultConfig: FullConfiguration = mergeObjects(
  {
    onLoadingSlow: noop,
    onSuccess: noop,
    onError: noop,
    onErrorRetry,
    onDiscarded: noop,

    revalidateOnFocus: true,
    revalidateOnReconnect: true,
    revalidateIfStale: true,
    shouldRetryOnError: true,

    errorRetryInterval: slowConnection ? 10000 : 5000,
    focusThrottleInterval: 5 * 1000,
    dedupingInterval: 2 * 1000,
    loadingTimeout: slowConnection ? 5000 : 3000,

    compare,
    isPaused: () => false,
    cache, //초기 캐시값
    mutate,
    fallback: {},
  },
  preset,
)
```

여기서 상위 `config`는 `SWRConfig`(`SWRConfigContext.Provider`)를 통해서 형성되며, 이는 하위 컴포넌트에 전달 합니다.

```tsx
// config-context.ts

export const SWRConfigContext = createContext<Partial<FullConfiguration>>({})

const SWRConfig: FC<
  PropsWithChildren<{
    value?: Config | ((parentConfig?: Config) => Config)
  }>
> = props => {
  const { value } = props
  const parentConfig = useContext(SWRConfigContext)
  const isFunctionalConfig = isFunction(value)
  const config = useMemo(
    () => (isFunctionalConfig ? value(parentConfig) : value),
    [isFunctionalConfig, parentConfig, value]
  )

  const extendedConfig = useMemo(
    () => (isFunctionalConfig ? config : mergeConfigs(parentConfig, config)),
    [isFunctionalConfig, parentConfig, config]
  )

	...

  return createElement(
    SWRConfigContext.Provider,
    mergeObjects(props, {
      value: extendedConfig
    })
  )
}

```

### useSwr

![useswr.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c0fc1a43-1127-4384-85d1-5fff37e02f46/324e81dc-6d30-401a-8287-d3323352e539/useswr.png)

`useSwr`은 컴포넌트 영역에서 직접 사용되는 `hook`입니다. 데이터를 패칭하고 `state`를 관리하며 필요에 따라 데이터를 재검증합니다.
`config`의 `cache`와 옵션 값을 전달받아 컴포넌트에 이용될 `state`를 관리합니다.

## 실행 과정

![흐름도.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/c0fc1a43-1127-4384-85d1-5fff37e02f46/0ed2d3a6-64db-4339-97c5-1e243b1b3c2d/%E1%84%92%E1%85%B3%E1%84%85%E1%85%B3%E1%86%B7%E1%84%83%E1%85%A9.png)

### 초기 실행

프로젝트 초기 실행시 `defaultConfig`가 기본으로 사용됩니다. 물론 상위 컴포넌트에 `SWRConfig`를 통해서 하위 컴포넌트의 `useSwr`에 제공할수 있지만 `SWRConfig`를 사용하지 않는 것을 가정으로 설명하겠습니다.

먼저 `initCache`를 호출을 통해 초기 `cache`를 반환합니다. 여기서 초기 `cache`는 맵 인스턴스로 `initCache`의 인자로 호출됩니다. 인자로 받은 `provider`를 `SWRGlobalState`에 `GlobalState`의 초기 값으로 맵핑하여 형성 합니다.

```tsx
// cache.ts

export const initCache = <Data = any>(
  provider: Cache<Data>,
  options?: Partial<ProviderConfiguration>
):
  | [Cache<Data>, ScopedMutator, () => void, () => void]
  | [Cache<Data>, ScopedMutator]
  | undefined => {

  if (!SWRGlobalState.has(provider)) {
    const opts = mergeObjects(defaultConfigOptions, options)

    const EVENT_REVALIDATORS = {}

    const mutate = internalMutate.bind(UNDEFINED, provider) as ScopedMutator
    let unmount = noop

    const subscriptions: Record<string, ((current: any, prev: any) => void)[]> =
      {}

    const subscribe = (
      key: string,
      callback: (current: any, prev: any) => void
    ) => {
      const subs = subscriptions[key] || []
      subscriptions[key] = subs

      subs.push(callback)
      return () => subs.splice(subs.indexOf(callback), 1)
    }

    const setter = (key: string, value: any, prev: any) => {
      provider.set(key, value)
      const subs = subscriptions[key]
      if (subs) {
        for (const fn of subs) {
          fn(value, prev)
        }
      }
    }

    const initProvider = () => {
      if (!SWRGlobalState.has(provider)) {
		    // SWRGlbalState에 provider(Cache)를 키로,
				// 배열에 값들(GlobalState)들이 밸류로 맵핑됩니다.
        SWRGlobalState.set(provider, [
					// 배열에는 GlobalState값인 Revalidators, Mutations,
					// Fetch, Preload, Mutator, Setter, Subscriber 등이 저장됩니다.
          EVENT_REVALIDATORS,
          {},
          {},
          {},
          mutate,
          setter,
          subscribe
        ])

				...
      }
    }

    initProvider()

		// SWRGLobalState가 맵핑된 후 Cache값인 provider를 반환합니다.
    return [provider, mutate, initProvider, unmount]
  }

	...
}

```

이렇게 만들어진 `cache`와 옵션으로 `defaultConfig`가 선언됩니다.

```tsx

// config.ts

const [cache, mutate] = initCache(new Map()) as [Cache<any>, ScopedMutator]
export { cache, mutate, compare }

// 기본 설정
export const defaultConfig: FullConfiguration = mergeObjects(
  {
    onLoadingSlow: noop,
    onSuccess: noop,
    onError: noop,
    onErrorRetry,
    onDiscarded: noop,

    revalidateOnFocus: true,
    revalidateOnReconnect: true,
    revalidateIfStale: true,
    shouldRetryOnError: true,

    errorRetryInterval: slowConnection ? 10000 : 5000,
    focusThrottleInterval: 5 * 1000,
    dedupingInterval: 2 * 1000,
    loadingTimeout: slowConnection ? 5000 : 3000,

    compare,
    isPaused: () => false,
		// 여기에 cache가 전달됩니다.
    cache,
    mutate,
    fallback: {}
  },
```

초기에 상위 `config`가 없기 때문에 `defaultConfig`만 `useSwrConfig`를 통해서 반환되고 이 `useSwrConfig`가 `withArgs`를 통해서 `fallbackConfig`로 전달됩니다.

```tsx
// use-swr-config.ts

export const useSWRConfig = (): FullConfiguration => {
  return mergeObjects(defaultConfig, useContext(SWRConfigContext))
}
```

```tsx
// resolve-args.ts

export const withArgs = <SWRType>(hook: any) => {
  return function useSWRArgs(...args: any) {
    const fallbackConfig = useSWRConfig()

    const [key, fn, _config] = normalize<any, any>(args)

    const config = mergeConfigs(fallbackConfig, _config)

		...

    return hook(key, fn || config.fetcher || null, config)
  } as unknown as SWRType
}

```

`config`를 가지고 있는 `withArgs`는 `useSwrHandler`를 인자로 호출하여 `useSwr`로 선언합니다. 이러한 형태로 `withArgs`를 통해 `config`를 최종적으로 `useSwrHandler`에 전달하며 사용됩니다.

```tsx
// use-swr.ts

export const useSWRHandler = <Data = any, Error = any>(
  _key: Key,
  fetcher: Fetcher<Data> | null,
  config: FullConfiguration & SWRConfiguration<Data, Error>
) => {
  const {
    cache,
	  ...
  } = config;

...

const useSWR = withArgs<SWRHook>(useSWRHandler)

export default useSWR
```

`useSwrHandler`는 `useSwr`의 시작점으로 `useSwr`의 내부를 핸들링하는 기본 함수입니다.

앞서 과정을 통해 `cache`가 담긴 `config`가 `useSwrHandler`에 전달됩니다.

전달받은 `cache`는 리액트에서 활용하는 내부 상태가 아닌 외부스토어로서 `useSyncExternalStore`를 통해 리액트에 반영하여 컴포넌트에서 이용될 `state`를 동기화 합니다.

여기서 `useSyncExternalStore`는 리액트 내부 상태가 아닌 외부 스토어를 구독하고, 이를 리액트 컴포넌트와 동기화하는데 도와주는 리액트 훅입니다.

이때 `useSyncExternalStore`에는 첫 번째 인자로 외부 상태값을 구독하는 함수, 그리고 두 번째 인자로 컴포넌트에 이용될 외부 스토어의 스냅샷이 필요합니다.

코드에서는 구독함수로는 `GlobalState`의 `subscribeCache`, 스냅샷으로는 `getSnapshot`이 이용됩니다.

`getSnapshot`은 `cache`에 대한 스냅샷이며, 스냅샷이 변경될 때마다 리렌더링을 발생시킵니다. 또 `subscribeCache`는 구독함수로서, `isEqual`을 통해서 `cache`의 변화가 생기면 `callback`을 호출하여 리렌더링을 발생시킵니다.

이러한 방식으로 외부데이터 `cache`의 변화를 감지하여 컴포넌트에 반영하게 됩니다.

```tsx
// use-swr.ts

export const useSWRHandler = <Data = any, Error = any>(
  _key: Key,
  fetcher: Fetcher<Data> | null,
  config: FullConfiguration & SWRConfiguration<Data, Error>
) => {
  const {
    cache,
	  ...
  } = config;
	const [key, fnArg] = serialize(_key);
  const [getCache, setCache, subscribeCache, getInitialCache] =
	  createCacheHelper<
      Data,
      State<Data, any> & {
        _k?: Key
      }
    >(cache, key);

	...

	const getSnapshot = useMemo(() => {
		...
		const cachedData = getCache();
		const clientSnapshot = getSelectedCache(cachedData)
    const serverSnapshot =
      cachedData === initialData
        ? clientSnapshot
        : getSelectedCache(initialData)

		...
		return [
			() => clientSnapshot,
			() => serverSnapshot
	}, [cache, key]);

  const cached = useSyncExternalStore(
    useCallback(
      (callback: () => void) =>
        subscribeCache(
          key,
          (current: State<Data, any>, prev: State<Data, any>) => {
            if (!isEqual(prev, current)) callback()
          }
        ),
      // eslint-disable-next-line react-hooks/exhaustive-deps
      [cache, key]
    ),
    getSnapshot[0],
    getSnapshot[1]
  )

	const cachedData = cached.data

	const returnedData = keepPreviousData
    ? isUndefined(cachedData)
      ? laggyDataRef.current
      : cachedData
    : data

  return {
    mutate: boundMutate,
    get data() {
      return returnedData
    },
		...
  } as SWRResponse<Data, Error>
}

const useSWR = withArgs<SWRHook>(useSWRHandler)

export default useSWR
```

### 상위 Config 전달

만약 상위에 `SWRConfig`를 이용하게 되면 전역적으로 `config`를 제공할수 있습니다.

상위 `context`를 통해 전달받은 `SwrConfigContext`를 `useSwrConfig`에서 `SwrConfigContext`를 우선순위로 `defaultConfig`와 병합합니다. 새로 병합된 `config`는 이전에 설명한 방식과 같은 방법으로 `withArgs`를 통해 최종적으로 `useSwr`에 전달됩니다.

```tsx
// use-swr-config.ts

export const useSWRConfig = (): FullConfiguration => {
  return mergeObjects(defaultConfig, useContext(SWRConfigContext))
}
```

추가로 `SWRConfig`내부 로직 중 `SWRConfigContext`, `extendedConfig`를 통해서 `SWRConfig`의 중첩을 통한 `config`전달도 가능하게 됩니다.

```tsx
// config-context.ts

export const SWRConfigContext = createContext<Partial<FullConfiguration>>({})

const SWRConfig: FC<
  PropsWithChildren<{
    value?: Config | ((parentConfig?: Config) => Config)
  }>
> = props => {
  const { value } = props
  const parentConfig = useContext(SWRConfigContext)
  const isFunctionalConfig = isFunction(value)
  const config = useMemo(
    () => (isFunctionalConfig ? value(parentConfig) : value),
    [isFunctionalConfig, parentConfig, value]
  )

  const extendedConfig = useMemo(
    () => (isFunctionalConfig ? config : mergeConfigs(parentConfig, config)),
    [isFunctionalConfig, parentConfig, config]
  )

	...

  return createElement(
    SWRConfigContext.Provider,
    mergeObjects(props, {
      value: extendedConfig
    })
  )
}

```

## 정리

최근에 swr 라이브러리를 파악해보고자 직접 기본적인구조만 구현해봤고, 이를 토대로 학습한 내용을 정리해봤습니다.

보통 라이브러리를 적용하고 내부 개념을 학습하는데서 끝났는데, 이번 기회에 내부 구조를 파악하면서 어떤 방식으로 개발하는지도 조금은 이해할 수 있었습니다.

과정자체는 굉장히 힘들고 오래걸렸지만 그만큼 더 많은 것 을 배울수 있었던 것 같습니다.
