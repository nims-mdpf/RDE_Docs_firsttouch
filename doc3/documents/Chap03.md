<div class="page" />

# サンプルデータの配置

実装に先立ち、チュートリアルで利用するデータを配置します。

> 本書で利用する入力データはテキストであり、かつ、大きなものではないので、本書から"コピー&ペースト"で準備することとします。

## sampla.data

入力データとして、`data/inputdata/sample.data`を以下の内容で作成します。

> コピーの際に、本書のフッタやヘッダが含まれないように注意してください。

data/inputdata/sample.data

```text
[METADATA]
data_title=data_title 2
measurement_date=2023/1/25 23:08
x_label=x(unit)
y_label=y(unit)
series_number=3
[DATA]
series1
16
1,101
2,102
3,103
4,104
5,105
6.2,106
7
8,108
9.5,109
10,101
11,103
12,105
13,104
14,100
15,98
16,82
[DATA]
series2
11
1,101
2,102
3,103
4,104
5,105
6,106
9,109
10,90
12,60
13,52
14,41
[DATA]
series3
12
1,101
1.5,101.2
2,102
3,103
4,104
5,105
6,106
9,109
10,90
12,60
13,52
14,41
```

## invoice.jsonファイルの更新

> 本来"invoice.json"ファイルは、RDEデータ登録画面からのデータ入力で自動的に作成された状態で構造化処理に渡ってきます。ローカル開発においては適切なinvoice.jsonを作成する手段がありませんので、本書では内容が入力されたinvoice.jsonを以下の様に手動作成することにします。

> 指定書式のExcelファイルに適切に設定することで、invoice.jsonを生成するツールを別途提供しています。以下のURLを参照ください。
>
>  https://github.com/nims-mdpf/RDE_datasettemplate-schemafile-make-tool

RDEToolKitのinit処理にて、invoice.jsonが作成されていますが、以下の内容で、`data/invoice/invoice.json`ファイルを上書きします。

```json
{
    "datasetId": "8fdec13d-bca2-4808-8753-2bb7e4e9c927",
    "basic": {
        "dateSubmitted": "2023-01-26",
        "dataOwnerId": "119cae3d3612df5f6cf7085fb8deaa2d1b85ce963536323462353734",
        "dataName": "NIMS_TRIAL_20230126b",
        "instrumentId": "413e53fb-aec9-41f8-ae55-3f88f6cd8d41",
        "experimentId": null,
        "description": ""
    },
    "custom": {
        "measurement_date": "2023-01-25",
        "invoice_string1": "送状文字入力値1",
        "invoice_string2": null,
        "is_devided": "devided",
        "is_private_raw": "private"
    },
    "sample": {
        "sampleId": "4d6de819-4b42-4dfe-9619-a0a0588653bc",
        "names": [
            "試料"
        ]
    }
}
```

> "invoice_string2"に指定しているnullは、データ登録時に入力が無かったことを意味します。RDEToolKitの古いバージョンでは、スキーマチェックで異常として検知されてしまうことがありました。その場合は、RDEToolKitのバージョンを新しいものに入れ替えてテストしてください。
>
> 本書の後半で、タイトル("basic"中の"dataName")に変更を加える処理を実施します。"dataName"の末尾に"/ (2024)"のような文言が付加されている場合は、当該処理が正常に稼働したかの確認が難しくなるので、この時点でその部分を削除しておいてください。

## invoice.schema.jsonの生成

> invoice.schema.jsonは、RDEデータ登録画面での入力画面を生成するのに用いられます。すなわち、invoice.jsonは、invoice.schema.jsonから、(少なくとも骨組みについては)生成されることになります。invoice.schema.jsonの作成は、開発者が実行する"開発"業務に含まれますが、それについては別途記述します。ここでは、以下の内容でinvoce.schema.jsonを作成(コピー&ペースト)してください。

`data/tasksupport/invoice.schema.json`を以下の内容で上書きします。

```json
{
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "$id": "TEST_NIMS_Sample_Template_v0.0",
    "description": "None",
    "type": "object",
    "required": [
        "custom",
        "sample"
    ],
    "properties": {
        "custom": {
            "type": "object",
            "label": {
                "ja": "固有情報",
                "en": "Custom Information"
            },
            "required": [],
            "properties": {
                "measurement_date": {
                    "label": {
                        "ja": "計測年月日",
                        "en": "measurement_date"
                    },
                    "type": "string",
                    "format": "date"
                },
                "invoice_string1": {
                    "label": {
                        "ja": "文字列1",
                        "en": "invoice_string1"
                    },
                    "type": "string"
                },
                "invoice_string2": {
                    "label": {
                        "ja": "文字列2",
                        "en": "invoice_string2"
                    },
                    "type": "string"
                },
                "is_devided": {
                    "label": {
                        "ja": "複数ファイル区分",
                        "en": "is_devided"
                    },
                    "type": "string"
                },
                "is_private_raw": {
                    "label": {
                        "ja": "公開区分",
                        "en": "is_private_raw"
                    },
                    "type": "string"
                }
            }
        },
        "sample" : {
            "type": "object",
            "label": {
                "ja": "試料情報",
                "en": "Sample Information"
            },
            "required": [
                "names"
            ],
            "properties": {
                "generalAttributes": {
                    "type": "array",
                    "items": [
                    ]
                }
            }
        }
    }
}
```

> RDEToolKit v1.3.2より前のバージョンでは、`data/tasksupport/invoice.schema.json`のルートノード直下の"required"句に"sample"の指定が無く、かつ、`data/invoice/invoice.json`にsample句が記述されている場合でもエラーにはなりませんでしたが、RDEToolKit v1.3.2以降のバージョンではバリデーション機能の強化に伴いエラーとなります。試料情報を入力する場合は、適切にinvoice.schema.jsonを記述する必要があります。

## metadata-def.jsonの変更

どういった項目を"メタデータ"とするかについてはmetadata-def.jsonファイルに定義しておく必要があります。

> metadata.jsonに値を設定して書き出す処理は、構造化処理の中で実施する必要があります。

`data/tasksupport/metadata-def.json`を以下の内容で上書きします。

```json
{
    "data_title": {
        "name": {
            "ja": "データ名",
            "en": "data title"
        },
        "schema": {
            "type": "string"
        },
        "order": 1,
        "mode": "Derived from data"
    },
    "measurement_date": {
        "name": {
            "ja": "測定日時",
            "en": "measurement date"
        },
        "schema": {
            "type": "string",
            "format": "date-time"
        },
        "order": 2,
        "mode": "Derived from data"
    },
    "x_label": {
        "name": {
            "ja": "独立変数(X軸ラベル)",
            "en": "x-label"
        },
        "schema": {
            "type": "string"
        },
        "order": 3,
        "mode": "Derived from data"
    },
    "y_label": {
        "name": {
            "ja": "従属変数(Y軸ラベル)",
            "en": "y-label"
        },
        "schema": {
            "type": "string"
        },
        "order": 4,
        "mode": "Derived from data"
    },
    "series_number": {
        "name": {
            "ja": "系列数",
            "en": "series number"
        },
        "schema": {
            "type": "integer"
        },
        "order": 5,
        "unit": "PCS",
        "mode": "Derived from data"
    },
    "series_name": {
        "name": {
            "ja": "系列名",
            "en": "series name"
        },
        "schema": {
            "type": "string"
        },
        "order": 6,
        "mode": "Analysis value",
        "variable": 1
    },
    "series_data_count": {
        "name": {
            "ja": "系列ごとデータ数",
            "en": "data count by series"
        },
        "schema": {
            "type": "integer"
        },
        "order": 7,
        "unit": "PCS",
        "mode": "Analysis value",
        "variable": 1
    },
    "series_data_mean": {
        "name": {
            "ja": "系列ごと平均値",
            "en": "data mean by series"
        },
        "schema": {
            "type": "number"
        },
        "order": 8,
        "mode": "Analysis value",
        "variable": 1
    },
    "series_data_median": {
        "name": {
            "ja": "系列ごと中央値",
            "en": "data median by series"
        },
        "schema": {
            "type": "number"
        },
        "order": 9,
        "mode": "Analysis value",
        "variable": 1
    },
    "series_data_max": {
        "name": {
            "ja": "系列ごと最大値",
            "en": "data max by series"
        },
        "schema": {
            "type": "number"
        },
        "order": 10,
        "mode": "Analysis value",
        "variable": 1
    },
    "series_data_min": {
        "name": {
            "ja": "系列ごと最小値",
            "en": "data min by series"
        },
        "schema": {
            "type": "number"
        },
        "order": 11,
        "mode": "Analysis value",
        "variable": 1
    },
    "series_data_stdev": {
        "name": {
            "ja": "系列ごと標準偏差",
            "en": "data stdev by series"
        },
        "schema": {
            "type": "number"
        },
        "order": 12,
        "mode": "Analysis value",
        "variable": 1
    },
    "measurement date": {
        "name": {
            "ja": "送状測定日時",
            "en": "measurement date from invoice"
        },
        "schema": {
            "type": "string",
            "format": "date-time"
        },
        "order": 13,
        "mode": "Invoice"
    },
    "invoice_number1": {
        "name": {
            "ja": "送状数値入力値1",
            "en": "invoice_number1"
        },
        "schema": {
            "type": "number"
        },
        "order": 14,
        "mode": "Invoice"
    },
    "invoice_number2": {
        "name": {
            "ja": "送状数値入力値2",
            "en": "invoice_number2"
        },
        "schema": {
            "type": "number"
        },
        "order": 15,
        "mode": "Invoice"
    },
    "invoice_string1": {
        "name": {
            "ja": "送状文字入力値1",
            "en": "invoice_string1"
        },
        "schema": {
            "type": "string"
        },
        "order": 16,
        "mode": "Invoice"
    },
    "invoice_string2": {
        "name": {
            "ja": "送状文字入力値2",
            "en": "inboice_string2"
        },
        "schema": {
            "type": "string"
        },
        "order": 17,
        "mode": "Invoice"
    },
    "invoice_list1": {
        "name": {
            "ja": "送状選択値1",
            "en": "invoice_list1"
        },
        "schema": {
            "type": "string"
        },
        "order": 18,
        "mode": "Invoice"
    }
}
```
> 実際のところ、上のファイルにはスペルミスが含まれています。本書中で修正しますので、ここでは上記内容のままコピー&ペーストにてファイルを上書き保存してください。


## samples.zipの利用

本書同時に配布されている`samples.zip`に、上で用意したファイルと同じものが用意されています。そちらを展開し、適切なフォルダにコピーすることでも上記と同様のことが可能です。

> samples.zipに含まれる内容は、上記の内容と同じものです。必要に応じて利用してください。
>
> 同時配布がない場合は、本書の取得元等、関係者におたずねください。

unzipコマンドがない場合はインストールします。

```bash
sudo apt update
sudo apt install unzip
```

以下ホームディレクトリに`samples.zip`が存在しているものとして、例を示します。

```bash
cd $HOME
unzip samples.zip
```

これにより`samples/`フォルダ下にファイルが作成されます。

```bash
(tenv) $ unzip samples.zip
Archive:  samples.zip
   creating: samples/
  inflating: samples/metadata-def.json
  inflating: samples/invoice.schema.json
  inflating: samples/sample.data
  inflating: samples/invoice.json
```

所定の場所にコピーします。

```bash
cp samples/sample.data  tutorial/container/data/inputdata/
cp samples/invoice.json  tutorial/container/data/invoice/
cp samples/invoice.schema.json  tutorial/container/data/tasksupport/
cp samples/metadata-def.json  tutorial/container/data/tasksupport/
```

> 同様に、Pythonコードなどが含まれた`handson_codes.zip`も同時配布されます。こちらはこれ以降で作成されるPythonスクリプトが含まれています。こちらも必要に応じて利用してください。

