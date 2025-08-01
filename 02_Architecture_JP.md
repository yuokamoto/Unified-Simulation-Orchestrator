# 2. 統合アーキテクチャ

![](./simulation_archi.drawio.svg)

## 層構造

本フレームワークは、複数種類のシミュレーションエンジンを統一的に扱い、  
資産やロジックの再利用を実現するため、以下の層で構成される：

1. **Snapshot Management Layer（スナップショット管理層）**  
   - 世界の状態を**フルスナップショット（時刻tでの完全な状態）**および**差分スナップショット（前ステップとの差分）**として管理。  
   - 初期化、状態同期、ログ、リプレイの基盤として機能。
   - 分散モードではRedisなどのKVストアを用い、ノード間での同期やリカバリを支援。

2. **Simulation Layer（シミュレーション層）**  
   - 実際のシミュレーションを行うエンジン層。  
   - **SimPy, Gazebo, Mujoco, Genesis, Isaac Sim, Unreal Engine** などを用途に応じて選択できる。  
   - ロジックシミュレーションのみ（物理無効）から、高精度物理や高品質3D表示まで切り替え可能。
   - コンベアやエレベータなど、物理演算を無効にしたオブジェクトも扱える。

3. **Logic Control Layer（ロジック制御層）**  
   - **内部ロジック制御**：ノード内部のBehavior Tree (BT) によりアセット（ロボット、人など）を制御。  
   - **外部ロジック制御**：ROS2やgRPCを通じて、外部アプリケーションや実ロボット用アルゴリズムを利用可能。

4. **Interface Layer（インターフェース層）**  
   - シミュレーション層と外部制御や状態管理を接続する抽象化レイヤー。
   - **gRPCをコア通信手段**とし、必要に応じてROS2ラッパーを提供。
   - スナップショット同期やイベント通知、制御コマンドを統一APIで扱う。

5. **Tools Layer（ツール層）**  
   - **スナップショット編集ツール**：GUIまたはスクリプトで世界状態を編集。  
   - **シナリオエディタ**：YAML/JSON形式のシナリオを作成・編集し、スナップショットを参照してプレビュー可能。  
   - **Behavior Tree Editor**：共通BT記述を編集・エクスポート可能。  
   - **フォーマット変換ツール**：ROSBagや他形式からスナップショットへ変換可能。

---

## データフローとファイルの関係

1. **シナリオファイル（YAML/JSON）**  
   - シミュレーションの「意図や流れ」を記述する。  
   - 具体的にはミッションのゴール、タスクスケジュール、イベント発火条件など。  
   - **初期状態スナップショットへの参照を持つ**ことで、世界の開始状態を明示。

2. **スナップショットファイル（YAML/JSON）**  
   - 特定の時刻における世界の「実際の状態」を記述。  
   - 位置・姿勢・速度・内部状態・接続状態（ロボットがパレットを運んでいる等）を保持。  
   - 初期化、分散ノード間の同期、バグ再現、ログ/リプレイに利用。

3. **Behavior Tree (BT)**  
   - SCXMLまたはBT XMLで共通記述し、各ノードが自動変換して利用。  
   - SimPyでは`py_trees`、Gazeboや他エンジンでは`BehaviorTree.CPP`などを使用。

4. **シミュレーション実行時**  
   - シナリオ → 参照された初期スナップショットをロード → シミュレーション開始。  
   - ノードは各ステップで差分スナップショットを生成し、Simulation Masterに送信。  
   - Simulation Masterは統合状態を管理し、必要に応じてログ化・リプレイ用に保存。

---

## モード別動作

1. **シングルモード**  
   - MasterとNodeを1プロセスで動作させ、通信を省略。
   - 開発・小規模テスト向け。

2. **分散モード**  
   - 複数ノード（SimPy, Gazebo, 他エンジン）をサーバー群で動作させる。  
   - Simulation Masterが差分スナップショットで同期を行い、統合状態を管理。

3. **マルチエンジン切替**  
   - シナリオ・スナップショット・BTは共通のまま、使用するシミュレーションエンジンだけ切り替える。  
   - 例:  
     - SimPyで大規模なロジック検証 → Gazeboで物理動作検証 → Unreal Engineでデモ。

