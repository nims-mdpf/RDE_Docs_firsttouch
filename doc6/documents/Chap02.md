<div class="page"/>

# 共通処理

次章以降で説明する各モード共通の処理をまとめます。

## RDEToolKitのインストール

他の文書などを参考に、RDEToolKitをインストールします。

本文書では、Pythonの仮想環境`venv`に対してRDEToolKitが導入されているものとして進めます。

```bash
$ source venv/bin/activate

(venv) $ python -m rdetoolkit version
1.7.1
```

> RDEToolKitは随時アップデートされますので上記とは異なる表記となる場合があります。

現時点(2026年09月)では、`1.7.1`が最新となっています。それより前のバージョンをお使いの場合は、最新バージョンにアップデートすることができます。

```bash
pip install rdetoolkit --upgrade
```

> 新規開発の場合は、RDEToolKitの最新バージョンをお使いください。既存バージョンの修正の場合は、以前と同じバージョンでの利用でも問題ありませんが、RDEToolKitの最新バージョンの導入とそれに合わせたPythonスクリプトの修正を推奨します。

## 初期フォルダ構成作成

本書では、`$HOME/rde-sample/<モード名>` 下に構成すること想定します。

"<モード名>"の部分は、例えばマルチデータタイルモードでは"multidatatile"が、エクセルインボイスモードでは"excelinvoice"のようになります。

| # | モード名 | フォルダ名 | 備考 |
| ---- | ---- | ---- | ---- |
| 1 | マルチデータタイルモード | $HOME/rde-sample/multidatatile | |
| 2 | エクセルインボイスモード | $HOME/rde-sample/excelinvoice | |
| 3 | RDEFormatモード | $HOME/rde-sample/rdeformat | |
| 4 | インボイスモード | $HOME/rde-sample/invoice | 本書ではエクセルインボイスの一部として作成するので、明示的なフォルダは作成しません。 |
| 5 | スマートテーブルインボイスモード | $HOME/rde-sample/smarttableinvoice |

> 上記フォルダ名は便宜的に決めたものです。ローカル開発においては、自由にフォルダ名を決めることができます。

以下、RDEFormatモードの場合の例を示します。

```bash
cd ${HOME}
mkdir -p rde-sample/rdeformat
```

当該フォルダに移動し、RDEToolKitの初期化コマンドを実行します。

```bash
(venv) $ cd rde-sample/rdeformat

(venv) $ python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /home/devel/rde-sample/rdeformat/container/requirements.txt
Created: /home/devel/rde-sample/rdeformat/container/Dockerfile
Created: /home/devel/rde-sample/rdeformat/container/data/invoice/invoice.json
Created: /home/devel/rde-sample/rdeformat/container/data/tasksupport/invoice.schema.json
Created: /home/devel/rde-sample/rdeformat/container/data/tasksupport/metadata-def.json
Created: /home/devel/rde-sample/rdeformat/templates/tasksupport/invoice.schema.json
Created: /home/devel/rde-sample/rdeformat/templates/tasksupport/metadata-def.json
Created: /home/devel/rde-sample/rdeformat/input/invoice/invoice.json

Check the folder: /home/devel/rde-sample/rdeformat
Done!
```

> 上記例は、ユーザ`devel`にて実行した場合の例です。実行ユーザに関しては以下同様とします。

構造化処理プログラムは`container/`フォルダ下に作成しますので、移動します。

```bash
cd container/
```

## モード指定方法およびRDEToolKit設定ファイルの利用方法

前述の様に、マルチデータタイルモードおよびRDEフォーマットモードを使用する場合は、その旨を指定する必要があります。

指定の仕方はいくつかありますが、大きくは以下の2つの方法になります。

1. rdetoolkit.workflows.run()実行時に、Configオブジェクトを指定し、その中で指定する方法
2. 設定ファイル(data/tasksupport/rdeconfig.yml または data/tasksupport/rdeconfig.yaml)に指定する方法

> 上記1.と2.を同時に使用することはできません。1.の方法を使った場合、2.の設定ファイルを設置しても無視されます。

なお、設定できる内容はモードの指定だけではなく、いくつかのRDEToolKitの振る舞いを変更することができます。本章では、モード指定の方法以外の`設定`の指定内容や、その方法についての記述も合わせて行います。

### Config オブジェクトを指定する方法

main.py中で、rdetoolkit.workflows.run()を実行する前に、以下の様に設定します。

```python
import rdetoolkit
from rdetoolkit.config import Config, SystemSettings

config = Config(
    system=SystemSettings(
        extended_mode="rdeformat",
        save_thumbnail_image=True
    ),
)
rdetoolkit.workflows.run(config=config)
```

> main.pyを上記の様に変更しただけでは、実行時に`ERROR: no zipped input files`となります。RDEFormatモードを利用する場合の例については、次章以降で説明します。

```bash
(venv) $ python main.py
:
:
Exception Type: StructuredError
Error: ERROR: no zipped input files
```

このように、Configオブジェクトにはいくつかのプロパティを設定することができます。

`system`プロパティは、RDEToolKitの設定として利用されます。

`multidata_tile`プロパティは、`system`プロパティの`extended_mode`で"MultiDataTile"が指定されている場合に、その設定として利用されます。

`smarttable`プロパティは、スマートテーブルインボイスモードを利用時の設定として利用されます。

`traceback`プロパティは、構造化処理プログラムが何らかの原因で異常終了した際に表示されるスタックトレース表示の変更に利用されます。

その他のプロパティは、開発者が自由に設定、利用することができます。

`system`プロパティで設定できる内容は以下の通りです。

| 項目名 | 意味 | フォーマット | 規定値 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| extended_mode | 拡張モードの使用を宣言 | 文字列 | 指定なし | |
| save_raw | inputdataにあるファイルをrawフォルダにコピーするか? | Boolean | False | |
| save_nonshared_raw | inputdataにあるファイルをnonshared_rawフォルダにコピーするか? | Boolean | True | |
| save_thumbnail_image | main_imageにある画像ファイルをthumbnailとして保存するか? | Boolean | True | |
| magic_variable | マジック変数({$filename})をファイル名に置き換えるか? | Boolean | False | |
| save_invoice_to_structured | invoice.jsonをstructuredフォルダにコピーするか? | Boolean | False | |
| feature_description | 説明欄への自動転記を制御 | Boolean | True | |

> extended_modeで指定できるのは、'rdeformat' または 'MultiDataTile' (大文字小文字を**区別**)です。それ以外を指定した場合は、指定しない場合と同様に解釈されます。
>
> これらの項目は、すべて任意指定です。指定されない場合は、規定値が指定されたものとして実行されます。

`multidata_tile`プロパティで設定できる内容は以下の通りです。

| 項目名 | 意味 | フォーマット | 規定値 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| ignore_errors | エラー発生時に処理を継続するか否か? | Boolean | False | デフォルトでは処理を止める |

`smarttable`プロパティで設定できる内容は以下の通りです。

| 項目名 | 意味 | フォーマット | 規定値 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| save_table_file | スマートテーブルファイルをデータタイルの1つとして登録するか? | Boolean | False | |

`traceback`プロパティは、AI利用時のための設定となっているため、ここでは割愛します。

なお、Configオブジェクトは、何も設定しないと既定値を指定したのと同じ設定となりますので、以下の様にすることも出来ます。

```python
import rdetoolkit

config = rdetoolkit.config.Config()
config.system.extended_mode = "rdeformat"
config.system.save_thumbnail_image = True

rdetoolkit.workflows.run(config=config)
```

> 最初の`config = rdetoolkit.config.Config()`で、すべて既定値がセットされたConfigオブジェクトを生成し、その下の2行で、既定値とは違う値をセットするもののみをセットします。

ただし、この機能を利用する場合でも、`smarttable`プロパティと`traceback`プロパティについては、明示的に設定する必要があります。

だめな例:

```python
config = rdetoolkit.config.Config()
config.smarttable.save_table_file = True
```

正しい設定例:

```python
from rdetoolkit.config import Config, SystemSettings, SmartTableSettings

config = Config()
config.smarttable = SmartTableSettings(save_table_file=True)
```

### 設定ファイルを使用する方法

main.pyを、以下の様に変更します。

```python
import rdetoolkit

from modules import datasets_process


rdetoolkit.workflows.run(
    custom_dataset_function=datasets_process.dataset
)
```

上と同じ内容を、data/tasksupport/rdeconfig.yml または data/tasksupport/rdeconfig.yamlに記述することで、Configを指定します。

data/tasksupport/rdeconfig.yml

```yaml
# コメント
system:
    save_raw: False
    # ↓ 空白行もOK

    save_nonshared_raw: False
```

`modules/datasets_process.py`を以下の様にして確認します。

```python
from pprint import pprint

from rdetoolkit.models.rde2types import RdeDatasetPaths


def dataset(paths: RdeDatasetPaths) -> None:
    config = srcpaths.config
    print(type(config))
    print("----")
    pprint(config)
```

実行してみます。

```bash
(venv) $ python main.py
<class 'rdetoolkit.models.config.Config'>
----
Config(system=SystemSettings(extended_mode=None, save_raw=False, save_nonshared_raw=False, save_thumbnail_image=False, magic_variable=False, save_invoice_to_structured=False, feature_description=True), multidata_tile=MultiDataTileSettings(ignore_errors=False), smarttable=None, traceback=None)
```

> 設定内容が反映されている(→ save_raw=False, save_nonshared_raw=False)ことが確認できます。

### ユーザ指定設定項目

上で示したプロパティ以外のプロパティは、"ユーザ指定設定"となります。上記とかぶらなければ任意のプロパティ名を構造化処理プログラムの中で利用することができます。

例えば、以下の様に設定できます。

data/tasksupport/rdeconfig.yml

```yaml
# コメント
system:
    save_raw: False
    # ↓ 空白行もOK

    save_nonshared_raw: False

user:
    x-label: "X軸"
    y-label: "Y軸"
```

利用する場合は以下の様にします。

```python
from pprint import pprint

from rdetoolkit.models.rde2types import RdeDatasetPaths


def dataset(paths: RdeDatasetPaths) -> None:
    user_config = paths.config.user
    print(type(user_config))
    print("----")
    pprint(user_config)
```

上記の結果は以下の様になります。

```bash
(venv) $ python main.py
<class 'dict'>
----
{'x-label': 'X軸', 'y-label': 'Y軸'}
```

> 取得した設定は、通常のDictとして利用できます。