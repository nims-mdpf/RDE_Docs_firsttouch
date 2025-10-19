<div class="page"/>

# マルチデータタイルモード

続いて、`マルチデータタイルモード`でのコード例を示します。

すでに示したように、"マルチデータタイルモード"は、以下のような処理を実行します。

* 複数のファイルを、それぞれ別のデータタイルとして登録する。
* そのとき利用される送り状(invoice.json)は、すべて同じものが利用される。
* ファイルは全く別のフォーマットの場合もあり、基本的にメタデータの抽出やデータの可視化など、必要最小限のものを除き"構造化処理"は行わない。

> 前章で行った、`RDEフォーマットモードの、マルチデータタイル登録`とは異なり、マルチデータタイルモードでは、"1ファイル = 1データタイル"として登録されます。

## RDEToolKitのインストールと初期フォルダ構成作成

『共通処理』の章で示したように、RDEToolKitのインストールと、初期フォルダ構成の作成を実施します。

```bash
$ cd ${HOME}/rde-sample
$ mkdir multidatatile
$ cd multidatatile/

$ source ~/venv/bin/activate
(venv) $ python -m rdetoolkit init
：
(venv) $ cd container
```

以下、双方が完了しているものとして進めます。

## テスト用データ配置

テスト用のデータとして、以下を配置します。

* Mo50Ti15C-20230111-10mN-00.tif (894KB)
* f27.bmp (377KB)
* materials.svg (32KB)
* report.pdf (1.4MB)

実際に格納されると、以下の様になります。

```bash
(venv) $ cd container/
(venv) $ ls -l data/inputdata/
total 2720
-rw-rw-r-- 1 devel devel  894137  5月 16 07:10 Mo50Ti15C-20230111-10mN-00.tif
-rw-rw-r-- 1 devel devel  376734  5月 16 07:10 f27.bmp
-rw-rw-r-- 1 devel devel   31755  5月 16 07:10 materials.svg
-rw-rw-r-- 1 devel devel 1475968  5月 16 07:11 report.pdf
```

> 日付については、上記と異なる可能性があります。違っていても特に問題ありません。

### フォルダ初期化

前章同様、以下の内容で初期化用スクリプトを作成します。

reinit.sh

```bash
#!/bin/bash


# ./data/inputdata
#  -> 何もしない

#./data/job.failed
if [ -f ./data/job.failed ];then
    echo "./data/job.failed was removed"
    rm -f ./data/job.failed
fi

# ./data/invoice
#  -> 何もしない

#./data/tasksupport
#  -> 何もしない

#./modules
#  -> 何もしない

#./requirements.txt
#  -> 何もしない

D="
./data/divided
./data/logs
./data/attachment
./data/invoice_patch
./data/meta
./data/main_image
./data/other_image
./data/raw
./data/nonshared_raw
./data/structured
./data/temp
./data/thumbnail
"
for d in ${D};do
    echo "${d} was removed"
    rm -rf ${d}
done

# Python cache
rm -rf modules/__pycache__
```

実行権限を付与します。

```bash
chmod a+x reinit.sh
```

## invoice.schema.json、invoice.jsonファイルの配置

> 実際のところ、下記2つのファイルは、`python -m rdetoolkit init`にて作成された内容から変更していません。ここでは内容を確認すればOKです。

data/tasksupport/invoice.schema.jsonを、以下の内容で作成します。

```json
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
}
```

data/invoice/invoice.jsonを以下の内容で作成します。

```json
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
}
```

## 構造化処理作成

ここから、実際に構造化処理プログラムを作成していきます。

> 以下"container/"フォルダからの相対パスとしてファイル名を記述します。

### main.py

初期化にて作成されたmain.pyに以下を加えていきます。

1. "MultiDataTile"モードを使う指定
2. "modules/datasets_process.py"にある`dataset()`関数を実行する。

> 詳細な処理については、この`dataset()`関数から呼び出される関数にて実行することを想定します。

main.py

```python
import rdetoolkit
from rdetoolkit.config import Config, SystemSettings, MultiDataTileSettings

from modules.datasets_process import dataset

def main() -> None:
    config = Config(
        system=SystemSettings(
            extended_mode="MultiDataTile",
        ),
        multidata_tile=MultiDataTileSettings(
            ignore_errors=True
        ),
    )
    rdetoolkit.workflows.run(
        config=config,
        custom_dataset_function=dataset,
    )

if __name__ == "__main__":
    main()
```

### modules/datasets_process.py

最初に'Hello RDE!'をプリントするだけの処理を実行し、何が起きるのかを確認します。

modules/datasets_process.py

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    print('Hello RDE!')
```

実行します。

```bash
(venv) $ python main.py
Hello RDE!
Hello RDE!
Hello RDE!
Hello RDE!
```

"Hello RDE!"が4回出力されました。

どうして4回出力されたのでしょうか?

謎を解明するために、pprintを使って確認してみます。

> pprintは変数の構造を含めて表示することが出来ます。同様のことをデバッガを利用して実行できます。普段お使いのデバッガがある場合はそちらをご利用いただいても問題ありません。

modules/datasets_process.pyの、上の方でpprintをimportします。

```python
：
from pprint import pprint
:
```

次に、`dataset()`関数内で、srcpathsの設定内容を表示するようにします。

```python
：
    print('Hello RDE!')
    pprint(srcpaths)
：
```

実行してみます。

```bash
(venv) $ python main.py 
Hello RDE!
RdeInputDirPaths(inputdata=PosixPath('data/inputdata'),
                 invoice=PosixPath('data/invoice'),
                 tasksupport=PosixPath('data/tasksupport'),
                 config=Config(system=SystemSettings(extended_mode='MultiDataTile', save_raw=False, save_nonshared_raw=True, save_thumbnail_image=False, magic_variable=False), multidata_tile=MultiDataTileSettings(ignore_errors=True), smarttable=None))
Hello RDE!
RdeInputDirPaths(inputdata=PosixPath('data/inputdata'),
                 invoice=PosixPath('data/invoice'),
                 tasksupport=PosixPath('data/tasksupport'),
                 config=Config(system=SystemSettings(extended_mode='MultiDataTile', save_raw=False, save_nonshared_raw=True, save_thumbnail_image=False, magic_variable=False), multidata_tile=MultiDataTileSettings(ignore_errors=True), smarttable=None))
Hello RDE!
RdeInputDirPaths(inputdata=PosixPath('data/inputdata'),
                 invoice=PosixPath('data/invoice'),
                 tasksupport=PosixPath('data/tasksupport'),
                 config=Config(system=SystemSettings(extended_mode='MultiDataTile', save_raw=False, save_nonshared_raw=True, save_thumbnail_image=False, magic_variable=False), multidata_tile=MultiDataTileSettings(ignore_errors=True), smarttable=None))
Hello RDE!
RdeInputDirPaths(inputdata=PosixPath('data/inputdata'),
                 invoice=PosixPath('data/invoice'),
                 tasksupport=PosixPath('data/tasksupport'),
                 config=Config(system=SystemSettings(extended_mode='MultiDataTile', save_raw=False, save_nonshared_raw=True, save_thumbnail_image=False, magic_variable=False), multidata_tile=MultiDataTileSettings(ignore_errors=True), smarttable=None))
```

同じものが4回出力されました。

まだ"なぜ4回出力されるのか?"が不明です。今度はsrcpathsの代わりに、もう1つの引数であるresource_pathsを表示してみます。

```python
：
    print('Hello RDE!')
    #pprint(srcpaths) #この行をコメントアウト
    pprint(resource_paths) #この行を追加
：
```

実行してみます。

```bash
(venv) $ python main.py 
Hello RDE!
RdeOutputResourcePath(raw=PosixPath('data/raw'),
                      nonshared_raw=PosixPath('data/nonshared_raw'),
                      rawfiles=(PosixPath('data/inputdata/Mo50Ti15C-20230111-10mN-00.tif'),),
                      struct=PosixPath('data/structured'),
                      main_image=PosixPath('data/main_image'),
                      other_image=PosixPath('data/other_image'),
                      meta=PosixPath('data/meta'),
                      thumbnail=PosixPath('data/thumbnail'),
                      logs=PosixPath('data/logs'),
                      invoice=PosixPath('data/invoice'),
                      invoice_schema_json=PosixPath('data/tasksupport/invoice.schema.json'),
                      invoice_org=PosixPath('data/temp/invoice_org.json'),
                      temp=PosixPath('data/temp'),
                      invoice_patch=PosixPath('data/invoice_patch'),
                      attachment=PosixPath('data/attachment'))
Hello RDE!
RdeOutputResourcePath(raw=PosixPath('data/divided/0001/raw'),
                      nonshared_raw=PosixPath('data/divided/0001/nonshared_raw'),
                      rawfiles=(PosixPath('data/inputdata/f27.bmp'),),
                      struct=PosixPath('data/divided/0001/structured'),
                      main_image=PosixPath('data/divided/0001/main_image'),
                      other_image=PosixPath('data/divided/0001/other_image'),
                      meta=PosixPath('data/divided/0001/meta'),
                      thumbnail=PosixPath('data/divided/0001/thumbnail'),
                      logs=PosixPath('data/divided/0001/logs'),
                      invoice=PosixPath('data/divided/0001/invoice'),
                      invoice_schema_json=PosixPath('data/tasksupport/invoice.schema.json'),
                      invoice_org=PosixPath('data/temp/invoice_org.json'),
                      temp=PosixPath('data/divided/0001/temp'),
                      invoice_patch=PosixPath('data/divided/0001/invoice_patch'),
                      attachment=PosixPath('data/divided/0001/attachment'))
Hello RDE!
RdeOutputResourcePath(raw=PosixPath('data/divided/0002/raw'),
                      nonshared_raw=PosixPath('data/divided/0002/nonshared_raw'),
                      rawfiles=(PosixPath('data/inputdata/materials.svg'),),
                      struct=PosixPath('data/divided/0002/structured'),
                      main_image=PosixPath('data/divided/0002/main_image'),
                      other_image=PosixPath('data/divided/0002/other_image'),
                      meta=PosixPath('data/divided/0002/meta'),
                      thumbnail=PosixPath('data/divided/0002/thumbnail'),
                      logs=PosixPath('data/divided/0002/logs'),
                      invoice=PosixPath('data/divided/0002/invoice'),
                      invoice_schema_json=PosixPath('data/tasksupport/invoice.schema.json'),
                      invoice_org=PosixPath('data/temp/invoice_org.json'),
                      temp=PosixPath('data/divided/0002/temp'),
                      invoice_patch=PosixPath('data/divided/0002/invoice_patch'),
                      attachment=PosixPath('data/divided/0002/attachment'))
Hello RDE!
RdeOutputResourcePath(raw=PosixPath('data/divided/0003/raw'),
                      nonshared_raw=PosixPath('data/divided/0003/nonshared_raw'),
                      rawfiles=(PosixPath('data/inputdata/report.pdf'),),
                      struct=PosixPath('data/divided/0003/structured'),
                      main_image=PosixPath('data/divided/0003/main_image'),
                      other_image=PosixPath('data/divided/0003/other_image'),
                      meta=PosixPath('data/divided/0003/meta'),
                      thumbnail=PosixPath('data/divided/0003/thumbnail'),
                      logs=PosixPath('data/divided/0003/logs'),
                      invoice=PosixPath('data/divided/0003/invoice'),
                      invoice_schema_json=PosixPath('data/tasksupport/invoice.schema.json'),
                      invoice_org=PosixPath('data/temp/invoice_org.json'),
                      temp=PosixPath('data/divided/0003/temp'),
                      invoice_patch=PosixPath('data/divided/0003/invoice_patch'),
                      attachment=PosixPath('data/divided/0003/attachment'))
```

resource_pathsには、出力されるフォルダや出力されるファイルが入っています。

少しわかりにくいので、この中からさらに"rawfiles"の値だけを取り出してみます。

```python
：
    #pprint(resource_paths)  #この行はコメントアウト
    pprint(resource_paths.rawfiles)
：
```

実行すると以下の様になります。

```bash
(venv) $ python main.py
Hello RDE!
(PosixPath('data/inputdata/Mo50Ti15C-20230111-10mN-00.tif'),)
Hello RDE!
(PosixPath('data/inputdata/f27.bmp'),)
Hello RDE!
(PosixPath('data/inputdata/materials.svg'),)
Hello RDE!
(PosixPath('data/inputdata/report.pdf'),)
```

そうです、"Hello RDE!"が4回表示されたのは、入力ファイル、つまり`data/inputdata/`フォルダ内に存在するファイルが4つ有り、そのそれぞれに対して構造化処理プログラムが実行されるため、なのです。

> `resource_paths.rawfiles`は、上記の様に入力ファイルが1個の場合でも配列として提供されます。構造化処理プログラム内で利用する場合は注意してください。

この"単純な"プログラムの結果どうなったかを確認してみます。

```bash
(venv) $ tree data
data
├── attachment
├── divided
│   ├── 0001
│   │   ├── attachment
│   │   ├── invoice
│   │   │   └── invoice.json
│   │   ├── invoice_patch
│   │   ├── logs
│   │   ├── main_image
│   │   ├── meta
│   │   ├── nonshared_raw
│   │   │   └── f27.bmp
│   │   ├── other_image
│   │   ├── raw
│   │   ├── structured
│   │   ├── temp
│   │   └── thumbnail
│   ├── 0002
│   │   ├── attachment
│   │   ├── invoice
│   │   │   └── invoice.json
│   │   ├── invoice_patch
│   │   ├── logs
│   │   ├── main_image
│   │   ├── meta
│   │   ├── nonshared_raw
│   │   │   └── materials.svg
│   │   ├── other_image
│   │   ├── raw
│   │   ├── structured
│   │   ├── temp
│   │   └── thumbnail
│   └── 0003
│       ├── attachment
│       ├── invoice
│       │   └── invoice.json
│       ├── invoice_patch
│       ├── logs
│       ├── main_image
│       ├── meta
│       ├── nonshared_raw
│       │   └── report.pdf
│       ├── other_image
│       ├── raw
│       ├── structured
│       ├── temp
│       └── thumbnail
├── inputdata
│   ├── Mo50Ti15C-20230111-10mN-00.tif
│   ├── f27.bmp
│   ├── materials.svg
│   └── report.pdf
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys.log
├── main_image
├── meta
├── nonshared_raw
│   └── Mo50Ti15C-20230111-10mN-00.tif
├── other_image
├── raw
├── structured
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
│   └── invoice_org.json
└── thumbnail

55 directories, 16 files
```

`data/inputdata/`フォルダには、先に用意した4つのファイルがあることが分かります。

```text
：
├── inputdata
│   ├── Mo50Ti15C-20230111-10mN-00.tif
│   ├── f27.bmp
│   ├── materials.svg
│   └── report.pdf
```

主な出力先の1つである"nonshared_raw/"フォルダについて確認すると
* data/nonshared_raw

の他に

* data/divided/0001/nonshared_raw
* data/divided/0002/nonshared_raw
* data/divided/0003/nonshared_raw

の3つがあることが分かります。

そして、そのそれぞれに、入力ファイルが1つずつ格納されています。

> 入力ファイルを、`nonshared_raw/`フォルダに出力する機能は、RDEToolKitにより標準提供されている機能です。`nonshared_raw/`フォルダではなく、`raw/`フォルダに出力したい場合など、デフォルトの挙動を変更するには、(main.pyで与えるConfig()内の指定などの)設定内容を変更する必要があります。

## イメージ処理

上述のように、この例で使われる入力ファイルはtif形式、bmp形式、svg形式およびpdf形式とそれぞれ異なっています。

tif形式(= tiff形式)やbmp形式(BitMap形式)、jpg形式(=JPEG形式)などの画像イメージは問題なくRDE内で利用可能である一方、SVG形式では、そのまま画像用のフォルダ(`main_image/`あるいは`other_image/`)に格納した場合、格納は可能であっても、登録されたデータのダウンロードに支障が発生してしまいます。

> 現時点(2025.05)のRDEの制限となっています。RDEの将来バージョンでSVG形式がサポートされた場合はこの限りではありません。

またpdf形式は画像形式ではないので、何らかの方法で画像化する必要があります。

> 画像化しなくても"No Image"という画像が表示されるだけですので、厳密には必須ではありませんが、画像が表示された方がわかりやすいなどの理由から画像化することを考えることにします。

マルチデータタイルモードでは、1ファイルが1データタイルとして扱われるので、それぞれのファイルから`main_image/`フォルダに格納する"画像"ファイルを生成します。

### 入力ファイル形式判断

入力ファイルの形式を判断するために、python-magic モジュールを利用します。

> 同様のモジュールとして、Python標準モジュールのmimetypesがあります。mimetypesは、ファイル名拡張子によって形式を返します。つまりファイル名に拡張子が付いていないファイルについて、その形式が判断できません。そのため、ここではpython-magicモジュールを使うことにします。

python-magicモジュールの実行には、libmagicが必要となります。

インストール状況を確認します。テストしたUbuntu 24.04の場合、以下のようになります。

```bash
(venv) $ sudo apt list --installed | grep libmagic*
：
libmagic-mgc/noble,now 1:5.45-3build1 amd64 [インストール済み、自動]
libmagic1t64/noble,now 1:5.45-3build1 amd64 [インストール済み、自動]
```

つまり、libmagicパッケージが既にインストールされた状態でした。インストールされていることが確認できない場合は、それぞれの環境に合わせてlibmagic(または相当する)パッケージをインストールしてください。

続いて、Pythonの`python-magic`モジュールをインストールします。

```bash
pip install python-magic
```

確認します。

```bash
(venv) $ pip list
Package                   Version
------------------------- -----------------
:
python-magic              0.4.27
:
```

> 上記のように`python-magic`が表示されればインストールされていることが確認できます。

スクリプト(modules/datasets_process.py)の先頭で、`magic`をインポートします。

> パッケージ名は`python-magic`ですが、importする際は`magic`だけとなります。

modules/datasets_process.py

```python
import magic
```

ファイル出力先を表示するようにします。以下、全文を提示します。

modules/datasets_process.py

```python
from pprint import pprint

import magic
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    image_to = resource_paths.main_image
    pprint(image_to)
```

実行します。

```bash
(venv) $ python main.py
PosixPath('data/main_image')
PosixPath('data/divided/0001/main_image')
PosixPath('data/divided/0002/main_image')
PosixPath('data/divided/0003/main_image')
```

このように、今回の場合、image_toは、ファイルそれぞれで以下の様になります。

* data/main_image
* data/divided/0001/main_image
* data/divided/0002/main_image
* data/divided/0003/main_image

> 2個目以降のフォルダ名を構造化処理プログラム内で都度生成するのは大変なので、上の様にRDEToolKitが提供している機能を利用します。

次に、それぞれのファイル名を取得し、python-magicモジュールによる判定を行います。

modules/datasets_process.py

```python
from pprint import pprint

import magic
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    image_to = resource_paths.main_image
    for inputfile in resource_paths.rawfiles:
        mymime = magic.from_file(inputfile, mime=True)
        pprint(mymime)   
```

実行します。

```bash
(venv) $ python main.py
'image/tiff'
'image/bmp'
'image/svg+xml'
'application/pdf'
```

それぞれのファイルの形式が正しく判断されていることが分かります。

### ファイルコピー または 画像ファイルへの変換

続いて、判定した形式(=mymime)の値に応じて処理を実行します。RDEが画像としてサポートしている形式であればそのままコピーします。それ以外の形式であれば、何らかの変換処理を実施し、画像として登録します。

#### "image/jpeg"、"image/png" または "image/gif"の場合

変換操作など実行せずに、main_imageフォルダにコピーする。コピーにはshutilパッケージを利用します。

modules/datasets_process.py

```python
import shutil
：

        if mymime == "image/jpeg" or mymime == "image/png" or mymime == "image/gif":
            shutil.copy(inputfile, image_to)
```

> 今回は、この3つの形式のファイルではないので、このif文では何も実行されません。

#### "application/pdf"の場合

pdfファイルの1ページ目の見た目を画像として生成します。

様々な方法がありますが、PyMuPDFを使うことにします。

Pythonモジュールをインストールします。

```bash
pip install PyMuPDF
```

インポートします。このモジュールは、インストール時に指定するパッケージ名とインポートする名称が大きく異なります(→ "fitz")ので注意してください。またpdfファイル名から拡張子を取り除き、新たな拡張子(.png)を付けて保存するため、pathlibもインポートしておきます。

modules/datasets_process.py

```python
from pathlib import Path
import fitz
：
```

変換処理は、上のif文に続く形とします。

```python
：
        elif mymime == "application/pdf":
            base_file_name = inputfile.stem

            # PDFファイルを開く
            pdf = fitz.open(inputfile)

            # 1ページずつ画像ファイルとして保存する
            for i, page in enumerate(pdf):
                page_num = i + 1 # iは0から始まるため、1足してページ数とする。
                pix = page.get_pixmap()
                image_f_name = f'{base_file_name}_{page_num:02}.png'
                image_f_path = image_to / image_f_name
                pix.save(image_f_path)
                # 1 ページだけでよいので、ここで抜ける
                break
```

> 上の例では、pdfのファイル名から拡張子を除き、"_"(アンダースコア)に続きページ番号を前ゼロ埋め2桁(つまり"01")、さらに拡張子(.png)を付加して画像ファイル名としています。

#### "image/svg+xml"の場合

前述の様に、現時点のRDEは、画像形式としてのsvg形式をサポートしていません。そのため、png形式などRDEがサポートしている形式に変換する必要があります。

ここでは、svg形式をpng形式に変換することとし、変換に`cairosvg`を利用することにします。

Pythonモジュールをインストールします。
```bash
pip install cairosvg
```

OSの必要パッケージもインストールします。

```bash
sudo apt install libcairo2
```

インポートします。

modules/datasets_process.py

```python
import os
import cairosvg
：
```

> 前者、つまりosパッケージのインポートは、Pythonスクリプト内で`環境変数`を設定するために必要となります。

変換処理は、上のif文に続く形とします。

modules/datasets_process.py

```python
：
        elif mymime == "image/svg+xml":
            base_file_name = inputfile.stem
            image_f_name = base_file_name + '.png'
            os.environ['LC_CTYPE'] = "ja_JP.UTF-8"
            cairosvg.svg2png(url=str(inputfile),write_to=str(image_to / image_f_name))
```

> プログラム中で環境変数`$LC_CTYPE`の設定を行っています。この設定は必須ではありませんが、英語環境で日本語文字列を含むsvg形式のファイルを変換する際には文字化けを避けるために設定します。

#### その他の画像形式の場合 (tif形式、bmp形式を含む)

その他の形式の場合は、PILを用いてpng画像への変換を試みます。PILサポート外の画像形式や画像以外のファイル形式(例えばCSV形式など)の場合は、何もしないようにします。

変換処理は、上のif文に続く形とします。

```python
:
from PIL import Image
：
        else:
            try:
                base_file_name = inputfile.stem
                image_f_name = base_file_name + '.png'
                img = Image.open(inputfile)
                img.save(str(image_to / image_f_name))
            except Exception:
                pass
```

> 場合によっては、png画像に変換出来なかった場合に`raise StructuredError(……)`を発行することを考慮してください。

以上をまとめると、以下のようになります。

modules/datasets_process.py

```python
import magic
import shutil
import os
from pathlib import Path

import fitz
import cairosvg
from PIL import Image
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    image_to = resource_paths.main_image
    for inputfile in resource_paths.rawfiles:
        mymime = magic.from_file(inputfile, mime=True)
        if mymime == "image/jpeg" or mymime == "image/png" or mymime == "image/gif":
            shutil.copy(inputfile, image_to)
        elif mymime == "application/pdf":
            base_file_name = inputfile.stem

            # PDFファイルを開く
            pdf = fitz.open(inputfile)

            # 1ページずつ画像ファイルとして保存する
            for i, page in enumerate(pdf):
                page_num = i + 1 # iは0から始まるため、1足してページ数とする。
                pix = page.get_pixmap()
                image_f_name = f'{base_file_name}_{page_num:02}.png'
                image_f_path = image_to / image_f_name
                pix.save(image_f_path)
                # 1 ページだけでよいので、ここで抜ける
                break
        elif mymime == "image/svg+xml":
            base_file_name = inputfile.stem
            image_f_name = base_file_name + '.png'
            os.environ['LC_CTYPE'] = "ja_JP.UTF-8"
            cairosvg.svg2png(url=str(inputfile),write_to=str(image_to / image_f_name))
        else:
            try:
                base_file_name = inputfile.stem
                image_f_name = base_file_name + '.png'
                img = Image.open(inputfile)
                img.save(str(image_to / image_f_name))
            except Exception:
                pass
```

実行します。
```bash
python main.py
```

出来上がったフォルダ構成を確認してみます。

```bash
(venv) $ tree data
data
├── attachment
├── divided
│   ├── 0001
│   │   ├── attachment
│   │   ├── invoice
│   │   │   └── invoice.json
│   │   ├── invoice_patch
│   │   ├── logs
│   │   ├── main_image
│   │   │   └── f27.png
│   │   ├── meta
│   │   ├── nonshared_raw
│   │   │   └── f27.bmp
│   │   ├── other_image
│   │   ├── raw
│   │   ├── structured
│   │   ├── temp
│   │   └── thumbnail
│   ├── 0002
│   │   ├── attachment
│   │   ├── invoice
│   │   │   └── invoice.json
│   │   ├── invoice_patch
│   │   ├── logs
│   │   ├── main_image
│   │   │   └── materials.png
│   │   ├── meta
│   │   ├── nonshared_raw
│   │   │   └── materials.svg
│   │   ├── other_image
│   │   ├── raw
│   │   ├── structured
│   │   ├── temp
│   │   └── thumbnail
│   └── 0003
│       ├── attachment
│       ├── invoice
│       │   └── invoice.json
│       ├── invoice_patch
│       ├── logs
│       ├── main_image
│       │   └── report_01.png
│       ├── meta
│       ├── nonshared_raw
│       │   └── report.pdf
│       ├── other_image
│       ├── raw
│       ├── structured
│       ├── temp
│       └── thumbnail
├── inputdata
│   ├── Mo50Ti15C-20230111-10mN-00.tif
│   ├── f27.bmp
│   ├── materials.svg
│   └── report.pdf
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys.log
├── main_image
│   └── Mo50Ti15C-20230111-10mN-00.png
├── meta
├── nonshared_raw
│   └── Mo50Ti15C-20230111-10mN-00.tif
├── other_image
├── raw
├── structured
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
│   └── invoice_org.json
└── thumbnail

55 directories, 20 files
```

> `inputdata/`フォルダにある、tiff形式、bmp形式、svg形式およびpdf形式ファイルは、それぞれpng画像に変換され、`main_image/`フォルダに格納されます。本章の例としては扱っていませんがjpeg形式、gif形式およびpng形式は(変換などの作業を行わず)そのまま`main_image/`フォルダに複製が格納されます。

## 非共有フォルダ格納と共有フォルダへの格納

RDEToolKitのデフォルト設定では、入力ファイルは非共有フォルダ(→ `nonshared_raw/`)へのコピー処理が実行されます。そのため非共有フォルダへの格納のために処理を追加する必要は有りません。

逆に共有フォルダ(raw/フォルダ)へ書き込み対場合は、設定を変更する必要があります。

以下に、非共有フォルダではなく、共有フォルダにコピーする場合の`main.py`の例を示します。

```python
import rdetoolkit
from rdetoolkit.config import Config, SystemSettings, MultiDataTileSettings

from modules.datasets_process import dataset

def main() -> None:
    config = Config(
        system=SystemSettings(
            extended_mode="MultiDataTile",
            save_nonshared_raw=False,
            save_raw=True,
        ),
        multidata_tile=MultiDataTileSettings(
            ignore_errors=True
        ),
    )
    rdetoolkit.workflows.run(
        config=config,
        custom_dataset_function=dataset,
    )

if __name__ == "__main__":
    main()
```

> `save_nonshared_raw`に`False`をセットすることで、nonshared_raw/フォルダへのコピーを抑止し、`save_raw`に`True`をセットすることでraw/フォルダへのコピーを実施するようにしています。
>
> また別述のように、RDEToolKitの設定は外部ファイル(`tasksupport/rdeconfig.yml`など)で指定することも可能です。詳しくは関連する章や別途公開されているオンラインマニュアルなどを参照ください。

