# Pomodoro Timer - Design Document

## Overview
ターミナルで動作するポモドーロタイマー。TDD（テスト駆動開発）で実装する学習プロジェクト。

## Requirements

### Functional Requirements
1. **タイマー機能**
   - 25分の作業セッション
   - 5分の短い休憩
   - 15分の長い休憩（4セッションごと）
   - カウントダウン表示

2. **ユーザー操作**
   - 一時停止/再開（Spaceキー）
   - セッションのスキップ（Enterキー）
   - 終了（Qキー）

3. **表示機能**
   - 残り時間の表示（MM:SS形式）
   - プログレスバー
   - 現在のモード表示（作業/休憩）
   - セッション数カウント

4. **通知機能**
   - セッション完了時にビープ音

### Non-Functional Requirements
- 読みやすいコード構造
- テストカバレッジ重視
- 小さな単位での動作確認

## Architecture

### Module Structure
```
playground/
├── main.rs           # エントリーポイント、UI制御
├── timer.rs          # タイマーのコアロジック（テスト対象）
└── tests/
    └── timer_test.rs # ユニットテスト
```

### Core Components

#### 1. `Timer` (timer.rs)
タイマーの状態管理とビジネスロジック

```rust
pub struct Timer {
    mode: TimerMode,
    state: TimerState,
    remaining_seconds: u64,
    session_count: u32,
}

pub enum TimerMode {
    Work,
    ShortBreak,
    LongBreak,
}

pub enum TimerState {
    Running,
    Paused,
    Completed,
}
```

**責務:**
- 時間のカウントダウン
- 状態遷移（実行中/一時停止/完了）
- セッション管理
- モード切り替えロジック

#### 2. `UI Layer` (main.rs)
ターミナル表示とユーザー入力処理

**責務:**
- タイマー情報の描画
- キーボード入力の処理
- ターミナル制御

## Design Principles

### 1. Separation of Concerns
- **ビジネスロジック** (`timer.rs`): 時間計算、状態管理
- **UI** (`main.rs`): 表示、入力処理

→ ビジネスロジックを単独でテスト可能に

### 2. Pure Functions First
できる限り純粋関数で実装し、副作用を最小化

### 3. Small Steps
TDDのサイクルを短く保つ
- Red: 失敗するテストを書く
- Green: 最小限の実装で通す
- Refactor: リファクタリング

## State Machine

```
[Work Session] --完了--> [Short Break] --完了--> [Work Session]
                   |                                    |
                4回目                                  継続
                   |                                    |
                   v                                    v
            [Long Break] --完了--> [Work Session]    [Work Session]
```

## Testing Strategy

### Unit Tests (timer.rs)
1. **初期状態のテスト**
   - 新規タイマーは作業モード
   - 残り時間は25分
   - セッション数は0

2. **時間経過のテスト**
   - 1秒経過後、残り時間が減る
   - 時間が0になったら完了状態になる

3. **状態遷移のテスト**
   - 一時停止 → 再開
   - 完了 → 次のセッション

4. **セッション管理のテスト**
   - 1-3回目: Work → ShortBreak
   - 4回目: Work → LongBreak
   - Break → Work

5. **エッジケースのテスト**
   - 一時停止中の時間経過
   - 完了後の操作

### Integration Tests
- UIとタイマーの連携（手動テスト）

## Implementation Order (TDD)

### Phase 1: Core Timer Logic
1. タイマーの初期化
2. 時間経過の処理
3. 状態管理（実行中/一時停止）
4. セッション完了の検出

### Phase 2: Session Management
5. 次のセッションへの遷移
6. セッション数のカウント
7. モード切り替えロジック

### Phase 3: UI Integration
8. タイマー情報の表示
9. キーボード入力処理
10. プログレスバー

### Phase 4: Polish
11. ビープ音
12. エラーハンドリング
13. リファクタリング

## Dependencies

```toml
[dependencies]
crossterm = "0.27"  # ターミナル制御・キー入力

[dev-dependencies]
# 標準のtest frameworkを使用
```

## File Time Estimates

- `timer.rs` 実装: 30-40分
- テスト作成: 20-30分
- `main.rs` UI実装: 15-20分
- 合計: 65-90分（学習含む）

## Learning Goals

1. **Rustの基礎**
   - 構造体とEnum
   - パターンマッチング
   - 所有権とライフタイム

2. **TDDの実践**
   - Red-Green-Refactorサイクル
   - テスタブルな設計
   - リファクタリングの安全性

3. **実践的スキル**
   - 状態管理
   - 時間処理
   - ターミナルアプリ開発

## Notes

- 最初はテストしやすい部分から始める
- UIは後回し（手動テストでOK）
- 完璧を目指さず、動くものを作ることを優先
- リファクタリングはテストがある安心感の中で行う