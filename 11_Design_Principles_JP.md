# 11. USOを貫く設計原則

**Status:** Draft
**Date:** 2026-08-17

> この文書は、設計議論で到達した結論を記録したものであり、この時点で新たに設計を発明するものではない。未解決の論点は [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) にまとめている。

## 解きたい問題

シミュレータやロボットが変わるたびに、本来同じであるはずのもの（環境レイアウト、ロボット構造、タスク論理、リカバリーロジックなど）を作り直している。これは USO 以前の状態で実際に起きている無駄であり、USO はこの**「本質と無関係な作り直し」をなくすこと**を目的とする。

## 原則1：変わる理由が違うものを分け、その境界を「共通の形式」でつなぐ

- ある要素がシミュレータやロボットを変えるたびに変わる理由は、常に同じではない。「本質的にそのタスク・ロボットが持つ性質」が変わる場合と、「そのシミュレータ固有の都合（摩擦モデル、レンダラー、DOFの持ち方など）」で変わる場合がある。
- この2つを分けずに扱うと、シミュレータを増やすたびに本質部分まで巻き込んで作り直すことになる。
- したがって USO は、**変わる理由が異なるものを構造的に分離し、その境界を共通フォーマット（OpenUSD, URDF/SDF, Behavior Tree XML, スナップショット）で橋渡しする**、という設計を貫く。
- この原則は本文書群のほぼすべての章（[12](./12_Layering_and_ML_Boundary_JP.md)〜[16](./16_Current_Status_and_Rollout_Approach_JP.md)）に通底する基準線であり、個別の設計判断に迷ったときはここに立ち戻る。

## 原則2（派生）：source of truth は情報量の多い側に置き、情報を減らす方向にのみ派生させる

- 複数の表現（詳細な関節構造 ⇄ 簡略化されたキネマティクス表現、フル物理 ⇄ ロジックのみ、など）が同じ対象を指す場合、どちらを正とするかを決めておく必要がある。
- 情報量の多い表現から情報量の少ない表現への変換（詳細 → 簡略）は、情報を捨てる操作なので常に可能。逆（簡略 → 詳細）は失われた情報を復元できないため、原理的に不可能。
- したがって、**最も情報量の多いものを source of truth とし、そこから情報を減らす方向にのみ派生させる**。
- この原則は、フォーマット選択（[13](./13_Reuse_Layers_and_Format_Selection_JP.md)）、個別調整の扱い（[14](./14_Engine_Specific_Adjustment_JP.md)）、簡略化の自動化戦略（[15](./15_Simplification_Automation_Strategy_JP.md)）で具体的に適用されている、この文書群全体を貫くもう一つの軸である。

## なぜこの2原則が起点になるのか（根拠）

- 「変わる理由の分離」がないと、エンジン追加のたびに本質（タスク論理・ロボット構造・環境レイアウト）まで壊れる。これは USO が最初に解決したい問題そのものである。
- 「source of truth の一方向性」がないと、どの表現を正としてメンテナンスすべきかが曖昧になり、詳細版と簡略版が互いに矛盾したまま放置される。一方向性を明示することで、変換ツールの設計（どちらからどちらへ変換を書けばよいか）が自動的に決まる。

## 関連ドキュメント

- [12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md) — レイヤーと責務の分離、MLエコシステムとの関係
- [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md) — 層別の流用可能性とフォーマット選択
- [14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md) — 個別調整（overlay / 変換ツール）の扱い
- [15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md) — 簡略化の自動化戦略
- [16_Current_Status_and_Rollout_Approach_JP.md](./16_Current_Status_and_Rollout_Approach_JP.md) — 既存アプリ／specとの接続状況
- [18_Snapshot_Alignment_with_Learning_Data_JP.md](./18_Snapshot_Alignment_with_Learning_Data_JP.md) — 原則2を、状態 vs. レンダリング画像（スナップショット形式の外部利用者向け）に適用したもの
- [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) — 原則1を適用し、アセット単位の状態・再現情報・自由記述メタデータをスナップショットの別々のフィールドに分割したもの
- [20_MetaData_Extensibility_Patterns_JP.md](./20_MetaData_Extensibility_Patterns_JP.md) — `meta_data`/`properties`内の将来の型付き拡張機構についての先行事例と候補案（探索段階、未決定）
- [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) — 未解決の論点
- [02_Architecture_JP.md](./02_Architecture_JP.md) — 本原則が実装されているレイヤー構造
