<div class="page"/>

# テスト用データの用意

次章以降で利用するテスト用のデータを取得します。

## テスト用データ取得

テスト用のデータファイルは、以下のサイトにて`sample_data.zip`として配布されています。

* https://github.com/nims-mdpf/RDE_Docs_firsttouch/tree/main/doc4

> ダウンロードの際は、Rawファイルのダウンロードを実行してください。URLの末尾が"sample_data.zip"だったとしても、HTMLファイルがダウ
ンロードされる場合があります。その場合は、展開(unzip)実行時に"…… this file is not a zipfile ……"のようなメッセージが表示されます。

ダウンロードし、展開します。

```bash
cd $HOME/rde-sample
wget https://github.com/nims-mdpf/RDE_Docs_firsttouch/raw/refs/heads/main/doc4/sample_data.zip
```

> wget実行にプロキシ設定が必要な場合は、適切に設定後実行してください。

## テスト用データの展開

続いて、取得した`sample_data.zip`を展開します。

```bash
(venv) $ mkdir sample_data
(venv) $ unzip sample_data.zip -d sample_data/
Archive:  sample_data.zip
   creating: sample_data/excelinvoice/
 extracting: sample_data/excelinvoice/data.zip
 extracting: sample_data/excelinvoice/sample-data.txt
  inflating: sample_data/excelinvoice/sample_excel_invoice.xlsx
  inflating: sample_data/excelinvoice/Template_excel_invoice.xlsx
   creating: sample_data/multidatatile/
  inflating: sample_data/multidatatile/Mo50Ti15C-20230111-10mN-00.tif
  inflating: sample_data/multidatatile/materials.svg
  inflating: sample_data/multidatatile/f27.bmp
  inflating: sample_data/multidatatile/report.pdf
   creating: sample_data/rdeformat/
 extracting: sample_data/rdeformat/data.zip
 extracting: sample_data/rdeformat/structured.zip
```

`sample_data/`フォルダは以下の様になります。

```bash
$ tree sample_data
sample_data
├── excelinvoice
│   ├── Template_excel_invoice.xlsx
│   ├── data.zip
│   ├── sample-data.txt
│   └── sample_excel_invoice.xlsx
├── multidatatile
│   ├── Mo50Ti15C-20230111-10mN-00.tif
│   ├── f27.bmp
│   ├── materials.svg
│   └── report.pdf
├── rdeformat
│   ├── data.zip
│   └── structured.zip
└── smarttableinvoice
    ├── common.zip
    ├── data.zip
    ├── data0.zip
    ├── data1.zip
    └── data2.zip

5 directories, 15 files
```

> コピーなどによりファイルの日付が変わることがあります。日付が違っていても実行には問題ありません。

以降の章では、この`sample_data/`から必要なファイルをコピーし、適切なフォルダに配置することで実行できるようにします。
