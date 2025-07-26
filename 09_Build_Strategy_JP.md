# 9. ビルド戦略とディレクトリ構成

## 目的
本章では、シミュレーションフレームワークのビルドおよびコード生成戦略、  
多言語（Python, C++, JavaScript）環境での統合的なビルドフローを定義する。  
将来的な **Bazelによるモノリポ統合** も見据える。

---

## ビルド戦略

### 1. 現在のビルド方針
- **Python**  
  - `setuptools` と `pip` を利用し、ライブラリやツールをパッケージング。
  - `poetry` をオプションでサポート（依存管理の簡略化）。
- **C++**  
  - CMakeベースでのビルドを採用。
  - シミュレーションノード（Gazebo、Genesis、Mujoco等）やgRPCサーバーのビルドを担当。
- **JavaScript (Node.js)**  
  - `npm`/`yarn` を利用し、Web GUIとWebSocketサーバーの依存管理を実施。

### 2. gRPCコード生成
- すべての言語で統一された `.proto` 定義を利用。
- `protoc` と各言語向けプラグインを用いてコード生成。
  - Python: `grpcio-tools`
  - C++: `grpc_cpp_plugin`
  - JavaScript: `grpc-tools`, `@grpc/proto-loader`
- コード生成は `scripts/gen_grpc.sh` で一括管理。

### 3. Bazelによる将来的な統合
- 各コンポーネントをBazelのビルドルールに移行し、モノリポジトリ化を進める。
- Bazel導入のメリット:
  - 多言語間依存の統一管理。
  - キャッシュとインクリメンタルビルドによるCI/CD効率化。
  - gRPCコード生成をBazelターゲットとして統合可能。

---

## ディレクトリ構成案

```
/project-root
  /proto                  # gRPCの.protoファイル（全言語共通）
  /scripts                # ビルド/コード生成/CIスクリプト
  /python
    /core                 # シミュレーションマスター、SimPyノード
    /tools                # スナップショット/シナリオ編集ツール
  /cpp
    /nodes                # Gazebo/Genesis/Mujocoノード
    /libs                 # 共通ライブラリ（スナップショット、BT、通信）
  /web
    /frontend             # React/Three.jsフロントエンド
    /server               # WebSocket/gRPC-Webサーバー（Node.js）
  /config                 # 環境設定、ビルド設定ファイル
  /tests                  # 統合テストとユニットテスト
  /docs                   # ドキュメント
```

---

## CI/CD
- **GitHub Actions** または **GitLab CI** を利用。
- 各言語ごとにビルド・テストジョブを設定。
- gRPCコード生成をビルド前に自動実行。
- 将来的にBazelに移行する場合は、CI/CDを1ジョブで統一可能。

---

## 拡張ポイント
- Bazelを正式導入することで、マルチプラットフォームクロスビルドやキャッシュ効率化を実現。
- Dockerを用いた環境統一（各ノード・ツールをコンテナでデプロイ可能）。
