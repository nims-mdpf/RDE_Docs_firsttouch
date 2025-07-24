<div class="page"/>

# RDEフォーマットモード

前述の様にRDEフォーマットモードは、入力データをRDEデータセットの形式にそのまま登録します。

シングルデータタイルとして登録する場合とマルチデータタイルとして登録する場合で、構造化処理プログラムに変更があるわけではありません。入力データ、つまりzipファイル内のフォルダ構成を変更することで、自動的に切り替わります。

## RDEフォーマットとは

同一のフォルダ下に、以下のサブフォルダが存在し、その中にそれぞれが想定するファイルが格納されているものを"RDEフォーマット"と称します。

* invoice (インボイスファイルを格納) ： 必須
* main_image (主たるイメージを格納) ： 任意
* meta (抽出したメタデータファイルを格納) ： 任意
* nonshared_raw (展開した入力ファイルを格納) ： 任意 (入力ファイルを"非公開"にする場合に使用)
* other_image (その他のイメージを格納) ： 任意
* raw (展開した入力ファイルを格納) ： 任意 (入力ファイルを"公開"にする場合に使用)
* structured (構造化処理後の可読ファイルを格納) ： 任意
* thumbmail (サムネイル画像を格納) ： 任意

```bash
|- sample.zip
    |- invoice/
    |   |- invoice.json
    |- main_image/
    |   |- xxxx.png
    |- other_image/
    |   |- xxxxx.png
    |- structured/
    |   |- xxxxxx.csv
```

> 上記に記述のない名称のフォルダは取り込み処理にて無視されます。

複数のデータタイルに登録する場合は、divided/XXXX(XXXXは0001から始まる数字連番)フォルダに、上記と同じ構造を取ることで取り込みを実現出来ます。

```bash
|- sample.zip
    |- divided
    |   |- 0001 
    |      |- invoice/
    |          |- invoice.json
    |      |- main_image/
    |          |- yyyy.png
    |      |- other_image/
    |          |- yyyyy.png
    |      |- structured/
    |          |- yyyyyy.csv
    |   |- 0002 
    |      |- invoice/
    |          |- invoice.json
    |      |- main_image/
    |          |- zzzz.png
    |      |- other_image/
    |          |- zzzzz.png
    |      |- structured/
    |          |- zzzzzz.csv
    |- invoice/
    |   |- invoice.json
    |- main_image/
    |   |- xxxx.png
    |- other_image/
    |   |- xxxxx.png
    |- structured/
    |   |- xxxxxx.csv
```

> 上記例だと、3つのデータタイルとして取り込み処理が実施されます。
>
> いずれのフォルダにあるinvoice.jsonも、最上位tasksupportフォルダにあるinvoice.schema.jsonの定義に沿った内容である必要があります。

本書では、前者を"シングルデータタイル登録"、後者を"マルチデータタイル登録"と呼ぶことにします。

## RDEToolKitのインストールと初期フォルダ構成作成

『共通処理』の章で示したように、RDEToolKitのインストールと、初期フォルダ構成の作成を実施します。

```bash
$ cd rde-sample
$ mkdir rdeformat
$ cd rdeformat/

$ source ~/venv/bin/activate
(venv) $ python -m rdetoolkit init
：
(venv) $ cd container
```

以下、双方が完了しているものとして進めます。

## シングルデータタイル登録

> 先に"シングルデータタイル"での実行例を示し、その後"マルチデータタイル"での実行例を示します。

### テスト用データ配置

テスト用のデータとして、本書と同梱で配布されている、以下のファイルを配置します。

* structured.zip (370KB)

実際に格納されると、以下の様になります。

```bash
(venv) $ ls -l data/inputdata/
total 372
-rwxr-xr-x 1 devel devel 377909  1月 31 10:50 structured.zip
```

### フォルダ初期化

開発においては、構造化処理プログラムを何度も実行することになります。

都度手動にて初期化するのは大変ですので、以下の内容で初期化用スクリプトを作成します。

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
./data/devided
./data/logs
./data/divided
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

### 構造化処理作成

ここから、実際に構造化処理プログラムを作成していきます。

> 以下"container/"フォルダからの相対パスとしてファイル名を記述します。

#### main.py

初期化にて作成されたmain.pyに以下を加えていきます。

1. RDEFormatモードを使用するように"設定"を追加
2. 同時にデフォルトではthumbnailフォルダへの画像ファイルコピーが実行されないので、実行するように設定を追加

> 別述のように、これらの設定内容は、main.py内に記述する方法の他、tasksupportフォルダにrdeconfig.ymlファイルを配置することでも実現できます。その方法を使う場合は、main.pyを変更する必要はありません。

main.py

```python
import rdetoolkit
from rdetoolkit.config import Config, SystemSettings


def main() -> None:
    config = Config(
        system=SystemSettings(
            extended_mode="rdeformat",
            save_thumbnail_image=True
        ),
    )

    rdetoolkit.workflows.run(
        config=config
    )

if __name__ == '__main__':
    main()
```

この例では、上で示したように、RDEフォーマットモードの使用(extended_mode="rdeformat")と、サムネイル画像の自動保存(save_thumbnail_image=True)の2つを設定しています。

> 基本的に指定されたデータを取り込むだけですので、custom_dataset_function句の設定やmodulesフォルダへ構造化処理プログラムの配置は行いません。

### 実行

実行してみます。

まず実行前のdataフォルダは以下の様になっています。

```bash
(venv) $ tree data
data
├── inputdata
│   └── structured.zip
├── invoice
│   └── invoice.json
└── tasksupport
    ├── invoice.schema.json
    └── metadata-def.json

4 directories, 4 files
```

> RDEToolKitのバージョンによっては、上記の他、`data/logs/`フォルダおよび`data/logs/rdesys.log`ファイルが存在している場合もあります。

実行します。

```bash
python main.py
```

実行後のディレクトリ構成は、以下の様になっています。

```bash
(venv) $ tree data
data
├── attachment
├── inputdata
│   └── structured.zip
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
├── main_image
│   └── gbp_rank2_numPeak2_shirley_mSG1504_result.png
├── meta
│   └── metadata.json
├── nonshared_raw
├── other_image
│   ├── BIC_vs_NumPeak.png
│   └── parameter_table.png
├── raw
│   └── Cs2O_O2s.zip
├── structured
│   ├── cmd_mrs.log
│   ├── gbp_rank2_numPeak2_shirley_mSG1504_parameters.csv
│   ├── settings_Linear.inp
│   └── settings_Shirley.inp
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
│   ├── invoice_org.json
│   ├── main_image
│   │   └── gbp_rank2_numPeak2_shirley_mSG1504_result.png
│   ├── meta
│   │   └── metadata.json
│   ├── other_image
│   │   ├── BIC_vs_NumPeak.png
│   │   └── parameter_table.png
│   ├── raw
│   │   └── Cs2O_O2s.zip
│   ├── structured
│   │   ├── cmd_mrs.log
│   │   ├── gbp_rank2_numPeak2_shirley_mSG1504_parameters.csv
│   │   ├── settings_Linear.inp
│   │   └── settings_Shirley.inp
│   └── thumbnail
│       └── gbp_rank2_numPeak2_shirley_mSG1504_result.png
└── thumbnail
    └── gbp_rank2_numPeak2_shirley_mSG1504_result.png

21 directories, 25 files
```

> tempフォルダは、invoice_org.jsonを除き、zipファイルを展開したフォルダ/ファイルが展開されています。このフォルダは、RDE取り込み処理では無視され、取り込み対象とはならないことに注意してください。
>
> 上記のようにならず、`nonshared_raw/`フォルダに`structured.zip`がそのままコピーされた状態となった場合は、rdeformatモードを使う設定が正しくなっていないことが考えられます。前章などを参考に、モード設定が正しく行われているかを確認してください。

## マルチデータタイル登録

続いて"マルチデータタイル"での実行例を示します。

### フォルダ初期化

フォルダを初期化します。

```bash
(venv) $ ./reinit.sh
./data/devided was removed
./data/logs was removed
./data/divided was removed
./data/attachment was removed
./data/invoice_patch was removed
./data/meta was removed
./data/main_image was removed
./data/other_image was removed
./data/raw was removed
./data/nonshared_raw was removed
./data/structured was removed
./data/temp was removed
./data/thumbnail was removed
```

### テスト用データ配置

上で使用したstructured.zipを削除します。

```bash
rm data/inputdata/structured.zip
```

テスト用のデータとして、本書と同梱で配布されている、以下のファイルを配置します。

* data.zip (676KB)

実際に格納されると、以下の様になります。

```bash
(venv) $ ls -l data/inputdata/
total 676
-rwxrwxrwx 1 devel devel 691369  1月 31 13:36 data.zip
```

## 構造化処理作成

シングルデータタイル登録で使用したものから変更する必要はありません。

## 実行

実行してみます。

まず実行前のdataフォルダは以下の様になっています。

```bash
(venv) $ tree data
data
├── inputdata
│   └── data.zip
├── invoice
│   └── invoice.json
├── logs
└── tasksupport
    ├── invoice.schema.json
    └── metadata-def.json

5 directories, 4 files
```

実行します。

```bash
python main.py
```

実行後のディレクトリ構成は、以下の様になっています。

```bash
(venv) $ tree data
data
├── attachment
├── divided
│   └── 0001
│       ├── attachment
│       ├── invoice
│       │   └── invoice.json
│       ├── invoice_patch
│       ├── logs
│       ├── main_image
│       │   └── gbp_rank1_numPeak4_shirley_mSG0112_result.png
│       ├── meta
│       │   └── metadata.json
│       ├── nonshared_raw
│       ├── other_image
│       │   ├── BIC_vs_NumPeak.png
│       │   └── parameter_table.png
│       ├── raw
│       │   └── La2O3_O2s.zip
│       ├── structured
│       │   ├── cmd_mrs.log
│       │   ├── gbp_rank1_numPeak4_shirley_mSG0112_parameters.csv
│       │   ├── settings_Linear.inp
│       │   └── settings_Shirley.inp
│       ├── temp
│       └── thumbnail
│           └── gbp_rank1_numPeak4_shirley_mSG0112_result.png
├── inputdata
│   └── data.zip
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
├── main_image
│   └── gbp_rank1_numPeak2_shirley_mSG0672_result.png
├── meta
│   └── metadata.json
├── nonshared_raw
├── other_image
│   ├── BIC_vs_NumPeak.png
│   └── parameter_table.png
├── raw
│   └── Cs2O_O1s.zip
├── structured
│   ├── cmd_mrs.log
│   ├── gbp_rank1_numPeak2_shirley_mSG0672_parameters.csv
│   ├── settings_Linear.inp
│   └── settings_Shirley.inp
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
│   ├── data
│   │   ├── divided
│   │   │   └── 0001
│   │   │       ├── invoice
│   │   │       │   └── invoice.json
│   │   │       ├── main_image
│   │   │       │   └── gbp_rank1_numPeak4_shirley_mSG0112_result.png
│   │   │       ├── meta
│   │   │       │   └── metadata.json
│   │   │       ├── other_image
│   │   │       │   ├── BIC_vs_NumPeak.png
│   │   │       │   └── parameter_table.png
│   │   │       ├── raw
│   │   │       │   └── La2O3_O2s.zip
│   │   │       └── structured
│   │   │           ├── cmd_mrs.log
│   │   │           ├── gbp_rank1_numPeak4_shirley_mSG0112_parameters.csv
│   │   │           ├── settings_Linear.inp
│   │   │           └── settings_Shirley.inp
│   │   ├── invoice
│   │   │   └── invoice.json
│   │   ├── main_image
│   │   │   └── gbp_rank1_numPeak2_shirley_mSG0672_result.png
│   │   ├── meta
│   │   │   └── metadata.json
│   │   ├── other_image
│   │   │   ├── BIC_vs_NumPeak.png
│   │   │   └── parameter_table.png
│   │   ├── raw
│   │   │   └── Cs2O_O1s.zip
│   │   └── structured
│   │       ├── cmd_mrs.log
│   │       ├── gbp_rank1_numPeak2_shirley_mSG0672_parameters.csv
│   │       ├── settings_Linear.inp
│   │       └── settings_Shirley.inp
│   └── invoice_org.json
└── thumbnail
    └── gbp_rank1_numPeak2_shirley_mSG0672_result.png

44 directories, 46 files
```

> tempフォルダは、invoice_org.jsonを除き、zipファイルを展開したフォルダ/ファイルが展開されています。このフォルダは、RDE取り込み処理では無視され、取り込み対象とはならないことに注意してください。
>
> 上の例では、入力のzipファイル中の最上位フォルダは"data"になります。このフォルダは任意で、"data2/"でも、シングルデータタイルモード時と同様main_imageやstructuredフォルダが直接並ぶ形式でも構いません。つまり「zipファイルを展開したときに、なんらかのフォルダがあって、その下にraw、structuredといったフォルダがあるケースと、フォルダなしでraw、structuredといったファイルがあるケースは、いずれの場合も同じ扱い」となります。
