# 13. 流用できるものの層別とフォーマット選択

**Status:** Draft
**Date:** 2026-08-17

> [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) の2原則（変わる理由の分離／source of truthの一方向性）を、具体的な「何を」「どのフォーマットで」流用するかに落とし込んだ結論を記録する。

## 流用できるものの層別

エンジンをまたいだ流用の効き方は、層によって異なる。

| 層 | 流用範囲 | 備考 |
|---|---|---|
| アセット（USD/URDF） | 全エンジン横断 | 最も広く共有できる |
| Behavior Tree（タスク論理・リカバリーフロー） | 物理の有無を問わず共有 | リーフノード（低レベル動作）はエンジン依存 |
| 認識モデル（画像→状態） | レンダリングを持つエンジンのみ | 学習条件に依存 |
| 方策モデル（状態→行動） | 物理を持つエンジンのみ | SimPy では意味を持たない |

下の層（アセット）ほど流用範囲が広く、上の層（方策モデル）に近づくほどエンジンの性質（物理の有無、レンダリングの有無）に依存して流用範囲が狭まる。

## フォーマット選択（アセット種別ごとに横断性の高いものを選ぶ）

| 対象 | フォーマット | 選定理由 |
|---|---|---|
| 環境・シーン | OpenUSD | Isaac 資産が豊富、overlay 機能を持つ |
| ロボット構造 | URDF/SDF | ロボティクスの横断デファクト。PyBullet/MuJoCo/Gazebo が読める |
| タスク論理 | Behavior Tree（BehaviorTree.CPP v4 XML を canonical に） | GUI編集・C++/Python双方への変換実績がある共通フォーマット（[04_BehaviorTree_Specification](./04_BehaviorTree_Specification_JP.md) 参照） |
| 状態 | スナップショット（フル／デルタ） | 初期化・同期・バグ再現・リプレイを統一的に扱える（[03_Snapshot_Specification](./03_Snapshot_Specification_JP.md) 参照） |

## 重要な含意：URDF→USDの一方向性（Isaac Simの制約）

- Isaac Sim ではロボットもネイティブ表現は USD であり、URDF は import 時に USD 化される。この変換は**一方向**（URDF → USD）であり、逆（USD → URDF）は一般には成立しない。
- これは [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) の原則2（source of truth は情報量の多い側、派生は情報を減らす方向のみ）が、フォーマット選択の場面で具体的に現れたケースである。
- ロボット定義を複数エンジンで**双方向に**共有したいのであれば、URDF を source of truth として保持し、Isaac 固有の作り込み（マテリアル、パーティクル効果など Isaac 側でしか使わない情報）は URDF に混ぜ込まず、**別レイヤー（USD overlay）に分離する**必要がある。
- このoverlayによる分離の考え方は、個別調整全般の扱い（[14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md)）で扱う「(a) パラメータ調整」の一例でもある。

## 関連ドキュメント

- [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) — 本章が適用している2原則
- [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) — 状態フォーマット（スナップショット）の既存仕様
- [04_BehaviorTree_Specification_JP.md](./04_BehaviorTree_Specification_JP.md) — タスク論理フォーマット（BT）の既存仕様
- [14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md) — overlayによるパラメータ調整の分離
- [16_Current_Status_and_Rollout_Approach_JP.md](./16_Current_Status_and_Rollout_Approach_JP.md) — PyBulletFleet側のUSD/環境ローダーの既存決定事項
- [18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md) — 外部利用者に対して状態フォーマットが復元十分であり続けるための拡張
