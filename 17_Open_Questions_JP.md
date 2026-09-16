# 17. Open Questions（未解決の論点）

**Status:** Draft
**Date:** 2026-08-17（項目1〜5）／2026-08-31 更新（項目6〜7を追加）／2026-09-16 更新（項目7を解決、項目8〜12を追加）

> ここに列挙する論点は、上記各時点の設計議論では結論が出ていない。agent が勝手に埋めるものではなく、意思決定者が判断した時点で、該当ドキュメントとあわせて更新すること。

## 1. overlayの「目的ラベル」はMVPでやるか、後回しか

- 背景：[14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md) の (a) パラメータ調整で、overlay に「実機再現用」「学習高速化用」のような目的ラベルを持たせる案が出た。overlay が増えても管理しやすくなる利点がある一方、MVPの段階では過剰かもしれない。
- 関連ドキュメント：[14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md)

## 2. 簡略化ルールの記録フォーマット

- 背景：[15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md) で、意味・用途レベルの簡略化判断を「簡略化ルール」として資産化する方針は決まったが、そのルールをどう表現し、どこに（リポジトリ内のどのファイル／どの形式で）置くかは未定。
- 関連ドキュメント：[15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md)

## 3. 意味・用途レベルの簡略化をagentに問う際の、構造化した問い方の標準

- 背景：[15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md) で「agent／人間に構造化して問う」方針は決まったが、その問い方（プロンプトの形式、あるいはスキーマ）の標準は未定義。
- 関連ドキュメント：[15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md)

## 4. state/data supply API（envの顔）の最小仕様

- 背景：[12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md) で、USOがenv的な顔（reset/step相当）を持つことは決まったが、最小仕様（インターフェースの具体的なシグネチャ、スナップショットとの対応関係など）は未確定。
- 進め方の候補：まず PyBulletFleet 単体での最小 replay を実装して感覚を掴み、そのあとで共通スキーマを確定する、という順序が提案されているが、この順序自体もまだ合意事項ではない。
- 関連ドキュメント：[12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md)

## 5. 個別調整（overlay）と変換ツールの、どちらをUSOの最初の実装対象にするか

- 背景：[14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md) で (a) overlay と (b) 変換ツールという2つの仕組みの必要性は整理されたが、どちらを先に実装するかは未決定。
- 関連ドキュメント：[14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md)

## 6. 関節レベルのスナップショットフィールドの正確なスキーマ

- 背景：[18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md) で、多関節アセットの復元十分性のためにスナップショットへ `joint_positions` / `joint_velocities` フィールドが必要と結論づけたが、正確な形状（命名規則、単位、自由度が多いアセットでの関節のキー付け方、差分スナップショットとの組み合わせ方）は未確定。
- 関連ドキュメント：[18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md)、[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)

## 7. シーン全体の「再現情報」（当初はカメラ・照明・Domain Randomization・シードとして提起）をどこに持つか — 解決済み

- 背景：[18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md) で、シミュレートされた状態から画像を再生成する（毎ステップ画像を保存する代わりに）には、カメラの位置・姿勢・内部パラメータ、照明、Domain Randomization 設定、乱数シードといった、アセットごとの世界状態には含まれない追加情報が要ることが判明した。これをスナップショット形式の内側に持つか、外部利用者側のメタデータとして完全に外側に置くかは未決定だった。
- **解決（2026-09-16）**：スナップショット形式の内側に、独立した `reproduction_info` セクション（グローバル、アセット単位ではない）として持つことになった。ただし、どのアセットにも属さない設定——照明・Domain Randomization・シード——のみを持つ。同時に新設された汎用の `meta_data` フィールドとは明確に区別する。カメラの位置・姿勢・内部パラメータは、さらなる検討の結果 `reproduction_info` から明示的に除外された：カメラそのものがアセットであり、シーン全体の状態ではないため。[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)（この除外についても記載）と [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) を参照。
- 関連ドキュメント：[18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md)、[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)

## 8. `reproduction_info` / `meta_data` の正確なサブフィールド形状

- 背景：[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) で、`reproduction_info`（照明・Domain Randomization・シード。カメラは意図的に対象外——詳細は同章を参照）と `meta_data`（自由記述）が、アセット単位の状態とは別のグローバルフィールドであることは決まったが、内部の正確な形状（Domain Randomization パラメータのキー付け方、整数の`seed`だけで十分か、それともシミュレーション開始後に消費した乱数を実際に再現するにはRNGアルゴリズム・ストリーム位置まで記録する必要があるか、`meta_data` の値に型制約を課すか、差分スナップショットでの部分更新をどう表現するか、さらに——これらのフィールドを書き込めるのはMasterのみであるため（[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)、[05_SimulationMaster_NodeDesign_JP.md](./05_SimulationMaster_NodeDesign_JP.md) 参照）——ノードや外部ツールが変更を要求する仕組みをどうするか）はまだ確定していない。
- 関連ドキュメント：[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)、[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)

## 9. `meta_data` / `properties` は、登録可能な型付き拡張の仕組みを持つべきか

- 背景：[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) で、`reproduction_info`を`meta_data`の中の特定キーに統合しても、required/自由記述の区別自体はなくならず1階層内側に移るだけだと指摘された。これにより、さらに将来を見据えた論点が浮かんだ：USOコア以外の特定の外部利用者が、`meta_data`や`properties`の中に自分専用の名前付き・型付きスキーマを登録し、USOコアが事前にそのスキーマを知らなくても、`reproduction_info`の将来のスキーマで意図されているのと同種の検証・必須フィールドの保証（その正確な形状自体もまだ未確定——項目8参照）を得られるようにすべきか。[20_MetaData_Extensibility_Patterns_JP.md](./20_MetaData_Extensibility_Patterns_JP.md) が先行事例（Kubernetes CRD、Protobufの`Any`、CloudEvents、glTFの拡張機構、OpenUSDのスキーマ等）と候補案を整理しているが、決定はしていない。現時点の既知の要件には不要。
- 関連ドキュメント：[20_MetaData_Extensibility_Patterns_JP.md](./20_MetaData_Extensibility_Patterns_JP.md)、[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)

## 10. スナップショットのデータバインディングは言語ごとに生成し、別途リプレイ／検証の共有実装を持つべきか、作るとしたらいつか

- 背景：スナップショットのスキーマ（および項目6・8・9で扱う拡張機構）が固まったら、各言語向けにスキーマバリデーション、ネイティブなデータ構造（例：Pythonのdict）への変換、時間指定での状態復元（「フルスナップショット＋デルタ群から、任意時刻tの状態を再構成する」——[06_Logging_Replay_JP.md](./06_Logging_Replay_JP.md)のリプレイ機能）を提供する共通ライブラリを持てば、準拠するすべてのシミュレーションノードが同じロジックを共有でき、各ノードが個別に再実装せずに済む。ただしスキーマが固まる前に今作ると、変更のたびに手戻りが発生し、まだ意図的に開いたままにしている論点を実装によって既成事実化してしまうリスクがある。
- 作るとしても、[09_Build_Strategy_JP.md](./09_Build_Strategy_JP.md) でgRPCについてすでに決めている「一箇所で定義し、各言語向けに生成する」方式（`.proto` → `protoc` → 一元管理された生成スクリプト経由で各言語のコードを生成——同章のビルド方針に記述されているだけで、本リポジトリにまだ実装済みファイルとして存在するわけではない）に倣うべきであり、言語ごとに独立して手書き・保守するライブラリにはすべきではない。後者は、原則1がまさに避けようとしている「同じロジックの重複実装が言語間でズレていく」問題を再現してしまう。ただし生成で解決できる範囲には限りがある：`protoc`的な生成ツールが作るのは型付きデータバインディングとシリアライズ／デシリアライズのコードであり、スナップショットのマージ／リプレイアルゴリズムや、より踏み込んだ検証制約（例：項目6の「関節フィールドは両方セットで必要」）までは生成しない。これらは生成物であるかどうかにかかわらず、別途意図的に共有された実装を持つ必要があり、そうしないと同じように言語間でズレていく。
- 想定される実装の進め方（未合意）：まずPyBulletFleet単体で（本リポジトリの設計を参照しながら）実装する。ただしこれは、共通層を抽象化する前に2〜3個の具体アプリを作るべきとする [16_Current_Status_and_Rollout_Approach_JP.md](./16_Current_Status_and_Rollout_Approach_JP.md) のボトムアップ方針とすり合わせが必要である——単一のアプリから抽出すると、PyBulletFleet固有のリプレイ意味論を、あたかも一般的なシミュレータ横断の契約であるかのように固定してしまうリスクがある。2つ目の具体実装ができるまでは、この段階で抽出するライブラリは、共通のシミュレータ横断ライブラリに格上げするのではなく、PyBulletFleet専用と明記してスコープを限定すべきである。
- 関連ドキュメント：[06_Logging_Replay_JP.md](./06_Logging_Replay_JP.md)、[09_Build_Strategy_JP.md](./09_Build_Strategy_JP.md)、[16_Current_Status_and_Rollout_Approach_JP.md](./16_Current_Status_and_Rollout_Approach_JP.md)、[20_MetaData_Extensibility_Patterns_JP.md](./20_MetaData_Extensibility_Patterns_JP.md)

## 11. カメラアセットの静的な較正情報（内部パラメータ）の正確な契約

- 背景：[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) と [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) は、カメラの内部パラメータ（焦点距離・解像度・画角）は`reproduction_info`ではなくそのアセットの静的なモデル記述に属すると結論づけたが、その静的カメラ契約自体はまだ定義されていない——スキーマの`model`フィールドは単なるパスであり、較正用スキーマや参照規約は未指定である。カメラの内部パラメータを`reproduction_info`から除外するという結論は、この静的契約（例：アセットの`properties`配下のフィールドとして、または参照先モデルファイル自体のフォーマットの一部として）が定義されて初めて完結する。
- 関連ドキュメント：[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)、[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)

## 12. エピソードごとにサンプリングされる、動力学に影響するDomain Randomizationの値はどこに持つか

- 背景：[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) は、動力学に影響するDomain Randomization（摩擦、質量、アクチュエータゲインなど）を`reproduction_info`から除外している。これらの値はシミュレーションの続行のされ方を変えるため、任意扱いのreproduction infoではなくシミュレーション状態として扱うべきだという理由による。しかし、これらを保持する規範的なフィールドは現状存在しない：唯一のアセット単位の拡張ポイントは自由記述の`properties`フィールドだが、これは特定のキーの存在を利用者が当てにできる保証を一切与えないと明記されている。リプレイがこれらの値を決定的に再構成できるよう、どこに記録すべきか（新たな型付きのアセット単位フィールド、`properties`の規約、あるいは他の方法）はまだ決まっていない。
- 関連ドキュメント：[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)、[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)

## 関連ドキュメント

- [11_Design_Principles_JP.md](./11_Design_Principles_JP.md)
- [12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md)
- [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md)
- [14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md)
- [15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md)
- [16_Current_Status_and_Rollout_Approach_JP.md](./16_Current_Status_and_Rollout_Approach_JP.md)
- [18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md)
- [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md)
- [20_MetaData_Extensibility_Patterns_JP.md](./20_MetaData_Extensibility_Patterns_JP.md)
