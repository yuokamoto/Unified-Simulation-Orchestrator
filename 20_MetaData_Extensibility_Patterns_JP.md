# 20. `meta_data` / `properties` の拡張パターン：参考にした先行事例と候補案

**Status:** Draft — 探索段階（未決定）
**Date:** 2026-09-16

> このドキュメントは設計上の決定を記録するものではない。設計議論の中で調べた先行事例と、そこから浮かんだ候補案を、[17_Open_Questions_JP.md](./17_Open_Questions_JP.md) の項目9に取り組む人のために記録するものである。[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) はすでに「`reproduction_info`（コア・型付き・機能上必須）」と「`meta_data`/`properties`（自由記述・無保証）」を構造的に分離することを決定しているが、本ドキュメントはその決定からさらに浮かんだ、別の論点を探索する。

## 論点

[19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) では、`reproduction_info`を`meta_data`の中の特定キーとして統合しても（そのキーだけにrequiredなスキーマを与えることで）、required/自由記述という区別自体はなくならず、1階層内側に移るだけだと結論づけた。この整理から、さらに別の、将来を見据えた論点が浮かぶ：**USOコア以外の、特定の外部利用者が、`meta_data`（あるいはアセット単位の`properties`）の中に自分専用の名前付き・型付きスキーマを登録し、USOコアがその利用者のスキーマを事前に知らなくても、`reproduction_info`自身の将来のスキーマ（それ自体もまだ未確定——項目8参照）で意図されているのと同種の信頼性（検証・必須フィールド）を得られるようにすべきか？**

これは現時点の既知の要件（`reproduction_info`、関節state）には不要である。この議論で、まさに同じ課題に対する社内の実例と、複数の確立されたオープンソースのパターンが見つかったため、後で一から再発見しなくて済むよう、設計の近くに記録しておく。

## 参考：先行事例

「**typeタグがどのスキーマに従うペイロードかを識別し、あるtypeのスキーマはそれを定義した者が所有・バージョン管理する**」という考え方のバリエーションは、広く使われている複数のシステムに繰り返し現れる。ただし細部は異なる（認識できないtypeを黙って無視するか、剪定するか、拒否するか。そもそも全てのペイロードが明示的なtypeタグを持つとは限らない）：

- **Kubernetes** — `labels`/`annotations`（`<domain>/<name>`の名前空間付きキー。Kubernetes組み込みの構文・長さ制約はあるが、利用者定義スキーマによる検証はない） vs. **CustomResourceDefinition**（`kind`ごとに登録されたOpenAPI v3スキーマを持ち、`required`フィールドや検証を持てる。未知フィールドの保持も可能）
- **Protocol Buffers** — `google.protobuf.Any`（`type_url`＋シリアライズされたペイロード。そのtypeを知っているconsumerだけが展開する）と、`Struct`/`Value`というwell-known type（公式に用意された「自由記述JSON」の逃げ道。厳密に型付けされたメッセージと対比される）
- **CloudEvents**（CNCF） — イベントは`type`（イベント種別を識別）と`data`ペイロードを持つ。`data`が従うスキーマは別属性の任意項目`dataschema`が指し示し、それを定義した者が所有・バージョン管理する。`type`を認識しない（あるいはスキーマを持たない）consumerでも、イベントを不透明なままルーティング・保存することは可能
- **OCI**（コンテナ／アーティファクト仕様） — `mediaType`/`artifactType`フィールドが、それ自体は不透明なペイロードの解釈方法を宣言する
- **JSON Schemaの`oneOf` ＋ OpenAPI 3の`discriminator`** — `oneOf`（JSON Schemaのキーワード）は、値が複数のスキーマのうちちょうど1つに一致することを要求し、実際の検証を担うのはこちらである。`discriminator`（JSON Schema自体には無い、OpenAPI独自の拡張）は新しいフィールドを追加するのではなく、`propertyName`によってペイロード中の**既存の**プロパティを指し示し、その値をスキーマへ任意にマッピングするだけであり、これにより`oneOf`の各候補を総当たりで試す代わりに、ツールが直接正しい分岐を選べるようにする
- **glTF**（Khronos） — コア仕様に加え、登録された`extensions`名前空間を持つ。`extensionsUsed`/`extensionsRequired`という配列により、ファイルが「ローダーがどの拡張を理解しなければ正しく解釈できないか」と「無視してよい拡張はどれか」を明示的に宣言できる
- **OpenUSD**（Pixar） — USOがすでにアセット形式として依拠しているため直接関連が深い：プラグイン登録された **IsA / API schema** が、フォールバック／デフォルト値を伴う型付き属性セットを定義する（これはUSD自体のスキーマ／デフォルト値の意味論であり、JSON Schemaのような必須フィールド検証を強制するものではない）。**`customData`/`assetInfo`** はそれ以外の自由記述辞書として残る。これはUSOがすでに`reproduction_info` vs. `meta_data`で行った二層分割と同じものが、USOがすでに使っているフォーマット自体にネイティブに存在する例
- **OpenTelemetry** のsemantic conventions — 名前空間付きのwell-knownな属性キーと、任意のカスタム属性が同じオブジェクト上に共存する
- **Confluent / Apicurio Schema Registry** — 中央レジストリがsubject（≒type名）をバージョン管理されたスキーマに対応づけ、バージョン間の互換性ルールを強制する。USOが`reproduction_info`や関節単位のフィールド（[17_Open_Questions_JP.md](./17_Open_Questions_JP.md) の項目6・8）を、既存利用者を壊さずにどう進化させるかの参考になる

## USOへの候補案（順不同、未決定）

- **名前空間規約のみ** — `meta_data`/`properties`内のキーに`<domain>/<name>`形式を義務付ける（Kubernetesのannotationと同様）。インフラ不要。利用者間の衝突は防げるが、検証・必須性までは解決しない
- **型付き値のラッパー** — `{"type": "<name>", "value": {...}}`という形（`Any`パターン）にし、`type`を認識したconsumerだけが自分が所有するスキーマで`value`を検証できるようにする
- **スナップショット単位の`schema_version`** — スナップショット形式自体にバージョン標識を持たせ、`reproduction_info`や関節stateフィールドの形状を、`meta_data`の論点とは独立に、互換性・変換の道筋を持ちながら進化させられるようにする
- **外部登録スキーマ**（CRD的な仕組み） — 利用者が自分の名前付き拡張のスキーマ（例：JSON Schema）を既知の場所に公開し、ツールがその名前に一致する`meta_data`/`properties`エントリだけをそれに照らして検証し、それ以外はそのまま扱う

これらのどれを選ぶか（あるいは組み合わせるか）、そもそも実際の外部利用者から要望が出る前に作る価値があるかは、未決定のまま残す。

## 関連ドキュメント

- [19_Snapshot_MetaData_and_Reproduction_Info_JP.md](./19_Snapshot_MetaData_and_Reproduction_Info_JP.md) — 本ドキュメントが続きの論点を探索している決定
- [03_Snapshot_Specification_JP.md](./03_Snapshot_Specification_JP.md) — `meta_data`/`properties`が現在定義されている場所（自由記述・スキーマなし）
- [17_Open_Questions_JP.md](./17_Open_Questions_JP.md) — 本探索から記録された項目9
