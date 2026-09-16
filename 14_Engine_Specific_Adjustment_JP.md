# 14. シミュレータごとの個別調整の扱い

**Status:** Draft
**Date:** 2026-08-17

> USOの使いやすさを左右する核心的な論点。個別調整は必ず発生する前提のもと、「なくせるか」ではなく「本質を汚さずどう分離するか」を扱う。

## 前提

シミュレータごとの個別調整（摩擦係数の違い、センサー位置の違い、簡略化の度合いの違い、表現の粒度そのものの違いなど）は、USO を導入しても必ず発生する。したがって問うべきは「個別調整をなくせるか」ではなく、**「個別調整を本質（source of truth）を汚さずにどう分離するか」**である。

個別調整はその性質によって2種類に分けられる。

## (a) パラメータ調整 → overlay で分離

- 対象：摩擦係数、センサー位置、数値的な簡略化のしきい値（例：メッシュ簡略化の許容誤差やLOD距離）など、**値の違い**。注：リンク・関節・自由度そのものを削除するような、構造を変える簡略化は下記の(b)構造の違いに該当し、値の調整ではない。
- 扱い方：**overlay（本質への差分）** として分離する。本質（source of truth）自体は書き換えず、その上に差分を重ね、実行時に「本質 + エンジン用 overlay」を合成する。
- フォーマットレベルの裏付け：USD の sublayer/variant 機能が、この「本質 + overlay の実行時合成」をフォーマットレベルで直接支援する。
- 拡張案（要検討）：overlay に「目的ラベル」（例：実機再現用／学習高速化用）を持たせておくと、overlay の数が増えても目的別に管理できる。ただし MVP でこれをやるかどうかは過剰である可能性があり、**Open Question**として残す（[17_Open_Questions_JP.md](./17_Open_Questions_JP.md) 参照）。

## (b) 構造の違い → 変換ツールで分離、一方向性が前提

- 対象：詳細な関節ロボット ⇄ キネマティクスの点、のように**表現の粒度そのものが違う**場合。overlay では吸収できない（値の差分ではなく、構造そのものが異なるため）。
- 扱い方：**変換ツール**が、source of truth から各エンジン向けの表現を作り分けることで対応する。
- **一方向性が前提**：[11_Design_Principles_JP.md](./11_Design_Principles_JP.md) の原則2の直接適用として、詳細 → 簡略の変換は情報を捨てる操作なので可能だが、簡略 → 詳細は失われた情報を復元できないため不可能。したがって**詳細版を source of truth に保つ**ことが前提になる。
- 具体例：[13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md) で述べた URDF → USD の一方向変換は、この (b) の一事例である。
- 「詳細 → 簡略」という変換そのものをどこまで自動化できるかは、[15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md) で扱う。

## なぜこの2分類が核心なのか（根拠）

- (a) と (b) は必要な仕組みが根本的に違う。(a) は「差分の合成」で解決できるが、(b) は「構造の作り分け」が必要で、overlay という仕組みだけでは対応できない。この区別を曖昧にすると、パラメータの違いに過ぎないものを変換ツールで扱おうとして過剰実装になったり、逆に構造の違いを overlay で無理に吸収しようとして破綻したりする。
- USO の使いやすさは、個別調整が発生するたびに開発者が「これは (a) か (b) か」を素早く判断し、適切な仕組み（overlayか変換ツールか）に振り分けられるかどうかにかかっている。

## 関連ドキュメント

- [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) — 一方向性の原則
- [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md) — URDF→USDという(b)の具体例
- [15_Simplification_Automation_Strategy_JP.md](./15_Simplification_Automation_Strategy_JP.md) — (b)の変換をどこまで自動化するか
- [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) — overlayの目的ラベル、(a)/(b)どちらを最初の実装対象にするか
