# RDE開発者向け--FirstTouch--資料

## RDEの構造化処理開発は、まずはこの資料から始めましょう

RDE (Research Data Express) は、物質・材料についての研究データをオンラインで迅速に登録するため国立研究開発法人物質・材料研究機構（以下、NIMS）が開発したシステムです。生データを登録すると自動的にデータ駆動型のマテリアル研究に適した形に構造化してクラウドに蓄積します。これによりユーザーや研究グループ内での再利用や他の研究グループとのデータの共用が容易となり、マテリアル研究開発のDX化を支援します。

ここで提供する資料は、RDEにおけるデータセットテンプレート開発をするための解説とハンズオン資料です。

## 提供する資料について

1. RDEデータ構造化処理開発のための予備知識
    - RDEのデータセットテンプレート開発を始めるための予備知識の説明書です。
    - [doc1/RDEデータセットテンプレートの開発を始める前に.pdf](doc1/RDEデータセットテンプレートの開発を始める前に.pdf)
2. RDEデータ構造化とデータセットテンプレート解説
    -   RDEデータセットテンプレートとデータ構造化に関する詳細な説明資料です。
    -   ハンズオンで実践において不明な用語や、テンプレート開発で仕様の確認をしたいときにご利用ください。
    -   [doc2/RDEデータ構造化とデータセットテンプレート解説.pdf](doc2/RDEデータ構造化とデータセットテンプレート解説.pdf)
    -   [doc2/rde-datasettemplate-instructions_appendix.zip](doc2/rde-datasettemplate-instructions_appendix.zip)
3. RDEToolKitを使ったRDE構造化処理ハンズオンチュートリアル
    - データセットテンプレートのサンプルを動かしてみるハンズオン資料です。
    - まず手を動かしてみてください。
    - そして、用語のことや仕組みをもっと知りたくなったらRDEデータ構造化とデータセットテンプレート解説を確認してみてください。
    - [doc3/RDEToolKit%28invoiceモード%29を利用したシンプルなRDEデータ構造化処理プログラムハンズオン.pdf]([doc3/RDEToolKit(invoiceモード)を利用したシンプルなRDEデータ構造化処理プログラムハンズオン.pdf](https://github.com/nims-mdpf/RDE_Docs_firsttouch/blob/main/doc3/RDEToolKit(invoice%E3%83%A2%E3%83%BC%E3%83%89)%E3%82%92%E5%88%A9%E7%94%A8%E3%81%97%E3%81%9F%E3%82%B7%E3%83%B3%E3%83%97%E3%83%AB%E3%81%AARDE%E3%83%87%E3%83%BC%E3%82%BF%E6%A7%8B%E9%80%A0%E5%8C%96%E5%87%A6%E7%90%86%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E3%83%8F%E3%83%B3%E3%82%BA%E3%82%AA%E3%83%B3%20.pdf))
    - [doc3/handson_codes.zip](doc3/handson_codes.zip)
    - [doc3/samples.zip](doc3/samples.zip)
4. RDEToolKitの各登録モードの利用事例ハンズオン
    - RDEToolKitには予め用意された登録モードが数種類あります。各モードを動かしてみるためのハンズオン資料です。
    - [doc4/RDEToolKitの各登録モードの利用事例.pdf](doc4/RDEToolKitの各登録モードの利用事例.pdf)
    - [doc4/sample_data.zip](doc4/sample_data.zip)
5. RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書
    - [doc5/RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書.pdf](doc5/RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書.pdf)

## 関連リンク
- [RDEToolKit](https://github.com/nims-mdpf/rdetoolkit)
- [RDEToolKit in PyPi](https://pypi.org/project/rdetoolkit/)
- [RDE/データセットテンプレート生成、確認ツール](https://github.com/nims-mdpf/RDE_datasettemplate-schemafile-make-tool)
- [RDE構造化処理結果プレビューツール](https://github.com/nims-mdpf/RDE_structured-result-preview-tool)
## 利用ルールおよびライセンス
 
* 本プログラムはMITライセンスで提供されています。


### RDEおよび本ツール関するお問い合わせ

ご不明な点につきましては、以下にお問い合わせください。

国立研究開発法人物質・材料研究機構
技術開発・共用部門 材料データプラットフォーム

お問い合わせ フォーム<br>
https://dice.nims.go.jp/contact/form.html

2025.5.29 公開