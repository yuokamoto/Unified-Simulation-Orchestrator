# 4. ビヘイビアツリー仕様

## 目的
本フレームワークでは、ロボットやアセットの動作ロジックを **Behavior Tree (BT)** で記述し、  
異なるシミュレーションエンジン間で **同一のロジックを可能な限り再利用可能にする**。

本フレームワークでは、BTを利用することで以下を実現する：
- ロジックの統一的な管理と再利用（SimPy、Gazebo、Unrealなどで共通）。
- 外部ロジック（ROS2/gRPC経由）と内部ロジック（シミュレータ内BT）どちらで実行するかを適宜選択可能

---

## フォーマット
1. **SCXML (State Chart XML)**  
   - 標準化されたステートマシン表現を用い、BTへのマッピングが可能。  
   - ツールや既存エディタでの編集が容易。

2. **BT XML (BehaviorTree.CPP形式)**  
   - GazeboやC++系のBTランタイムで直接利用可能。  
   - SimPyではPython用ライブラリ（`py_trees`）への変換を行う。


---

## シミュレーション環境別の適用

### SimPy ノード
- Pythonベースで動作。
- `py_trees` ライブラリを用いてBTを実行。
- 共通BT XMLを **Pythonクラスに自動変換**し、必要に応じてユーザ定義アクション（Python関数）をバインド。

### Gazebo ノード
- C++ベースで動作。
- `BehaviorTree.CPP` を用いてBTを実行。
- 共通BT XMLを **C++ノードツリーに変換**し、各アクションをC++の関数やROS2ノードに接続。

### その他のシミュレーションエンジン（Mujoco, Genesis, Unreal, Isaacなど）
- それぞれに適したBTランタイムを利用しつつ、共通XMLを **変換または直読み** できるよう、**プラグインを提供**。

---

## Blackboard との連携
- 各ノードが保持する **Blackboard** を通じて、BTノード間で変数やフラグを共有する。
- Blackboardは **ノードローカルであり、シミュレーション間では同期しない**。
  - 例外として、**必要な状態（タスク目標やフラグ）だけがスナップショットを介して共有**される。
- Blackboardの状態は、**オプションでログに含めることが可能**で、デバッグや再現性向上に活用できる。

---

## 実行フロー
1. **BTファイルのロード**  
   - SCXMLまたはBT XML形式のBTを、各シミュレーションノードがロード。
2. **エンジンごとの変換**  
   - SimPy: SCXMLを`py_trees`に変換。
   - Gazebo/Unreal: SCXMLまたはBT XMLを`BehaviorTree.CPP`で実行。
3. **実行と更新**  
   - 各シミュレーションステップでBTが更新され、アセットのアクションが決定される。
4. **外部ロジック連携**  
   - ROS2/gRPCを介して外部アルゴリズムと統合可能。

---

## サンプル（簡易BT XML）
```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <Sequence name="MissionSequence">
      <Condition ID="BatteryCheck"/>
      <Action ID="NavigateToTarget"/>
      <Action ID="PickObject"/>
    </Sequence>
  </BehaviorTree>
</root>
```
