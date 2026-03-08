# 7. インターフェースとメッセージ仕様

## 通信アーキテクチャ概要

本フレームワークでは、通信経路を **内部（Simulation Master ⇔ Node）** と **外部（GUI/制御系 ⇔ Simulation Master）** に分け、
以下のプロトコル構成を採用する。

1. **Simulation Master ⇔ Node 間（内部通信）**
   - **ZeroMQ** を唯一の通信手段として採用。
   - 軽量かつ低遅延で、大量の差分スナップショットをスケーラブルに伝送可能。

2. **外部システム ⇔ Simulation Master 間（外部通信）**
   - **gRPC** をコア通信プロトコルとして採用し、シミュレーションライフサイクル管理、外部制御、GUI連携に使用。
   - 既存のROS 2ノード群とのシームレスな統合のために **ROS 2 ラッパー** を提供。

---

## メッセージ種別

### 1. スナップショット同期
- ノードが生成する **差分スナップショット** を Simulation Master に送信。
- Simulation Master が **統合スナップショット** を生成し、ノードやツールに配布。
- スナップショットの仕様は第3章（スキーマ仕様）と一致。

**ZeroMQメッセージ例（差分スナップショット）**
```json
{
  "type": "state_update",
  "timestamp": 123.45,
  "node_id": "simpy_A",
  "delta_snapshot": {
    "updated_assets": {
      "robot_1": {
        "position": [1.2, 3.4, 0.0],
        "linear_velocity": [0.5, 0.0, 0.0],
        "status": "moving"
      }
    },
    "removed_assets": [],
    "new_assets": {}
  }
}
```

### 2. シミュレーション制御
- **ステップ進行**: Master → Node に次ステップ開始を通知。
- **リセット**: 指定スナップショットに状態を戻す。
- **停止**: シミュレーションを終了する。

**ZeroMQメッセージ例（ステップ開始）**
```json
{
  "type": "step_start",
  "delta_t": 0.01
}
```

### 3. イベント通知
- ノードが検知したイベント（タスク完了、BT状態遷移、エラー）をMasterに通知。

**ZeroMQメッセージ例（イベント）**
```json
{
  "type": "event",
  "event_type": "task_completed",
  "asset_id": "robot_1",
  "details": {"task_id": "pickup_001"}
}
```

### 4. ログ・リプレイ制御
- CLIやGUIからの指示で、Masterにログ再生や記録開始を要求。
- ZeroMQ経由で全ノードにブロードキャスト可能。

**ZeroMQメッセージ例（ログ再生開始）**
```json
{
  "type": "start_replay",
  "log_id": "session_2025_07_21",
  "speed": 1.0
}
```

---

## APIの構成

1. **ZeroMQ API**
   - 軽量なバイナリまたはJSONで、ステップ制御・スナップショット同期・イベントを伝送。
   - 内部通信（Master ⇔ Node）専用。

2. **gRPC API**
   - シミュレーションの開始、停止、リプレイ、外部監視ツールとの連携を管理。
   - GUI、CLI、サードパーティ統合のためのコア外部通信プロトコル。

3. **ROS 2 API**
   - gRPCの機能をラップし、既存のROS 2 ノードから容易に利用可能。
   - 通信middlewareには zenoh を優先採用。

---

## 実装上のポイント
  - ZeroMQのメッセージは軽量化のため、差分スナップショットのみを基本伝送。
  - gRPCは状態管理と高レベル操作（スタート/ストップ/リプレイ）に限定。
  - ROS 2 の利用は、既存シミュレーションエンジンの互換性を最優先し、不足部分のみ追加実装。
