# learn-React18

## Automatic Batching

- 概要

  ```

  > npm run start

  > yarn start

  ```

  以下のコマンドを打った際に、
  マウント → アンマウント → マウント
  二回マウントされるように変更になった。

  ```

  > npm run build

  > yarn build

  ```

  などの本番環境では適応されない。

- 理由

  マウント → アンマウント → マウント
  とすることで、予期しないバグやエラーを防ぐことが目的。

- Next.js で on にする方法

  https://nextjs.org/docs/api-reference/next.config.js/react-strict-mode

  ```tsx
  // next.config.js
  module.exports = {
    reactStrictMode: true,
  };
  ```

- Promise や setTimeout も Batch されるように

  - コード

    ```tsx
    // React17では、イベントのみのBatch。
    const clickHandler = () => {
      setUsers(res.data);
      setFetchCount((fetchCount) => fetchCount + 1);
    };

    // React18では、Promise や setTimeout は Batch される。
    const clickHandler = () => {
      axios.get("https://jsonplaceholder.typicode.com/users").then((res) => {
        setUsers(res.data);
        setFetchCount((fetchCount) => fetchCount + 1);
      });
    };
    ```

  - Batch を off にしたい場合

    ```tsx
    import { flushSync } from "react-dom";

    const clickHandler = () => {
      flushSync(() => {
        setUsers(res.data);
      });
      flushSync(() => {
        setFetchCount((count) => count + 1);
      });
    };
    ```

## Suspense

- 概要

  ```tsx
  if (status === "loading") return <p>Loading...</p>;
  if (status === "error") return <p>Error</p>;
  ```

  子コンポーネントに今まで書いていた上記のようなコードを、親側で定義できるようになった

- コード

  ```tsx

  const MyComponent = () => {
    return (
      <ErrorBoundary
        fallback={
          <ExclamationCircleIcon className="my-5 h-10 w-10 text-pink-500" />
        }
      >
        <Suspense fallback={<Spinner />}>
          <Users />
        </Suspense>
      </ErrorBoundary>;
    );
  };

  ```

## Nested Suspense

- 概要

  ```tsx
  const NestedSuspense = () => {
    return (
      <Layout>
        <p className="mb-3 text-xl font-bold text-blue-500">Nested Suspense</p>
        <Suspense
          fallback={
            <>
              <p className="my-5 text-green-500">Showing outer skelton...</p>
              <Spinner />
            </>
          }
        >
          <FooComponents />
          <Suspense
            fallback={
              <>
                <p className="my-5 text-pink-500">Showing inner skelton...</p>
                <Spinner />
              </>
            }
          >
            <BarComponents />
          </Suspense>
        </Suspense>
      </Layout>
    );
  };
  ```

  Suspense をネストしている場合、
  FooComponents → BarComponents の順番で読み込まれる

## startTransition (Concurrent feature)

- 概要

  state の更新に、優先順位をつけることができる

- コード

  ```tsx
  const [isPending, startTransition] = useTransition();
  const [input, setInput] = useState("");
  const [searchKey, setSearchKey] = useState("");

  const updateHandler = (e) => {
    setInput(e.target.value);
    startTransition(() => setSearchKey(e.target.value));
  };

  // startTransitionで優先順位の低いstateを指定
  // isPendingは、inputとsearchKeyの値に違いがある時にtrueになる
  ```
