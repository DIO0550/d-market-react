---
name: component
description: Reactコンポーネント開発ルール。関数コンポーネント、Props定義、条件付きレンダリング、アクセシビリティの規約を定義。コンポーネント実装時に参照。
---

# コンポーネント開発ルール

Reactコンポーネントの規約を定義するスキル。

## 基本ルール

- 関数コンポーネントを使用（`React.FC` は使用しない）
- 再描画コストが高い場合は `React.memo` を使用
- Propsは同ファイル内で `type Props` として定義
- 単一責任の原則（1コンポーネント = 1責任）
- ビジネスロジックはcustom hooksに抽出

## 命名規則

- コンポーネント名: `PascalCase`
- props/変数名: `camelCase`

## 条件付きレンダリング

```tsx
// 推奨: 三項演算子
{isLoading ? <Spinner /> : <Content />}

// 非推奨: 論理演算子（falsy値の問題）
{count && <Badge count={count} />}
```

## アクセシビリティ

- aria属性を適切に設定
- セマンティックHTMLを使用
- キーボード操作を考慮
