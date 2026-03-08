# Unified Simulation Orchestrator (USO)

複数のシミュレーションエンジン（SimPy, Gazebo, Mujoco, Genesis, Isaac Sim, Unreal Engine 等）を切り替え・統合しながら、アセット・ロジック・シナリオを最大限再利用できる**統合シミュレーションフレームワーク**。

---

## 主な特徴

- **マルチエンジン対応** — 軽量ロジックシミュレーション（SimPy）、高精度物理（Gazebo/Mujoco）、高品質3Dビジュアライゼーション（Isaac Sim/Unreal Engine）を切り替え可能
- **アセット・ロジックの再利用** — 3Dモデル（OpenUSD）、ロボット記述（URDF/SDF）、動作ロジック（BT XML）、世界状態（YAMLスナップショット）を全エンジン間で共有
- **スナップショット中心の状態管理** — フル・差分スナップショットにより、初期化・同期・バグ再現・ログ・リプレイを統一
- **シングル・分散モード対応** — 単一プロセスのデバッグから数千エージェントのマルチサーバーシミュレーションまでシームレスに対応
- **既存ワークフローの維持** — 既存シミュレータの使用感を最小限の変更で維持（例: Gazeboは引き続き `roslaunch` で起動可能）

---

## ドキュメント

| # | ドキュメント | 概要 |
|---|-------------|------|
| 1 | [コンセプト概要 (EN)](01_Concept_Overview_EN.md) / [JP](01_Concept_Overview_JP.md) | プロジェクトのビジョン、背景、核心コンセプト |
| 2 | [アーキテクチャ (EN)](02_Architecture_EN.md) / [JP](02_Architecture_JP.md) | 層構造、データフロー、動作モード |
| 3 | [スナップショット仕様 (EN)](03_Snapshot_Specification_EN.md) / [JP](03_Snapshot_Specification_JP.md) | フル・差分スナップショットのスキーマと運用ポリシー |
| 4 | [ビヘイビアツリー仕様 (EN)](04_BehaviorTree_Specification_EN.md) / [JP](04_BehaviorTree_Specification_JP.md) | BT XMLを正式フォーマットとし、エンジン別の実行方式を定義 |
| 5 | [SimMaster・ノード設計 (EN)](05_SimulationMaster_NodeDesign_EN.md) / [JP](05_SimulationMaster_NodeDesign_JP.md) | Master/Nodeの役割、共通インターフェース、アセットオーナーシップルール |
| 6 | [ログ・リプレイ (EN)](06_Logging_Replay_EN.md) / [JP](06_Logging_Replay_JP.md) | ログ構造、リプレイ仕様、CLI/GUIツール |
| 7 | [インターフェース・メッセージ (EN)](07_Interface_Messaging_EN.md) / [JP](07_Interface_Messaging_JP.md) | ZeroMQ（内部）、gRPC（外部）、ROS 2ラッパー |
| 8 | [GUI設計 (EN)](08_GUI_Design_EN.md) / [JP](08_GUI_Design_JP.md) | Webベース GUI: 3Dビジュアライザ、エディタ、リプレイ |
| 9 | [ビルド戦略 (EN)](09_Build_Strategy_EN.md) / [JP](09_Build_Strategy_JP.md) | 多言語ビルド、ディレクトリ構成、CI/CD |
| 10 | [エンジン選定・分散Sim (EN)](10_Engine_Selection_Distributed_EN.md) / [JP](10_Engine_Selection_Distributed_JP.md) | エンジン・ミドルウェア選定の根拠 |

### 計画ドキュメント

| ドキュメント | 概要 |
|-------------|------|
| [開発計画](USO_Development_Plan.md) | タイムラインとマイルストーン |
| [実装タスクリスト](USO_Implementation_Task_List.md) | 実装の詳細タスク分解 |

---

## アーキテクチャ概要

```
┌─────────────────────────────────────────────────────┐
│                   ツール層                            │
│  スナップショット編集 │ シナリオ編集 │ BT編集 │ ...    │
├─────────────────────────────────────────────────────┤
│                インターフェース層                      │
│   内部: ZeroMQ (Master⇔Node)                         │
│   外部: gRPC (GUI/CLI⇔Master), ROS 2ラッパー          │
├─────────────────────────────────────────────────────┤
│               ロジック制御層                           │
│   内部BT (py_trees / BehaviorTree.CPP)                │
│   外部ロジック (ROS2 / gRPC)                          │
├─────────────────────────────────────────────────────┤
│              シミュレーション層                        │
│   SimPy │ Gazebo │ Mujoco │ Genesis │ Isaac │ Unreal  │
├─────────────────────────────────────────────────────┤
│           スナップショット管理層                       │
│   フルスナップショット │ 差分スナップショット │ Redis   │
└─────────────────────────────────────────────────────┘
```

---

## 通信プロトコル概要

| 経路 | プロトコル | 用途 |
|------|-----------|------|
| Master ⇔ Node（内部） | **ZeroMQ** | 軽量な差分スナップショット同期、ステップ制御、イベント |
| 外部 ⇔ Master | **gRPC** | シミュレーションライフサイクル、GUI/CLI制御、モニタリング |
| ROS 2エコシステム | **ROS 2ラッパー**（gRPC上） | 既存ROS 2ノードとの統合 |

---

## 未解決事項 / TODO

> **注記:** 以下はオープンな設計課題であり、今後のイテレーションで対応予定。

- [ ] **シナリオファイル仕様** — ミッション、タスクスケジュール、イベントトリガーを定義するシナリオファイルスキーマ（YAML/JSON）はドキュメント全体で参照されているが、正式な仕様が未定義。第3章への追加または独立した章が必要。
- [ ] **WebブラウザでのOpenUSD表示** — 第8章で3Dビジュアライザ（Three.js）でのOpenUSDアセットの読み込みを記載しているが、Three.jsにはUSDのネイティブサポートがない。変換パイプライン（USD → glTF）または代替レンダリングライブラリの評価が必要。
- [ ] **セキュリティ（認証 / TLS）** — 分散モードのマルチサーバー通信に対する認証・認可・暗号化の仕様が未定義。
- [ ] **パフォーマンス目標** — 具体的な目標値（例: 目標ステップレートでのエージェント数、差分スナップショットのレイテンシ上限等）が未定義。

---

## ライセンス

詳細は [LICENSE](LICENSE) を参照。
