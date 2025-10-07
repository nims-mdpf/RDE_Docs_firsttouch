<div class="page" />

# 実験データの永続化

RDE では、アップロードされたデータは構造化処理前は、`data/inputdata/`フォルダに保存されています。

> タイトルに「実験データ」と付いていますが、入力されたデータファイル全般を、同様に考えることができます。

この「inputdata」フォルダに保存されたデータは、構造化処理後の"永続化"フォルダには保存されません。そのため、実験データを永続的に保存する、つまり他のユーザがダウンロードして利用できるようにするためには「inputdata」フォルダから「raw」または「nonshared_raw」フォルダにコピーする必要があります。

実験データを永続化したい場合は、公開／非公開の取扱いに応じてコピーすることになります。

* 実験データを公開する場合は、rawフォルダに保存
* 実験データを非公開にする場合は、nonshared_rawフォルダに保存

> 公開されるのは、決められた期日(→エンバーゴ日付)を過ぎてからになります。それまではrawフォルダにあったとしても、アクセス出来るのは、当該データセットを利用可能な研究グループに所属するユーザのみとなります。

## 想定

登録したデータが、"公開"なのか"非公開"なのかについては、通常データセット開設時に決定することになります。RDEToolKitのデフォルトは"非公開"となります。

設定を変更することで、すべての入力ファイルを"公開"とすることができます。

また、送り状(invoice.json)などの指定値を読み込んで、1つのデータセットの中で、公開/非公開を選択することができます。

しかし、その"判定"方法についての決まった方式は存在せず、それぞれの構造化処理で自由に決定することができます。

本書では、以下の様に想定します。

* data/invoice/invoice.jsonの"custom" → "is_private_raw"の値から、データ毎に決定する。
* "is_provate_raw"の値が"share"の場合は"公開"とする。それ以外の値の場合は非公開とする。
* 公開の場合は"data/raw"フォルダに、非公開の場合は"data/nonshared_raw"フォルダにコピーする。

## custom_module()にロジックを実装

RDEToolKitのデフォルト設定では、追加コーディング無しで、nonshared_rawフォルダ、つまり非公開フォルダにコピーされます。

> 古いバージョンのRDEToolKit(→ v1.0.2より前のバージョン)では、"公開"つまりrawフォルダへのコピーがデフォルトでした。なにも設定しない場合、どちらのフォルダにコピーされるのかは、RDEToolKitのバージョンによって変わる可能性がありますのでご注意ください。

```bash
(tenv) $ python main.py 
(tenv) $ tree
：
│   ├── nonshared_raw
│   │   └── sample.data
：
```

すべての入力ファイルをnonshared_rawフォルダに置く場合は、デフォルト処理で問題ありません。

逆に、すべての入力ファイルをrawフォルダに置く場合は以下の様にします。

```python
import rdetoolkit
from rdetoolkit.config import Config, MultiDataTileSettings, SystemSettings

from modules import datasets_process


def main():
    config = Config(
        system=SystemSettings(
            save_raw=True,
            save_nonshared_raw=False,
        ),
    )

    rdetoolkit.workflows.run(
        config=config,
        custom_dataset_function=datasets_process.dataset
    )

if __name__ == '__main__':
    main()
```

> "save_raw"に`True`を、"save_nonshared_raw"に`False`を設定します(デフォルトはそれぞれ逆になっています)。
>
> 前述のようにコンフィグ設定(上記プログラム例の"config=～"の部分)を、外部ファイル(data/tasksupport/rdeconfig.ymlなど)に置く場合は、main.pyの修正は不要となります。

前章で示した"想定"の様に、入力内容によって変えるには、自動的にnonshared_rawまたはrawフォルダにコピーされてしまっては問題があります。そのため、`raw/`にコピーする処理および`nonshared_raw/`にコピーする処理の**双方を無効**にします。

自動でnonshared_rowフォルダまたはrawフォルダにコピーする処理を無効にするために、main.pyを以下の様にします。

```python
import rdetoolkit
from rdetoolkit.config import Config, MultiDataTileSettings, SystemSettings

from modules import datasets_process


def main():
    config = Config(
        system=SystemSettings(
            save_raw=False,
            save_nonshared_raw=False,
        ),
    )

    rdetoolkit.workflows.run(
        config=config,
        custom_dataset_function=datasets_process.dataset
    )

if __name__ == '__main__':
    main()
```

続いて、datasets_process.pyで、rawフォルダかnonshared_rawフォルダのどちらのフォルダにコピーすれば良いかを取得し、それに応じて入力データを、適切なフォルダにコピーするように処理を記述します。

modules/datasets_process.py

```python
import shutil
：
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    ：
    # Copy inputdata to public (raw/) or non_public (nonshared_raw/)

    # raw_dir = resource_paths.nonshared_raw if is_private_raw else resource_paths.raw
    # input_file = resource_paths.rawfiles[0] # read one file only
    shutil.copy(input_file, raw_dir)
```

> 入力ファイルおよびコピー先のフォルダは、ここまでの処理で変数にセットされているため、そのまま利用します。そのため、上記ではコメントとしてあります。

実行すると以下の様になります。

> 送り状にて、is_private_raw項目で"private"を選択した場合の例です。

```bash
(tenv) $ python main.py

(tenv) $ tree
.
├── data
│   ├── raw
│   ├── nonshared_raw
│   │   ├── invoice.json.orig
│   │   └── sample.data
:
```

