<div class="page" />

# 開発

前章で作成したDockerイメージを使って構造化処理プログラムの開発していきます。

## ソースフォルダ作成

ソースコードを保存するフォルダを、ローカル環境に用意します。

`app/`フォルダを作成します。

```bash
cd $HOME/rde-docker
mkdir app
```

> プロンプトを確認し、コンテナ内から抜けていることを確認してから上記を実行してください。コンテナ内にいる場合は`exit`コマンドでコンテナから抜けます。
> また、作業ディレクトリ(カレントディレクトリ)が`dev_image/`でないことを確認してから`app/`フォルダを作成してください。

## データフォルダ作成

実際の運用環境では`data/`フォルダは、`app/container/`フォルダの下に用意され、画面で入力された内容や入力ファイルなどが自動的に配置されます。

```bash
app
├── container
│   ├── data (本来の配置位置)
:
```

ここでは開発の都合上`app/`と同じ階層に作ることにします。

```bash
cd $HOME/rde-docker
mkdir data
```

`data/`フォルダ下は以下の様になります。

```bash
app
├── container (この時点では未作成)
│   ├── data (本来の配置位置) (この時点では未作成)
:
data (今回の設置位置)
:
```

## コンテナ起動

最終的には、コンテナ内の"/app"フォルダに開発済みのスクリプトを配置したDockerイメージを作成しますが、この時点のコンテナ内には、前章で作成したダミーのスクリプトファイル(main.py)が配置されています。

それらダミーとして作成したものと、これからコンテナ上で新規に作成するものを区別するために、ここでは`/app2`にプログラムを、`/app2/data`の下にデータを配置することにします。

> Dockerイメージを再作成する際に考慮すればいいので、そのまま"/app"を使ってもいいですし、別の名前のディレクトリを使用しても構いません。

上記ディレクトリを使用する形でコンテナを起動します。

```bash
$ docker run -it --rm -v ${PWD}/app:/app2 -v ${PWD}/data:/app2/data rde-sample:v0.1 /bin/bash
root@cfafcd0f658a:/app#
```

上記例では、`-v`フラグを使って、2つのフォルダ(`app/`と`data/`)を、コンテナ内の別のフォルダに配置しています。

> 前述の様に、`-v ${PWD}/app:/app2`は、ホスト上(→Dockerを実行しているサーバ)の`${PWD}/app`(→上で用意したソースフォルダ)を、コンテナ内の`/app2`フォルダにマウントし、もう1つの`-v ${PWD}/data:/app2/data`は、`${PWD}/app`(→上で用意したデータフォルダ)を、コンテナ内の`/app2/data`フォルダにマウントします。

## フォルダ構成初期化

`/app2/`フォルダの下に、RDEToolKitが想定するフォルダ構成を、**コンテナ内で**"初期化"します。

最初に、`/app2/`フォルダに移動します。

> 別のフォルダを利用する場合は、そのフォルダに移動してください。

```bash
root@cfafcd0f658a:/app# cd /app2
```

続いて、RDEToolKitの`init`サブコマンドを実行し、フォルダ構成を初期化し、必要最低限のファイルを生成します。

```bash
root@cfafcd0f658a:/app2# python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /app2/container/requirements.txt
Created: /app2/container/Dockerfile
Created: /app2/container/data/invoice/invoice.json
Created: /app2/container/data/tasksupport/invoice.schema.json
Created: /app2/container/data/tasksupport/metadata-def.json
Created: /app2/templates/tasksupport/invoice.schema.json
Created: /app2/templates/tasksupport/metadata-def.json
Created: /app2/input/invoice/invoice.json

Check the folder: /app2
Done
```

> 上述の様に、ローカル環境の`${PWD}/data`フォルダは、コンテ内の`/app2/data`にマウントされていますが、上で作成した`data/`フォルダは`/app2/container`の下にあるものです。そのため、`/app2/container/data/`下の内容を、ローカル環境の`${PWD}/data`フォルダに移す必要があります(後述)。

出来たばかりの構造化処理プログラムを実行してみます。

```bash
root@cfafcd0f658a:/app2# cd container/
root@cfafcd0f658a:/app2/container# ls -F
Dockerfile  data/  main.py  modules/  requirements.txt

root@cfafcd0f658a:/app2/container# python main.py 
root@cfafcd0f658a:/app2/container#
```

ここでは、エラー表示なく終了したことを確認します。

フォルダ構成の初期化が完了しましたので、いったんコンテナから抜けます。

```bash
root@cfafcd0f658a:/app2/container# exit
exit
$
```

ローカル環境のフォルダを確認します。

```bash
$ tree app
app
├── container
│   ├── Dockerfile
│   ├── data
│   │   ├── attachment
│   │   ├── inputdata
│   │   ├── invoice
│   │   │   └── invoice.json
│   │   ├── invoice_patch
│   │   ├── logs
│   │   │   └── rdesys.log
│   │   ├── main_image
│   │   ├── meta
│   │   ├── nonshared_raw
│   │   ├── other_image
│   │   ├── raw
│   │   ├── structured
│   │   ├── tasksupport
│   │   │   ├── invoice.schema.json
│   │   │   └── metadata-def.json
│   │   ├── temp
│   │   └── thumbnail
│   ├── main.py
│   ├── modules
│   └── requirements.txt
├── data
├── input
│   ├── inputdata
│   └── invoice
│       └── invoice.json
└── templates
    └── tasksupport
        ├── invoice.schema.json
        └── metadata-def.json

24 directories, 10 files
```

> `python main.py`を実行しないで確認すると、initサブコマンド実行のメッセージに示されたとおり、`data/invoice/invoice.json`フォルダと`data/tasksupport`フォルダの2つだけ存在することになります。他のフォルダが存在しない場合は、上記で示している手順で`python main.py`を実行してください。

## データフォルダの初期化

この時点で、ローカル環境上に用意した`data/`フォルダには何も入っていません。

```
$ tree data
data

0 directories, 0 files
```

上の`init`サブコマンドで初期化された`data/`フォルダの内容を、カレントディレクトリにある`data/`フォルダに移動します。

```bash
$ cd ${HOME}/rde-docker
$ sudo mv app/container/data/* ./data

$ tree data
data
├── attachment
├── inputdata
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys.log
├── main_image
├── meta
├── nonshared_raw
├── other_image
├── raw
├── structured
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
└── thumbnail

15 directories, 4 files
```

> `data/`フォルダの下にフォルダが3つしかない場合は、コンテナ内で`python main.py`が未実施な場合が考えられます。再度コンテナを起動し、その中で(`init`サブコマンドで作成された)`main.py`を実行してください。

次に、ローカル環境での書き換え時に"sudo"コマンドを使わなくてもよいように、`app/`と`data/`のオーナーを"root"から、開発で使用しているユーザ(→本マニュアル上では"devel"ユーザ)に変更しておきます。

```bash
sudo chown -R ${USER} app/
sudo chown -R ${USER} data/
```

カレントディレクトリを確認します。

```bash
$ pwd
/home/devel/rde-docker
```

現在のmain.pyの内容を確認します。

```bash
$ cat app/container/main.py 
# The following script is a template for the source code.

import rdetoolkit

rdetoolkit.workflows.run()
```

> RDEToolKitのinit処理にて生成された内容のままです。

この状態で、`-v`オプションの設定を本番実行用のDockerイメージを想定したものに変更して、再度コンテナを実行します。

```bash
docker run -it --rm -v ${PWD}/app/container:/app2 -v ${PWD}/data:/app2/data rde-sample:v0.1 /bin/bash
```

> 実際の本番環境ではスクリプトおよびデータは、`/app2/`フォルダではなく、`/app/`フォルダに配置されます。開発途中であることを明確に示す意味で`/app2`フォルダを使っています。

* 1個目のマウント元が`${PWD}/app`から`${PWD}/app/container`に変更になったことに注意してください。
* 2個目のマウント設定により、コンテナ内での`main.py`実行で作成されたファイルは、マウント元の${HOME}/rde-docker/dataで確認することができるようになります。

フォルダの構成状況を確認すると以下の様になります。

```bash
root@7d92e5033f5e:/app# cd /app2
root@7d92e5033f5e:/app2# tree
.
├── Dockerfile
├── data
│   ├── attachment
│   ├── inputdata
│   ├── invoice
│   │   └── invoice.json
│   ├── invoice_patch
│   ├── logs
│   │   └── rdesys.log
│   ├── main_image
│   ├── meta
│   ├── nonshared_raw
│   ├── other_image
│   ├── raw
│   ├── structured
│   ├── tasksupport
│   │   ├── invoice.schema.json
│   │   └── metadata-def.json
│   ├── temp
│   └── thumbnail
├── main.py
├── modules
└── requirements.txt

17 directories, 7 files
```

念のため、エラーなく実行できることを確認します。

```bash
root@7d92e5033f5e:/app2# python main.py 
root@7d92e5033f5e:/app2#
```

コンテナ内から抜けます。

```bash
root@7d92e5033f5e:/app2# exit
exit
```

## データ配置

開発で実際に使うデータを用意します。

本来の開発であれば、ここで実際に利用されるデータなどを配置しますが、今回は**ダミーデータを用意する**ことにします。

> ハンズオンチュートリアルで使用したものを使います。すでに同名のファイルがある場合は上書きしてください。
>
> コンテナ内で作成しても、コンテナ外で作成しても構いません。コンテナ内で作成する場合は、`data/inputdata/`部分を`/app2/data/inputdata/`と読み替えて作成してください。

### 入力データファイル

以下の内容で`data/inputdata/sample.data`を作成します。

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

### 送り状ファイル

以下の内容で`data/invoice/invoice.json`を作成します。

> 既にあるファイルを一度削除の上、以下の内容で作成してください。

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
        ],
        "composition": null,
        "referenceUrl": null,
        "description": null,
        "generalAttributes": [
            {
                "termId": "3adf9874-7bcb-e5f8-99cb-3d6fd9d7b55e",
                "value": null
            },
            {
                "termId": "0aadfff2-37de-411f-883a-38b62b2abbce",
                "value": null
            },
            {
                "termId": "0444cf53-db47-b208-7b5f-54429291a140",
                "value": null
            },
            {
                "termId": "e2d20d02-2e38-2cd3-b1b3-66fdb8a11057",
                "value": null
            }
        ],
        "specificAttributes": [
            {
                "classId": "52148afb-6759-23e8-c8b8-33912ec5bfcf",
                "termId": "70c2c751-5404-19b7-4a5e-981e6cebbb15",
                "value": null
            },
            {
                "classId": "961c9637-9b83-0e9d-e60e-ffc1e2517afd",
                "termId": "70c2c751-5404-19b7-4a5e-981e6cebbb15",
                "value": null
            },
            {
                "classId": "01cb3c01-37a4-5a43-d8ca-f523ca99a75b",
                "termId": "dc27a956-263e-f920-e574-5beec912a247",
                "value": null
            }
        ],
        "ownerId": "119cae3d3612df5f6cf7085fb8deaa2d1b85ce963536323462353734"
    }
}
```

### tasksupportフォルダに置くファイル

ここでは、`data/tasksupport/invoice.schema.json`と`data/tasksupport/metadata-def.json`を用意します。

> 既にあるファイル一度削除の上、以下の内容で作成してください。

以下の内容で`data/tasksupport/invoice.schema.json`を作成します。

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

以下の内容で`data/tasksupport/metadata-def.json`を作成します。

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
        "mode": "Derived from data",
        "original_name": "data_title"
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
        "mode": "Derived from data",
        "original_name": "measurement_date"
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
        "mode": "Derived from data",
        "original_name": "x_label"
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
        "mode": "Derived from data",
        "original_name": "y_label"
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
        "mode": "Derived from data",
        "original_name": "series_number"
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
        "variable": 1,
        "original_name": "series_name"
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
        "variable": 1,
        "original_name": "series_data_count"
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
        "variable": 1,
        "original_name": "series_data_mean"
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
        "variable": 1,
        "original_name": "series_data_median"
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
        "variable": 1,
        "original_name": "series_data_max"
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
        "variable": 1,
        "original_name": "series_data_min"
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
        "variable": 1,
        "original_name": "series_data_stdev"
    },
    "measurement date": {
        "name": {
            "ja": "送状状測定日時",
            "en": "measurement date from invoice"
        },
        "schema": {
            "type": "string",
            "format": "date-time"
        },
        "order": 13,
        "mode": "Invoice",
        "original_name": "measurement date"
    },
    "invoice_number1": {
        "name": {
            "ja": "送状状数値入力値1",
            "en": "invoice_number1"
        },
        "schema": {
            "type": "number"
        },
        "order": 14,
        "mode": "Invoice",
        "original_name": "invoice_number1"
    },
    "invoice_number2": {
        "name": {
            "ja": "送状状数値入力値2",
            "en": "invoice_number2"
        },
        "schema": {
            "type": "number"
        },
        "order": 15,
        "mode": "Invoice",
        "original_name": "invoice_number2"
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
        "mode": "Invoice",
        "original_name": "invoice_string1"
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
        "mode": "Invoice",
        "original_name": "inboice_string2"
    },
    "invoice_list1": {
        "name": {
            "ja": "送状状選択値1",
            "en": "invoice_list1"
        },
        "schema": {
            "type": "string"
        },
        "order": 18,
        "mode": "Invoice",
        "original_name": "invoice_list1"
    }
}
```

## コンテナ起動

コンテナを起動します。

```bash
$ cd ${HOME}/rde-docker/
$ docker run -it --rm -v ${PWD}/app/container:/app2 -v ${PWD}/data:/app2/data rde-sample:v0.1 /bin/bash
root@b103229368dc:/app#
```

`/app2/`フォルダに移動します。

```bash
cd /app2
```

## スクリプト開発

本来の開発であれば、main.pyおよびmodules/フォルダ下のスクリプトを作成していくことになりますが、本章ではデータ同様に**ダミースクリプトを用意する**ことにします。

> ここでは、別途提供されている『ハンズオンチュートリアル』で使用したものを使います。この構成が必ずしも推奨のものではないことに留意ください。

```bash
root@b103229368dc:/app2# ls -F
Dockerfile  data/  main.py  modules/  requirements.txt
```

### main.py

```python
import rdetoolkit
from rdetoolkit.config import Config, SystemSettings

from modules.datasets_process import dataset


def main():
    config = Config(
         system=SystemSettings(
             save_raw=False,
             save_nonshared_raw=False
         )
    )
    rdetoolkit.workflows.run(
        custom_dataset_function=dataset,
        config=config
    )

if __name__ == '__main__':
    main()
```

> すでにある`main.py`を、上記内容にて上書きします。
> inputdata/フォルダにあるファイルを、自動的にraw/フォルダおよびnonshared_raw/フォルダにコピーする機能を**抑止する**ような設定を追加しています。

### modules/datasets_process.py

`/app2/modules/`フォルダ配下に、実行したい処理が記述されたファイルを配置していきます。

最初に`main.py`の`custom_dataset_function`に指定した`datasets_process.py`を、以下の内容で作成します。

```python
import io
import shutil

import pandas as pd
import statistics as st
import matplotlib.pyplot as plt
from PIL import Image
from rdetoolkit.rde2util import Meta
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.exceptions import StructuredError
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
from rdetoolkit.invoicefile import InvoiceFile


def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    # Check input file
    input_files = resource_paths.rawfiles
    if len(input_files) == 0:
        raise StructuredError("ERROR: input data not found")

    if len(input_files) > 1:
        raise StructuredError("ERROR: input data should be one file")

    raw_file_path = input_files[0]
    if raw_file_path.suffix.lower() != ".data":
        raise StructuredError(
            f"ERROR: input file is not '*.data' : {raw_file_path.name}"
        )

    # Read invoice file
    invoice_file = resource_paths.invoice  / "invoice.json"
    invoice = InvoiceFile(invoice_file)

    # Check public or private
    is_private_raw = False if invoice["custom"]["is_private_raw"] == "share" else True

    # Backup(=Copy) invoice.json to shared/nonshared folder
    raw_dir = resource_paths.nonshared_raw if is_private_raw else resource_paths.raw
    invoice_file_backup = raw_dir / "invoice.json.orig"
    InvoiceFile.copy_original_invoice(invoice_file, invoice_file_backup)

    # Rewrite invoice
    original_data_name = invoice.invoice_obj["basic"]["dataName"]
    additional_title = "(2024)"
    if original_data_name.find(additional_title) < 0:
        # update title if not applied yet
        invoice.invoice_obj["basic"]["dataName"] = original_data_name + " / " + additional_title
        # overwrite original invoice
        invoice_file_new = resource_paths.invoice  / "invoice.json"
        invoice.overwrite(invoice_file_new)

    # Read input data
    DELIM = "="
    raw_data_df = None
    raw_meta_obj = None
    #
    input_file = resource_paths.rawfiles[0] # read one file only

    with open(input_file) as f:
        lines = f.readlines()
    # omit new line codes (\r and \n)
    lines_strip = [line.strip() for line in lines]

    meta_row = [i for i, line in enumerate(lines_strip) if "[METADATA]" in line]
    data_row = [i for i, line in enumerate(lines_strip) if "[DATA]" in line]
    if (meta_row != []) & (data_row !=[]):
        meta = lines_strip[(meta_row[0]+1):data_row[0]]
        # metadata to dict
        raw_meta_obj = dict(map(lambda x: tuple([x.split(DELIM)[0],
                    DELIM.join(x.split(DELIM)[1:])]), meta))
    else:
        raise StructuredError("ERROR: invalid RAW METADATA or DATA")
    # read data to data.frame
    if (int(raw_meta_obj["series_number"])!= len(data_row)):
        raise StructuredError("ERROR: unmatch series number")
    raw_data_df = []
    for i in data_row:
        series_name = lines_strip[i+1]
        cnt = int(lines_strip[i+2])
        csv = "".join(lines[(i+3):(i+3+cnt)])
        temp_df = pd.read_csv(io.StringIO(csv),header=None)
        temp_df.columns = ["x", series_name]
        raw_data_df.append(temp_df)

    metadata_def_file = srcpaths.tasksupport / "metadata-def.json"
    meta_obj = Meta(metadata_def_file)

    # get meta data
    # #1 Read Input File
    # -> input fileは前述の処理にて作成済み

    # #2 Read Inovoice
    # -> invoice.invoice_obj["custom"] は前述の処理にて作成(→読み込み)済み

    # #3 From raw data numeric data part
    s_name = []
    s_count = []
    s_mean = []
    s_median = []
    s_max = []
    s_min = []
    s_stdev = []
    for df in raw_data_df:
        d = df.dropna(axis=0)
        y = d.iloc[:, 1]
        s_name.append(d.columns[1])
        s_count.append(len(y))
        s_mean.append("{:.2f}".format(st.mean(y)))
        s_median.append("{:.2f}".format(st.median(y)))
        s_max.append(max(y))
        s_min.append(min(y))
        s_stdev.append("{:.2f}".format(st.stdev(y)))                                                                             
    meta_vars = {
        "series_name": s_name,
        "series_data_count": s_count,
        "series_data_mean": s_mean,
        "series_data_median": s_median,
        "series_data_max": s_max,
        "series_data_min": s_min,
        "series_data_stdev": s_stdev,
    }

    # Merge 2 types of metadata
    const_meta_info = raw_meta_obj | invoice.invoice_obj["custom"]
    repeated_meta_info = meta_vars

    # Set metadata to meta instance
    meta_obj.assign_vals(const_meta_info)
    meta_obj.assign_vals(repeated_meta_info)

    # Write metadata
    metadata_json = resource_paths.meta / "metadata.json"
    meta_obj.writefile(metadata_json)

    # Write CSV file(s)
    for d in raw_data_df:
        fname = d.columns[1].replace(" ", "") + ".csv"
        csv_file_path = resource_paths.struct / fname
        d.to_csv(csv_file_path, header=True, index=False)

    # Write Graph
    title = const_meta_info["data_title"]
    x_label = const_meta_info["x_label"]
    y_label = const_meta_info["y_label"]

    ## by series
    for d in raw_data_df:
        x = d.iloc[:, 0]
        y = d.iloc[:, 1]
        label = d.columns[1]
        fname = resource_paths.other_image / f"{label}.png"
        fig, ax = plt.subplots(figsize=(5, 5), facecolor="white")
        ax.plot(x, y, label=label)
        ax.set_title(title)
        ax.set_xlabel(x_label)
        ax.set_ylabel(y_label)
        ax.legend()
        fig.savefig(fname)
        plt.close(fig)

    ## all series
    fig, ax = plt.subplots(figsize=(5, 5), facecolor="lightblue")
    ax.set_xlabel(x_label)
    ax.set_ylabel(y_label)
    ax.set_title(title)
    for d in raw_data_df:
        x = d.iloc[:, 0]
        y = d.iloc[:, 1]
        label = d.columns[1]
        ax.plot(x, y, label=label)
    ax.legend()
    fname = resource_paths.main_image / "all_series.png"
    fig.savefig(fname)
    plt.close(fig)

    # Write thumbnail images
    src_img_file_path = resource_paths.main_image / "all_series.png"
    out_img_file_path = resource_paths.thumbnail / "thumbnail.png"
    closure_w = 286
    closure_h = 200

    Image.MAX_IMAGE_PIXELS = None   # オープンする画像のピクセルサイズ制限を解除する
    img_org = Image.open(src_img_file_path)
    ratio_w = closure_w / img_org.width
    ratio_h = closure_h / img_org.height
    ratio = min(ratio_w, ratio_h)
    img_re = img_org.resize((int(img_org.width * ratio), int(img_org.height * ratio)), Image.BILINEAR)  # image magickのデフォルトに合わせてバイリニアとしている
    img_re.save(out_img_file_path)

    # Copy inputdata to public (raw/) or non_public (nonshared_raw/)

    # raw_dir = resource_paths.nonshared_raw if is_private_raw else resource_paths.raw
    # input_file = resource_paths.rawfiles[0] # read one file only
    shutil.copy(input_file, raw_dir)

@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    custom_module(srcpaths, resource_paths)
```

## 実行

構造化処理プログラムを動かしてみます。

```bash
root@633a74823e8b:/app2# python main.py 
Traceback (most recent call last):
  File "/app2/main.py", line 4, in <module>
    from modules.datasets_process import dataset
  File "/app2/modules/datasets_process.py", line 6, in <module>
    import matplotlib.pyplot as plt
ModuleNotFoundError: No module named 'matplotlib'
```

上記スクリプト実行のために必要なモジュールがインストールされていません。

Dockerイメージを作り直します。

`exit`コマンドを使いDockerコンテナから抜け、requirements.txtを以下の様に変更します。

```bash
$ vi dev_image/requirements.txt

$ cat dev_image/requirements.txt 
rdetoolkit==1.3.4
matplotlib==3.10.7
```

> `matplotlib==3.10.7`の行を追記します。matplotlibのバージョンが不明の場合は、"==3.10.7"部分を付けずにbuildおよび実行し、`pip list`コマンドにて当該モジュールのバージョンを確認後、再度requirements.txtを修正してください。

Dockerイメージを再作成します。

```bash
cd dev_image
docker build -t rde-sample:v0.1  ./
```

> 同じ名前(とバージョン)で作っていますので、上書きになります。

再度実行(docker run)します。

```bash
$ cd ${HOME}/rde-docker
$ docker run -it --rm -v ${PWD}/app/container:/app2 -v ${PWD}/data:/app2/data rde-sample:v0.1 /bin/bash

root@38dc486bc3a5:/app# cd /app2
root@38dc486bc3a5:/app2# python main.py 
root@38dc486bc3a5:/app2#
```

正常に終了しました。出力されるファイルを確認すると以下の様になっています。

```bash
root@38dc486bc3a5:/app2# tree data
data
├── attachment
├── inputdata
│   └── sample.data
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys.log
├── main_image
│   └── all_series.png
├── meta
│   └── metadata.json
├── nonshared_raw
│   ├── invoice.json.orig
│   └── sample.data
├── other_image
│   ├── series1.png
│   ├── series2.png
│   └── series3.png
├── raw
├── structured
│   ├── series1.csv
│   ├── series2.csv
│   └── series3.csv
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
└── thumbnail
    └── thumbnail.png

15 directories, 16 files
```

上記の例の場合、ファイル構成から、問題無く想定のファイルが出来ていることが確認できます。実際の開発では、ファイルの内容も含めて確認する必要があります。

なんらかの異常がある場合は、`data/job.faild`ファイルの内容や端末に表示されるエラー表示を確認して修正します。

## 2回目以降の実行

PythonスクリプトやJSONファイル、YAMLファイルその他のファイルを修正して、再度実行する場合は、以下のような点に注意してください。

### job.failedファイルの削除

前回処理の結果と混同するなどの混乱を避けるため、`data/job.failed`が存在している場合は削除します。

```bash
root@38dc486bc3a5:/app# cd /app2
root@38dc486bc3a5:/app2# rm -f data/job.failed
```

> 前回の処理が正常に終了した場合は、`data/job.failed`ファイルは生成されません。削除するのは、当該ファイルが存在する場合のみです。

### 生成ファイルの削除

構造化処理プログラムの実行により生成したファイルを削除します。

> 削除せずに実行した場合、存在しているファイルが最新の実行によって生成されたのか、あるいは前回の実行によって生成されたのかが、すぐには判別できない場合があります。そういった事態を避けるために、実行の前に、前回の実行で生成したファイルの削除処理を行うことをお勧めします。

### ログファイルのクリア

必要に応じてログをクリア(→ 0バイトのファイルへの置き換え)や削除を実行してください。

> ログファイルが存在していなければ、次の実行時に新規に作成されますので、クリアも削除もどちらも同じ意味になります。
>
> ログは基本的に追記型なので、ログファイルのクリアや削除を実行しなくても特に問題はありません。ログが大きくなりすぎて見づらくなったなど、問題がある場合にクリア(または削除)してください。

RDEToolKitを使った構造化処理プログラムは、`data/logs/rdesys.log`にログを出力します。

また作成したスクリプトによっては、別のファイルにログを出力する(ユーザログファイル)こともあります。

以下にログファイルを削除する場合の実行例を示します。

```bash
root@38dc486bc3a5:/app2# rm -f data/logs/*.log
```

> ユーザログファイルも、`data/logs`フォルダに出力するように作成します。

### 実行 (あるいはデバッグ)

初回実行と同様に、構造化処理プログラムを実行します。

何らかの異常がある場合は、`main.py`や`modules/`フォルダ下のスクリプトファイルを修正します。

以後、望む状態になるまで、修正と実行を繰り返します。

## (参考)スクリプトファイルやその他生成ファイルのオーナー

本書の手順通りに作成したDockerコンテナ内で作成したファイル、あるいは構造化処理プログラムの実行により生成されたファイルのオーナーは`root`ユーザです。

> 前章で、一部ファイルのオーナーの変更処理を実行しています。そのため、すべてのファイルのオーナーが`root`ユーザではありません。

**Dockerコンテナ外**からファイルを編集したい場合など、`Permission denied`により編集出来ない場合があります。

その場合は、`sudo`コマンドを利用するか、ファイル/フォルダのオーナーを変更する必要があります。

`sudo`コマンドを利用する例：

```bash
cd app/container
sudo vi modules/datasets_process.py
```

オーナーを変更する場合の例：

```bash
$ sudo chown -R devel modules  (ユーザ"devel"を利用している場合)
$ vi modules/datasets_process.py
```

> ファイルを削除する場合も、上と同様に、`sudo`コマンドを利用するか、オーナーを変更してから削除処理を実行する必要があります。

