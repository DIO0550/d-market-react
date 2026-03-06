---
name: component
description: Reactコンポーネント開発ルール。関数コンポーネント、Props API設計、条件付きレンダー、アクセシビリティ、テストの規約を定義。コンポーネント実装時に参照。
---

# コンポーネント開発ルール

Reactコンポーネントの規約を定義するスキル。

## 基本ルール

- 関数コンポーネントを使用（`React.FC` は使用しない）
- Propsは同ファイル内で `type Props` として定義
- 単一責任の原則（1コンポーネント = 1責任）
- ビジネスロジックはcustom hooksに抽出
- 可能な限り `composition` を優先し、意味のないラッパー要素を増やさない

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

- パフォーマンス最適化は計測結果にもとづいて行う
- `React.memo` は再描画コストが高い箇所かつ効果が計測で確認できる場合に限定
- `memo` 前提の子に関数propsを渡す場合は `useCallback` を検討
- 高コスト計算は `useMemo` でキャッシュし、依存配列を厳密に管理

## テスト

- `@testing-library/react` でユーザー視点のクエリ（`getByRole` など）を優先
- 最低限、主要状態（loading / error / empty / success）と主要イベントを検証
- snapshotのみのテストに依存しない
