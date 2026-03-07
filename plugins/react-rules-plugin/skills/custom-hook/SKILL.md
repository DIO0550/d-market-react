---
name: custom-hook
description: Reactカスタムフック開発ルール。いつhookに切り出すか、API設計、effectの使いどころ、状態管理、テスト観点、アンチパターンを定義。カスタムフック実装時に参照。
---

# カスタムフック開発ルール

Reactカスタムフックは「見た目を持たない再利用ロジック」を切り出すためのもの。  
コンポーネントを薄くするために使うが、抽象化のための抽象化はしない。

## まず判断すること

- 複数画面で再利用したい状態管理・副作用ならhookにする
- browser API、subscription、fetch、form stateなど、UIから分離したいロジックならhookにする
- 1コンポーネント内でしか使わず、分岐も少ないならローカル実装から始める
- JSXを返したくなったらhookではなくcomponent設計を見直す
- propsを受けて少し名前を変えて返すだけなら、hook化しない

## 基本ルール

- 必ず `use` で始める
- 1 hook = 1責務。取得とモーダル制御のような別責務を混ぜない
- hookの単位はAPI endpointではなく、画面や機能が必要とする振る舞いの単位で決める
- hookはUIを知らない。DOM参照、class名、JSXを公開APIに含めない
- 戻り値は原則object。tupleは要素数が少なく、各要素の役割が慣習的に明確な小さいAPIだけに限る
- 再利用実績や明確な責務分離がない段階で、先回りした抽象化をしない
- 戻り値の関数名は `open` `close` `submit` `retry` のような動詞にする
- 戻り値のイベントハンドラ名に `handleXxx` は使わない。`handle` は component 内部向け

## hookにする価値があるパターン

| パターン              | hookに向く理由                                             | 典型例          |
| :-------------------- | :--------------------------------------------------------- | :-------------- |
| 非同期データ取得      | loading / error / retry をまとめられる                     | `useUserQuery`  |
| UI状態の再利用        | 開閉、選択、ページングの振る舞いを共有できる               | `useDisclosure` |
| 外部システム同期      | event listener や subscription の cleanup を閉じ込められる | `useMediaQuery` |
| 複数stateの整合性維持 | state遷移を1つの公開APIにまとめられる                      | `useAsyncTask`  |

## hookにしない方がよいパターン

- propsからそのまま計算できる値を返すだけ
- ただの `useState` ラッパーで、意味のある振る舞い追加がない
- その画面だけで完結する条件分岐を隠したいだけ
- 返り値が増え続けて、利用側が半分以上を使っていない
- 将来使うかもしれない差分を吸収するためだけに抽象化している

## API設計

- 引数が3つ以上、または任意引数を含む場合はoptions objectを使う
- options objectは「必須」と「任意」が読めるshapeにする
- consumerが必要とする最小APIだけ返す。内部stateや生のsetterをむやみに公開しない
- booleanは `isOpen` `isLoading` `hasNextPage` のように意味が読める名前にする
- 公開関数は「何が起きるか」が分かる命名にする。`setVisible(true)` より `open()`
- derived valueは `useEffect + setState` で保持せず、render中に導出する
- tupleを返すなら、要素数は2-3個までを目安にし、callerが分割代入だけで意味を把握できる形にする

## useState と useReducer の使い分け

- 独立したstateが1-2個で、更新規則も単純なら `useState` を優先する
- 複数stateが同時に変わり、遷移の組み合わせを管理したいなら `useReducer` を検討する
- `loading` `data` `error` `isDirty` のように、状態遷移をactionで整理した方が読める場合は `useReducer` が向く
- 不正な組み合わせを減らしたい場合にも `useReducer` が向く。例: `isLoading && error && data` のような曖昧状態を避けたいとき
- reducerは「状態遷移を明示したいから使う」のであって、単にstateが多いから使うわけではない
- 非同期処理そのものをreducerに埋め込まず、`dispatch` は状態遷移の記述に使う

```typescript
type UseDisclosureResult = {
  isOpen: boolean;
  open: () => void;
  close: () => void;
  toggle: () => void;
};

export const useDisclosure = (initialOpen = false): UseDisclosureResult => {
  const [isOpen, setIsOpen] = useState(initialOpen);

  const open = () => setIsOpen(true);
  const close = () => setIsOpen(false);
  const toggle = () => setIsOpen((prev) => !prev);

  return { isOpen, open, close, toggle };
};
```

## effectのルール

- `useEffect` は外部システムとの同期にだけ使う
- mount時/unmount時だけでよい処理は `useEffect(() => { ...; return cleanup; }, [])` を使ってよい
- ただし `[]` は「初回だけ実行したいから」ではなく、「現在のprops/stateに追従する必要がない外部同期」に限る
- mount/unmount 専用であることが明確なら、`react-hooks/exhaustive-deps` の無効化は許容できる。その場合は「なぜ追従不要か」をコメントで残す
- props/stateから計算できる値の導出に `useEffect` を使わない
- effect 1つにつき同期対象は1つに絞る
- event listener、timer、subscription、requestは必ずcleanupする
- 非同期処理は競合やstale responseを考慮する
- ユーザー操作起点の処理は、まずevent handlerで実行できないか考える
- 依存配列の問題を隠すための `eslint-disable` はしない

## 非同期hook設計

- 最低限 `data` `isLoading` `error` を検討する
- 再取得可能なら `refetch` を返す
- hookをAPIごとに機械的に分けない。endpoint単位ではなく、画面や機能が必要とする取得・再取得・キャンセルのまとまりで設計する
- mount時に必ず必要なデータだけ `useEffect` で自動取得する。ユーザー操作起点の取得まで mount時実行に寄せない
- 検索、送信、ページ送りのようにユーザー操作で始まる処理は `search()` `submit()` `loadNext()` のような公開関数から実行する
- 成功/失敗時の状態遷移を呼び出し側から追える形にする
- 失敗時に古い成功値を残すかクリアするかを決めて一貫させる
- API連続呼び出しがあり得るなら、latest-wins / queue / ignore duplicate のどれかを決める
- 連続呼び出し時は `AbortController`、request id、実行中フラグなどで競合とstale responseを防ぐ
- 1回目のレスポンスが2回目を上書きしないことを前提に設計・テストする

```typescript
type UseUserSearchResult = {
  users: User[];
  isLoading: boolean;
  error: Error | null;
  search: (keyword: string) => Promise<void>;
};

export const useUserSearch = (): UseUserSearchResult => {
  const [users, setUsers] = useState<User[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const latestRequestIdRef = useRef(0);

  const search = useCallback(async (keyword: string) => {
    const requestId = latestRequestIdRef.current + 1;
    latestRequestIdRef.current = requestId;

    setIsLoading(true);
    setError(null);

    try {
      const nextUsers = await searchUsers(keyword);

      if (latestRequestIdRef.current !== requestId) {
        return;
      }

      setUsers(nextUsers);
    } catch (nextError) {
      if (latestRequestIdRef.current !== requestId) {
        return;
      }

      setError(nextError as Error);
    } finally {
      if (latestRequestIdRef.current === requestId) {
        setIsLoading(false);
      }
    }
  }, []);

  return { users, isLoading, error, search };
};
```

## テスト観点

- 初期状態が公開APIどおりか
- 主要操作で状態遷移が正しいか
- props変更時に再評価されるか
- 非同期hookで loading -> success / error が観測できるか
- unmount時に cleanup されるか
- 古い非同期結果が新しい状態を上書きしないか
- API連続呼び出し時に、意図した競合制御になっているか

```typescript
import { renderHook, waitFor } from "@testing-library/react";

test("後から呼んだ search の結果を採用する", async () => {
  const { result } = renderHook(() => useUserSearch());

  void result.current.search("a");
  void result.current.search("ab");

  await waitFor(() =>
    expect(result.current.users).toEqual([{ id: "2", name: "ab" }]),
  );
});
```

- `renderHook` では実装詳細ではなく公開APIを検証する
- callbackの内部実装や private state に依存したテストを書かない
- effect cleanup が重要なhookでは `unmount` を使って副作用解除を確認する
- latest-wins を採るなら、遅いレスポンスが新しい結果を上書きしないことを確認する

## アンチパターン

- `useEffect(() => setDerived(...), [source])` でderived valueをstate化する
- 条件分岐の中でhookを呼ぶ
- mount/unmount 専用ではない effect に `eslint-disable-next-line react-hooks/exhaustive-deps` を付けて依存配列問題を隠す
- 返り値に `foo`, `setFoo`, `bar`, `setBar`, `baz`, `setBaz` を並べた巨大hookを作る
- endpointごとに `useFetchXxx` を量産し、画面側で複数hookの結果を `useEffect` でつなぎ込む
- libraryの生APIをそのまま漏らし、hookとしての責務がない
- consumerが毎回同じ組み立てコードを書くほど公開APIが薄い
- まだ1箇所でしか使っていないのに、将来の拡張を見越して分岐やoptionsを増やし続ける

## レビュー時のチェックリスト

- このhookは本当に再利用単位として成立しているか
- 公開APIだけ見て使い方が想像できるか
- effectは外部同期だけに使われているか
- cleanup漏れ、競合、stale closure の余地がないか
- `useState` より `useReducer` が適切な状態遷移なのに、更新が散らばっていないか
- endpoint単位の分割で、かえって画面側の `useEffect` オーケストレーションが増えていないか
- componentを薄くしつつ、挙動を隠しすぎていないか
