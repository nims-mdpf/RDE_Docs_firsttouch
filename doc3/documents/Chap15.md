<div class="page" />

# 応用編2

"応用編2"として、別の実装例を示します。

こちらは、以下の様の実装例となっています。

* 入力ファイルごとに処理クラスとして記述する。
* 設定ファイルとして、`pyptoject.toml`を使用する。
* 実行ログを、コンソール画面上、またはユーザ用のログファイルに出力する。

## 準備

応用編と同じように準備していきます。

### RDEToolKit

初級編、応用編で使用したRDEToolKitをそのまま利用します。従ってPython仮想環境の"tenv"をそのまま利用します。

仮想環境を有効にしていない場合は有効にします。

```bash
$ source tenv/bin/activate
(tenv) $
```

### フォルダ構成

応用編2では"$HOME/handson/advanced2"にファイルを設置することとします。

フォルダを作成し、RDEToolKitの初期化を実施します。

```bash
(tenv) $ cd $HOME/handson

(tenv) $ mkdir advanced2
(tenv) $ cd advanced2

(tenv) $ python -m rdetoolkit init
Ready to develop a structured program for RDE.
Created: /home/devel/advanced2/container/requirements.txt
Created: /home/devel/advanced2/container/Dockerfile
Created: /home/devel/advanced2/container/data/invoice/invoice.json
Created: /home/devel/advanced2/container/data/tasksupport/invoice.schema.json
Created: /home/devel/advanced2/container/data/tasksupport/metadata-def.json
Created: /home/devel/advanced2/templates/tasksupport/invoice.schema.json
Created: /home/devel/advanced2/templates/tasksupport/metadata-def.json
Created: /home/devel/advanced2/input/invoice/invoice.json

Check the folder: /home/devel/advanced2
Done!
```

> 他と同様、`devel`ユーザとして実行した場合の例となります。

### 入力ファイル

入力ファイルとしてのsample.data、invoice.json、invoice.schema.json および matadata-def.jsonは、初級編と同じものを使うので、コピーしておきます。

> 初級編と同様、本テキストからのコピー&ペーストにより作成しても構いません。

```bash
(tenv) $ cd container/

(tenv) $ pwd
/home/devel/handson/advanced2/container

(tenv) $ cp $HOME/handson/tutorial/container/data/inputdata/sample.data data/inputdata/
(tenv) $ cp $HOME/handson/tutorial/container/data/invoice/invoice.json data/invoice/
(tenv) $ cp $HOME/handson/tutorial/container/data/tasksupport/invoice.schema.json data/tasksupport/
(tenv) $ cp $HOME/handson/tutorial/container/data/tasksupport/metadata-def.json data/tasksupport/
```

### 初期化用スクリプト

初期化用スクリプトは初級編で使用したものと同じものを利用するのでコピーします。

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

### invoice.json

invoice.jsonの"basic" → "dataName"に"/ (2024)"を付加したままかどうかを確認し、付加されている場合は修正します。

data/invoice/invoice.json

変更前

```
        "dataName": "NIMS_TRIAL_20230126b / (2024)",
```

変更後

```
        "dataName": "NIMS_TRIAL_20230126b",
```

<div class="page" />

## 実装

> 処理内容はいままでの例と同じですので、コードのみ示します。また今までと同じ処理であっても、"こういう書き方もできる"ということを示すために、少し変更したバージョンとなっているものもあります。軽微な修正の場合は特にコメントを記述しない場合があります。

main.py

```python
from rdetoolkit import workflows

from modules.datasets_process import dataset


def main():
    workflows.run(
        custom_dataset_function=dataset
    )

if __name__ == '__main__':
    main()
```

### 設定ファイル

本章のはじめに示したように`pyproject.toml`を用意して設定ファイルとします。

> 設置場所は、`data/`フォルダと同じフォルダ(つまり通常は`container/`フォルダ)下に設置します。

pyproject.toml

```toml
[tool.rdetoolkit.system]
save_raw = false
save_nonshared_raw = false
```

> `multidata_tile`にデータを設定する場合は、`[tool.rdetoolkit.multidata_tile]`節に設定します。その他独自設定を追加する場合は`[tool.rdetoolkit.(独自設定名)]`、具体的には`[tool.rdetoolkit.user]`のような節に設定します。独自設定項目は、他の方法で設定した場合と同様、スクリプト内ではdict形式で利用可能です。
>
> tomlファイルでの真偽値は、小文字(`true` または `false`)で指定します。

### ログ出力

RDEToolKitは、RDEToolKitが行った`作業`を、`data/logs/rdesys.log`ファイルにログ出力します。
ここでは、上記とは別に、独自のログファイル出力を実装します。

RDEToolKit v1.1.0から、ユーザログ出力用の専用クラスが実装されましたので、それを使うことにします。

> ユーザログは`data/logs/rdeuser.log`への書き込みと、画面への出力の片方、または双方を実装時に選択することができます。

利用するには、datasets_process.pyなどで、以下のような設定を行います。

modules/datasets_process.py

```python
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
from rdetoolkit.rdelogger import CustomLog, log_decorator


logger = CustomLog().get_logger()

@log_decorator()
def Huga():
    pass

@log_decorator()
def Hoge():
    logger.debug('Sample log!')

def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    Hoge()
    Huga()
```

関数の定義文(def文)にデコレートする(上部に`@log_decorator()`を前置する)ことで、デコレートした関数の実行のはじめと終わりにログを出力するようになります。

また関数内部で、取得した`logger`に対してdebug()やwarning()関数を実行することで、任意の場所で任意のメッセージを出力することができます。

実行すると、以下のようになります。

```bash
(tenv) $ python main.py 
2025-09-08 11:14:29 +0900 (INFO) Hoge            --> Start
2025-09-08 11:14:29 +0900 (DEBUG) Sample log!
2025-09-08 11:14:29 +0900 (INFO) Hoge            <-- End
2025-09-08 11:14:29 +0900 (INFO) Huga            --> Start
2025-09-08 11:14:29 +0900 (INFO) Huga            <-- End
```

同時にログファイル(data/logs/rdeuser.log)にも同じ内容がセットされます。

```bash
(tenv) $ cat data/logs/rdeuser.log 
2025-09-08 11:14:29 +0900 (INFO) Hoge            --> Start
2025-09-08 11:14:29 +0900 (DEBUG) Sample log!
2025-09-08 11:14:29 +0900 (INFO) Hoge            <-- End
2025-09-08 11:14:29 +0900 (INFO) Huga            --> Start
2025-09-08 11:14:29 +0900 (INFO) Huga            <-- End
```

開発が終了し、ログ出力が不要となった場合は、datasets_process.pyの上の方でセットした`get_logger()`の引数として`False`を与えればログ表示、および書き込みがなくなります。

変更前 (ログ表示あり)

```python
logger = CustomLog().get_logger()
```

変更後 (ログ表示無し)

```python
logger = CustomLog().get_logger(False)
```

> get_logger()の引数にFalseを指定するだけです。ロジック内のソースを変更する必要はありません。
>
> ログ出力が抑止されるのは、ユーザログ(`data/logs/rdeuser.log`)のみです。システムログ(`data/logs/rdesys.log`)は引き続き出力されます。

```bash
(tenv) $ python main.py 
(tenv) $ 
```

> 画面(→標準出力)には、何も表示されなくなりました。
>
> なお、ログファイル(→data/logs/rdeuser.log)に追記はされませんが、ファイルのクリア(0バイトのファイルとして更新)や削除は行われません。ファイルのクリアや削除を実施したい場合は、別途処理(`reinit.sh`など)を実施してください。

```bash
$ ./reinit.sh 
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

再度構造化処理プログラムを実行し、ログ出力がされていないことを確認します。

```bash
(tenv) $ python main.py 

(tenv) $ ls -l data/logs/rdeuser.log
合計 0
```

> ログファイルが存在していないことが確認できます。ただしこの場合でも、システムログ(`data/logs/rdesys.log`)は出力されます。

### それ以外の実装

ユーザログ出力が確認できたので、前述のように、前章と同様の処理を実装していきます。

> 上記でテストした`modules/datasets_process.py`がある場合は、下記内容で上書きしてください。

また、前述の様に、前章とはクラス構成を変更しています。こういう実装もある、という参考にしてください。

modules/datasets_process.py

```python
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
from rdetoolkit.errors import catch_exception_with_message

from modules.custom_process import CustomProcess


@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    # Create Instance of Custom Process
    custom_process = CustomProcess(srcpaths, resource_paths)

    # Do some ...
    custom_process.check_input()
    custom_process.parse_invoice()
    custom_process.backup_invoice_file("invoice.json.orig")
    custom_process.update_invoice()
    custom_process.parse_input()
    custom_process.make_meta()
    custom_process.make_struct()
    custom_process.make_graph()
    custom_process.make_thumbnail()
    custom_process.copy_raw_files()
```

modules/StructuredProcessBase.py

```python
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
from rdetoolkit.exceptions import StructuredError


class StructuredProcessBase:
    def __init__(self, srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath):
        self.srcpaths = srcpaths
        self.resource_paths = resource_paths

    def check_input(self):
        raise NotImplementedError

    def parse_invoice(self):
        raise NotImplementedError

    def update_invoice(self, newInvoiceDict = None):
        raise NotImplementedError

    def copy_raw_files(self):
        raise NotImplementedError

    def make_meta(self):
        raise NotImplementedError

    def make_struct(self):
        raise NotImplementedError

    def make_graph(self):
        raise NotImplementedError

    def make_thumbnail(self):
        raise NotImplementedError
```

modules/custom_process.py

```python
import io
import statistics as st
import shutil

import pandas as pd
import matplotlib.pyplot as plt
from PIL import Image

from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
from rdetoolkit.errors import StructuredError
from rdetoolkit.fileops import readf_json, writef_json
from rdetoolkit.rde2util import Meta
from rdetoolkit.rdelogger import CustomLog, log_decorator

from modules.StructuredProcessBase import StructuredProcessBase


logger = CustomLog().get_logger()

class CustomProcess(StructuredProcessBase):
    additional_title = "(2024)"

    @log_decorator()
    def __init__(self, srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath):
        super().__init__(srcpaths, resource_paths)
        self.invoice_file = resource_paths.invoice.joinpath('invoice.json')
        self.data_df = None
        self.const_meta_info = dict()
        self.repeated_meta_info = dict()
        self.is_private_row = None

    @log_decorator()
    def check_input(self) -> bool:
        # Check input file
        input_files = self.resource_paths.rawfiles
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

    @log_decorator()
    def parse_input(self) -> None:
        # Read input data
        DELIM = "="
        raw_data_df = None
        raw_meta_obj = None
        #
        input_file = self.resource_paths.rawfiles[0]

        with open(input_file) as f:
            lines = f.readlines()
        # omit new line codes (\r and \n)
        lines_strip = [line.strip() for line in lines]

        meta_row = [i for i, line in enumerate(lines_strip) if "[METADATA]" in line]
        data_row = [i for i, line in enumerate(lines_strip) if "[DATA]" in line]
        if (meta_row != []) & (data_row != []):
            meta = lines_strip[(meta_row[0]+1):data_row[0]]
            # metadata to dict
            raw_meta_obj = dict(map(lambda x: tuple([x.split(DELIM)[0],
                        DELIM.join(x.split(DELIM)[1:])]), meta))
        else:
            raise StructuredError("ERROR: invalid RAW METADATA or DATA")

        # read data to data.frame
        if (int(raw_meta_obj["series_number"]) != len(data_row)):
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
        self.const_meta_info = self.const_meta_info | raw_meta_obj
        self.data_df = raw_data_df

    @log_decorator()
    def parse_invoice(self) -> None:
        self.invoice_dict = readf_json(self.invoice_file)
        self.is_private_raw = self._is_private_raw()

    def _is_private_raw(self) -> bool:
        invoice_dict = self.invoice_dict
        # Check private or not
        custom = invoice_dict.get("custom")
        if custom is None:
            return False
        is_private_raw = custom.get("is_private_raw")
        if is_private_raw == "share":
            return False
        # other case -> "private"
        return True

    @log_decorator()
    def update_invoice(self, new_invoice_dict = None) -> None:
        if new_invoice_dict is not None:
            self.invoice_dict = new_invoice_dict
        else:
            self._update_invoice()
        # overwrite invoice.json
        self._overwrite_invoice_file()

    def _update_invoice(self) -> None:
        # Update invoice title
        original_data_name = self.invoice_dict["basic"]["dataName"]
        additional_title = self.additional_title
        if original_data_name.find(additional_title) < 0:
            # update title if not applied yet
            self.invoice_dict["basic"]["dataName"] = \
                original_data_name + " / " + additional_title

    def _overwrite_invoice_file(self) -> None:
        # Overwrite Invoice file
        invoice_file_new = self.invoice_file
        writef_json(invoice_file_new, self.invoice_dict)

    def backup_invoice_file(self, backup_file = None) -> None:
        # Backup(=Copy) invoice.json to shared/nonshared folder
        is_private_raw = self.is_private_raw
        if is_private_raw:
            raw_dir = self.resource_paths.nonshared_raw
        else:
            raw_dir = self.resource_paths.raw
        if backup_file is None:
            backup_file = "invoice_backup.json"
        invoice_file_new = raw_dir.joinpath(backup_file)
        writef_json(invoice_file_new, self.invoice_dict)

    @log_decorator()
    def copy_raw_files(self) -> None:
        # Copy inputdata to public (raw/) or non_public (nonshared_raw/)
        is_private_raw = self.is_private_raw
        raw_dir = self.resource_paths.nonshared_raw if is_private_raw else self.resource_paths.raw
        for input_file in self.resource_paths.rawfiles:
            shutil.copy(input_file, raw_dir)

    @log_decorator()
    def make_meta(self) -> None:
        # create meta and save meta
        self._parse_from_invoice()
        self._parse_from_inputdata()
        save_path = self.resource_paths.meta.joinpath("metadata.json")
        self._save_meta(save_path)

    def _parse_from_invoice(self) -> None:
        # Merge invoice info to meta
        if self.invoice_dict["custom"] is not None:
            self.const_meta_info = self.const_meta_info | self.invoice_dict["custom"]

    def _parse_from_inputdata(self) -> None:
        data = self.data_df

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

        meta_vars = {
            "series_name": s_name,
            "series_data_count": s_count,
            "series_data_mean": s_mean,
            "series_data_median": s_median,
            "series_data_max": s_max,
            "series_data_min": s_min,
            "series_data_stdev": s_stdev,
        }

        # Set variable meta
        self.repeated_meta_info = meta_vars

    def _save_meta(self, save_path) -> None:
        # Save metadata to file
        metadef_filepath = self.srcpaths.tasksupport.joinpath("metadata-def.json")
        metaobj = Meta(metadef_filepath)
        metaobj.assign_vals(self.const_meta_info)
        metaobj.assign_vals(self.repeated_meta_info)

        metaobj.writefile(save_path)


    @log_decorator()
    def make_struct(self) -> None:
        # Write CSV file(s)
        for d in self.data_df:
            fname = d.columns[1].replace(" ", "") + ".csv"
            csv_file_path = self.resource_paths.struct.joinpath(fname)
            d.to_csv(csv_file_path, header=True, index=False)

    @log_decorator()
    def make_graph(self,title = None, xlabel = None , ylabel = None) -> None:
        # Write Graph
        title = title if title is not None else "No title"
        xlabel = xlabel if xlabel is not None else "X-label"
        ylabel = ylabel if ylabel is not None else "Y-label"

        ## by series
        for d in self.data_df:
            x = d.iloc[:, 0]
            y = d.iloc[:, 1]
            label = d.columns[1]
            fname = self.resource_paths.other_image.joinpath(f"{label}.png")
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
        for d in self.data_df:
            x = d.iloc[:, 0]
            y = d.iloc[:, 1]
            label = d.columns[1]
            ax.plot(x, y, label=label)
        ax.legend()
        fname = self.resource_paths.main_image.joinpath("all_series.png")
        fig.savefig(fname)
        plt.close(fig)

    @log_decorator()
    def make_thumbnail(self) -> None:
        # Write thumbnail images
        src_path = self.resource_paths.main_image
        src_img_file_path = src_path.joinpath("all_series.png")
        save_path = self.resource_paths.thumbnail
        out_img_file_path = save_path.joinpath("thumbnail.png")

        # Size of thumbnail
        closure_w = 286
        closure_h = 200

        Image.MAX_IMAGE_PIXELS = None
        img_org = Image.open(src_img_file_path)
        ratio_w = closure_w / img_org.width
        ratio_h = closure_h / img_org.height
        ratio = min(ratio_w, ratio_h)
        img_re = img_org.resize(
            (int(img_org.width * ratio), int(img_org.height * ratio)),
            Image.BILINEAR
        )
        img_re.save(out_img_file_path)
```

> サムネイル画像作成処理は、応用編1で示したresize_image()関数を使う方法もあります。

実行してみると、以下の様になります。

```bash
(tenv) $ ./reinit.sh
:
(tenv) $ python main.py
2025-09-08 11:49:50 +0900 (INFO) __init__        --> Start
2025-09-08 11:49:50 +0900 (INFO) __init__        <-- End
2025-09-08 11:49:50 +0900 (INFO) check_input     --> Start
2025-09-08 11:49:50 +0900 (INFO) check_input     <-- End
2025-09-08 11:49:50 +0900 (INFO) parse_invoice   --> Start
2025-09-08 11:49:50 +0900 (INFO) parse_invoice   <-- End
2025-09-08 11:49:50 +0900 (INFO) update_invoice  --> Start
2025-09-08 11:49:50 +0900 (INFO) update_invoice  <-- End
2025-09-08 11:49:50 +0900 (INFO) parse_input     --> Start
2025-09-08 11:49:50 +0900 (INFO) parse_input     <-- End
2025-09-08 11:49:50 +0900 (INFO) make_meta       --> Start
2025-09-08 11:49:50 +0900 (INFO) make_meta       <-- End
2025-09-08 11:49:50 +0900 (INFO) make_struct     --> Start
2025-09-08 11:49:50 +0900 (INFO) make_struct     <-- End
2025-09-08 11:49:50 +0900 (INFO) make_graph      --> Start
2025-09-08 11:49:50 +0900 (INFO) make_graph      <-- End
2025-09-08 11:49:50 +0900 (INFO) make_thumbnail  --> Start
2025-09-08 11:49:50 +0900 (INFO) make_thumbnail  <-- End
2025-09-08 11:49:50 +0900 (INFO) copy_raw_files  --> Start
2025-09-08 11:49:50 +0900 (INFO) copy_raw_files  <-- End
```

どういったファイルが生成されているかの確認をします。

```bash
(tenv) $ tree data
data
├── attachment
├── inputdata
│   └── sample.data
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   ├── rdesys.log
│   └── rdeuser.log
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

15 directories, 17 files
```

> 初級編、応用編1と同じ結果となっています。

### インボイスの値によって処理を変える

さらなる応用例として、以下を想定した実装を考えてみます。

* 上記をベースの処理とする。
* invoice.jsonの"custom"->"data_type"をいう項目を新設する。invoice.schema.jsonも合わせて修正する。
* "data_type"の値が"base"なら、いままで通り"dataName"に"/ (2024)"を付加する。"variant"なら"/ (2025)"を付加する。

#### invoice.schema.json

data/tasksupport/invoice.schema.json

```json
：
    "properties": {
        "custom": {
            "type": "object",
            "label": {
                "ja": "固有情報",
                "en": "Custom Information"
            },
            "required": [
                "data_type"
            ],
            "properties": {
                "data_type": {
                    "label": {
                        "ja": "データタイプ",
                        "en": "data_type"
                    },
                    "type": "string"
                },
                "measurement_date": {
                    "label": {
                        "ja": "計測年月日",
                        "en": "measurement_date"
                    },
：
```

> "required"にdata_typeを追加し、その項目のラベルやtypeの設定を追加します。

#### invoice.json

data/invoice/invoice.json

```json
:
    "custom": {
        "data_type": "variant",
        "measurement_date": "2023-01-25",
        "invoice_string1": "送状文字入力値1",
        "invoice_string2": null,
        "is_devided": "devided",
        "is_private_raw": "private"
    },
:
```

> `"data_type": "variant",`の行を追加します。

#### custom_process2.py

続いて、「タイトルに"/ (2024)"を付加する」処理を「タイトルに"/ (2025)"を付加する」処理に変更したクラスを新規に作成します。

modules/custom_process2.py

```python
from modules.custom_process import CustomProcess


class CustomProcess2(CustomProcess):
    additional_title = "(2025)"
```

> CustomProcessを継承しているため、CustomProcessクラスからの変更点のみの記述となっていることに注意してください。つまり、CustomProcessから変更のない処理についての記述は不要です。今回の例では、追加するテキストが変更になる("(2024)" → "(2025)")だけですのでクラス変数部分の指定だけで済みます。
>
> もちろん他の方法での実装(初期化時の引数として、`additional_title`の内容を指定する、など)もあります。上記は1つの実装例として考えてください。

#### datasets_process.py

条件を判定して、上記で作成したクラスを使う処理を追加します。

modules/datasets_process.py

```python
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.invoicefile import InvoiceFile

from modules.custom_process import CustomProcess
from modules.custom_process2 import CustomProcess2


@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    # Parse invoice
    invoice_file = resource_paths.invoice.joinpath("invoice.json")
    invoice = InvoiceFile(invoice_file).invoice_obj

    ## Create Instance of Custom Process
    if invoice["custom"]["data_type"] == 'variant':
        custom_process = CustomProcess2(srcpaths, resource_paths)
    else:
        custom_process = CustomProcess(srcpaths, resource_paths)

    # Do some ...
    custom_process.check_input()
    custom_process.parse_invoice()
    custom_process.backup_invoice_file("invoice.json.orig")
    custom_process.update_invoice()
    custom_process.parse_input()
    custom_process.make_meta()
    custom_process.make_struct()
    custom_process.make_graph()
    custom_process.make_thumbnail()
    custom_process.copy_raw_files()
```

> invoiceインスタンスを作成の後、"Create Instance ……"部分を`invoiceで指定された値に応じて処理を変更する`ように変更します。同時にCustomProcess2のimportを忘れずに追加してください。

#### 実行確認

"data_type"が"variant"であることを確認し、実行します。

実行前に、invoice.jsonの内容を確認します。、

```bash
(tenv) $ grep data_type data/invoice/invoice.json
        "data_type": "variant",
(tenv) $ grep Name data/invoice/invoice.json
        "dataName": "NIMS_TRIAL_20230126b",
```

> "/ (2024)"が付加されている場合は、エディタを使い取り除いてください。

```bash
(tenv) $ python main.py
:
(tenv) $ grep Name data/invoice/invoice.json
        "dataName": "NIMS_TRIAL_20230126b / (2025)",
```

> "dataName"に"/ (2025)"が付加されていることがわかります。

上記が確認できたら、次にエディタを使い、"/ (2025)"を取り除き、"data_type"の値を"base"に変更します。

```bash
(tenv) $ vi data/invoice/invoice.json

(tenv) $ grep data_type data/invoice/invoice.json
        "data_type": "base",
```

実行します。

```bash
(tenv) $ grep Name data/invoice/invoice.json
        "dataName": "NIMS_TRIAL_20230126b",

(tenv) $ python main.py
:
(tenv) $ grep Name data/invoice/invoice.json
        "dataName": "NIMS_TRIAL_20230126b / (2024)",
```

想定通り、"dataName"には"/ (2024)"が付加されています。

> 上記ソースでは、`dataName`項目にすでに`/ (2024)`が含まれている場合でも、`/ (2025)`を付加してしまう不具合があります。サンプルプログラムということで、ここでは対処しないことにします。
