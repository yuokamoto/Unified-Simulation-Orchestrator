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
      joint_positions:  # 非規範的な例——正確な形状（命名規則、単位、球関節・自由関節のような
                        # 多自由度関節のキー付け方）は未確定。項目6参照。非多関節アセットでは
                        # joint_positions・joint_velocities とも省略する。多関節アセットでは、
                        # 復元十分なフルスナップショットとして両方がセットで必要（ポーズだけでは
                        # 動作中の状態を復元できない）——ただし必ずしもこの「関節ごとに1つの
                        # float」という形状とは限らない。
        <joint_name>: <float>   # あくまで例示：単一自由度（回転／直動）関節の角度（rad）または
                                 # 変位（m）。多自由度関節にはこのままでは使えない。
      joint_velocities: # 多関節アセットでは joint_positions とセットで必要——上記と同じ
                        # 非規範的な注意点が当てはまる
        <joint_name>: <float>   # あくまで例示——joint_positions 参照
      status: <string>  # 任意の状態ラベル（"idle", "moving", "error"等）
      connected_to: [<asset_id>, ...]  # 接続されている他アセットのリスト（例: ロボットがパレットを運んでいる場合）。接続なしの場合は空リスト。
      properties:       # 内部状態や追加属性
        battery_level: 0.85
        custom_flags:
          carrying_load: true
  reproduction_info:   # グローバル（アセット単位ではない）。状態からレンダリング画像を再生成する
                        # 利用者にのみ必須 — 詳細は下記「グローバル（非アセット）フィールド」参照
    lighting: <構造化された照明の記述>
    domain_randomization: {<param_name>: <value>, ...}
    seed: <int>
  meta_data:            # 任意、グローバル、自由記述 — 詳細は下記「グローバル（非アセット）フィールド」参照
    <key>: <value>
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
        "connected_to": ["pallet_1"]
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
        "connected_to": []
      }
    },
    "updated_reproduction_info": {
      "seed": 42
    },
    "updated_meta_data": {
      "experiment_tag": "run_017"
    }
  }
}

```

> **非規範的な例示：** 上記の `updated_reproduction_info` / `updated_meta_data` は意図の例示に過ぎない。省略されたフィールドやキーを受信側が保持・置換・削除のどれとして扱うかはまだ定義されていない——項目8参照。この例をもって意味論が確定したものと扱わないこと。

---

## 接続状態の表現
- `connected_to` フィールドは**アセットIDのリスト**であり、物理的な拘束を伴わない「論理的な接続」を表現可能。
- これにより、以下のような状態を表現できる：
  - ロボットが複数のパレットを運搬: `connected_to: ["pallet_1", "pallet_2"]`
  - 棚が複数のアイテムを保持: `connected_to: ["item_1", "item_2", "item_3"]`
  - コンベア上の荷物: `connected_to: ["package_1", "package_2"]`
- 空リスト `[]` は接続なしを示す。
- 物理シミュレーションを用いずとも、論理的な関係性のモデリングが可能。

---

## グローバル（非アセット）フィールド

スナップショットが持つのはアセット単位の状態だけではない。`world` の直下に、`assets` と並んで2つのフィールドが置かれる：

- **`reproduction_info`** — 状態から、画像そのものを保存せずにレンダリング画像を再生成するために特に必要となる、**シーン全体に関わる**構造化された型付きデータ（照明、Domain Randomization 設定、乱数シード）。シミュレーションを続行するためには**不要**（物理演算はこれを必要としない）。エピソード単位のスコープを持ち、通常は実行中変化しないため、通常はフルスナップショットに一度だけ乗せる。
  - **カメラは `reproduction_info` には含まれない。** 観測に使うカメラそのものはグローバルな状態ではなく、アセットである：ロボットに剛体的に取り付けられたセンサー（例：手首カメラ）の世界座標での姿勢は、親アセットの追跡済みの状態——ルート姿勢、および多関節アセットであればすでに追跡されている`joint_positions`——と、アセットのモデルで定義された固定の取り付けオフセット・キネマティックチェーンを組み合わせることで常に導出できるため、専用のスナップショットフィールドは一切不要である（ルート姿勢からの単一の固定オフセットだけで十分なのは、カメラが非多関節の剛体に直接取り付けられている場合に限る）。据え置き型の観測カメラ（例：作業エリアを見下ろす固定俯瞰カメラ）は、他のアセットと同様に（`type: "camera"`、通常通り `position`/`orientation` を持つ形で）追跡する。内部パラメータ（焦点距離など）は、毎ステップの動的データではなく、そのアセットのモデルの静的な属性として持つ——ただし、その較正情報（焦点距離・解像度・画角）をどこに・どう表現するかという正確な契約はまだ標準化されていない（`model`は単なるパスであり、較正用スキーマや参照規約は未定義）ため、この静的カメラ契約が存在して初めてこの除外は完結する——項目11参照。詳細は [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) を参照。
- **`meta_data`** — 復元のためには不要な、グローバル（シミュレーション全体）スコープの自由記述データを持つための、ユーザー定義のバッグ。既存のアセット単位 `properties` フィールドのグローバル版にあたる。実験タグ、デバッグ用フラグ、その他任意のカスタムデータに使う。形状は意図的に制約されていないため、フレームワーク側や外部利用者側は、このフィールドの存在や特定のキーの有無に依存すべきではない。

どちらも任意フィールドである。この分割に至った設計議論は [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)、未解決点（サブフィールドの正確な形状）は [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) を参照。

---

## 運用ポリシー
- スナップショットは主にアセット単位の世界の状態（位置・速度などの動的状態。更新は各シミュレータのロジックで行う）を記述する。グローバルな `reproduction_info` / `meta_data` フィールド（上記「グローバル（非アセット）フィールド」参照）はこの原則の意図的な例外であり、シミュレートされた世界の状態ではなく、レンダリング・再現設定とユーザーメタデータを持つ。
- 利用場面：
  1. **初期化** – シミュレーション開始時に適用。
  2. **同期** – 分散モードでの状態共有。
  3. **ログ保存** – 状態の履歴を保存し、リプレイや解析に利用。

---

## 関連：設計議論の記録

スナップショットを「状態」を表すsource of truthとして扱う位置づけ、および学習パイプラインとの境界（state/data supply API）における役割は [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md) と [12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md) に記録されている。

上記の `joint_positions` / `joint_velocities` フィールド（および正確な形状）は、本フォーマットの外部利用者によって顕在化した、まだ未確定の拡張である。[18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md) と [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) を参照。

上記の `reproduction_info` / `meta_data` というグローバルフィールドは、アセット単位ではないデータ（レンダリング・再現設定、ユーザー任意のタグ）の置き場所を決めるために追加された。[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) を参照。
