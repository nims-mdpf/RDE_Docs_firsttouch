<div class="page" />

# メタデータの出力

構造化処理プログラムの大きな目的の一つは"メタデータの抽出"です。基本的に、どういった構造化処理プログラムでも必ずメタデータの抽出は行われることが想定されます(溜めるだけを想定したデータセットではメタデータファイルの作成が行われない場合もあります)。

> 図の作成や数値データの可読化(→ CSVファイルへの出力) などはオプション実装、つまり要件次第で実装する場合もあれば実装しない場合もあります。

RDEでは抽出したメタデータを、"metadata.json" というJSON形式のファイルに出力する必要があります。

抽出されたメタデータがどのような項目を持ちうるかは、データセットを開設前に事前に準備する "metadata-def.json"という JSON ファイルを用意して記述しておく必要があります。
本書では、すでに述べたようにmetadata-def.jsonがすでに用意された状態を想定しますが、通常は開発者が記述/作成する必要があります。

多くの場合、「入力ファイルを読み込み取得したメタデータを、ファイル(medatada.json)として出力する」ことが想定されます。本書では、それ以外にもいくつかの方法でメタデータを収集する手段について実装例を示すことにします。

## 想定

* 以下の順でメタデータを収集する。
  * 入力ファイルを読み込んだ際にパースしたメタデータ部分
  * invoiceファイルの`custom`部分
  * 入力データの、数値部分
* 上記データをマージします。同じキーのデータがある場合は、**後から指定したもので上書き**する。
* マージしたメタデータを、ファイル(medatada.json)として出力する。

> ここでいう"パース"とは、`文字データを解析して一定の形式のデータ構造に変換する処理`であり、`変換した結果から、メタデータや数値データを取り出す`ことを含みます。

メタデータには、大きく分けて2つのパートがあります。

1つ目は"constant"部で、本書の場合、前者の2つ、つまり「入力ファイルのメタデータ部」と「invoiceの"custom"部分」が出力される部分となります。

2つ目は、"variable"部で、上記想定の3つ目、つまり「入力ファイルの数値部」から出力されます。

## custom_module()にロジックを実装

おおよその流れとしては、以下の様になります。

1. metadata-def.jsonを指定してMetaオブジェクト(RDEToolKitが提供するメタデータ操作用オブジェクト)を作成する。なお、metadata-def.jsonは、"data/tasksupport"フォルダに格納されている前提なので、フォルダ名を含む形で指定する必要があります。
2. 何らかの方法で、メタデータを収集する。
3. 上で収集したメタデータを、Metaオブジェクトに登録する。
4. metadata.jsonを出力する。

> 上記の様に作成することで、RDEToolKitがmetadata-def.jsonと連携して、metadata.jsonを作成します。

上記の流れに沿って、以下のように処理を追記します。

modules/datasets-process.py
```python
import statistics as st
from rdetoolkit.rde2util import Meta

def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
    # Retrieve Meta data

    # create Meta instance
    metadata_def_file = srcpaths.tasksupport / "metadata-def.json"
    meta_obj = Meta(metadata_def_file)

    # get meta data
    # #1 Read Input File
    # -> input fileは前述の処理にて作成済み

    # #2 Read Inovoice
    # -> invoice.invoice_obj["custom"] は前述の処理にて作成(→読み込み)済み

    # #3 From raw data numeric data part
    s_name = []
    s_count = []
    s_mean = []
    s_median = []
    s_max = []
    s_min = []
    s_stdev = []
    for df in raw_data_df:
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

    # Merge 2 types of metadata
    const_meta_info = raw_meta_obj | invoice.invoice_obj["custom"] 
    repeated_meta_info = meta_vars

    # Set metadata to meta instance
    meta_obj.assign_vals(const_meta_info)
    meta_obj.assign_vals(repeated_meta_info)

    # Write metadata
    metadata_json = resource_paths.meta / "metadata.json"
    meta_obj.writefile(metadata_json)
```

## (参考) metadata.json

参考のため、上記(サンプル)コードでの出力結果を以下に示します。

```bash
(tenv) $ cat data/meta/metadata.json
{
    "constant": {
        "data_title": {
            "value": "data_title 2"
        },
        "measurement_date": {
            "value": "2023-01-25T00:00:00"
        },
        "x_label": {
            "value": "x(unit)"
        },
        "y_label": {
            "value": "y(unit)"
        },
        "series_number": {
            "value": 3,
            "unit": "PCS"
        },
        "invoice_string1": {
            "value": "送状文字入力値1"
        }
    },
    "variable": [
        {
            "series_name": {
                "value": "series1"
            },
            "series_data_count": {
                "value": 15,
                "unit": "PCS"
            },
            "series_data_mean": {
                "value": 102.07
            },
            "series_data_median": {
                "value": 103.0
            },
            "series_data_max": {
                "value": 109.0
            },
            "series_data_min": {
                "value": 82.0
            },
            "series_data_stdev": {
                "value": 6.27
            }
        },
        {
            "series_name": {
                "value": "series2"
            },
            "series_data_count": {
                "value": 11,
                "unit": "PCS"
            },
            "series_data_mean": {
                "value": 88.45
            },
            "series_data_median": {
                "value": 102.0
            },
            "series_data_max": {
                "value": 109
            },
            "series_data_min": {
                "value": 41
            },
            "series_data_stdev": {
                "value": 24.88
            }
        },
        {
            "series_name": {
                "value": "series3"
            },
            "series_data_count": {
                "value": 12,
                "unit": "PCS"
            },
            "series_data_mean": {
                "value": 89.52
            },
            "series_data_median": {
                "value": 101.6
            },
            "series_data_max": {
                "value": 109.0
            },
            "series_data_min": {
                "value": 41.0
            },
            "series_data_stdev": {
                "value": 24.01
            }
        }
    ]
}(tenv) $ 
```

> (参考)metadata-def.jsonで定義されている"invoice_string2"の項目は、metadata.jsonには**出力されません**。これは基となるinvoice.json中で値が"Null" (ダブルクォーテーションで囲まれていないNull)、つまり画面上での入力がないためです。invoice.json中の"invoice_string2"の値(Null)をダブルクォーテーションで囲んで処理を実行すれば、文字列として処理されますのでmetadata.jsonに書き出されることが確認できます。

## おまけ

処理には影響しませんがmetadata-def.jsonにタイポ(誤字)があります。

以下の部分にあります。分かりますでしょうか?

```json
：
    "invoice_string2": {
        "name": { 
            "ja": "送状文字入力値2",
            "en": "inboice_string2"
        },
        "schema": {
            "type": "string"
        },
        "order": 17,
        "mode": "Invoice"
    },
：
```

> 答え：inboice_string2(誤) --> invoice_string2(正) 

今回は"表示名部分"のタイポ(誤字)なので英語表示のブラウザで実行した場合には、表示がおかしいことになります。その後の構造化処理には影響しません。

しかし、表示名ではなく"項目名"部分を間違えた場合は、構造化処理プログラム内での指定と異なることになり、意図しない動きになる可能性があります。タイポ(誤字)には十分注意してください。

