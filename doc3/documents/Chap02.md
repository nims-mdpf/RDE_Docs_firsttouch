<div class="page"/>

# 環境準備

チュートリアルを実施していくにあたって、以下の環境を準備します。

## Linux環境

手元で利用できるLinux環境を用意します。

Pythonが稼働すればよいので、ディストリビューションに特に制限はありません。

> Pythonを実行するディストリビューションとして、現時点(2025年05月)では`Ubuntu 24.04`の利用を推奨します。以降本書では、Ubuntu 24.04を使った場合の実行例を示します。また、バージョン番号を明記しない`Ubuntu`は、`Ubuntu 24.04`を指すものとします。

また、実マシン上に構築されたOSでもいいですし、VirtualBoxなどを利用したVM(Virtual Machine)でも問題ありません。Windowsを利用している場合は、WSL2の利用を推奨します。

python処理系がインストールされていて、利用可能である必要があります。

```bash
$ python3 --version
Python 3.12.3
```

また後述するように、venvも利用しますので、インストールされていることを確認します。Debianの場合、以下の様に確認できます。

```bash
$ sudo apt list python3-venv 
一覧表示... 完了
python3-venv/noble-updates,now 3.12.3-0ubuntu2 amd64 [インストール済み]
N: 追加バージョンが 1 件あります。表示するには '-a' スイッチを付けてください。
```

> 以下Linux環境のロケールは日本語であることを想定しています。その他の場合(英語など)は表示が異なることがありますので、適宜読み替えるか、または日本語ロケールに変更してください。

以下に変更する場合のコマンド例を示します。

```bash
$ localectl
System Locale: LANG=en_US.UTF-8
    VC Keymap: (unset)         
   X11 Layout: jp
    X11 Model: pc105

$ sudo apt update
$ sudo apt install language-pack-ja

$ sudo localectl set-locale ja_JP.UTF-8

$ localectl
System Locale: LANG=ja_JP.UTF-8
    VC Keymap: (unset)         
   X11 Layout: jp
    X11 Model: pc105
```


インストールされていない場合は、インストールします。

```
sudo apt install python3-venv
```

## Python仮想環境作成

> ここでの"Python仮想環境"は、pipモジュールを他の環境とは隔離して利用することを主目的として利用します。そのため、"Python仮想環境"の利用は必須ではありません。

Python仮想環境として、`tenv`という名前で環境を作成します。

```bash
python3 -m venv tenv
```

> `-m venv`でモジュールの読み込みを、最後の`tenv`で仮想環境の名前を、それぞれ指定しています。後者は任意に命名して構いません。
> すでに仮想環境として`tenv`が存在している場合は、適宜変更してください。本書では、Python仮想環境として`tenv`を利用しているものとして記述します。

## Python仮想環境の有効化

```bash
$ source tenv/bin/activate
(tenv) $
```

上記の例では、"tenv"ディレクトリは、ホームディレクトリに作成されていますので、以下の様にしても同じ結果となります。

```bash
$ source $HOME/tenv/bin/activate
(tenv) $
```

> sourceコマンド実行の結果、プロンプトが変わります(→ 元のプロンプトの前に、Python仮想環境名が括弧付きで表示されます)ので、当該仮想環境が有効になったことが分かります。

python仮想環境を使った場合、`python3`コマンドだけでなく、`python`コマンドも利用可能になります。

```bash
(tenv) $ python3 --version
Python 3.12.3

(tenv) $ python --version
Python 3.12.3
```

> どちらも同じものです。本書では`python3`ではなく、`python`コマンドを用いることにします。

## RDEToolKitのインストール

この時点では、pipだけがインストールされた状態になっているはずです。

```bash
(tenv) $ pip list
Package Version
------- -------
pip     24.0
```

> インストールしたPythonバージョンによっては、"setuptools"が表示されることがあります。特に問題はありませんのでそのまま先に進みます。

pip自身を最新にします。

```bash
pip install pip -U
```

> Proxyの設定やSSLエラー対応などの設定がすでに済んでいるものとします。

RDEToolKitをインストールします。

```bash
pip install rdetoolkit
```

> 依存パッケージも同時にインストールされます。

インストールが正常に終わった場合、pipモジュールは以下の様になります。

```bash
(tenv) $ pip list
Package                   Version
------------------------- -----------------
annotated-types           0.7.0
attrs                     25.3.0
build                     1.2.2.post1
chardet                   5.2.0
charset-normalizer        3.4.2
click                     8.1.8
et_xmlfile                2.0.0
eval_type_backport        0.2.2
jsonschema                4.23.0
jsonschema-specifications 2025.4.1
Markdown                  3.8
numpy                     2.2.5
openpyxl                  3.1.5
packaging                 25.0
pandas                    2.2.3
pip                       25.1.1
polars                    1.29.0
pyarrow                   20.0.0
pydantic                  2.11.4
pydantic_core             2.33.2
pyproject_hooks           1.2.0
python-dateutil           2.9.0.post0
pytz                      2025.2
PyYAML                    6.0.2
rdetoolkit                1.2.0
referencing               0.36.2
rpds-py                   0.24.0
six                       1.17.0
toml                      0.10.2
tomlkit                   0.13.2
tqdm                      4.67.1
types-pytz                2025.2.0.20250326
typing_extensions         4.13.2
typing-inspection         0.4.0
tzdata                    2025.2
```

> 他のバージョンのRDEToolKitの場合は、pipモジュールのバージョンも変わっている可能性があります。RDEToolKitでは、バージョン毎に必要なpipモジュールのバージョンも固定していますので注意してください。
>
> 他のプロダクトとの関係で、特定のpipモジュールバージョンだけ変更する必要がある場合もあります。その場合はそちらに合わせてみて、実行に問題がないかを確認する必要があります。

## ディレクトリ作成

最初に作業用のディレクトリを作成します。

> 開発の次のフェーズでは、Dockerイメージを作りますが、そのDockerイメージにはこのディレクトリ下にあるフォルダ/ファイルだけを格納します。従って、このディレクトリ名は任意です。

ユーザのホームディレクトリに`tutorial`というディレクトリを作成します。

```bash
mkdir $HOME/tutorial
```

`tutorial`ディレクトリに移動します。

```bash
cd $HOME/tutorial
```

## 初期化

RDEToolKitを用いて、RDE用のディレクトリ構成を作成します。

最初に、現在のディレクトリが、ホームディレクトリ(ここでは/home/devel)の`tutorial`であることを確認します。

```bash
(tenv) $ pwd
/home/devel/tutorial
```

想定通りであることが確認できたので、"初期化"処理を実施します。"初期化"処理をすることで、構造化処理に必要な最小限のディレクトリ、ファイルが作成されます。

```bash
(tenv) $ python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /home/devel/tutorial/container/requirements.txt
Created: /home/devel/tutorial/container/Dockerfile
Created: /home/devel/tutorial/container/data/invoice/invoice.json
Created: /home/devel/tutorial/container/data/tasksupport/invoice.schema.json
Created: /home/devel/tutorial/container/data/tasksupport/metadata-def.json
Created: /home/devel/tutorial/templates/tasksupport/invoice.schema.json
Created: /home/devel/tutorial/templates/tasksupport/metadata-def.json
Created: /home/devel/tutorial/input/invoice/invoice.json

Check the folder: /home/devel/tutorial
Done!
```

このバージョンのRDEToolKitでは、以下のディレクトリ/ファイルが作成されます。

```bash
(tenv) $ tree -F
./
├── container/
│   ├── Dockerfile
│   ├── data/
│   │   ├── inputdata/
│   │   ├── invoice/
│   │   │   └── invoice.json
│   │   └── tasksupport/
│   │       ├── invoice.schema.json
│   │       └── metadata-def.json
│   ├── main.py
│   ├── modules/
│   └── requirements.txt
├── input/
│   ├── inputdata/
│   └── invoice/
│       └── invoice.json
└── templates/
    └── tasksupport/
        ├── invoice.schema.json
        └── metadata-def.json

12 directories, 9 files
```

> ダミーデータを使ったファイルが設定され、この時点で、最小限実行可能な状態となっています。後述するようにこれらのファイルを置き換えていく必要があります。

> また、`input/`ディレクトリにあるファイルや`templates/`にあるファイルは、実際の構造化処理においては**利用されません**。構造化処理で利用されるのは、`container/`ディレクトリ下にある`data/`ディレクトリ以下のファイルとなります。

初期化直後の、各ファイルは以下の様になっています。

```bash
(tenv) $ cat container/data/invoice/invoice.json
{
    "datasetId": "3f976089-7b0b-4c66-a035-f48773b018e6",
    "basic": {
        "dateSubmitted": "",
        "dataOwnerId": "7e4792d1a8440bcfa08925d35e9d92b234a963449f03df441234569e",
        "dataName": "toy dataset",
        "instrumentId": null,
        "experimentId": null,
        "description": null
    },
    "custom": {
        "toy_data1": "2023-01-01",
        "toy_data2": 1.0
    },
    "sample": {
        "sampleId": "",
        "names": [
            "<Please enter a sample name>"
        ],
        "composition": null,
        "referenceUrl": null,
        "description": null,
        "generalAttributes": [],
        "specificAttributes": [],
        "ownerId": "1234567e4792d1a8440bcfa08925d35e9d92b234a963449f03df449e"
    }
}(tenv) $
```

```bash
(tenv) $ cat container/data/tasksupport/invoice.schema.json
{
    "version": "https://json-schema.org/draft/2020-12/schema",
    "schema_id": "https://rde.nims.go.jp/rde/dataset-templates/dataset_template_custom_sample/invoice.schema.json",
    "description": "RDEデータセットテンプレートテスト用ファイル",
    "value_type": "object",
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
            "required": [
                "toy_data1"
            ],
            "properties": {
                "toy_data1": {
                    "label": {
                        "ja": "トイデータ１",
                        "en": "toy_data1"
                    },
                    "type": "string"
                },
                "sample2": {
                    "label": {
                        "ja": "トイデータ２",
                        "en": "toy_data2"
                    },
                    "type": "number"
                }
            }
        },
        "sample": {
            "type": "object",
            "label": {
                "ja": "試料情報",
                "en": "Sample Information"
            },
            "properties": {}
        }
    }
}(tenv) $
```

```bash
(tenv) $ cat container/data/tasksupport/metadata-def.json
{}(tenv) $
```

> RDEToolKitのバージョンにより内容が変更になる可能性があることに注意してください。

この時点で、RDETooKitが導入されたことを確認します。

```bash
(tenv) $ cd container
(tenv) $ ls
Dockerfile  data  main.py  modules  requirements.txt
(tenv) $ python -m rdetoolkit version
1.2.0
```

* 上は本書執筆時点でのバージョンとなります。随時アップデートされます。

次に、なにも変更せずに、"構造化処理"を実行してみます。

```bash
(tenv) $ python main.py
100%|████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 134.63it/s]
(tenv) $
```

RDEToolKit v1.1.0より、上記のように、構造化処理プログラム実行時に`プログレスバー(進捗表示)`が表示されるようになりました。

>本書では、以降処理結果を示す際に、この**プログレスバーの提示を省略します**。

RDEToolKitのバージョンによっては、上記実行時に"FileNotFoundError: The schema and path do not exist: metadata.json"といったエラーが表示されることがあります。

その場合は、下記のようにダミーのmetadata.jsonを作成してください。

> 最新のRDEToolKitではこのエラーは発生しないため、この処理は**不要**です。

```bash
(tenv) $ echo -e '{\n  "constant": {},\n  "variable": []\n}' > data/meta/metadata.json
```

`vi`などのエディタを利用して`metadata.json`を作成しても構いません。

```json
{
  "constant": {},
  "variable": []
}
```

実行し、エラー表示が行われないことを確認します。

```bash
(tenv) $ python main.py
```

## 本書でのディレクトリ表記

以下、本書でのディレクトリまたはディレクトリを含むファイルの表記をする場合、init処理で作成された`container/`ディレクトリを起点として表記することにします。

例： data/inputodata/sample.dat (本書の場合$HOME/tutorial/container/data/inputdata/sample.datを意味します。

> `container/`配下以外の場所のファイルについて記述する場合は、フルパスで記述するか、$HOMEに続いて記述するなどして、それと分かるように記述します。

例1： "/etc/passwd" (絶対パスの場合)

例2： $HOME/.bashrc (ユーザのホームディレクトリ直下のファイルの場合)

また、本書では"ディレクトリ"と"フォルダ"を同じ意味で使用します。
