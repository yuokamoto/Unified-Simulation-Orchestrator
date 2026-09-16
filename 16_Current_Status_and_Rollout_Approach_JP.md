# 16. USOと既存アプリ／specの接続状況（現状の位置づけ）

**Status:** Draft
**Date:** 2026-08-17

## 既存アプリ

- アーム（Isaac Sim）
- PyBulletFleet（キネマティクス／フリート）
- 今後追加予定：AGV（Isaac Sim または MuJoCo）

## 進め方の方針：ボトムアップ

- 具体アプリを2〜3個作ってから、統一層（USOの共通レイヤー）を抽象化する、という順序で進める。
- 抽象化には複数の具体例が要る。1つのアプリだけから抽象化すると、それがそのアプリ固有の都合なのか本質なのかを区別できない（[11_Design_Principles_JP.md](./11_Design_Principles_JP.md) の原則1に基づく判断）。

## 既存のUSD loader spec（PyBulletFleet側）との関係

PyBulletFleet側の既存の USD loader spec（PyBulletFleetリポジトリの [`docs/design/usd-behavior-tree/spec.md`](https://github.com/yuokamoto/PyBulletFleet/blob/main/docs/design/usd-behavior-tree/spec.md)。実装は `pybullet_fleet/usd_loader.py`）は、以下を決めている：
- 環境を USD で読む — サポート対象のメッシュを静的な `SimObject` に変換する、任意（optional）のOpenUSD静的世界インポータ
- prim パスを安定 ID として扱う — 「PyBulletのbody IDではなく、prim pathが外部から見た安定したsource IDである」（spec本文より）
- 変換を一度だけ正規化する — インポート時に、姿勢とメッシュスケールをPyBulletFleetのZ-up・メートル規約へ一度だけ正規化し、後段で再導出しない

この決定は、後続の snapshot/replay（[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)、[06_Logging_Replay_JP.md](./06_Logging_Replay_JP.md)）や USO の変換ツール（[14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md)）が前提とする「安定した参照」を正しく準備している。

## 棚卸しの観点

各アプリを次の5つの観点で棚卸しし、要素ごとに「本質」か「シミュレータ固有」かを仕分ける。

1. 状態
2. リセット
3. アセット
4. 時間
5. 表現

「シミュレータ固有」と判定されたものは、さらに [14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md) の分類に従って二分する：

- (a) パラメータ調整 → overlay
- (b) 構造の違い → 変換ツール

## 関連ドキュメント

- [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) — ボトムアップで進める根拠となる原則
- [14_Engine_Specific_Adjustment_JP.md](./14_Engine_Specific_Adjustment_JP.md) — 棚卸し後の(a)/(b)分類
- [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md), [06_Logging_Replay_JP.md](./06_Logging_Replay_JP.md) — USD loader specの決定事項が前提とする既存仕様
- PyBulletFleet リポジトリ、[`docs/design/usd-behavior-tree/spec.md`](https://github.com/yuokamoto/PyBulletFleet/blob/main/docs/design/usd-behavior-tree/spec.md) — 上記で参照しているUSD loader spec本体
- PyBulletFleet リポジトリ `docs/design/unified-spawn/spec.md` — エンティティ生成のYAML統一・登録の既存仕様（近縁のloader/spawn関連spec）
