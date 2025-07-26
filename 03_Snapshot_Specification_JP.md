# 3. スナップショット仕様

## 役割
スナップショットは、シミュレーション世界の状態を表現し、次の目的で使用される：
- 初期化：シナリオで定義された初期状態を各シミュレーションエンジンに適用。
- 同期：分散モードで各ノード間の状態を統合・同期。
- リプレイ：シミュレーションの再生やバグ再現。
- ログ：シミュレーション結果の保存・解析。

---

## スナップショットの種類
1. **フルスナップショット**  
   世界の完全な状態（位置、速度、内部状態、接続関係など）を記録。
2. **差分スナップショット**  
   前回のスナップショットとの差分のみを記録し、大規模シミュレーションの通信量を削減。

---

## フルスナップショット例 (YAML)
```yaml
timestamp: 123.45   # スナップショットが表すシミュレーション時刻（秒）
world:
  assets:
    <asset_id>:         # アセットごとの一意な識別子
      type: <string>    # "robot", "human", "object", "conveyor", "elevator" など
      model: <string>   # OpenUSD, URDF, SDFなど、アセットの3D/物理モデルへのパス
      position: [x, y, z]  # 世界座標系での位置
      orientation: [qx, qy, qz, qw]  # クォータニオンでの姿勢
      linear_velocity: [vx, vy, vz]  # 速度ベクトル
      angular_velocity: [wx, wy, wz] # 角速度
      status: <string>  # 任意の状態ラベル（"idle", "moving", "error"等）
      connected_to: <asset_id or null>  # 接続されている他アセット（例: ロボットがパレットを運んでいる場合）
      properties:       # 内部状態や追加属性
        battery_level: 0.85
        custom_flags:
          carrying_load: true
```

---

## 差分スナップショット例 (JSON)
```json
{
  "type": "state_update",
  "timestamp": 130.00,
  "node_id": "simpy_A",
  "delta_snapshot": {
    "updated_assets": {
      "robot_1": {
        "position": [5.0, 1.0, 0.0],
        "linear_velocity": [0.5, 0.0, 0.0],
        "status": "moving",
        "connected_to": "pallet_1"
      }
    },
    "removed_assets": ["human_2"],
    "new_assets": {
      "robot_3": {
        "type": "robot",
        "model": "urdf/robot3.urdf",
        "position": [0.0, 0.0, 0.0],
        "orientation": [0, 0, 0, 1],
        "linear_velocity": [0, 0, 0],
        "angular_velocity": [0, 0, 0],
        "status": "idle",
        "connected_to": null
      }
    }
  }
}

```

---

## 接続状態の表現
- `connected_to` フィールドを用いて、物理的な拘束を伴わない「論理的な接続」を表現可能。
- これにより、物理シミュレーションを用いずとも、ロボットがパレットを運ぶ等の表現が可能。

---

## 運用ポリシー
- スナップショットは**世界の状態を記述するのみ**であり、位置や速度の更新は各シミュレータのロジックで行う。
- 利用場面：
  1. **初期化** – シミュレーション開始時に適用。
  2. **同期** – 分散モードでの状態共有。
  3. **ログ保存** – 状態の履歴を保存し、リプレイや解析に利用。
