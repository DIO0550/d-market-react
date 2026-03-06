---
name: custom-hook
description: Reactカスタムフック開発ルール。use*命名規則、状態管理パターン、戻り値の型定義、テスト方法を定義。カスタムフック実装時に参照。
---

# カスタムフック開発ルール

Reactカスタムフックの規約を定義するスキル。

## 基本ルール

- **命名規則**: 必ず `use` で始める（例: `useUserData`）
- **単一責任**: 1フック = 1機能
- **戻り値**: 配列またはオブジェクトで統一
- **型定義**: 戻り値の型を明示的に定義
- **エラーハンドリング**: エラー状態も戻り値に含める

## 推奨パターン

```typescript
type UseUserDataResult = {
  user: User | null;
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
};

export const useUserData = (userId: string): UseUserDataResult => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  // ...
  return { user, isLoading, error, refetch };
};
```

## テスト

```typescript
import { renderHook, act } from "@testing-library/react";

test("初期値が0である", () => {
  const { result } = renderHook(() => useCounter());
  expect(result.current.count).toBe(0);
});
```
