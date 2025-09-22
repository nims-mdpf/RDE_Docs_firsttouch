<div class="page" />

# 入力データの読み込み

続いて、データ登録者がアップロードした入力ファイル(実験データ)を読み込む処理を考えます。

ここは、構造化処理プログラムを書く上で難しい箇所の1つです。

なぜなら、装置ごとに実験データの形式は全く別物であり、データの読み込み方は多種多様になるためです。
つまり、基本的には`実験データ`それぞれに特化したデータ読み取り関数を実装する必要があります。

## 想定

* `data/inputdata/`フォルダにある、`sample.data`ファイルを読み込み、適切なオブジェクトとして保持する。
* 実験データのメタデータ部と計測データ部をそれぞれrawMetaObj とrawDataDf という変数に保存する。関数化した場合は、その順に返すように実装する。

## custom_module()に読み込みロジックを実装

```python
import io

import pandas as pd
:
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
    # Read input data
    DELIM = "="
    raw_data_df = None
    raw_meta_obj = None
    #
    input_file = resource_paths.rawfiles[0] # read one file only

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
```

ここでは、ファイルを指定して、それを読み込み、dict形式オブジェクトにするまでを実装しています。 実装内容は、利用する入力ファイルにより異なります。

上記により、2つのオブジェクト(raw_data_dfとraw_meta_obj)の2つが作成されます。

pprint()などを使って内容を確認できます。現在の`custom_module`関数の末尾に以下を挿入します。

```python
：
    from pprint import pprint
    pprint(raw_data_df)
    print("----")
    pprint(raw_meta_obj)
：
```

上記4行を追記したのち、実行すると以下の様になります。

```bash
(tenv) $ python main.py 
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
----
{'data_title': 'data_title 2',
 'measurement_date': '2023/1/25 23:08',
 'series_number': '3',
 'x_label': 'x(unit)',
 'y_label': 'y(unit)'}

```

> 以降の章で、これら(raw_data_dfとraw_meta_obj)を利用します。

> 上で確認のために追記した4行は、対象のオブジェクトの内容確認のために追加したものです。後続の処理では使いませんので、確認後削除してください。

