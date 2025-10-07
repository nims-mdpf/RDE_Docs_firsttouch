<div class="page"/>

# 環境準備

ローカル開発環境として求めれるのは、以下のようなものです。

| 項目 | 内容 | 備考 |
| ---- | ---- | ---- |
| 開発マシン | 実機、仮想マシンいずれも可 | | 
| 開発OS | Linux | ディストリビューションは問わない |
| 開発言語 | Python v3.10以上 | 仮想環境(venv)が利用できること |
| RDEToolKit | v1.3.4 | 開発時点の最新版の利用を推奨 |
| Docker | Docker Engine、 Docker CLI | |
| 開発補助ツール | テンプレート生成ツール、構造化処理結果プレビューツール | NIMS提供のツール。 テンプレート生成ツールの利用にはMS Excelが必要 |

なおRDE実行環境では、主に`4vcpu、16GBメモリ、総ディスク容量10GB`の環境を利用して構造化処理プログラムが実行されますので、それを参考に実機、仮想マシンを用意してください。

> 上記を超えるような性能を必要とする場合は、開発前にご相談ください。

またRDEでは、内部でDocker機能を使用していますが、登録するコンテナはNIMSにてbuildするため、ローカル開発ではDocker機能の準備は必須ではありません。ただし、開発者は、Dockerイメージを作成するために使用する`Dockerfile`の作成までを求められますので、そのためにDocker Engine、Docker CLIの準備が必要となります。本書ではDockerを使った開発に関しては扱いません。別途提供している『RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書』を参照ください。

> 『RDEToolKitを用いたRDE構造化処理用Dockerコンテナ作成手順書』は以下のURLにて公開されています。
>
> https://github.com/nims-mdpf/RDE_Docs_firsttouch

チュートリアルを実施していくにあたって、以下の環境を準備します。

| 項目 | 内容 | 備考 |
| ---- | ---- | ---- |
| 開発OS | Ubuntu 24.04 | |
| 開発言語 | Python v3.12 | 仮想環境(venv)を含む |
| パッケージ | RDEToolKit v1.3.4 | RDEToolKitの依存パッケージを含む |

## Linux環境

ローカル、つまり手元で利用できるLinux環境を用意します。

Pythonが稼働すればよいので、ディストリビューションに特に制限はありません。

> Pythonを実行するディストリビューションとして、現時点(2025年09月)では`Ubuntu 24.04`の利用を推奨します。以降本書では、Ubuntu 24.04を使った場合の実行例を示します。また、バージョン番号を明記しない`Ubuntu`は、`Ubuntu 24.04`を指すものとします。

また、実マシン上に構築されたOSでもいいですし、VirtualBoxなどを利用したVM(Virtual Machine)でも問題ありません。Windowsを利用している場合は、WSL2の利用を推奨します。

> 以下本書では、Linux環境のロケールは日本語であることを想定しています。英語環境などその他の場合は表示が異なりますので、適宜読み替えるか、または日本語ロケールに変更してください。

以下に、日本語ロケールに変更する場合のコマンド例を示します。

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

> 日本語パックを導入し、デフォルトロケールを変更します。

## Python環境

python処理系がインストールされていて、利用可能である必要があります。

```bash
$ python3 --version
Python 3.12.3
```

また仮想環境も利用しますので、インストールされていることを確認します。Ubuntuの場合、以下の様に確認できます。

```bash
$ sudo apt list python3-venv 
一覧表示... 完了
python3-venv/noble-updates,now 3.12.3-0ubuntu2 amd64 [インストール済み]
N: 追加バージョンが 1 件あります。表示するには '-a' スイッチを付けてください。
```

インストールされていない場合は、インストールします。

```
sudo apt install python3-venv
```

## Python仮想環境作成

> ここでの"Python仮想環境"は、pipモジュールを他の環境とは隔離して利用することを主目的として利用します。そのため、"Python仮想環境"の利用は必須ではありません。

Python仮想環境として、`tenv`という名前で環境を作成します。

```bash
cd $HOME
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

> 上記コマンドにて、必要な依存パッケージも同時にインストールされます。

インストールが正常に終わった場合、pipモジュールは以下の様になります。

```bash
Package                   Version
------------------------- -----------------
annotated-types           0.7.0
attrs                     25.3.0
build                     1.3.0
chardet                   5.2.0
charset-normalizer        3.4.3
click                     8.2.1
et_xmlfile                2.0.0
eval_type_backport        0.2.2
jsonschema                4.25.1
jsonschema-specifications 2025.4.1
Markdown                  3.8.2
numpy                     2.3.2
openpyxl                  3.1.5
packaging                 25.0
pandas                    2.3.2
pip                       25.2
polars                    1.32.3
pyarrow                   21.0.0
pydantic                  2.11.7
pydantic_core             2.33.2
pyproject_hooks           1.2.0
python-dateutil           2.9.0.post0
pytz                      2025.2
PyYAML                    6.0.2
rdetoolkit                1.3.4
referencing               0.36.2
rpds-py                   0.27.0
six                       1.17.0
toml                      0.10.2
tomlkit                   0.13.3
types-pytz                2025.2.0.20250809
typing_extensions         4.15.0
typing-inspection         0.4.1
tzdata                    2025.2
```

> 他のバージョンのRDEToolKitの場合は、依存パッケージのバージョンも変わっている可能性があります。RDEToolKitでは、バージョン毎に必要なpipモジュールのバージョンも固定していますので注意してください。また、インストール時期により依存パッケージバージョンが変わるものもありますので、同じバージョンのRDEToolKitを利用していた場合でも、必ずしも上記と同じバージョンになるとは限りません。
>
> 他のプロダクトとの関係で、特定のpipモジュールバージョンだけ変更する必要がある場合もあり得ます。その場合はそちらに合わせてみて、構造化処理プログラム実行に問題がないかを確認する必要があります。

## ディレクトリ作成

次に`作業用のディレクトリ`を作成します。

> 開発の後半フェーズでは、Dockerイメージを作りますが、そのDockerイメージにはこのディレクトリ下にあるフォルダ/ファイルだけを格納します。従って、このディレクトリ名は任意です。

ユーザのホームディレクトリに`handson`というディレクトリを作成し、さらにその下に`tutorial`というディレクトリを作成します。

```bash
mkdir $HOME/handson
mkdir $HOME/handson/tutorial
```

`tutorial`ディレクトリに移動します。

```bash
cd $HOME/handson/tutorial
```

## 初期化

RDEToolKitを用いて、RDE用のディレクトリ構成を作成します。

最初に、現在のディレクトリが、ホームディレクトリ(ここでは/home/devel)の下の`handson/tutorial`であることを確認します。

```bash
(tenv) $ pwd
/home/devel/handson/tutorial
```

想定通りであることが確認できたので、"初期化"処理を実施します。"初期化"処理をすることで、構造化処理に必要な最小限のディレクトリ、ファイルが作成されます。

```bash
(tenv) $ python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /home/devel/handson/tutorial/container/requirements.txt
Created: /home/devel/handson/tutorial/container/Dockerfile
Created: /home/devel/handson/tutorial/container/data/invoice/invoice.json
Created: /home/devel/handson/tutorial/container/data/tasksupport/invoice.schema.json
Created: /home/devel/handson/tutorial/container/data/tasksupport/metadata-def.json
Created: /home/devel/handson/tutorial/templates/tasksupport/invoice.schema.json
Created: /home/devel/handson/tutorial/templates/tasksupport/metadata-def.json
Created: /home/devel/handson/tutorial/input/invoice/invoice.json

Check the folder: /home/devel/handson/tutorial
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
    "$schema": "https://json-schema.org/draft/2020-12/schema",
    "$id": "https://rde.nims.go.jp/rde/dataset-templates/dataset_template_custom_sample/invoice.schema.json",
    "description": "RDEデータセットテンプレートテスト用ファイル",
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

> RDEToolKit v1.3.2より前のバージョンでは、ローカル開発時は問題ないものの、そのままRDEシステム上で利用すると問題が発生するものが生成されていました。最初の4行が"\$schema"、"\$id"、"description" および "type"となっていることを確認してください。

```bash
(tenv) $ cat container/data/tasksupport/metadata-def.json
{}(tenv) $
```

> RDEToolKitのバージョンにより内容が変更になる可能性があることに注意してください。

この時点で、RDEToolKitが導入されたことを確認します。

```bash
(tenv) $ cd container
(tenv) $ ls
Dockerfile  data  main.py  modules  requirements.txt
(tenv) $ python -m rdetoolkit version
1.3.4
```

* 上は本書執筆時点でのバージョンとなります。随時アップデートされます。

次に、なにも変更せずに、"構造化処理"を実行してみます。

```bash
(tenv) $ python main.py
(tenv) $
```

RDEToolKitの過去のバージョンでは、構造化処理プログラム実行時に`プログレスバー(進捗表示)`が表示されるものがありましたが、現在はプログレスバーは表示されません。

また、RDEToolKitのバージョンによっては、上記実行時に"FileNotFoundError: The schema and path do not exist: metadata.json"といったエラーが表示されることがあります。

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

実行し、エラーが表示されないことを確認します。

```bash
(tenv) $ python main.py
(tenv) $
```

## 禁止事項

以下のようなものは、RDE環境で利用することが出来ませんので、ローカル開発でも使用しないようにしてください。

* 公開利用が禁止されているもの
* 有償の開発環境、パッケージ、コマンド

上記の例として、以下のようなものがあります。

* 事例1 装置メーカー提供のデータ変換ツール(配布が禁止されているもの)
* 事例2 Anaconda パブリック リポジトリからパッケージのインストール

> 本制限は、`RDE上で実行される構造化処理プログラム`についての禁止事項となります。RDEにデータを登録する`前処理`として装置メーカー提供のデータ変換ツールやAnacondaパブリックリポジトリのパッケージを利用することは問題ありません(使用許諾やライセンス的に問題がない場合)。


## 本書でのディレクトリ表記

以下、本書でのディレクトリまたはディレクトリを含むファイルの表記をする場合、init処理で作成された`container/`ディレクトリを起点として表記することにします。

例： data/inputdata/sample.dat (本書の場合$HOME/handson/tutorial/container/data/inputdata/sample.datを意味します。

> `container/`配下以外の場所のファイルについて記述する場合は、フルパスで記述するか、$HOMEに続いて記述するなどして、それと分かるように記述します。

例1： "/etc/passwd" (絶対パスの場合)

例2： $HOME/.bashrc (ユーザのホームディレクトリ直下のファイルの場合)

また、本書では"ディレクトリ"と"フォルダ"を同じ意味で使用します。

## OS提供コマンドのインストールについて

本書では、`tree`や`vi`など多くのLinuxディストリビューションにて標準提供されているコマンドを利用しています。それらのコマンドはインストール直後には利用できない場合があります。

必要に応じて、インストールして利用してください。

Ubuntuにて、treeコマンドをインストールする場合の例:

```bash
sudo apt install tree
```

