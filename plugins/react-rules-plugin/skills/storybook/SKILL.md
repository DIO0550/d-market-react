---
name: storybook
description: Storybook開発ルール。CSF3.0形式、Meta/StoryObj型定義、Story種類（Default/AllProps/EdgeCases + UI状態）、argTypes/controls/play/decoratorsの規約を定義。Storybook作成時に参照。
---

# Storybook開発ルール

StorybookのStory作成規約を定義するスキル。

## 基本ルール

- **ファイル名**: `ComponentName.stories.tsx`
- **CSF3.0**: Component Story Format 3.0 を使用
- **型定義**: `Meta<typeof Component>` と `StoryObj<typeof Component>`
- **Title設定**: 原則 `title` は設定しない（自動生成）。
  - ただし、公開UIライブラリなどで階層を固定したい場合は明示してよい。
- **データ責務**: Storyは表示責務に限定し、ビジネスロジックは持たない

## Story種類

最低限、以下を用意する。

| Story | 内容 |
|:-|:-|
| Default | 基本的な使用例 |
| AllProps | 主要propsを網羅した状態 |
| EdgeCases | 境界値・特殊状態 |
| Loading | 読み込み中状態（該当時） |
| Empty | 空状態（該当時） |
| Error | エラー表示状態（該当時） |

## Args / Controls / Actions

- `args` にデフォルト値を集約し、Story差分は最小化する
- `argTypes` で controls の公開範囲を明示する
  - 内部的なpropsは `control: false` を指定
- イベント系propsは `fn()` でモックし、実処理を持ち込まない

## Decorators / Providers

- テーマ、Router、i18n、State Provider などは decorator で注入する
- グローバルで共通化できるものは global decorators に寄せる
- Story固有の依存のみ、個別decoratorで付与する

## Interaction（play）

- `play` はUI操作の確認（クリック・入力・フォーカス）までに限定する
- 重い統合テストや外部通信検証は Storybook に持ち込まない
- `play` を書く場合は、ユーザー視点のアサーションを優先する

## 基本例

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import { fn } from "@storybook/test";
import { Button } from "./Button";

const meta: Meta<typeof Button> = {
  component: Button,
  args: { onClick: fn() },
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Default: Story = {
  args: { label: "Click me" },
};
```

## 禁止事項

- Story内でビジネスロジックを実装しない
- API呼び出しや外部サービスへの直接依存を避ける
- `play` 内で不安定な待機（過剰な `setTimeout` 依存）を行わない
