# 17. Open Questions（未解決の論点）

**Status:** Draft
**Date:** 2026-08-17

> ここに列挙する論点は、2026-08-17時点の設計議論では結論が出ていない。agent が勝手に埋めるものではなく、意思決定者が判断した時点で、該当ドキュメントとあわせて更新すること。

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

## 関連ドキュメント

- [11_Design_Principles_JP.md](./11_Design_Principles_JP.md)
- [12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md)
- [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md)
- [14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md)
- [15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md)
- [16_Current_Status_and_Rollout_Approach_JP.md](./16_Current_Status_and_Rollout_Approach_JP.md)
