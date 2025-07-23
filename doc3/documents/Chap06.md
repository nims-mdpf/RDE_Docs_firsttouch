<div class="page" />

# 入力ファイルのチェック

最初に、入力ファイル、つまり登録されたデータのチェックを行います。

## 想定

入力ファイルは様々なパターンが考えられますが、本書では、以下を想定します。

* "inputdata"ディレクトリ内に、ファイルが1個だけ存在している。
* 入力ファイルの拡張子は".data"である。

> どんな確認処理が必要かについては、それぞれのプロジェクト/データセットテンプレート毎に吟味する必要があります。

これらを順に確認し、条件に合致しない場合は、エラーとして処理します。

modules/datasets_process.pyは、以下の状態になっているものとして、ここからコードを修正していきます。

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.exceptions import StructuredError
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    print("Hello RDE!")

@catch_exception_with_message(error_message="ERROR: failed in data processing", error_code=50)
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    custom_module(srcpaths, resource_paths)
```

## custom_module()にチェックロジックを実装

custom_module()内に、入力ファイルのチェックロジックを追加していきます。

```python
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    # Check input file
    input_dir = srcpaths.inputdata
    input_files = list(input_dir.glob("*"))
    if len(input_files) == 0:
        raise StructuredError("ERROR: input data not found")
    
    if len(input_files) > 1:
        raise StructuredError("ERROR: input data should be one file")
    
    raw_file_path = input_files[0]
    if raw_file_path.suffix.lower() != ".data":
        raise StructuredError(
            f"ERROR: input file is not '*.data' : {raw_file_path.name}"
        )
```

> 前の処理で追加していた`print("Hello RDE!")`は不要ですので、削除します。

1. 1つ目のif文で、inputdata/フォルダに存在するファイルの個数が0かどうかをチェックし、0だったらエラーとして処理します。
2. 次のif文で、同ファイル個数が1より多いかどうかをチェックし、多かったらエラーとして処理します。
3. その後、ファイルが1個だけ存在することが確定していますので、そのファイルの拡張子が".data"かどうかをチェックし、そうでない場合はエラーとして処理します。

## 確認

ダミーファイルを置き、data/inpudataフォルダにファイルが"2個"存在する状態にし、エラーとなるかを確認します。

```bash
(tenv) $ touch data/inputdata/dummy.txt

(tenv) $ ls data/inputdata
dummy.txt  sample.data
```

この状態でmain.pyを実行すると以下の様になります。

```bash
(tenv) $ python main.py

Traceback (simplified message):
Call Path:
   File: /home/devel/tenv/lib/python3.11/site-packages/rdetoolkit/errors.py, Line: 43 in wrapper()
    └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 24 in dataset()
        └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 14 in custom_module()
            └─> L14: raise StructuredError("ERROR: input data should be one file") 🔥

Exception Type: StructuredError
Error: ERROR: input data should be one file
```

入力ファイルが1個でない場合のエラーメッセージ"input data should be one file"が出力されていることが確認できます。

> Tracebackに出力されるファイル名や行番号は、RDEToolKitのバージョンが変わると、リファクタリングやその他の理由で変更されることがあります。

`data/job.failed`の内容を確認してみます。

```bash
(tenv) $ cat data/job.failed
ErrorCode=1
ErrorMessage=Error: ERROR: input data should be one file
```

画面に出たエラーメッセージと同じものが格納されていることが分かります。

> このjob.failedのErrorMessage句にセットされた内容が、RDEのWeb画面の登録結果画面に表示されます。
>
> `data/job.failed`に画面表示と同じメッセージがセットされることが確認できましたので、以降は`data/job.failed`の確認は省略します。

次のチェックの確認に行く前に、上で作成したダミーファイルを消します。

```bash 
rm data/inputdata/dummy.txt
```

続いて、"data/inputdata/"フォルダにファイルが無い状態でどうなるかを確認します。

ファイルを消してもよいですが、あとでまた使いますのでもとに戻せるようにフォルダをリネームして、新規に同名のフォルダを作成することで、入力ファイルが存在しない状態を作ります。

```bash
mv data/inputdata/ data/inputdata.org
mkdir data/inputdata/
```

構造化処理プログラムを実行します。

```bash
(tenv) $ python main.py

Traceback (simplified message):
Call Path:
   File: /home/devel/tenv/lib/python3.11/site-packages/rdetoolkit/errors.py, Line: 43 in wrapper()
    └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 24 in dataset()
        └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 11 in custom_module()
            └─> L11: raise StructuredError("ERROR: input data not found") 🔥

Exception Type: StructuredError
Error: ERROR: input data not found
```

想定のエラーメッセージが出力されることが確認できました。

元に戻して、今度は入力ファイルの拡張子を".data"から".dat"に変更して、構造化処理プログラムを実行してみます。

```bash
(tenv) $ rm -rf data/inputdata
(tenv) $ mv data/inputdata.org/ data/inputdata
(tenv) $ mv data/inputdata/sample.data data/inputdata/sample.dat

(tenv) $ python main.py

Traceback (simplified message):
Call Path:
   File: /home/devel/tenv/lib/python3.11/site-packages/rdetoolkit/errors.py, Line: 43 in wrapper()
    └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 24 in dataset()
        └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 18 in custom_module()
            └─> L18: raise StructuredError( 🔥

Exception Type: StructuredError
Error: ERROR: input file is not '*.data' : sample.dat
```

エラーメッセージから、入力ファイルの拡張子が想定通りでないことを検知し、異常終了したことが分かります。

以上で、入力ファイルのチェック処理がうまく実施されました。

以降の処理のために元に戻します。

```bash
mv data/inputdata/sample.dat data/inputdata/sample.data
```

前章で作成した`reinit.sh`を実行し、main.pyが正常に終了することを確認します。

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

(tenv) $ python main.py
(tenv) $ echo $?
0
```

> 何かを表示する(→標準出力に出力する)処理を実装していませんので、この時点では(プログレスバー以外は)何も表示されません。
