---
name: component
description: Reactコンポーネント開発ルール。関数コンポーネント、レンダーの純粋性（Rules of React）、Props API設計、State設計（colocation・派生値・Context）、条件付きレンダー、アクセシビリティ、パフォーマンス（メモ化・Suspense・React Compiler）、React 19の書き方、テスト（Testing Library）の規約を定義。コンポーネント実装時に参照。
---

# コンポーネント開発ルール

Reactコンポーネントの規約を定義するスキル。

## 基本ルール

- 関数コンポーネントを使用（`React.FC` は使用しない）
- Propsは同ファイル内で `type Props` として定義
- 単一責任の原則（1コンポーネント = 1責任）。UIはデータモデルの構造に対応する階層に分解する
- ビジネスロジックはcustom hooksに抽出
- 可能な限り `composition` を優先し、意味のないラッパー要素を増やさない
- データフローは一方向に保つ。データはpropsで下へ、更新はコールバックで上へ流す
- リストの `key` は配列インデックスや乱数ではなく、データ由来の安定した一意なIDを使う（インデックスkeyは並べ替え・挿入時にstateの取り違えや不要な再マウントを起こす）

## レンダーの純粋性（Rules of React）

- コンポーネントは純粋に保つ。同じprops/state/contextに対して常に同じJSXを返し、レンダー中に副作用（state更新・DOM操作・外部通信）を実行しない
  - ReactはStrict Modeの二重実行を含めレンダーを複数回実行しうる。純粋性はReact本体とReact Compilerの最適化の前提条件
- props・state・hooksの引数/戻り値は不変として扱い、直接ミューテートしない（Reactは参照比較で変更を検知するため、ミューテーションは再レンダー漏れを生む）
- コンポーネント関数を直接呼び出さない（`Component()` ではなく `<Component />`）。レンダーのタイミングはReactが管理し、直接呼び出しはstateの関連付けを壊す

## State設計

- stateは「最小限かつ完全」に。propsから渡る値・時間経過で不変の値・既存のstate/propsから計算できる値はstateにしない
- 派生値はレンダー中に計算する。`useEffect + setState` で同期しない。高価な計算のみ `useMemo` を使う
- stateは使う場所の近くに置く（colocation）。複数コンポーネントで必要になったときだけ、最も近い共通親へリフトアップする（上位に置いたstateの更新は配下全体の再レンダーチェックを引き起こす）
- 矛盾しうる複数のbooleanフラグではなく、単一のstatus union（`"typing" | "sending" | "sent"`）で「ありえない状態」を表現不能にする
- 同じデータを複数のstateに重複させない（選択オブジェクトではなく選択IDを持つ）。深いネストは避けフラットに正規化する
- 常に一緒に更新される複数のstateは1つのオブジェクトにまとめる（別々に持つと更新漏れで不整合になる）
- propsの変化でstateをリセットしたい場合は `useEffect` ではなく `key` propでコンポーネントを再生成する
- Contextは最後の手段。まずpropsの受け渡し、次にcomposition（children）を検討し、テーマ・現在ユーザーなど「ツリーの離れた場所で広く必要な情報」にのみ使う。Providerはアプリのルートではなく必要なサブツリーの近くに置く

## Props設計

- callback props は `onClick` / `onChange` のように `onXxx` で命名
- `variant` / `size` などは文字列unionで表現し、許可値を型で制限
- `children` は必要なときだけ明示的に受け取る（暗黙的に許可しない）
- 排他的なpropsはunion型で表現し、不正な組み合わせを型で防ぐ
- ラッパー要素コンポーネントは `ComponentPropsWithoutRef<"button">` などを活用し、標準属性を透過

## 条件付きレンダー

- 画面の主要分岐（loading / error / empty など）は早期 `return` で分ける
- 単純なオプション表示のみ `&&` を使う（長いJSXや複数分岐には使わない）
- 三項演算子は1行で読める短い分岐だけに限定し、ネストは禁止
- 条件が2つ以上に増えたら、サブコンポーネント分割または `if` で明示分岐する

## 命名規則

- コンポーネント名: `PascalCase`
- props/変数名: `camelCase`

## アクセシビリティ

- icon-onlyボタンには `aria-label` を必ず付与
- フォーム要素は `label` と関連付ける（`htmlFor` / `id` または `aria-labelledby`）
- 画像には用途に応じた `alt` を設定（装飾画像は空文字）
- `button` 要素は `type` を明示（`button` / `submit`）
- セマンティックHTMLを使用
- キーボード操作を考慮

## 設計パターン（有効に働く場面）

| パターン | 有効な場面 | 使いすぎ注意 |
|:-|:-|:-|
| Composition (`children` / slot props) | レイアウトの差分だけを差し替えたい。DOMの入れ子を減らしたい。 | propsが増えすぎるなら分割を検討 |
| Compound Components (`X.Root`, `X.Item`) | 関連要素をセットで使い、構造の一貫性を保ちたい。 | Context依存が強すぎると追跡しづらい |
| Render Props (`children` as function) | 振る舞いは共通、描画は画面ごとに変えたい。Headlessな再利用をしたい。 | 通常のhooksで十分な場合は不要 |
| Controlled/Uncontrolled | フォーム入力や開閉状態を、外部管理/内部管理で使い分けたい。 | 両対応する場合は優先順位を明記 |

## 入れ子深さのガイド

- 同種のレイアウトラッパーが3段以上続く場合は、コンポジションまたは分割を検討
- スタイル適用だけが目的の `div` 追加は避ける（既存要素やsemantic要素を優先）
- 共通化は「再利用が2回以上見込める」場合を目安に行う

## UI状態の扱い

- データ取得・整形はhook側に置き、コンポーネントは表示責務に集中
- skeleton/spinner表示時もレイアウトシフトを最小化する

## パフォーマンス

- パフォーマンス最適化は計測結果にもとづいて行う（React DevTools Profiler等で遅いコンポーネントを特定してから）
- メモ化より先に、children compositionやstateのcolocationで再レンダー自体を減らせないか検討する
- `React.memo` は再描画コストが高い箇所かつ効果が計測で確認できる場合に限定
- `memo` 前提の子に関数propsを渡す場合は `useCallback` を検討
- 高コスト計算は `useMemo` でキャッシュし、依存配列を厳密に管理
- React Compiler導入済みのプロジェクトでは、新規コードに手動メモ化（`memo` / `useMemo` / `useCallback`）を書かない。Compilerの前提はRules of Reactの遵守
- 「再レンダーすると壊れる」コンポーネントはメモ化で隠さず、先にレンダーを純粋にしてバグ自体を直す
- 初期表示に不要な大きいコンポーネントは `React.lazy` + `Suspense` でコード分割する
- Suspense境界はデザイン上のローディング表示単位に合わせて配置し、必要以上に細かくしない（細かすぎる境界はちらつくスケルトンの乱立を招く）
- 表示済みコンテンツがfallbackに巻き戻るのを防ぐため、緊急でない更新（ナビゲーション・検索クエリ）は `useTransition` / `useDeferredValue` で行う

## React 19 の書き方

- 新規コードで `forwardRef` を使わない。`ref` は通常のpropとして受け取る
- Context Providerは `<Context.Provider value={...}>` ではなく `<Context value={...}>` と書く
- refコールバックはクリーンアップ関数を返す形で書く（アンマウント時の `null` 呼び出しに依存しない）
- フォーム送信のpending・エラー・リセットを手動 `useState` で管理せず、`<form action>` + `useActionState` を使う
- フォーム内の子コンポーネント（送信ボタン等）は `useFormStatus` で親フォームのpending状態を読み、propsドリリングしない
- 楽観的UI更新は `useOptimistic` で実装する（失敗時は自動で元の値に復帰する）

## テスト

- `@testing-library/react` でユーザー視点のクエリ（`getByRole` など）を優先。CSSクラスや `container.querySelector` でのクエリは実装詳細への依存であり禁止
- 操作は `fireEvent` ではなく `@testing-library/user-event` を使う（実際のユーザー操作に近いイベント列を発火する）
- 非同期に現れる要素は `await screen.findBy...` で待つ（`waitFor(() => getBy...)` より簡潔でエラーメッセージも良い）
- `waitFor` のコールバックには単一のアサーションのみを入れ、副作用（クリック・入力）を入れない（コールバックは不定回数リトライされる）
- `query*` 系は「存在しないことの検証」（`.not.toBeInTheDocument()`）にのみ使う。存在検証は `get*` / `find*`
- `@testing-library/jest-dom` の専用マッチャー（`toBeDisabled()` 等）と `screen` を使う
- `render` / `userEvent` を手動で `act()` にラップしない。act警告は未処理の非同期更新のシグナルであり、ラップで黙らせない
- テストのために `role="button"` のような冗長なARIA属性を足さない。セマンティックHTML要素を使う
- 最低限、主要状態（loading / error / empty / success）と主要イベントを検証
- snapshotのみのテストに依存しない
