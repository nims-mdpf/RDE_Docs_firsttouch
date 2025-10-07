<div class="page" />

# invoice.jsonの読み込みと保存

構造化処理プログラムが起動する時、データ登録者が生データファイルのアップロードと一緒にWeb画面から入力した情報は、RDEシステムにより送り状(Invoice)データ、つまり`data/invoice/invoice.json`ファイルとして生成され、生データファイルと共に提供されます。構造化処理プログラムは、これらのファイルを入力として処理を実行し、既定の出力ファイルを属性ごとにフォルダに出力します。

Invoiceデータを「ただ登録する」だけであれば、RDE構造化処理プログラムの中で読み込むことは必要はありません。

しかし、RDE構造化処理の内容によっては、プログラム中でInvoiceの情報を読み込んで、メタデータとして転記、グラフ作成処理時にパラメータとして使用するなど、Invoiceの情報を利用したい場合もあります。

送り状(Invoice)のデータを利用したい場合の例として以下の様な場合があります。

* グラフのXY 軸の設定（描画範囲）を指定したい
* 解析処理として`ガウス分布にフィッティングする処理を行う`が、そのとき使用する`ピーク数`を入力データ毎に指定したい

これらのように、事前にRDE構造化処理プログラムの中では定義できず、ユーザが実験データを登録時に自由に決めたいような場合です。

あるいは、Invoiceの内容を生データファイルの値によって書き換えたいという要望もあるかもしれません。

そういった場合に、構造化処理プログラムの初期段階で`invoice.json` を読み込んでおくと扱いやすくなります。

## 想定

本書では、以下のような処理が必要である、と想定することにします。

* 通常の構造化処理プログラムは、アップロードされたファイルの公開/非公開、つまり"生データファイル"の保存先を、`raw/`(→公開の場合)または`nonshared_raw/`(→非公開の場合)のいずれのフォルダにするのかが、データセット単位で決まっていることが多い。本ハンズオンではinvoice.json内で指定されたある項目を、アップロードされたファイルを公開/非公開のいずれのフォルダにするのかの判断に利用することにする。
  * 本章では、判断に利用するための変数をセットするところまでを実装する。入力ファイルを、公開または非公開フォルダにコピーする処理(→ 実験データの永続化)は、後述する。
  * 本章ではこの変数を、改変前のinvoice.jsonをバックアップとして保存するフォルダ名を決めるために使用する方法を、例として示す。
* RDE構造化処理プログラムの中でinvoice.jsonの一部を変更する。具体的には、タイトル部に'/ (2024)'と言う文字列を付加する。新しいinvoice.jsonは、もとのファイルを上書きする。

> 実際の構造化処理プログラムでは、タイトル部に"(2024)"という文字列を付与するような処理は行わないと思われますが、「こういうことも出来る」という例として実装します。

## custom_module()に読み込みロジックを実装

modules/datasets_process.py内に、以下の処理を追加します。

```python
:
from rdetoolkit.invoicefile import InvoiceFile
:
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
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
    :
```

> 上記を手入力にて確認する際には、import文の追加を忘れないようにしてください。
>
> `Exception Type: NameError`が表示される場合は、`from rdetoolkit.invoicefile import InvoiceFile`の追加を忘れている場合が考えられます。確認し、抜けている場合は追加してください。

本書の古い版では、invoice.jsonファイルのパスを求める箇所を以下の様にしていました。

```python
    invoice_file = srcpaths.invoice  / "invoice.json"
```

しかし、上記の様にした場合、Excelインボイス等複数ファイルを一度に導入するモードでの実行時に、常に1行目に記述したインボイスの内容が有効になるといった不具合が発生します。上に示したように、`resource_paths`を使用してください

```python
    invoice_file = resource_paths.invoice  / "invoice.json"
```

> 絶対にExcelインボイスを使用することはないという場合は、`srcpaths`の方を利用しても同じ結果となりますが、将来Excelインボイスモードを使用する場合に備え、srcpathsの利用は控えてください。


上記例では'(2024)'がタイトルに含まれていない場合にのみ追記しています。これは、べき等性、つまり何度同じ処理をしても同じ結果となることを想定しています。より厳密には、**末尾**に当該文字列があるかどうか、といった別のチェックが必要となる場合があることに注意してください。

また、今回の`invoice.json`では、以下の様にprivateつまり"非公開"を指定しています。

```json
：
        "is_private_raw": "private"
：
```

そのため、実行の結果として`invoice.json.orig`ファイルは、nonshared_raw/フォルダにコピーされます。

```bash
(tenv) $ python main.py

(tenv) $ tree data/nonshared_raw/
data/nonshared_raw/
├── invoice.json.orig
└── sample.data

1 directory, 2 files
```

入力ファイル(上記の場合は"sample.data")は、構造化処理プログラム内では何も処理を記述していませんが、RDEToolKitのデフォルト設定で"data_nonshared_raw"フォルダにコピーされるため、上記の様になります。

送り状ファイルのオリジナルと変更後のものを比べると以下の様になります。

```bash
(tenv) $ diff -ru data/nonshared_raw/invoice.json.orig data/invoice/invoice.json
--- data/nonshared_raw/invoice.json.orig        2025-09-01 17:18:56.038536493 +0900
+++ data/invoice/invoice.json   2025-09-01 17:18:56.038536493 +0900
@@ -3,7 +3,7 @@
     "basic": {
         "dateSubmitted": "2023-01-26",
         "dataOwnerId": "119cae3d3612df5f6cf7085fb8deaa2d1b85ce963536323462353734",
-        "dataName": "NIMS_TRIAL_20230126b",
+        "dataName": "NIMS_TRIAL_20230126b / (2024)",
         "instrumentId": "413e53fb-aec9-41f8-ae55-3f88f6cd8d41",
         "experimentId": null,
         "description": ""
```

> 想定通りにdataNameの値が変更されていることが分かります。
>
> もとのinvoice.jsonファイルの作り方により、最後に`ファイル末尾に改行がありません`と表示される場合があります。ファイル末尾の改行の有無は処理に影響しませんので、出力されなくても問題ありません。

なお、上記実装は、readf_jsonとwritef_jsonを使って以下の様に記述することもできます。

```python
:
from rdetoolkit.fileops import readf_json, writef_json
:
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
    # Read invoice file
    invoice_file = resource_paths.invoice  / "invoice.json"
    invoice = readf_json(invoice_file)

    # Check public or private
    is_private_raw = False if invoice["custom"]["is_private_raw"] == "share" else True

    # Backup(=Copy) invoice.json to shared/nonshared folder
    raw_dir = resource_paths.nonshared_raw if is_private_raw else resource_paths.raw
    invoice_file_backup = raw_dir / "invoice.json.orig"
    writef_json(invoice_file_backup, invoice)

    # Rewrite invoice
    original_data_name = invoice["basic"]["dataName"]
    additional_title = "(2024)"
    if original_data_name.find(additional_title) < 0:
        # update title if not applied yet
        invoice["basic"]["dataName"] = original_data_name + " / " + additional_title
        invoice_file_new = resource_paths.invoice  / "invoice.json"
        writef_json(invoice_file_new, invoice)
    ：    
```

> この方法を使う場合は、`import InvoiceFile`は不要となります。代わりに`import readf_json, writef_json`の追加が必要になります。
>
> こちらを使う場合も、import文の記述を忘れがちですので、注意してください。

