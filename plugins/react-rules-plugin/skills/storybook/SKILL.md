---
name: storybook
description: Storybook開発ルール。CSF3.0形式、Meta/StoryObj型定義、Story種類（Default/AllProps/EdgeCases）の規約を定義。Storybook作成時に参照。
---

# Storybook開発ルール

StorybookのStory作成規約を定義するスキル。

## 基本ルール

- **ファイル名**: `ComponentName.stories.tsx`
- **CSF3.0**: Component Story Format 3.0 を使用
- **型定義**: `Meta<typeof Component>` と `StoryObj<typeof Component>`
- **Title設定**: Metaに title は設定しない（自動生成）

## Story種類

| Story | 内容 |
|:-|:-|
| Default | 基本的な使用例 |
| AllProps | 全props設定状態 |
| EdgeCases | 境界値・特殊状態 |

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
