# RDE開発者向け--FirstTouch--資料

## RDEの構造化処理開発は、まずはこの資料から始めましょう

RDE (Research Data Express) は、物質・材料についての研究データをオンラインで迅速に登録するため国立研究開発法人物質・材料研究機構（以下、NIMS）が開発したシステムです。生データを登録すると自動的にデータ駆動型のマテリアル研究に適した形に構造化してクラウドに蓄積します。これによりユーザーや研究グループ内での再利用や他の研究グループとのデータの共用が容易となり、マテリアル研究開発のDX化を支援します。

ここで提供する資料は、RDEにおけるデータセットテンプレート開発をするための解説とハンズオン資料です。

英語版を用意していないため、各資料のmarkdown形式のファイルを掲載しています。翻訳をしてご利用ください。

## 提供する資料について

1. RDEデータ構造化処理開発のための予備知識
    - RDEのデータセットテンプレート開発を始めるための予備知識の説明書です。
    - [doc1/RDEデータセットテンプレートの開発を始める前に.pdf](doc1/RDEデータセットテンプレートの開発を始める前に.pdf)
    - [markdown形式ドキュメント](doc1/documents/)
2. RDEデータ構造化とデータセットテンプレート解説
    -   RDEデータセットテンプレートとデータ構造化に関する詳細な説明資料です。
    -   ハンズオンで実践において不明な用語や、テンプレート開発で仕様の確認をしたいときにご利用ください。
    -   [doc2/RDEデータ構造化とデータセットテンプレート解説.pdf](doc2/RDEデータ構造化とデータセットテンプレート解説.pdf)
    -   [doc2/rde-datasettemplate-instructions_appendix.zip](doc2/rde-datasettemplate-instructions_appendix.zip)
    -   [markdown形式ドキュメント](doc2/documents/)
3. RDEToolKitを使ったRDE構造化処理ハンズオンチュートリアル
    - データセットテンプレートのサンプルを動かしてみるハンズオン資料です。
    - まず手を動かしてみてください。
    - そして、用語のことや仕組みをもっと知りたくなったらRDEデータ構造化とデータセットテンプレート解説を確認してみてください。
    - [doc3/RDEToolKit(invoiceモード)を利用したシンプルなRDEデータ構造化処理プログラムハンズオン.pdf](doc3/RDEToolKit%28invoiceモード%29を利用したシンプルなRDEデータ構造化処理プログラムハンズオン.pdf)
    - [markdown形式ドキュメント](doc3/documents/)
    - [doc3/handson_codes.zip](doc3/handson_codes.zip)
    - [doc3/samples.zip](doc3/samples.zip)
4. RDEToolKitの各登録モードの利用事例ハンズオン
    - RDEToolKitには予め用意された登録モードが数種類あります。各モードを動かしてみるためのハンズオン資料です。
    - [doc4/RDEToolKitの各登録モードの利用事例.pdf](doc4/RDEToolKitの各登録モードの利用事例.pdf)
    - [markdown形式ドキュメント](doc4/documents/)
    - [doc4/sample_data.zip](doc4/sample_data.zip)
5. RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書
    - [doc5/RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書.pdf](doc5/RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書.pdf)
    - [markdown形式ドキュメント](doc5/documents/)


## オススメの読み方

1. まずはデータセットテンプレートとその開発について一読を
    - RDEデータセットテンプレート開発における用語や概略をまずは確認<br>
    [doc1/RDEデータセットテンプレートの開発を始める前に.pdf](doc1/RDEデータセットテンプレートの開発を始める前に.pdf)
    - でも、その前にRDEサービスについてこちらで確認しておいてください。[DICEサービスRDE紹介ページ](https://dice.nims.go.jp/services/RDE/)
2. とにかくハンズオン
    - 習うより慣れろ、ということでとにかく動かしてみましょう
    - [doc3/RDEToolKit(invoiceモード)を利用したシンプルなRDEデータ構造化処理プログラムハンズオン.pdf](doc3/RDEToolKit%28invoiceモード%29を利用したシンプルなRDEデータ構造化処理プログラムハンズオン.pdf)
    - 参考プログラムなどは[こちら](doc3)から取得してください
3. もっと知りたい、参考プログラムを改造したくなったら
    - テンプレートを改造してみたい、どういうしくみか知りたくなったらこちら<br>
    [doc2/RDEデータ構造化とデータセットテンプレート解説.pdf](doc2/RDEデータ構造化とデータセットテンプレート解説.pdf)
    - 付属のファイルはは[こちら](doc2)
4. 他に参考プログラムないの？
    - 4つの登録事例のサンプルプログラムを試してみてください
    - [doc4/RDEToolKitの各登録モードの利用事例.pdf](doc4/RDEToolKitの各登録モードの利用事例.pdf)
    - プログラムコードは[こちら](doc4)
5. 実際に使えるデータセットテンプレートはないの？
    - こちらを参考にしてください
      - [RDE_XRD](https://github.com/nims-mdpf/RDE_XRD)
      - [RDE_XPS](https://github.com/nims-mdpf/RDE_XPS)
      - [RDE_SIMPLE_REGISTRATION](https://github.com/nims-mdpf/RDE_SIMPLE_REGISTRATION)
    - 順次追加していきますので[NIMS-MDPF github](https://github.com/nims-mdpf/)をご確認ください
6. Dockerコンテナを使って開発をしたいという方はこちら
    - [doc5/RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書.pdf](doc5/RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書.pdf)
7. RDEToolKitのデータ登録モードについてもっと知りたい方はこちら
    - [RDE_rdetoolkit_5mode_templates](https://github.com/nims-mdpf/RDE_rdetoolkit_5mode_templates)
       - RDEToolKitのデータ登録モード5種類について動かしながら学べるようになっています



## 関連リンク
- [RDEToolKit](https://github.com/nims-mdpf/rdetoolkit)
- [RDEToolKit in PyPi](https://pypi.org/project/rdetoolkit/)
- [RDE/データセットテンプレート生成、確認ツール](https://github.com/nims-mdpf/RDE_datasettemplate-schemafile-make-tool)
- [RDE構造化処理結果プレビューツール](https://github.com/nims-mdpf/RDE_structured-result-preview-tool)
- [RDE_rdetoolkit_5mode_templates](https://github.com/nims-mdpf/RDE_rdetoolkit_5mode_templates)

## 利用ルールおよびライセンス
 
* 本プログラムはMITライセンスで提供されています。


### RDEおよび本ツール関するお問い合わせ

ご不明な点につきましては、以下にお問い合わせください。

国立研究開発法人物質・材料研究機構
技術開発・共用部門 材料データプラットフォーム

お問い合わせ フォーム<br>
https://dice.nims.go.jp/contact/form.html

2025.12.05 更新