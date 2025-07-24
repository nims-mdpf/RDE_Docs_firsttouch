<div class="page"/>

# 共通処理

次章以降で説明する各モード共通の処理をまとめます。

## RDEToolKitのインストール

他の文書などを参考に、RDEToolKitをインストールします。

本文書では、Pythonの仮想環境`venv`に対してRDEToolKitが導入されているものとして進めます。

```bash
$ source venv/bin/activate

(venv) $ python -m rdetoolkit version
1.2.0
```

> RDEToolKitは随時アップデートされますので上記とは異なる表記となる場合があります。

現時点(2025年5月)では、`v1.2.0`が最新となっています。それより前のバージョンをお使いの場合は、最新バージョンにアップデートすることができます。

```bash
pip install rdetoolkit --upgrade
```

> 新規開発の場合は、RDEToolKitの最新バージョンをお使いください。既存バージョンの修正の場合は、以前と同じバージョンでの利用でも問題ありませんが、RDEToolKitの最新バージョンの導入およびそれに合わせてPythonスクリプトの修正することを推奨します。

## 初期フォルダ構成作成

本書では、`$HOME/rde-sample/<モード名>` 下に構成すること想定します。

"<モード名>"の部分は、例えばマルチデータタイルモードでは"multidatatile"が、エクセルインボイスモードでは"excelinvoice"のようになります。

| # | モード名 | フォルダ名 | 備考 |
| ---- | ---- | ---- | ---- |
| 1 | マルチデータタイルモード | $HOME/rde-sample/multidatatile | |
| 2 | エクセルインボイスモード | $HOME/rde-sample/excelinvoice | |
| 3 | RDEFormatモード | $HOME/rde-sample/rdeformat | |
| 4 | インボイスモード | $HOME/rde-sample/invoice | 本書ではエクセルインボイスの一部として作成するので、明示的なフォルダは作成しません。 |

> RDEにて上記フォルダ名が決まっているわけではありません。ローカル開発においては、自由にフォルダ名を決めることができます。

以下、マルチデータタイルモードの場合の例を示します。

```bash
cd ${HOME}
mkdir -p rde-sample/multidatatile
```

当該フォルダに移動し、RDEToolKitの初期化コマンドを実行します。

```bash
(venv) $ cd rde-sample/multidatatile

(venv) $ python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /home/devel/rde-sample/multidatatile/container/requirements.txt
Created: /home/devel/rde-sample/multidatatile/container/Dockerfile
Created: /home/devel/rde-sample/multidatatile/container/data/invoice/invoice.json
Created: /home/devel/rde-sample/multidatatile/container/data/tasksupport/invoice.schema.json
Created: /home/devel/rde-sample/multidatatile/container/data/tasksupport/metadata-def.json
Created: /home/devel/rde-sample/multidatatile/templates/tasksupport/invoice.schema.json
Created: /home/devel/rde-sample/multidatatile/templates/tasksupport/metadata-def.json
Created: /home/devel/rde-sample/multidatatile/input/invoice/invoice.json

Check the folder: /home/devel/rde-sample/multidatatile
Done!
```

> 上記例は、ユーザ：develにて実行した場合の例です。実行ユーザに関しては以下同様とします。

## Config指定方法

前述の様に、マルチデータタイルモードおよびRDEフォーマットモードを使用する場合は、その旨を指定する必要があります。

指定の仕方はいくつかありますが、大きくは以下の2つの方法になります。

1. rdetoolkit.workflows.run()実行時に、Configオブジェクトを指定し、その中で指定する方法
2. 設定ファイル(data/tasksupport/rdeconfig.yml または data/tasksupport/rdeconfig.yaml)に指定する方法

> 上記1.と2.を同時に使用することはできません。1.の方法を使った場合、2.の設定ファイルを設置しても無視されます。

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

このように、Configオブジェクトにはいくつかのプロパティを設定することができます。

`system`プロパティは、RDEToolKitの設定として利用されます。

`multidata_tile`プロパティは、`system`プロパティの`extended_mode`で"MultiDataTile"が指定されている場合に、その設定として利用されます。

その他のプロパティは、開発者が自由に設定、利用することができます。

`system`プロパティで設定できる内容は以下の通りです。

| 項目名 | 意味 | フォーマット | 規定値 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| extended_mode | 拡張モードの使用を宣言 | 文字列 | 指定なし | |
| save_raw | inputdataにあるファイルをrawフォルダにコピーするか? | Boolean | False | |
| save_nonshared_raw | inputdataにあるファイルをnonshared_rawフォルダにコピーするか? | Boolean | True | |
| save_thumbnail_image | main_imageにある画像ファイルをthumbnailとして保存するか? | Boolean | True | |
| magic_variable | マジック変数({$filename})をファイル名に置き換えるか? | Boolean | False | |

> extended_modeで指定できるのは、'rdeformat' または 'MultiDataTile' (大文字小文字無視)です。それ以外を指定した場合は、指定しない場合と同様に解釈されます。
>
> これらの項目は、すべて任意指定です。指定されない場合は、規定値が指定されたものとして実行されます。

`multidata_tile`プロパティで設定できる内容は以下の通りです。

| 項目名 | 意味 | フォーマット | 規定値 | 備考 |
| ---- | ---- | ---- | ---- | ---- |
| ignore_errors | エラー発生時に処理を継続するか否か? | Boolean | False | デフォルトでは処理を止める |

なお、Configオブジェクトは、何も設定しないと既定値を指定したのと同じ設定となりますので、以下の様にすることも出来ます。

```python
import rdetoolkit

config = rdetoolkit.config.Config()
config.system.extended_mode = "rdeformat"
config.system.save_thumbnail_image = True

rdetoolkit.workflows.run(config=config)
```

### 設定ファイルを使用する方法

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

from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    config = srcpaths.config
    print(type(config))
    pprint(config)
```

実行してみます。

```bash
(venv) $ python main.py
<class 'rdetoolkit.models.config.Config'>
Config(system=SystemSettings(extended_mode=None, save_raw=False, save_nonshared_raw=False, save_thumbnail_image=False, magic_variable=False), multidata_tile=MultiDataTileSettings(ignore_errors=False))
```

> 設定内容が反映されていることが確認できます。

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

from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    user_config = srcpaths.config.user
    print(type(user_config))
    pprint(user_config)
```

上記の結果は以下の様になります。

```bash
(venv) $ python main.py
<class 'dict'>
{'x-label': 'X軸', 'y-label': 'Y軸'}
```

> 取得した設定は、通常のDictとして利用できます。
