<div class="page" />

# 応用編

ここからは、本書の前半部分(以下初級編と呼びます)を踏まえ"応用編"として、構成を変更した構造化処理プログラムについて記述します。

構成を変更する場合、以下の2つが考えられます。

* 入力ファイルのチェックやグラフ作成などを、処理毎に別々のファイルに処理を記述するパターン
* 複数の似たデータファイルに対応するため、入力ファイルごとに処理クラスとして記述するパターン

本章では、初級編と同じ処理内容を、入力ファイルのチェックやグラフ作成などを、処理毎に別々のファイルに処理を記述し、全体の見通しが良くなるように書き換えます。

## 準備

### RDEToolKit

初級編で使用したRDEToolKitをそのまま利用します。従ってPython仮想環境の"tenv"をそのまま利用します。

仮想環境を有効にしていない場合は有効にします。

```bash
$ source tenv/bin/activate
(tenv) $
```

### フォルダ構成

初級編では、`$HOME/handson/tutorial`フォルダを利用しましたが、応用編では`$HOME/handson/advanced`にファイルを設置することとします。

フォルダを作成し、RDEToolKitの初期化を実施します。

```bash
(tenv) $ cd $HOME/handson

(tenv) $ mkdir advanced
(tenv) $ cd advanced

(tenv) $ python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /home/devel/handson/advanced/container/requirements.txt
Created: /home/devel/handson/advanced/container/Dockerfile
Created: /home/devel/handson/advanced/container/data/invoice/invoice.json
Created: /home/devel/handson/advanced/container/data/tasksupport/invoice.schema.json
Created: /home/devel/handson/advanced/container/data/tasksupport/metadata-def.json
Created: /home/devel/handson/advanced/templates/tasksupport/invoice.schema.json
Created: /home/devel/handson/advanced/templates/tasksupport/metadata-def.json
Created: /home/devel/handson/advanced/input/invoice/invoice.json

Check the folder: /home/devel/handson/advanced
Done!
```

### 入力ファイル

入力ファイルとしてのsample.data、invoice.json、invoice.schema.json および metadata-def.jsonは、初級編と同じものを使うので、コピーします。

> 初級編と同様、本テキストからのコピー&ペーストにより作成しても構いません。

```bash
(tenv) $ cd container/

(tenv) $ pwd
/home/devel/handson/advanced/container

(tenv) $ cp $HOME/handson/tutorial/container/data/inputdata/sample.data data/inputdata/
(tenv) $ cp $HOME/handson/tutorial/container/data/invoice/invoice.json data/invoice/
(tenv) $ cp $HOME/handson/tutorial/container/data/tasksupport/invoice.schema.json data/tasksupport/
(tenv) $ cp $HOME/handson/tutorial/container/data/tasksupport/metadata-def.json data/tasksupport/
```

invoice.jsonを確認し、変更されている場合は、初期状態に戻します。

```bash
(tenv) $ vi data/invoice/invoice.json 
```

変更箇所は以下です。

変更前

```
        "dataName": "NIMS_TRIAL_20230126b / (2024)",
```

変更後

```
        "dataName": "NIMS_TRIAL_20230126b",
```

> " / (2024)"が追加されていたら、その部分を削除します。

### 初期化用スクリプト

初期化用スクリプトも初級編で使用したものと同じものを利用するのでコピーします。

```bash
(tenv) $ cp $HOME/handson/tutorial/container/reinit.sh .
```

初期化用スクリプトが正常に実行できることを確認します。
```bash
(tenv) $ ./reinit.sh 
./data/logs was removed
./data/meta was removed
./data/main_image was removed
./data/other_image was removed
./data/raw was removed
./data/nonshared_raw was removed
./data/structured was removed
./data/temp was removed
./data/thumbnail was removed
./data/attachment was removed
./data/invoice_patch was removed
```

### ベーススクリプト

応用編のベースとなるスクリプトを配置していきます。

> これらはNIMS内で標準的なサンプルとして利用されていたものをベースとしています。これを基に改変していきますので、ここでは以下のままスクリプトを配置してください。
>
> 初級編と同様コメント部分は削除しています。

#### modules/inputfile_handler.py

```python
from __future__ import annotations

from pathlib import Path
from typing import Any

import pandas as pd
from rdetoolkit.models.rde2types import MetaType

from modules.interfaces import IInputFileParser


class FileReader(IInputFileParser):

    def read(self, path: Path) -> tuple[MetaType, pd.DataFrame]:
        # Caution! dummy data
        self.data = pd.DataFrame([[1, 11], [2, 22], [3, 33]])
        self.meta: dict[str, str | int | float | list[Any] | bool] = {"meta1": "value1", "meta2": 2}  # Example with int value to match the expected type
        return self.meta, self.data
```

> readメソッドには、ダミー処理が記述されています。以後の処理で書き換えていきますので、ここではこのままコピーしてください。以後のファイルも同様とします。

#### modules/meta_handler.py

```python
from __future__ import annotations

from pathlib import Path
from typing import Any

from rdetoolkit import rde2util
from rdetoolkit.models.rde2types import MetaType, RepeatedMetaType

from modules.interfaces import IMetaParser


class MetaParser(IMetaParser[MetaType]):

    def parse(self, data: MetaType) -> tuple[MetaType, RepeatedMetaType]:
        # dummy return
        self.const_meta_info: MetaType = data
        self.repeated_meta_info: RepeatedMetaType = {"key": ["key_value1", "key_value2"], "sample": ["sample_value1", "sample_value2"]}
        return self.const_meta_info, self.repeated_meta_info

    def save_meta(  # noqa: ANN201
        self,
        save_path: Path,
        metaobj: rde2util.Meta,
        *,
        const_meta_info: MetaType | None = None,
        repeated_meta_info: RepeatedMetaType | None = None,
    ) -> Any:
        if const_meta_info is None:
            const_meta_info = self.const_meta_info
        if repeated_meta_info is None:
            repeated_meta_info = self.repeated_meta_info
        metaobj.assign_vals(const_meta_info)
        metaobj.assign_vals(repeated_meta_info)

        # dummy return
        return metaobj.writefile(str(save_path))
```

#### modules/structured_handler.py

```python
from __future__ import annotations

from pathlib import Path

import pandas as pd

from modules.interfaces import IStructuredDataProcessor


class StructuredDataProcessor(IStructuredDataProcessor):

    def to_csv(self, dataframe: pd.DataFrame, save_path: Path, *, header: list[str] | None = None) -> None:

        if header is not None:
            dataframe.to_csv(save_path, header=header, index=False)
        else:
            dataframe.to_csv(save_path, index=False)
```

#### modules/graph_handler.py

```python
from __future__ import annotations

from pathlib import Path

import pandas as pd

from modules.interfaces import IGraphPlotter


class GraphPlotter(IGraphPlotter[pd.DataFrame]):

    def plot(self, data: pd.DataFrame, save_path: Path, *, title: str | None = None, xlabel: str | None = None, ylabel: str | None = None) -> pd.DataFrame:

        return data  # dummy return value for testing purposes
```

#### modules/interfaces.py

```python
from __future__ import annotations

from abc import ABC, abstractmethod
from pathlib import Path
from typing import Any, Generic, TypeVar

import pandas as pd
from rdetoolkit.models.rde2types import MetaType, RepeatedMetaType
from rdetoolkit.rde2util import Meta

T = TypeVar("T")


class IInputFileParser(ABC):

    @abstractmethod
    def read(self, path: Path) -> Any:  # noqa: D102
        raise NotImplementedError


class IStructuredDataProcessor(ABC):

    @abstractmethod
    def to_csv(self, dataframe: pd.DataFrame, save_path: Path, *, header: list[str] | None = None) -> Any:  # noqa: D102
        raise NotImplementedError


class IMetaParser(Generic[T], ABC):

    @abstractmethod
    def parse(self, data: T) -> Any:  # noqa: D102
        raise NotImplementedError

    @abstractmethod
    def save_meta(
        self, save_path: Path, meta: Meta, *, const_meta_info: MetaType | None = None, repeated_meta_info: RepeatedMetaType | None = None) -> Any:
        raise NotImplementedError


class IGraphPlotter(Generic[T], ABC):

    @abstractmethod
    def plot(self, data: T, save_path: Path, *, title: str | None = None, xlabel: str | None = None, ylabel: str | None = None) -> Any:  # noqa: D102
        raise NotImplementedError
```

#### modules/datasets_process.py

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeDatasetPaths
from rdetoolkit.rde2util import Meta

from modules.graph_handler import GraphPlotter
from modules.inputfile_handler import FileReader
from modules.meta_handler import MetaParser
from modules.structured_handler import StructuredDataProcessor


class CustomProcessingCoordinator:

    def __init__(
        self,
        file_reader: FileReader,
        meta_parser: MetaParser,
        graph_plotter: GraphPlotter,
        structured_processor: StructuredDataProcessor,
    ):
        self.file_reader = file_reader
        self.meta_parser = meta_parser
        self.graph_plotter = graph_plotter
        self.structured_processor = structured_processor


def custom_module(paths: RdeDatasetPaths) -> None:

    module = CustomProcessingCoordinator(FileReader(), MetaParser(), GraphPlotter(), StructuredDataProcessor())
    # Read Input File
    meta, df_data = module.file_reader.read(paths.inputdata)
    # Save csv
    module.structured_processor.to_csv(df_data, paths.struct.joinpath("sample.csv"))
    # Meta
    module.meta_parser.parse(meta)
    module.meta_parser.save_meta(paths.meta.joinpath("metadata.json"), Meta(paths.tasksupport.joinpath("metadata-def.json")))
    # Graph
    module.graph_plotter.plot(df_data, paths.main_image.joinpath("sample.png"))


@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(paths: RdeDatasetPaths) -> None:

    custom_module(paths)
```

#### "`from __future__ import annotations`"について

上記で示したスクリプトの1行目には、"`from __future__ import annotations`"の記述があります。

`from __future__ import annotations` は、型ヒントを扱いやすくするための設定です。この行を書くと、型ヒントに書いた名前をすぐには評価せず、後で解釈するようになります。そのため、まだ定義されていないクラス名や、自分自身のクラス名を型ヒントに書きやすくなります。

<div class="page" />

## 実装

初級編で実装した内容を、そのまま上の構成を使って実装します。

> 前述の通り、上で用意したスクリプト内にはダミー処理が書かれています。ダミー処理は上書きしていきます。

### main.py

基本的に、初級編の実装と変わりません。

```python
import rdetoolkit

from modules import datasets_process


def main():
    rdetoolkit.workflows.run(
        custom_dataset_function=datasets_process.dataset
    )

if __name__ == '__main__':
    main()
```

前の章にて設定したmodules/dataset_process.pyにはダミー処理が組み込まれていますので、この時点で`python main.py`を実行すると以下の様になります。

最初に実行前のフォルダ構成を確認します。

```bash
(tenv) $ tree data
data
├── inputdata
│   └── sample.data
├── invoice
│   └── invoice.json
└── tasksupport
    ├── invoice.schema.json
    └── metadata-def.json

4 directories, 4 files
```

実行し、フォルダ構成を再度確認します。

```bash
(tenv) $ python main.py 

(tenv) $ tree data
data
├── attachment (★)
├── inputdata
│   └── sample.data
├── invoice
│   └── invoice.json
├── invoice_patch (★)
├── logs (★)
│   └── rdesys_20260930_165654.log (★)
├── main_image (★)
├── meta (★)
│   └── metadata.json (★)
├── nonshared_raw (★)
│   └── sample.data (★)
├── other_image (★)
├── raw (★)
├── structured (★)
│   └── sample.csv (★)
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp (★)
└── thumbnail (★)

15 directories, 8 files
```

* 実行により、新規に作成された部分に"(★)"を付けています。

> 必要なフォルダ構成が生成されていること(新規に11フォルダ作成)と、デフォルト設定に従い、入力ファイルが`nonshared_raw/`にコピーされていることが確認できます。
>
> ログファイル名に、実行時の日付時刻が含まれていますので、ログファイル名は実行の都度変わります。


### 生データのraw/フォルダへの自動コピー停止

RDEToolKitでは、生データ( data/inputdata/* )は、何も指定しないと自動的に"data/nonshared_raw"フォルダ(→非公開フォルダ)にコピーされます。

初級編で示したように、生データのコピーをスクリプト内で明示的に行うため、自動コピーを停止します。

自動コピーの停止は、`"main.py"の中で設定する`方法と、`設定ファイルを使って設定する`方法の2つから選択できます。

main.pyの中で設定する方法は初級編内で実施しましたので、応用編では、"設定ファイル"を置くことで同様の設定を行います。

設定ファイルのひな形(すべてデフォルト値で設定)を、以下の様に作成します。

```
rdetoolkit gen-config data/tasksupport/
```

> `rdetoolkit`コマンドは、`python -m rdetoolkit`と同じですので`python -m rdetoolkit gen-config data/tasksupport/`としても同じ意味となります。

以下の内容で、ファイルを修正します。

data/tasksupport/rdeconfig.yaml

```yaml
system:
    save_raw: false
    save_nonshared_raw: false # Falseも可
```

> 上記以外の設定については、デフォルト値が設定されているだけですので、削除しても問題ありません。

設定ファイルを使う場合、"main.py"内で設定内容を指定する必要はありません。逆に、"main.py"内で設定を行った場合は上記ファイルは**利用されません**ので注意してください。

設定ファイルの名称としては、`rdeconfig.yaml`の他、`rdeconfig.yml`や`pyproject.toml`が利用できます。

> RDEToolKit内では、main.py内の設定 → rdeconfig.yaml → rdeconfig.yml → pyproject.tomlの順に確認し、最初に見つかったもの**だけ**を使用します。

また一部のバージョンのRDEToolKit(v1.3.0など)で、`save_raw: False`と`save_nonshared_raw: False`のように双方を`False`にした場合にエラーとなる場合があります。本書の例のように、双方を`False`にしたい場合は、その他のバージョンのRDEToolKitを利用してください。
RDEToolKit v1.3.1以降ではエラーにならないように修正されていますので、特に制限がなければ最新版の利用を推奨します。

上記設定ファイルを設置後、それまでの実行で出力された出力ファイルを削除し、再度実行してみます。

```bash
(tenv) $ ./reinit.sh 
:

(tenv) $ python main.py 
(tenv) $ tree data
data
├── attachment
├── inputdata
│   └── sample.data
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys_20260930_171003.log
├── main_image
├── meta
│   └── metadata.json
├── nonshared_raw
├── other_image
├── raw
├── structured
│   └── sample.csv
├── tasksupport
│   ├── invoice.schema.json
│   ├── metadata-def.json
│   └── rdeconfig.yaml
├── temp
└── thumbnail

15 directories, 8 files
```

> `data/nonshared_raw/`"フォルダにも、`data/raw/`フォルダにも、生データ(→ data/inputdata/* )がコピーされて**いない**ことが確認できます。

### modules/datasets_process.pyから余計な処理を削除

先に示したdatasets_process.pyには、ダミー処理として不必要な処理が多く設定されていますので、処理を記述する前に削除します。

具体的には、"custom_module()"関数内の記述をすべて削除します。

modules/datasets_process.py

```python
def custom_module(paths: RdeDatasetPaths) -> None
    pass

```

> メソッド(関数)内に処理がないとエラーになりますので、何も実行しないことを意味する`pass`だけを記述します。後述する処理を追記した場合は、不要となりますので、削除してください。

この状態で`python main.py`を実行すると、以前同様出力用のフォルダが作成されます。

```bash
(tenv) $ ./reinit.sh
:

(tenv) $ python main.py 
(tenv) $ tree data
data
├── attachment
├── inputdata
│   └── sample.data
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys_20260930_172056.log
├── main_image
├── meta
├── nonshared_raw
├── other_image
├── raw
├── structured
├── tasksupport
│   ├── invoice.schema.json
│   ├── metadata-def.json
│   └── rdeconfig.yaml
├── temp
└── thumbnail

15 directories, 6 files
```

> ログファイル名には実行日時が含まれていますので、実行の都度変わります。

上記削除により、`data/meta/metadata.json`と`data/structured/sample.csv`の2つのファイルが生成されなくなったことが確認できます。

### コーディネータクラスのインスタンスの準備

各種処理を記述する際に利用すると便利なクラスをまとめた`CustomProcessingCoordinator`が用意されているので、このインスタンスを用意します。

modules/datasets_process.py

```python
：

def custom_module(paths: RdeDatasetPaths) -> None:

    coordinator = CustomProcessingCoordinator(
                      FileReader(),
                      MetaParser(),
                      GraphPlotter(),
                      StructuredDataProcessor(),
    )
：
```

> 変更前は、"module"という変数を用いていましたが、pythonモジュールの格納場所が`modules/`であり、区別が付けにくいので、本書では変数として`coordinator`を使用します。
>
> 何も実行しないという意味で記述してあった"pass"を削除します。

上記状態で保存して、次に進みます。

### 入力ファイルのチェック

入力ファイルのチェック部分をinputfile_handler.pyに実装します。

modules/inputfile_handler.py

```python
：
from rdetoolkit.errors import StructuredError
：
class FileReader(IInputFileParser):
    ：
    def check(self, input_files: list[Path]) -> bool:
        # Check input file
        if len(input_files) == 0:
            raise StructuredError("ERROR: input data not found")

        if len(input_files) > 1:
            raise StructuredError("ERROR: input data should be one file")

        raw_file_path = input_files[0]
        if raw_file_path.suffix.lower() != ".data":
            raise StructuredError(
                f"ERROR: input file is not '*.data' : {raw_file_path.name}"
            )

        return True
```

> 関数化に伴い、最後に"return True"が追加されています。真偽値の戻り値が期待されますが、True以外の場合は、raise処理が実行されるため、False(偽)が返ることはありません。同様にraise処理が実行された場合は、それ以降は実行されませんので、else句でうける必要もないので、ここでは、elif/else句を使わない形を使用しています。
>
> readメソッドについては後で修正しますので、この時点では変更しません。

呼び出し側(modules/datasets_process.py)の方を変更します。

```python
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Check input file
    coordinator.file_reader.check(paths.rawfiles)
：
```

生データが存在していることを確認し、構造化処理を実行してみます。

```bash
(tenv) $ cat data/inputdata/sample.data
[METADATA]
data_title=data_title 2
measurement_date=2023/1/25 23:08
:

(tenv) $ python main.py 
(tenv) $ 
```

問題無く終了しました。

続いて、エラー状態を試します。

```bash
(tenv) $ touch data/inputdata/test.data

(tenv) $ python main.py 

Traceback (simplified message):
Call Path:
   File: /home/devel/handson/tenv/lib/python3.12/site-packages/rdetoolkit/errors.py, Line: 46 in wrapper()
    └─ File: /home/devel/handson/advanced/container/modules/datasets_process.py, Line: 41 in dataset()
        └─ File: /home/devel/handson/advanced/container/modules/datasets_process.py, Line: 36 in custom_module()
            └─ File: /home/devel/handson/advanced/container/modules/inputfile_handler.py, Line: 20 in check()
                └─> L20: raise StructuredError("ERROR: input data should be one file") 🔥

Exception Type: StructuredError
Error: ERROR: input data should be one file
```

> 「入力ファイルが1個だけある」という前提が崩れているため、想定通りエラーとなります。

確認がすみましたので、先ほど作成したファイルを消します。

```bash
(tenv) $ rm data/inputdata/test.data
(tenv) $ tree data/inputdata
data/inputdata
└── sample.data

1 directory, 1 file
```

入力ファイルのチェックの内容は初級編と同じですので、他のエラー処理の確認は省略します。

### invoice.jsonの読み込みと保存

invoice.jsonを扱うクラスを**新規に**作成します。

modules/invoice_handler.py

```python
from pathlib import Path

from rdetoolkit.errors import StructuredError
from rdetoolkit.invoicefile import InvoiceFile


class InvoiceParser():
    """RDE invoice.json handle
    """
    additional_title = "(2024)"

    def __init__(self):
        self.invoice_file = None
        self.invoice_obj = None
        self.raw_dir = None
        self.nonshared_raw_dir = None

    def parse(self, invoice_file):
        self.invoice_path = invoice_file
        self.invoice_obj = InvoiceFile(invoice_file)

    @property
    def is_private_raw(self) -> bool:
        if self.invoice_obj is None:
            raise StructuredError("ERROR: invoice does not set")

        invoice_dict = self.invoice_obj.invoice_obj
        # Check private or not
        custom = invoice_dict.get("custom")
        if custom is None:
            return False
        is_private_raw = custom.get("is_private_raw")
        if is_private_raw == "share":
            return False
        # other case -> "private"
        return True

    def set_dirs(self, *, raw_dir: Path|None = None, nonshared_raw_dir: Path|None = None):
        self.raw_dir = raw_dir
        self.nonshared_raw_dir = nonshared_raw_dir

    def change_title(self):
        if self.invoice_obj is None:
            raise StructuredError("ERROR: invoice does not set")
        # Update invoice title
        original_data_name = self.invoice_obj["basic"]["dataName"]
        additional_title = self.additional_title
        if original_data_name.find(additional_title) < 0:
            # update title if not applied yet
            self.invoice_obj["basic"]["dataName"] = \
                original_data_name + " / " + additional_title
            # Overwrite
            invoice_file_new = self.invoice_path
            self.invoice_obj.overwrite(invoice_file_new)

    def backup(self):
        # Backup(=Copy) invoice.json to shared/nonshared folder
        is_private_raw = self.is_private_raw
        invoice_file = self.invoice_path
        if is_private_raw:
            backup_dir = self.nonshared_raw_dir
        else:
            backup_dir = self.raw_dir
        # check backup dir 
        if backup_dir is None:
            raise StructuredError("ERROR: backup dir does not set")

        invoice_backup_file = backup_dir / "invoice.json.orig"
        InvoiceFile.copy_original_invoice(invoice_file, invoice_backup_file)
```

> 上位(datasets_process.pyのdataset()関数、もしくはcustom_module()関数)で利用していた変数`paths`はここでは利用できませんので、`set_dirs()`メソッドを利用してセットするようにしています。
>
> `StorageDir`クラスや`DirectoryOps`クラスを使用しても実装できますが、Excelインボイス利用時の処理などが複雑になります。上記で示しているようにpathsの内容をそのまま利用することをお勧めします。

これらを使って処理を実行するように"modules/datasets_process.py"に追加します。

```python
：
from modules.invoice_handler import InvoiceParser
：
class CustomProcessingCoordinator:

    def __init__(
        self,
        file_reader: FileReader,
        meta_parser: MetaParser,
        graph_plotter: GraphPlotter,
        structured_processor: StructuredDataProcessor,
        invoice_parser: InvoiceParser,
    ):
        self.file_reader = file_reader
        self.meta_parser = meta_parser
        self.graph_plotter = graph_plotter
        self.structured_processor = structured_processor
        self.invoice_parser = invoice_parser


def custom_module(paths: RdeDatasetPaths) -> None:

    coordinator = CustomProcessingCoordinator(
             FileReader(),
             MetaParser(),
             GraphPlotter(),
             StructuredDataProcessor(),
             InvoiceParser(),
    )

    # Check input file
    coordinator.file_reader.check(paths.rawfiles)

    # Read and Update Invoice
    invoice = coordinator.invoice_parser
    invoice.parse(paths.invoice / "invoice.json")
    invoice.set_dirs(
        raw_dir = paths.raw,
        nonshared_raw_dir = paths.nonshared_raw,
    )
    invoice.backup()
    invoice.change_title()
：
```

> `CustomProcessingCoordinator`クラスに、`InvoiceParser`クラスのインスタンスも管理するように変更します。
>
> 実行する順番に注意してください。invoice.jsonの変更を実施するメソッドの前にbackupメソッドを実行しないと、変更後の内容を"バックアップ"することになります。

main.pyを実行します。

```bash
(tenv) $ python main.py
(tenv) $ tree
：
│   ├── nonshared_raw
│   │   └── invoice.json.orig
:
(tenv) $ diff -ru data/nonshared_raw/invoice.json.orig data/invoice/invoice.json
--- data/nonshared_raw/invoice.json.orig        2026-09-30 17:51:48.832566026 +0900
+++ data/invoice/invoice.json   2026-09-30 17:51:48.832566026 +0900
@@ -3,7 +3,7 @@
     "basic": {
         "dateSubmitted": "2023-01-26",
         "dataOwnerId": "6932eaf2ba96dc12812d07f3f3bbdca9de5e149c3837376539343134",
-        "dataName": "NIMS_TRIAL_20230126b",
+        "dataName": "NIMS_TRIAL_20230126b / (2024)",
         "instrumentId": "413e53fb-aec9-41f8-ae55-3f88f6cd8d41",
         "experimentId": null,
         "description": ""
```

次の処理に備え、上で出力したファイルを削除するなど、実行前の状態に戻しておきます。

```bash
(tenv) $ ./reinit.sh
：
```

> 本来は、invoice.jsonファイルもバックアップしたファイルに戻した方がよいですが、大きな影響はないため、ここではそのまま改変後のinvoice.jsonを入力値として扱うことにします。

### 入力データの読み込み

続いて、入力データを読み込む処理を記述します。

> すでに存在しているreadメソッド処理は削除してから、上の記述に変更してください。

modules/inputfile_handler.py

```python
import io
：
    def read(self, input_files: list[Path]) -> tuple[MetaType, list[pd.DataFrame]]:
        # Read input data
        DELIM = "="
        rawDataDf = None
        rawMetaObj = None
        #
        input_file = input_files[0]

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
        #
        return raw_meta_obj, raw_data_df
```

> 戻り値の型ヒント部分も変更されます。`tuple[MetaType, pd.DataFrame]` → `tuple[MetaType, list[pd.DataFrame]]`

これらを使って処理を実行するように"modules/datasets_process.py"に追加します。

```python
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Read input data
    meta, df_data = coordinator.file_reader.read(paths.rawfiles)

：
```

pprintなどを使って内容を確認すると以下の様になっています。

meta

```
{'data_title': 'data_title 2',
 'measurement_date': '2023/1/25 23:08',
 'series_number': '3',
 'x_label': 'x(unit)',
 'y_label': 'y(unit)'}
```

df_data

```
[       x  series1
0    1.0    101.0
1    2.0    102.0
2    3.0    103.0
3    4.0    104.0
4    5.0    105.0
5    6.2    106.0
6    7.0      NaN
7    8.0    108.0
8    9.5    109.0
9   10.0    101.0
10  11.0    103.0
11  12.0    105.0
12  13.0    104.0
13  14.0    100.0
14  15.0     98.0
15  16.0     82.0,
      x  series2
0    1      101
1    2      102
2    3      103
3    4      104
4    5      105
5    6      106
6    9      109
7   10       90
8   12       60
9   13       52
10  14       41,
        x  series3
0    1.0    101.0
1    1.5    101.2
2    2.0    102.0
3    3.0    103.0
4    4.0    104.0
5    5.0    105.0
6    6.0    106.0
7    9.0    109.0
8   10.0     90.0
9   12.0     60.0
10  13.0     52.0
11  14.0     41.0]
```

### メタデータの出力

メタデータの取得と出力を実装します。基本的なロジックは、初級編と同じです。

> もともとサンプルとして実装されていた部分はほとんど使いません。下記に置き換えてください。

modules/meta_handler.py

```python
from __future__ import annotations

from pathlib import Path
from typing import Any
import statistics as st

from rdetoolkit import rde2util
from rdetoolkit.models.rde2types import MetaType, RepeatedMetaType

from modules.interfaces import IMetaParser


class MetaParser(IMetaParser[MetaType]):

    def __init__(self):
        self.const_meta_info = dict()
        self.repeated_meta_info = dict()

    def parse_from_invoice(self, invoice) -> None:
        # Merge invoice info to meta
        if invoice["custom"] is not None:
            self.const_meta_info = self.const_meta_info | invoice["custom"]

    def parse_from_inputdata(self, meta, data) -> None:

        # From meta
        self.const_meta_info = self.const_meta_info | meta

        # From raw data numeric data part
        s_name = []
        s_count = []
        s_mean = []
        s_median = []
        s_max = []
        s_min = []
        s_stdev = []

        for df in data:
            d = df.dropna(axis=0)
            y = d.iloc[:, 1]
            s_name.append(d.columns[1])
            s_count.append(len(y))
            s_mean.append("{:.2f}".format(st.mean(y)))
            s_median.append("{:.2f}".format(st.median(y)))
            s_max.append(max(y))
            s_min.append(min(y))
            s_stdev.append("{:.2f}".format(st.stdev(y)))

        metaVars = {
            "series_name": s_name,
            "series_data_count": s_count,
            "series_data_mean": s_mean,
            "series_data_median": s_median,
            "series_data_max": s_max,
            "series_data_min": s_min,
            "series_data_stdev": s_stdev,
        }

        # Set variable meta
        self.repeated_meta_info = metaVars

    def parse(self, data: MetaType) -> tuple[MetaType, RepeatedMetaType]:
        #
        return self.const_meta_info, self.repeated_meta_info

    def save_meta(
        self,
        save_path: Path,
        metaobj: rde2util.Meta,
        *,
        const_meta_info: MetaType | None = None,
        repeated_meta_info: RepeatedMetaType | None = None
    ):
        """
        Save parsed metadata to a file using the provided Meta object
        """

        if const_meta_info is None:
            const_meta_info = self.const_meta_info
        if repeated_meta_info is None:
            repeated_meta_info = self.repeated_meta_info
        metaobj.assign_vals(const_meta_info)
        metaobj.assign_vals(repeated_meta_info)

        metaobj.writefile(save_path)
```

これらを使って処理を実行するように"modules/datasets_process.py"に追加します。

```python
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Read input data
    meta, df_data = coordinator.file_reader.read(paths.rawfiles)

    # Meta  (※ここ以下を追加する)
    meta_parser = coordinator.meta_parser 
    meta_parser.parse_from_invoice(invoice.invoice_obj)
    meta_parser.parse_from_inputdata(meta, df_data)
    meta_parser.save_meta(
        paths.meta.joinpath("metadata.json"),
        Meta(paths.tasksupport.joinpath("metadata-def.json"))
    )
：
```

### 生データの可読化

続いて、生データの可読化、つまり、生データをCSVファイルとして出力する部分を実装します。

modules/structured_handler.py

```python
from __future__ import annotations

from pathlib import Path

import pandas as pd

from modules.interfaces import IStructuredDataProcessor


class StructuredDataProcessor(IStructuredDataProcessor):

    def to_csv(self, dataframes: list[pd.DataFrame], save_path: Path, *, header: list[str] | None = None) -> None:
        # Write CSV file(s)
        for d in dataframes:
            fname = d.columns[1].replace(" ", "") + ".csv"
            csvFilePath = save_path / fname
            if header is not None:
                d.to_csv(csvFilePath, header=header, index=False)
            else:
                d.to_csv(csvFilePath, header=True, index=False)
```

これらを使って処理を実行するように"modules/datasets_process.py"に追加します。

```python
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Read input data
    meta, df_data = coordinator.file_reader.read(paths.rawfiles)
:
    # Save csv  (※ここ以下を追加する)
    coordinator.structured_processor.to_csv(df_data, paths.struct)

：
```

### 生データの可視化

可読化に続き、生データの可視化を実装します。

modules/graph_handler.py

```python
from __future__ import annotations

import pandas as pd
import matplotlib.pyplot as plt
from rdetoolkit.models.rde2types import RdeDatasetPaths

from modules.interfaces import IGraphPlotter


class GraphPlotter(IGraphPlotter[pd.DataFrame]):

    def plot(self, data: list[pd.DataFrame], save_path: RdeDatasetPaths, *, title: str | None = None, xlabel: str | None = None, ylabel: str | None = None):
        # Write Graph
        title = title if title is not None else "No title"
        xlabel = xlabel if xlabel is not None else "X-label"
        ylabel = ylabel if ylabel is not None else "Y-label"

        ## by series
        for d in data:
            x = d.iloc[:, 0]
            y = d.iloc[:, 1]
            label = d.columns[1]
            fname = save_path.other_image / f"{label}.png"
            fig, ax = plt.subplots(figsize=(5, 5), facecolor="white")
            ax.plot(x, y, label=label)
            ax.set_title(title)
            ax.set_xlabel(xlabel)
            ax.set_ylabel(ylabel)
            ax.legend()
            fig.savefig(fname)
            plt.close(fig)

        ## all series
        fig, ax = plt.subplots(figsize=(5, 5), facecolor="lightblue")
        ax.set_xlabel(xlabel)
        ax.set_ylabel(ylabel)
        ax.set_title(title)
        for d in data:
            x = d.iloc[:, 0]
            y = d.iloc[:, 1]
            label = d.columns[1]
            ax.plot(x, y, label=label)
        ax.legend()
        fname = save_path.main_image / "all_series.png"
        fig.savefig(fname)
        plt.close(fig)
```

これらを使って処理を実行するように"modules/datasets_process.py"に追加します。

```python
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Read input data
    meta, df_data = coordinator.file_reader.read(paths.rawfiles)
:
    # Graph  (※ここ以下を追加する)
    const_meta_info = meta # or coordinator.meta_parser.const_meta_info
    coordinator.graph_plotter.plot(
        df_data,
        paths,
        title=const_meta_info["data_title"],
        xlabel=const_meta_info["x_label"],
        ylabel=const_meta_info["y_label"]
    )

：
```

### サムネイル画像の作成

サムネイル画像を扱うクラスが用意されていませんので、新規に作成します。

基礎編のように、独自にコーディングしてもよいですが、RDEToolKit v1.0.2よりサムネイル画像作成に利用できる関数が追加になったので、ここではそれを使ってサムネイル画像をつくります。

以下の内容で、新規に作成します。

modules/thumbnail_handler.py

```python
from __future__ import annotations

from pathlib import Path

from rdetoolkit.models.rde2types import RdeDatasetPaths
from rdetoolkit.img2thumb import resize_image


class ThumbnailDrawer():

    def draw(self, paths: RdeDatasetPaths) -> None:
        # Write thumbnail images
        src_img_file_path = paths.main_image / "all_series.png"
        out_img_file_path = paths.thumbnail / "thumbnail.png"

        # Size of thumbnail
        closure_w = 286
        closure_h = 200

        resize_image(
            src_img_file_path,
            closure_w,
            closure_h,
            out_img_file_path
        )
```

これらを使って処理を実行するように"modules/datasets_process.py"に追加します。

```python
：
from modules.thumbnail_handler import ThumbnailDrawer
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Graph
    const_meta_info = meta # or coordinator.meta_parser.const_meta_info
    coordinator.graph_plotter.plot(
        df_data,
        paths,
        title=const_meta_info["data_title"],
        xlabel=const_meta_info["x_label"],
        ylabel=const_meta_info["y_label"]
    )
    ：
    # Thumbnail  (※ここ以下を追加する)
    thumbnail = ThumbnailDrawer()
    thumbnail.draw(paths)
：
```

> グラフ画像をもとにサムネイル画像を作成しますので、グラフ画像生成の**後に**実行する必要があります。
>
> また、基礎編でのサムネイル画像作成とはことなり、縦横比の調整などは行っていないため、上記実装ではサムネイル画像の左右端に白色帯が作成されてしまいます。問題がある場合は、作成されるサムネイル画像の縦横サイズ設定を変更するか、基礎編での実装を使ってください。
>
> invoiceファイルの場合は、`CustomProcessingCoordinator`クラスに追加しましたが、ここでは追加せずに利用するようにしています。追加する形で利用しても問題ありません。

デフォルト設定では、main_imageフォルダに画像がある場合は、サムネイル画像としてコピーされてしまいますので、その動作を抑止します。

data/tasksupport/rdeconfig.yaml

```yaml
system:
  :
  save_thumbnail_image: false
```

### 生データの永続化

設定により生データの永続化(nonshared_rawまたはrawフォルダへの自動コピー)を抑止しましたので、永続化処理を実装します。

> 別ファイルとして切り出すほどのロジックではないので、datasets_process.pyに直接記述することにします。

modules/datasets_process.py

```python
：
import shutil
：
def custom_module(paths: RdeDatasetPaths) -> None:
    ：
    # Copy inputdata to public (raw/) or non_public (nonshared_raw/)
    is_private_raw = invoice.is_private_raw
    raw_dir = paths.nonshared_raw if is_private_raw else paths.raw
    for input_file in paths.rawfiles:
        shutil.copy(input_file, raw_dir)
```

> invoice.jsonの設定内容を使いますので、invoiceの処理の**後に**に記述する必要があります。
>
> 今回の例では、入力ファイルは1個であることが確定していますのでfor文を使う必要はありませんが、複数の入力ファイルを扱う場合と同じコードで実行できるようfor文を使っています。

### まとめ

上記の様な記述を行うことにより、custom_module()関数内の記述がすっきりします。

念のため、最終的なmodules/datasets_process.pyを以下に示します。

```python
import shutil

from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeDatasetPaths
from rdetoolkit.rde2util import Meta

from modules.graph_handler import GraphPlotter
from modules.inputfile_handler import FileReader
from modules.meta_handler import MetaParser
from modules.structured_handler import StructuredDataProcessor
from modules.invoice_handler import InvoiceParser
from modules.thumbnail_handler import ThumbnailDrawer

class CustomProcessingCoordinator:

    def __init__(
        self,
        file_reader: FileReader,
        meta_parser: MetaParser,
        graph_plotter: GraphPlotter,
        structured_processor: StructuredDataProcessor,
        invoice_parser: InvoiceParser,
    ):
        self.file_reader = file_reader
        self.meta_parser = meta_parser
        self.graph_plotter = graph_plotter
        self.structured_processor = structured_processor
        self.invoice_parser = invoice_parser


def custom_module(paths: RdeDatasetPaths) -> None:

    coordinator = CustomProcessingCoordinator(
                      FileReader(),
                      MetaParser(),
                      GraphPlotter(),
                      StructuredDataProcessor(),
                      InvoiceParser(),
    )

    # Check input file
    coordinator.file_reader.check(paths.rawfiles)

    # Read and Update Invoice
    invoice = coordinator.invoice_parser
    invoice.parse(paths.invoice / "invoice.json")
    invoice.set_dirs(
        raw_dir = paths.raw,
        nonshared_raw_dir = paths.nonshared_raw,
    )
    invoice.backup()
    invoice.change_title()

    # Read input data
    meta, df_data = coordinator.file_reader.read(paths.rawfiles)

    #from pprint import pprint
    #pprint(meta)
    #print("====")
    #pprint(df_data)

    # Meta
    meta_parser = coordinator.meta_parser
    meta_parser.parse_from_invoice(invoice.invoice_obj)
    meta_parser.parse_from_inputdata(meta, df_data)
    meta_parser.save_meta(
        paths.meta.joinpath("metadata.json"),
        Meta(paths.tasksupport.joinpath("metadata-def.json"))
    )

    # Save csv
    coordinator.structured_processor.to_csv(df_data, paths.struct)

    # Graph
    const_meta_info = meta # or coordinator.meta_parser.const_meta_info
    coordinator.graph_plotter.plot(
        df_data,
        paths,
        title=const_meta_info["data_title"],
        xlabel=const_meta_info["x_label"],
        ylabel=const_meta_info["y_label"]
    )

    # Thumbnail
    thumbnail = ThumbnailDrawer()
    thumbnail.draw(paths)

    # Copy inputdata to public (raw/) or non_public (nonshared_raw/)
    is_private_raw = invoice.is_private_raw
    raw_dir = paths.nonshared_raw if is_private_raw else paths.raw
    for input_file in paths.rawfiles:
        shutil.copy(input_file, raw_dir)

@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(paths: RdeDatasetPaths) -> None:

    custom_module(paths)
```

> 前述のように`CustomProcessingCoordinator`クラスに、ThumbnailDrawerクラスを追記した方がよいかもしませんが、本章ではここまでにしておきます。
