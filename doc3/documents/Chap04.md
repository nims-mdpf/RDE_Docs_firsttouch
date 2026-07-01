<div class="page" />

# 構造化処理の記述開始

## main.py

以降、必要な処理を実装していきますが、それらは"どこ"に記述していけばよいでしょうか?

処理が実行される順に確認していきます。

RDE構造化処理では、最初に以下が実行されます。

```bash
(tenv) $ python main.py
```

> 上記が実行されるのは、テンプレート内で、"container/"フォルダにあるmain.pyを実行するように設定するためです。そのためこの設定を変更すれば別の名前のスクリプトを実行するように変更することもできます。ただし、プログラムのメンテナンスの観点から変更しないことを推奨します。

init処理(python -m rdetoolkit init)を実行しただけのmain.pyでは、空行、コメント行を除けば、以下の2行からなります。

```python
import rdetoolkit
rdetoolkit.workflows.run()
```

独自のRDE構造化処理を実行するには、この"rdetoolkit.workflows.run()"の名前付き引数"custom_dataset_function="に、実行される関数を指定する必要があります。

```python
：
rdetoolkit.workflows.run(custom_dataset_function=datasets_process.dataset)
：
```

独自処理を記述したPythonスクリプトは、(main.pyと同じ階層にある)"modules/"フォルダに配置することになっています。

従って、上記のように記述した場合、"modules/datasets_process.py"にある、"dataset()"関数が実行されます。

また、スクリプトの上部で、当該スクリプトを"import"する必要があります。

以上をまとめると、"main.py"は以下の様になります。

```python
import rdetoolkit
from modules import datasets_process


rdetoolkit.workflows.run(custom_dataset_function=datasets_process.dataset)
```

> 本書ではコメントについての記述は、一部を除いて行いません。実際のスクリプトにおいては、適切なコメントを付与することが推奨されます。
>
> スクリプトファイル名も、"rdetoolkit.workflows.run()"にて実行される関数名も、任意に決定することができますが、プログラムのメンテナンス性確保のため、スクリプトファイル名として"modules/datasets_process.py"を、実行される関数名として"dataset()"の利用を推奨します。
> 同様に格納するフォルダ名も"modules/"である必要はありませんが、こちらもメンテナンスの観点から変更しないことをお勧めします。

なお、main.pyは、よくあるpythonスクリプトの様に、以下の様に記述しても構いません。

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

まだ"modules/datasets_process.py"を作成していませんので、この時点でmain.pyを実行すると、以下の様なエラーメッセージが表示されます。

```bash
(tenv) $ python main.py 
：
ImportError: cannot import name 'datasets_process' from 'modules' (unknown location)
```

続いて、このdatasets_process.pyを記述していきます。

## datasets_process.py

最初の一歩として、"Hello RDE!"と表示されるような処理を実装してみます。

"modules/datasets_process.py"を、以下の内容で作成します。

* RDEToolKit v1.5以降

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.exceptions import StructuredError
from rdetoolkit.models.rde2types import RdeDatasetPaths


@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(paths: RdeDatasetPaths) -> None:
    print("Hello RDE!")
```

* RDEToolKit v1.5より前のバージョン

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.exceptions import StructuredError
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    print("Hello RDE!")
```

* "catch_exception_with_message"および"StructuredError"は、なんらかのエラーが発生した場合に利用されますので、"import"する必要があります。

main.pyの"custom_dataset_function=～"で指定される関数は、1つまたは2つの引数を取る必要があります。
RDEToolKit v1.5.0より前のバージョンでは、2つの引数指定が必要でしたが、RDEToolKit v1.5.0以降はこれら2つの引数の内容が整理され、1つの引数で扱うようになりました。

* 後方互換性維持のため、2つの引数をとる形式もしばらくの間はサポートされます。本書では1つのみの引数を取る形式を採用します。

指定する引数は、処理の入出力に関係するフォルダ/ファイルのpathlibオブジェクトが格納されています。これらはRDEToolKitが自動でセットしてくれるため、多くのRDE構造化処理プログラムで共通して利用することができます。

> 引数を2つ指定する場合は、1つ目はRdeInputDirPathsオブジェクトのインスタンスで、主に入力に関係するフォルダ/ファイルのpathlibオブジェクトが格納されています。2つ目はRdeOutputResourcePathオブジェクトのインスタンスで、主に出力に関係するフォルダ/ファイルのpathlibオブジェクトが格納されています。

実行すれば以下の様に出力されます。

```bash
(tenv) $ python main.py 
Hello RDE!
```

この時点で"Error in validating invoice.json"エラーがでる場合があります。これはRDEToolKit v1.3.3より前のバージョンにて自動生成された`invoice.json`に問題があったことが原因の可能性があります。RDEToolKit v1.3.3以降でバリデーション処理が変更になり、`invoice.schema.json`に定義のない項目が`invoice.json`に含まれている場合に異常として処理するようになったためです。

* 実際のRDEシステムで稼働する際は、invoice.schema.jsonに定義のない情報が含まれるinvoice.jsonが作成されることはないため、上記のようなエラーは発生しません。

このエラーの対処は、以下の2つが考えられます。

* invoice.jsonからsample部分を削除する。
* invoice.schema.jsonに、sampleに関する定義を追加し、requiredに`sample`も加える。

> 現在配布されている`samples.zip`に含まれるinvoice.jsonとinvoice.schema.jsonの組み合わせでは上記エラーが発生しないことを確認しています。`samples.zip`を利用して上記問題が発生する場合は、新しい`samples.zip`を使ってください。

また、Python コードのスタイルガイド(→ https://pep8-ja.readthedocs.io/ja/latest/ )には以下の様にあります。

```
import文 は次の順番でグループ化すべきです:
1. 標準ライブラリ
2. サードパーティに関連するもの
3. ローカルな アプリケーション/ライブラリ に特有のもの
```

本書も、基本的には上記に従います。

また、import文の書き方も重要となります。

例1)

```python
import rdetoolkit
from modules import datasets_process


rdetoolkit.workflows.run(custom_dataset_function=datasets_process.dataset)
```

例2)

```python
import rdetoolkit
from modules.datasets_process import dataset


rdetoolkit.workflows.run(custom_dataset_function=dataset)
```

例2の方が、ロジック部分の記述がすっきりするため、本書では例2の方の書き方を中心にします。

> 本質的には同じものとなります。詳しくはPython関連の情報を参照してください。

### datasets() → custom_module()

このままdatasets()関数内で処理を記述してもよいのですが、現在取り扱っている構造化プログラムでは、多くの場合datasets()内からcustom_module()関数を呼び出し、そこに必要な構造化処理を書いていくスタイルを取っています。

そのため本書でも、そのような書き方で処理を書いていくようにします。

スクリプト内のdatasets関数の上部に、custom_module()関数を定義し、サンプルで追加していたprint文をそちらに移動します。datasets()関数では、新たに定義したcustom_module()を実行するように変更します。

結果、"modules/datasets_process.py"は以下の様になります。

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.exceptions import StructuredError
from rdetoolkit.models.rde2types import RdeDatasetPaths


def custom_module(paths: RdeDatasetPaths) -> None:
    print("Hello RDE!")

@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(paths: RdeDatasetPaths) -> None:
    custom_module(paths)
```

> custom_module()関数は、datasets()関数と同じ引数を取るようにします。custom_module()関数を呼ぶ際に、dataset()が受け取った引数をそのまま渡しています。

実行結果は、同じものとなります。

```bash
(tenv) $ python main.py
Hello RDE!
```

### dataset()の引数

前述の様に、dataset()関数は、引数を1つ取ります。

* paths: RdeDatasetPaths 

pathsには、RDEToolKitにより自動的に適切なパス情報がセットされますので、開発者はこれらを使ってパス情報を組み立てる必要があります。

pythonにて標準提供されている`pprint`を使って内容を表示してみます。


```python
from pprint import pprint

from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.exceptions import StructuredError
from rdetoolkit.models.rde2types import RdeDatasetPaths


def custom_module(paths: RdeDatasetPaths) -> None:
    print("Hello RDE!")
    pprint(paths)

@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(paths: RdeDatasetPaths) -> None:
    custom_module(paths)
```

上記の様にして実行すると以下の様になります。

```bash
(tenv) $ python main.py
Hello RDE!
RdeDatasetPaths(input_paths=RdeInputDirPaths(inputdata=PosixPath('data/inputdata'),
                                             invoice=PosixPath('data/invoice'),
                                             tasksupport=PosixPath('data/tasksupport'),
                                             config=Config(system=SystemSettings(extended_mode=None, save_raw=False, save_nonshared_raw=True, save_thumbnail_image=False, magic_variable=False, save_invoice_to_structured=False, feature_description=True), multidata_tile=MultiDataTileSettings(ignore_errors=False), smarttable=None, traceback=None)),
                output_paths=RdeOutputResourcePath(raw=PosixPath('data/raw'),
                                                   nonshared_raw=PosixPath('data/nonshared_raw'),
                                                   rawfiles=(PosixPath('data/inputdata/sample.data'),),
                                                   struct=PosixPath('data/structured'),
                                                   main_image=PosixPath('data/main_image'),
                                                   other_image=PosixPath('data/other_image'),
                                                   meta=PosixPath('data/meta'),
                                                   thumbnail=PosixPath('data/thumbnail'),
                                                   logs=PosixPath('data/logs'),
                                                   invoice=PosixPath('data/invoice'),
                                                   invoice_schema_json=PosixPath('data/tasksupport/invoice.schema.json'),
                                                   invoice_org=PosixPath('data/invoice/invoice.json'),
                                                   smarttable_rowfile=None,
                                                   smarttable_row_data=None,
                                                   temp=PosixPath('data/temp'),
                                                   invoice_patch=PosixPath('data/invoice_patch'),
                                                   attachment=PosixPath('data/attachment')))
```

> 上記に含まれるの出力用のフォルダはRDEToolKitにより自動的に作成されます。RDEの想定しているフォルダ名と異なるフォルダ名とならないように、構造化処理プログラム中ではの"明示的な"フォルダの生成は行わずに、上記フォルダを使用するようにしてください。一時的な作業用フォルダなど、RDEに格納することを目的としないフォルダ名はこの限りではありません。

以上をまとめると、以下の様になります。

| # | 変数名 | 意味 | 備考 |
| ---- | ---- | ---- | ---- |
|  1 | paths.input_paths.inputdata | 入力ファイルのフォルダ | |
|  2 | paths.input_paths.invoice | invoice.jsonが格納されているフォルダ | |
|  3 | paths.input_paths.tasksupport | tasksupportファイル群の格納されているフォルダ | |
|  4 | paths.input_paths.config | 設定したコンフィグ内容 | |
|  5 | paths.output_paths.raw | raw出力フォルダ | 共有 生データの格納用フォルダ |
|  6 | paths.output_paths.nonshared_raw | nonshared_raw出力フォルダ | 非共有 生データ格納用のフォルダ |
|  7 | paths.output_paths.rawfiles | **入力された**ファイルを示す"配列" |  |
|  8 | paths.output_paths.struct | structured出力フォルダ |  |
|  9 | paths.output_paths.main_image | main_image出力フォルダ |  |
| 10 | paths.output_paths.other_image | other_image出力フォルダ |  |
| 11 | paths.output_paths.meta | metadata.json出力フォルダ |  |
| 12 | paths.output_paths.thumbnail | thumbnail出力フォルダ|
| 13 | paths.output_paths.logs | ログ出力フォルダ |  |
| 14 | paths.output_paths.invoice | invoice.json出力フォルダ | 入力フォルダと同じ |
| 15 | paths.output_paths.invoice_schema_json | invoice.schema.jsonファイル |  |
| 16 | paths.output_paths.invoice_org | invoice.json入力ファイル |  |
| 17 | paths.output_paths.smarttable_rowfile | 処理対象invoiceファイル(SmartTableインボイスモード利用時) |  |
| 18 | paths.output_paths.smarttable_row_data | 解釈後のinvoice情報(辞書形式、SmartTableインボイスモード利用時) |  |
| 19 | paths.output_paths.temp | 一時ファイル出力フォルダ |  |
| 20 | paths.output_paths.invoice_patch | invoice_patchフォルダ |  |
| 21 | paths.output_paths.attachment | attachment出力フォルダ | データ登録時に指定されなかった場合はNone |

フォルダを意味するPathオブジェクトを利用してファイルを指定するためには、joinpath()を使うか、"/ (ファイル名)"を使います。

例1)

```python
invoice_file = paths.input_paths.invoice.joinpath("invoice.json")
```

例2)

```python
invoice_file = paths.input_paths.invoice / "invoice.json"
```

> 上記例1と例2は、invoice.jsonファイルを示すPathオブジェクトです。どちらを指定しても同じ意味です。本書では主に後者を用いることにします。

ただし、`paths.input_paths.inputdata`と`paths.input_paths.invoice`の利用には注意が必要です。Excelインボイス方式など複数のファイルを一度に処理するモードや、複数のインボイスをExcel形式などで一度に登録して利用する際は、入力ファイルとして後述する`paths.output_paths.rawfiles`を、invoiceファイルが格納されているフォルダとして`paths.output_paths.invoice`を使う必要があります。

> (通常の)インボイスモードだけを利用する場合は、`paths.input_paths.invoice`を利用しても問題ありませんが、その場合Excelインボイス利用時にソース改変が必要となります。通常のインボイスモード利用時とExcelインボイス利用時でソース改変が不要になるよう、本書ではpaths.output_paths.invoiceを利用して必要な情報を取得しています。

`paths.output_paths`には、主に出力に関するフォルダ、ファイルを示すPathオブジェクトが格納されていますが、一部は入力関連のPathオブジェクトも格納されていますので注意してください。

例1) (生成した)サムネイル画像を格納するフォルダ

```python
thumbnail_path =  paths.output_paths.thumbnail
```

例2) (複数の)入力ファイルを表示

```python
input_files = paths.output_paths.rawfiles
for input_file in input_files: 
    print(input_file)
```

> `paths.output_paths.rawfiles`は、入力ファイルが1個しかない場合でも、"配列"としてセットされます。

例3) 入力ファイルのうち、拡張子が".csv"のものを表示

```python
for csv_file in paths.input_paths.inputdata.glob("*.csv"):
    print(f"Input CSV: {csv_file}")
```

なお、多くの場合、変数中の`.input_path`と`.output_path`は省略可能です。

つまり、以下の2つは同じ意味となります。

```python
print(paths.input_paths.inputdata)
```

```python
print(paths.inputdata)
```

* `invoice`に関しては、input_paths,output_pathsの両方にありますが、`paths.invoice`は、`paths.output_paths.invoice`を意味します。

明示的なプロパティ(変数)としてはinput_paths,output_paths双方に存在していませんが、`metadata-def.json`ファイルへのPathオブジェクトは以下の様に求めることができます。

```python
metadata_def = paths.metadata_def_json
```

これは以下の様にしても同じです。

```python
metadata_def = paths.tasksupport.joinpath('metadata-def.json')
```
