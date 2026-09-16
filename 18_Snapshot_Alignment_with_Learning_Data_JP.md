# 18. スナップショット仕様と外部の学習データ記録フォーマットとの整合

**Status:** Draft
**Date:** 2026-08-31

> このドキュメントは、[12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md) と [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md) を発展させた、後日の設計議論の結論を記録するものであり、新たな設計を発明するものではない。未解決点は [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) に追加する。

## 背景：スナップショット形式の2人目の独立した利用者

本リポジトリの外にある、テレオペレーション（遠隔操作介入）データ記録の取り組み（模倣学習的な用途を想定）は、各記録ステップにおける「世界の状態」を表す observation を必要としている。この取り組みは、独自の状態表現を新たに定義するのではなく、**USO の SimulationSnapshot と同じものとして observation を定義する**という判断を下している。

これは USO にとって重要な意味を持つ。スナップショット形式はもはや USO 内部の関心事（初期化・同期・リプレイ — [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)）にとどまらず、**外部の利用者にとっての状態表現の source of truth** としても頼られている、ということである。スナップショットのスキーマを変更する際、利害関係者がもう一人増えたことになる。

## 浮上した要件：「復元十分性」には関節レベルのフィールドが要る

外部の取り組みでは、observation が **復元十分（restoration-sufficient）** であることを要件としている。すなわち、その記録時点からシミュレーションを再開して同じ続きが得られるだけの情報を持つこと。多関節ロボットの場合、これは具体的には、ポーズだけでなく**速度**（ポーズだけでは動作中の状態が復元できない）、そして**関節角度・関節速度**を意味する。ロボット全体の剛体としてのポーズだけでは足りない。

現在の [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) のスキーマ例は、アセットごとに剛体のフィールド（`position`、`orientation`、`linear_velocity`、`angular_velocity`）のみを示しており、多関節アセットの関節ごとの状態を置く場所がない。これは単独では設計上の欠陥というより、外部のユースケースによって顕在化した具体的なギャップである。USO 自身はこれまで、多関節ロボットの内部関節状態までリプレイする必要に迫られていなかったため、この点はまだ検証されていなかった。

**結論**：フル／差分スナップショットのスキーマを、多関節アセットの関節ごとの状態（関節角度・関節速度）を含む形に拡張すべきである。これにより、ロボットのルートポーズだけでなく、内部自由度についてもスナップショットが復元十分であり続ける。具体的なフィールド形状は未定のまま残す（[17_Open_Questions_JP.md](./17_Open_Questions_JP.md) 参照）。本ドキュメントは、このギャップが存在すること、そしてそれを埋める場所は別フォーマットではなくスナップショットであるべきこと、のみを記録する。

これは [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) の原則2（source of truth は情報量の多い方であり、そこから情報を減らす方向にのみ派生する）が、状態 vs. レンダリング画像という形で具体的に現れた例である。

## 既存のフル／差分の分割は、このユースケースにもそのまま当てはまる

外部フォーマットは、「静的構造」（どんなアセットが存在するか、その type/model）と「動的状態」（あるステップでのポーズ・速度・関節状態）を別々に必要としている。これは既存のフル／差分スナップショットの分割（[03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md)）にそのまま対応する。静的構造は、あらゆるフルスナップショット——[06_Logging_Replay_JP.md](./06_Logging_Replay_JP.md) に記載の、定期的に保存されるリプレイアンカーとしてのフルスナップショットも含め、初回だけではなく——に自然に含まれる（フルスナップショットは定義上、常に完全な状態である）。その間の差分は、静的構造が変化しないため、単にそれを省略するだけである。ここに新しい仕組みは不要であり、本項は新たな決定ではなく、既存設計がこの外部ユースケースにもそのまま一般化することの確認として記録する。

## 再現情報（カメラ・照明・シード）は別の関心事

外部の取り組みでは、sim データについて、記録ステップごとにレンダリング画像を保存することを避け、後から状態を元に再生成したいと考えている。sim では状態が source of truth であり、画像はそこからの派生的・再現可能な産物である、という理由による。これを成立させるには、アセットごとの世界状態を超えた追加情報（カメラの位置・姿勢・内部パラメータ、照明、Domain Randomization 設定、使用した乱数シード）の保持が必要になる。

この「再現情報」は世界状態そのものの一部ではない（あるシミュレートされたアセットの状態を表すのではなく、その状態から観測者／レンダラーがどう画像を再構成するかを記述するものである）。**後日の議論で解決**：スナップショット形式の中に、独立した `reproduction_info` セクション（グローバル、アセット単位ではない）として含めることになった。同時に導入された汎用の `meta_data` フィールドとは明確に区別する。解決の経緯と根拠は [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) を参照。

## 関連ドキュメント

- [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) — 本章が拡張を提案しているスナップショットのスキーマ
- [11_Design_Principles_JP.md](./11_Design_Principles_JP.md) — 原則2（source of truth は詳細→簡略にのみ派生）を、状態 vs. レンダリング画像に適用したもの
- [12_Layering_and_ML_Boundary_JP.md](./12_Layering_and_ML_Boundary_JP.md) — この外部利用者が背後に立つ state/data supply API の境界
- [13_Reuse_Layers_and_Format_Selection_JP.md](./13_Reuse_Layers_and_Format_Selection_JP.md) — 状態レイヤーのフォーマット選定
- [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) — 本章が追加した未解決の論点
