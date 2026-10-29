<div class="page"/>

# スマートテーブルインボイスモード

"スマートテーブルインボイスモード"は、以下のような処理を実行します。

* 送り状(invoice.json)の代わりに、同等の内容を含むExcel形式ファイル、CSVファイルまたはTSVファイルを使って自動的にinvoice.jsonを生成し、複数のデータを一度に登録する。
* 1つのデータタイルに登録されるファイルは、1個でも複数でも構わない。
* 構造化処理は、通常の"インボイスモード"の場合と同様に実装される。
* 1つのデータタイルに登録される送り状データは、Excel/CSV/TSVファイルの1行に記述される。
* 画面から入力された送り状データ(invoice.json)は、Excel/CSV/TSVファイルの内容で**上書き**される。Excel/CSV/TSVファイル内で指定がない(空欄の)項目は画面入力値が利用される。

## スマートテーブルインボイスモード利用時の開発概要

スマートテーブルインボイスモードを利用した構造化処理プログラムの開発は以下のような流れで実施します。

1. (通常の)インボイスモードとして構造化処理プログラムを開発する。
2. 上で作成した送り状設定ファイル(invoice.schema.json)の形式に合わせて、スマートテーブルインボイスファイル(xlsx形式、CSV形式、またはTSVファイル形式)を作成する。必要であればinvoice.jsonの内容を、このExcel/CSV/TSVファイルに転記する。
3. 上記スマートテーブルインボイスで指定したファイル(入力ファイル)を作成する。
4. スマートテーブルインボイスモードとしての動作確認

> 構造化処理プログラムとしては、"インボイスモード"と"スマートテーブルインボイスモード"での違いはありません。RDEToolKitが自動的に判別し、適切に前処理を実行してくれるためです。

## 準備

### RDEToolKitのインストールと初期フォルダ構成作成

『共通処理』の章で示したように、RDEToolKitのインストールと、初期フォルダ構成の作成を実施します。

```bash
$ cd ${HOME}/rde-sample
$ mkdir smarttableinvoice
$ cd smarttableinvoice/

$ source ~/venv/bin/activate

(venv) $ python -m rdetoolkit init
：
(venv) $ cd container
```

以下、双方が完了しているものとして進めます。

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

## インボイスモードの開発

エクセルインボイスモードの場合と同様、スマートテーブルインボイスモードの場合も最初は通常のインボイスモードとして開発します。

先に関連するファイル群を作成していきます。エクセルインボイスモードで利用したものを使用することにします。

### invoice.json

送り状ファイルを用意します。

data/invoice/invoice.json

```json
{
    "datasetId": "e751fcc4-b926-4747-b236-cab40316fc49",
    "basic": {
        "dateSubmitted": "2023-03-14",
        "dataOwnerId": "16b51e451d6be669bb93b551420ec005b1df55b83530316137643764",
        "dataName": "test_spe1",
        "instrumentId": null,
        "experimentId": "test_230606_1",
        "description": "test_today"
    },
    "custom": {
        "key1": "test1",
        "key2": "test2",
        "key3": "test3",
        "key4": "test4",
        "key5": "test5"
    },
    "sample": {
        "sampleId": "8ea50153-7a87-44b7-9d6e-10179c507039",
        "names": [
            "testdev01_1"
        ],
        "composition": "AB2C3",
        "referenceUrl": "test_ref",
        "description": null,
        "generalAttributes": [
            {
                "termId": "3adf9874-7bcb-e5f8-99cb-3d6fd9d7b55e",
                "value": "test_ABC"
            },
            {
                "termId": "e2d20d02-2e38-2cd3-b1b3-66fdb8a11057",
                "value": "b"
            },
            {
                "termId": "efcf34e7-4308-c195-6691-6f4d28ffc9bb",
                "value": "c"
            },
            {
                "termId": "7cc57dfb-8b70-4b3a-5315-fbce4cbf73d0",
                "value": "d"
            },
            {
                "termId": "1e70d11d-cbdd-bfd1-9301-9612c29b4060",
                "value": "e"
            },
            {
                "termId": "5e166ac4-bfcd-457a-84bc-8626abe9188f",
                "value": "f"
            },
            {
                "termId": "0d0417a3-3c3b-496a-b0fb-5a26f8a74166",
                "value": "g"
            }
        ],
        "ownerId": "de17c7b3f0ff5126831c2d519f481055ba466ddb6238666132316439"
    }
}
```

### invoice.schema.json

data/tasksupport/invoice.schema.json

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "NIMS_ELN_Process_Pulsed_laser_deposition_Magnetic_thin_film",
  "description": "固有情報と試料情報のスキーマ",
  "type": "object",
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
          "required": [],
          "properties": {
              "key1": {
                  "label": {
                      "ja": "キー1",
                      "en": "key1"
                  },
                  "type": "string"
              },
              "key2": {
                  "label": {
                      "ja": "キー2",
                      "en": "key2"
                  },
                  "type": "string"
              },
              "key3": {
                  "label": {
                      "ja": "キー3",
                      "en": "key3"
                  },
                  "type": "string"
              },
              "key4": {
                  "label": {
                      "ja": "キー4",
                      "en": "key4"
                  },
                  "type": "string"
              },
              "key5": {
                  "label": {
                      "ja": "キー5",
                      "en": "key5"
                  },
                  "type": "string"
              }
          }
      },
      "sample": {
        "type": "object",
        "label": {
            "ja": "試料情報",
            "en": "Sample Information"
        },
        "required": [
        ],
        "properties": {
            "generalAttributes": {
                "type": "array",
                "items": [
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "3adf9874-7bcb-e5f8-99cb-3d6fd9d7b55e"
                            }
                        }
                    },
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "e2d20d02-2e38-2cd3-b1b3-66fdb8a11057"
                            }
                        }
                    },
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "efcf34e7-4308-c195-6691-6f4d28ffc9bb"
                            }
                        }
                    },
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "7cc57dfb-8b70-4b3a-5315-fbce4cbf73d0"
                            }
                        }
                    },
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "1e70d11d-cbdd-bfd1-9301-9612c29b4060"
                            }
                        }
                    },
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "5e166ac4-bfcd-457a-84bc-8626abe9188f"
                            }
                        }
                    },
                    {
                        "type": "object",
                        "required": [
                            "termId"
                        ],
                        "properties": {
                            "termId": {
                                "const": "0d0417a3-3c3b-496a-b0fb-5a26f8a74166"
                            }
                        }
                    }
                ]
            }
        }
    }
  }
}
```

### metadata-def.json

metadata-def.jsonを以下の内容で作成します。

data/tasksupport/metadata-def.json

```json
{
    "file_size": {
        "name": {
            "ja": "ファイルサイズ",
            "en": "file size"
        },
        "schema": {
            "type": "number"
        },
        "description": "ファイルサイズ",
        "variable":1,
        "unit" : "byte"
    },
    "file_created": {
        "name": {
            "ja": "ファイル作成日",
            "en": "file created at"
        },
        "schema": {
            "type": "string",
            "format": "date"
        },
        "description": "ファイル作成日"
    },
    "file_name": {
        "name": {
            "ja": "ファイル名",
            "en": "file name"
        },
        "schema": {
            "type": "string"
        },
        "description": "ファイル名"
    }
}
```

続いて、構造化処理を実行するようにプログラムを作成していきます。

> エクセルインボイスモードの時と同様、ここではサンプルを表示することを主題にしていますので、一部の処理が意味の無い処理のままとなっています。

### modules/interfaces.py

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
        self, save_path: Path, meta: Meta, *, const_meta_info: MetaType | None = None, repeated_meta_info: RepeatedMetaType | None = None,
    ) -> Any:
        raise NotImplementedError


class IGraphPlotter(Generic[T], ABC):
    @abstractmethod
    def plot(self, data: T, save_path: Path, *, title: str | None = None, xlabel: str | None = None, ylabel: str | None = None) -> Any:  # noqa: D102
        raise NotImplementedError
```

### modules/inputfile_handler.py

```python
from __future__ import annotations

from pathlib import Path
from typing import Any
from datetime import datetime

import pandas as pd
from rdetoolkit.models.rde2types import MetaType

from modules.interfaces import IInputFileParser


class FileReader(IInputFileParser):
    def read(self, srcpath: Path) -> tuple[MetaType, pd.DataFrame]:
        file_size = srcpath.stat().st_size
        file_created = datetime.fromtimestamp(srcpath.stat().st_mtime).strftime("%Y-%m-%d")
        self.data = pd.DataFrame([[1, 11], [2, 22], [3, 33]]) # Caution! dummy data
        self.meta: dict[str, str | int | float | list[Any] | bool] = {
            "file_size": file_size,
            "file_created": file_created,
            "file_name": srcpath.name
        }

        return self.meta, self.data
```

> 上記例では、ソース中のコメントにもあるように、返却されるdataは(固定の)ダミーデータです。

### modules/meta_handler.py

```python
from __future__ import annotations

from pathlib import Path
from typing import Any

from rdetoolkit import rde2util
from rdetoolkit.models.rde2types import MetaType, RepeatedMetaType

from modules.interfaces import IMetaParser


class MetaParser(IMetaParser[MetaType]):
    def parse(self, data: MetaType) -> tuple[MetaType, RepeatedMetaType]:
        """Parse and extract constant and repeated metadata from the provided data."""
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
        """Save parsed metadata to a file using the provided Meta object."""
        if const_meta_info is None:
            const_meta_info = self.const_meta_info
        if repeated_meta_info is None:
            repeated_meta_info = self.repeated_meta_info
        metaobj.assign_vals(const_meta_info)
        metaobj.assign_vals(repeated_meta_info)

        # dummy return
        return metaobj.writefile(str(save_path))
```

> 上記例では、parseメソッドは全くのダミーです。

### modules/graph_handler.py

```python
from __future__ import annotations

from pathlib import Path

import pandas as pd

from modules.interfaces import IGraphPlotter


class GraphPlotter(IGraphPlotter[pd.DataFrame]):
    def plot(self, data: pd.DataFrame, save_path: Path, *, title: str | None = None, xlabel: str | None = None, ylabel: str | None = None) -> pd.DataFrame:
        return data  # dummy return value for testing purposes
```

> 上記例では、plotメソッドはダミー実装であり、実際には何も実行されません。

### modules/structured_handler.py

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

### modules/datasets_process.py

上で記述した処理を使うようにdatasetメソッドを記述します。

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeDatasetPaths
from rdetoolkit.rde2util import Meta

from modules.inputfile_handler import FileReader
from modules.meta_handler import MetaParser
from modules.graph_handler import GraphPlotter
from modules.structured_handler import StructuredDataProcessor


class CustomProcessingCoordinator:
    def __init__(
        self,
        file_reader: FileReader,
        meta_parser: MetaParser,
        graph_plotter: GraphPlotter,
        structured_processer: StructuredDataProcessor
    ):
        self.file_reader = file_reader
        self.meta_parser = meta_parser
        self.graph_plotter = graph_plotter
        self.structured_processer = structured_processer


def custom_module(paths: RdeDatasetPaths) -> None:
    co = CustomProcessingCoordinator(
             FileReader(),
             MetaParser(),
             GraphPlotter(),
             StructuredDataProcessor()
         )

    # Read Input File
    inputfile = paths.rawfiles[0] # Only one input file is expected
    meta, df_data = co.file_reader.read(inputfile)

    # Save csv
    co.structured_processer.to_csv(
        df_data,
        paths.struct / "sample.csv"
    )

    # Meta
    co.meta_parser.parse(meta)
    co.meta_parser.save_meta(
        paths.meta / "metadata.json",
        Meta(paths.tasksupport / "metadata-def.json")
    )

    # Graph
    co.graph_plotter.plot(
        df_data,
        paths.main_image / "sample.png"
    )

@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(paths: RdeDatasetPaths) -> None:
    custom_module(paths)
```

> 上記例では、ファイル名をプログラム内で固定で記述するなど、実際の構築の際にはそうするべきでない実装が含まれています。

### main.py

```python
import rdetoolkit

from modules import datasets_process


def main() -> None:
    rdetoolkit.workflows.run(
        custom_dataset_function=datasets_process.dataset,
    )

if __name__ == "__main__":
    main()
```

この状態でダミーファイルを用意し実行すると、以下の様になります。

```bash
touch data/inputdata/sample-data.txt
python main.py 
```

> インボイスファイルを入力としていませんので、この時点では"インボイスモード"となっています。

実行の結果、以下の様に入力ファイルをnonshared_rawフォルダにコピーする処理だけが実施されます。グラフ作成処理がダミーなので、イメージ系のファイル作成はありません。

```bash
((venv) $ tree data
data
├── attachment
├── inputdata
│   └── sample-data.txt
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
│   └── rdesys_20260630_112129.log
├── main_image
├── meta
│   └── metadata.json
├── nonshared_raw
│   └── sample-data.txt
├── other_image
├── raw
├── structured
│   └── sample.csv
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
└── thumbnail

15 directories, 8 files
```

> 他の箇所と同様に、ログファイル名は実行時間によりますので、ファイル名は異なります。

metadata.jsonファイルを確認してみます。

```bash
(venv) $ cat data/meta/metadata.json 
{
    "constant": {
        "file_created": {
            "value": "2026-06-30"
        },
        "file_name": {
            "value": "sample-data.txt"
        }
    },
    "variable": [
        {
            "file_size": {
                "value": 0,
                "unit": "byte"
            }
        }
    ]
}
```

意図通りに、ファイル名やファイルサイズが取得できていることが確認できます。

> 本ファイルの末尾に改行がありませんので、実際に上記コマンドを実行すると最後尾の`{`に続いてプロンプトが表示されます。

## スマートテーブルインボイスモードの開発

> data/inputdata/フォルダに、条件を満たすファイルが存在する場合、自動的に"スマートテーブルインボイスモード"となります。従って、`スマートテーブルインボイスモード`を使うために、「main.pyを修正してConfig()を使ったモード指定する」といった**追加指定は不要**です。

### テスト用データ配置

通常のインボイスモードでの開発が済んでいるので、スマートテーブルインボイスでの確認を進めます。

フォルダ構成を初期化します。

```bash
./reinit.sh 
```

上で使用したデータファイル(`sample-data.txt`)は、不要ですので削除します。

```bash
rm data/inputdata/sample-data.txt
```

また、invoice.jsonファイルは、以下の構造化処理プログラム実行で上書きされますので、念のためバックアップを取っておきます。

```bash
cp data/invoice/invoice.json data/invoice/invoice.json.orig
```

> このコピーからの戻し処理は、必要に応じて実施してください。

スマートテーブルインボイスモードのテスト用のデータを作成します。

```bash
echo '0000' > data0000.dat
echo '0001' > data0001.dat
$ echo '0002' > data0002.dat

zip data.zip ./data0000.dat ./data0001.dat ./data0002.dat
```

以下の様なファイルが作成されます。

```bash
(venv) $ ls -l data.zip
-rw-rw-r-- 1 devel devel 493  6月 30 11:58 data.zip
```

> zipコマンドがない場合は、`sudo apt install zip`にてインストールしてから作業してください。

所定の場所に配置します。

```bash 
cp data.zip data/inputdata/
```

以下の様になります。

```bash
(venv) $ ls -l data/inputdata/
total 4
-rw-rw-r-- 1 devel devel 493  6月 30 11:59 data.zip
```

> cpコマンドにて新規にファイルをコピーしているため、ファイルの最終更新日時に差異が生じます。

構造化処理プログラムが、`インボイスモード`か`スマートテーブルインボイスモード`かのいずれで実行されるかを判断するポイントは、`data/inputdata`フォルダに、ファイル名の先頭が`smarttable_`という接頭辞がついたExcel/CSV/TSVファイルが存在するかどうか、です。
当該ファイルが存在すればスマートテーブルインボイスモードとして稼働し、存在しなければ通常のインボイスモードとして稼働します。

### スマートテーブルインボイスファイルの作成

スマートテーブルインボイスファイルを作成していきます。

前述の様に、スマートテーブルインボイスファイルとしては、Excel/CSV/TSVファイルいずれの形式でも利用可能ですが、ここではCSV形式を選択します。

ファイル名も`smarttable_`から始まれば、他には特に制限がないので、`smarttable_test.csv`として作成します。

data/inputdata/smarttable_test.csv

```csv
# for smarttableinvoice mode,,,
basic/dataName,inputdata1,sample/names,custom/key1
データ0,data0000.dat,Inu,あ1
データ1,data0001.dat,Neko,あ2
データ2,data0002.dat,,あ3
```

> 上記CSVファイルは、説明のため、必要最小限の項目だけを含めています。

スマートテーブルインボイスファイルは、2行ヘッダー形式を採用しています。1行目に表示名、2行目にマッピングキーを配置することが想定されています。

例:

```
データ名,入力ファイル1,サイクル,厚さ,温度,試料名,試料ID,一般項目
basic/dataName,inputdata1,custom/cycle,custom/thickness,custom/temperature,sample/names,sample/sampleId,sample/generalAttributes.3adf9874-7bcb-e5f8-99cb-3d6fd9d7b55e
: (3行目以降は、実際のインボイスデータを、必要な数だけセット)
```

ただし、1行目はユーザー向けの説明用なので、上の`smarttable_test.csv`のように"# for smarttableinvoice mode"といったファイルの説明として利用しても問題ありません。つまり1行目は構造化処理プログラムでは無視されます。

> 1行目のの先頭の"#"は、ユーザ向けにコメントであることを明示しているだけです。実際には先頭の"#"が無くても処理に影響はありません。

2行目はマッピングキーをセットします。この行を読み取って、3行目以降の当該列の内容が、invocie.jsonやメタデータ等に自動でマッピングされます。

basic/、custom/、sample/といったプレフィックスにより、どのデータにセットされるかが決まります。マッピングされるルールは以下の様になっています。

| # | 設定内容 | マッピングルール |
| ---- | ---- | ---- |
| 1 | basic/<invoice.jsonのキー名> | invoice.jsonのbasicセクションにマッピングされます。 |
| 2 | custom/<invoice.jsonのキー名> | invoice.jsonのcustomセクションにマッピングされます。 |
| 3 | sample/<invoice.jsonのキー名> | invoice.jsonのsampleセクションにマッピングされます。 |
| 4 | sample/generalAttributes.<termId> | generalAttributes配列内の該当するtermIdのvalueにマッピングされます。 |
| 5 | sample/specificAttributes.<classId>.<termId> | specificAttributes配列内の該当するclassIdとtermIdのvalueにマッピングされます。|
| 6 | meta/<metadata-defのキー> | metadata-def.jsonの定義に従ってmetadata.jsonのconstantセクションへ書き込み（schema.typeで型変換し、unitがあれば付与）。variable定義は現時点では対応しません。 |
| 7 | inputdataX | zipファイル内のファイルパスを指定 (X=1,2,3...) |

> いずれも、存在しないキーを指定した場合は、StructuredErrorとなります。キーの値を確認してください。

#### ファイル形式の選択

前述の様に、スマートテーブルインボイスファイルとしてExcel形式、CSV形式およびTSV形式が利用できます。

いずれを利用しても問題ありませんが、以下のような基準でファイル形式を決めてください。

* 通常はCSV形式を利用します。Excelなどの表計算アプリの他、テキストエディタでも編集できますので、手軽に利用できます。
* 設定する値にカンマ(,)が含まれる場合は、TSV形式を選択してください。値をダブルクォーテーションで囲むなどの追加処理なしで指定できます。
* 項目数や行数が多い場合はExcel形式を選択します。ただし、その場合でもExcelアプリを使う場合にはほぼ同じ操作感となるためCSV形式を利用することもできます。
* 動的に値を決める場合、例えば別のシート上の値を参照する場合や、計算により値を求めるといった場合はExcel形式を選択します。

#### 文字コード

* CSV形式やTSV形式のスマートテーブルインボイスファイルを作成する場合で、かつ、その内容に日本語の文字列を含む場合、文字コードは"UTF-8"として保存してください。それ以外の文字コードを利用した場合、エラーとなります。

```bash
(venv) $ python main.py 
:
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x83 in position 106: invalid start byte
:
:
Exception Type: StructuredError
Error: Failed to read SmartTable file data/inputdata/smarttable_test.csv: 'utf-8' codec can't decode byte 0x83 in position 106: invalid start byte
```

> Excelアプリなどを利用して生成/修正する場合、CSVファイル出力時に文字コード('UTF-8')を指定して出力するようにしてください。
>
> Excel形式のファイルを利用する場合は、文字コードを意識する必要はありません。

#### 指定するスマートテーブルインボイスファイルは1つだけ

1回の構造化処理プログラム内で使用できるスマートテーブルは1つだけです。

```bash
(venv) $ ls -1 data/inputdata/
:
smarttable_test.csv
smarttable_test_3.tsv
```

上記の様にして、構造化処理プログラムを実行すると、以下のようなエラーとなります。

```bash
(venv) $ python main.py
:
Exception Type: StructuredError
Error: Multiple SmartTable files found: ['smarttable_test_3.tsv', 'smarttable_test.csv']. Only one SmartTable file is allowed.
```

スマートテーブルインボイスファイルが1つだけの状態にする必要があります。

```
(venv) $ rm data/inputdata/smarttable_test_3.tsv
(venv) $ python main.py
(venv) $
```

#### Excel形式を利用する場合の注意点

スマートテーブルインボイスファイルとしてExcel形式を使用する場合、以下に注意してください。

* シート名は任意でよい。
* シートは複数あってもよいが、利用するシートは左端のシートのみとなる。

左端のシートが、スマートテーブルインボイスファイルの形式に則っていない場合は以下のようなエラーが表示されます。

```
Exception Type: StructuredError
Error: SmartTable file must have mapping keys with prefixes: basic/, custom/, sample/, meta/, inputdata
```

* 左端のシート内の値が、他のシートへの参照を含んでいても良い。

### rawfilesの扱い

出力されたmetadata.json (たとえば`data/divided/0001/meta/metadata.json`)を確認すると以下の様になっている場合があります。

```json
{
    "constant": {
        "file_created": {
            "value": "2026-07-02"
        },
        "file_name": {
            "value": "fsmarttable_test_0000.csv" ★★★ 
        }
    },
    "variable": [
        {
            "file_size": {
                "value": 8,
                "unit": "byte"
            }
        },
        {
            "file_size": {
                "value": 4,
                "unit": "byte"
            }
        }
    ]
}
```

"★"で示した`file_name"の値が、入力ファイル(スマートテーブルインボイスファイルにおいて、表題に`inputdata1`が指定された列のデータ)として指定したファイルとは異なるファイルとなっています。

RDEToolKit v1.7.0より前のバージョンでは、スマートテーブルインボイスファイルで指定した送り状データが、それぞれの行毎に分割され新たなファイル(中間ファイル)として生成され、rawfiles配列に含まれているためです。

RDEToolKit v1.7.0以降では、そういった中間ファイルはrawfiles配列には含まれなくなりました。(上記の動きで不都合がある場合は)RDEToolKit v1.7.0以降の最新版をお使いください。

### スマートテーブルインボイスファイルをRDEに登録する

スマートテーブルインボイスファイル(Excel/CSV/TSV)は、構造化処理プログラム内で利用されますが、RDE登録の対象とはなりません。

スマートテーブルインボイスファイルそのものをRDE上に登録する場合は、設定値`smarttable.save_table_file'に"true"を設定する必要があります。

設定ファイルを使って設定する場合、`data/tasksupport/rdeconfig.yml`ファイルに以下を追記します。

```yml
smarttable:
    save_table_file: true
```

main.py内で設定する場合は、以下の様にします。

```python
import rdetoolkit
from rdetoolkit.config import Config, SmartTableSettings

from modules import datasets_process


def main():
    config = Config(
        smarttable=SmartTableSettings(
            save_table_file=True,
        ),
    )

    rdetoolkit.workflows.run(
        config=config,
        custom_dataset_function=datasets_process.dataset
    )

if __name__ == '__main__':
    main()
```

この設定により、登録されるデータタイルが1つ追加され、追加されたデータタイルにスマートテーブルインボイスファイルが格納されます。

> そのデータタイルの`basic/dataName`には、スマートテーブルインボイスファイル名がセットされます。
>
> スマートテーブルインボイスファイルは、登録される一連のデータタイルの先頭に登録されます。

次に進む前に、このオプションを追加前に戻します。

```bash
rm data/tasksupport/rdeconfig.yml
```

> 他の用途でrdeconfig.ymlを利用している場合は、ファイルを消さずに、その設定内容だけを消してください。
>
> あるいは設定される値を`false`に変更しても構いません。

### 複数のデータファイル

複数のデータタイル、つまりスマートテーブルインボイスの複数行で共通するファイルを指定する場合は、2つめのデータファイルを提供しインプットファイルとして利用することができます。

> 1つのデータファイル中にすべてのファイルを含める形でも問題ありません。

```bash
echo -e "[common]\nDATE = 2025-12-25" > common.dat
zip common.zip ./common.dat 
```

このファイルを`inputdata`フォルダに用意します。

```bash
cp ./common.zip data/inputdata
```

スマートテーブルインボイスファイルの2行目に`,inputdata2`を追加し、3行目以降にcommon.datを追記していきます。

```csv
# for smarttableinvoice mode,,,
basic/dataName,inputdata1,sample/names,custom/key1,custom/key2,inputdata2
データ0,data0000.dat,Inu,あ1,enami1,common.dat
データ1,data0001.dat,Neko,あ2,enami2,common.dat
データ2,data0002.dat,,あ3,enami3,common.dat
```

この状態で、構造化処理プログラムを実行します。

```bash
python main.py
```

結果以下の様になります。

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
│   │   │   └── metadata.json
│   │   ├── nonshared_raw
│   │   │   ├── common.dat
│   │   │   └── data0000.dat
│   │   ├── other_image
│   │   ├── raw
│   │   ├── structured
│   │   │   └── sample.csv
│   │   ├── temp
│   │   └── thumbnail
│   └── 0002
│       ├── attachment
│       ├── invoice
│       │   └── invoice.json
│       ├── invoice_patch
│       ├── logs
│       ├── main_image
│       ├── meta
│       │   └── metadata.json
│       ├── nonshared_raw
│       │   ├── common.dat
│       │   └── data0001.dat
│       ├── other_image
│       ├── raw
│       ├── structured
│       │   └── sample.csv
│       ├── temp
│       └── thumbnail
├── inputdata
│   ├── common.zip
│   ├── data.zip
│   └── smarttable_test.csv
├── invoice
│   ├── invoice.json
│   └── invoice.json.orig
├── invoice_patch
├── logs
│   └── rdesys_20260702_103109.log
├── main_image
├── meta
│   └── metadata.json
├── nonshared_raw
│   ├── common.dat
│   └── data0002.dat
├── other_image
├── raw
├── structured
│   └── sample.csv
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
│   ├── common.dat
│   ├── data0000.dat
│   ├── data0001.dat
│   ├── data0002.dat
│   ├── fsmarttable_test_0000.csv
│   ├── fsmarttable_test_0001.csv
│   └── fsmarttable_test_0002.csv
└── thumbnail

42 directories, 29 files
```

上記から、以下のようなことが行われることが確認できます。
* 2つのzipファイルが展開されて`data/temp`フォルダに存在する
* スマートテーブルインボイスファイルで指定された内容が、1行毎に別々のファイルに分割されて`fsmarttable_`で始まるファイルとして存在する
* スマートテーブルインボイスファイルで指定されたinputdata1とinputdata2で指定したファイルが、それぞれの`nonshared_raw`フォルダに存在する

続いて、送り状毎に入力ファイルをzip圧縮する場合を考えます。

```bash
echo '000'> data0000.dat 
echo '05-03' > common.dat
zip data0.zip data0000.dat common.dat

echo '001' > data0001.dat
echo '05-04' > common.dat
zip data1.zip data0001.dat common.dat

echo '002' > data0002.dat
echo '05-05' > common.dat
zip data2.zip data0002.dat common.dat
```

> それぞれのzipファイルには、それぞれ内容が異なる、同名のファイルが格納されている状態です。

不要なファイルを消して、上で作成したzipファイルをコピーします。

```bash
rm data/inputdata/*.zip
cp data0.zip data1.zip data2.zip data/inputdata
```

スマートテーブルインボイスファイルは、上と同じ`smarttable_test.csv`を使います。

構造化処理プログラムを実行します。

```
./reinit.sh
python main.py
```

`common.dat`の内容を確認してみます。

```bash
(venv) $ cat data/divided/0001/nonshared_raw/common.dat
05-03
(venv) $ cat data/divided/0002/nonshared_raw/common.dat
05-03
(venv) $ cat data/nonshared_raw/common.dat
05-03
```

それぞれのzipファイルに、異なる内容のcommon.datファイルを配置しましたが、構造化処理タスク終了後はすべて同じ内容のファイルとなっています。

このことから、複数のzipにある同名ファイルは、最初に展開されるものが残り、それ以外のzipにあるファイルでの**上書きはされない**ことが分かります。

> 必要な場合は、フォルダを分けてファイルを格納し、フォルダ毎zipファイルに格納、スマートテーブルインボイスファイル内で展開後のフォルダ毎指定するようにしてください。

### 試料情報登録

スマートテーブルインボイスファイル内で、試料の名前(`sample/names`)が指定された場合は、新規に試料が登録されます(エクセルインボイスモードと同じ挙動)。

スマートテーブルインボイスモードの invoice.json 生成は、元の invoice.json をベースに各行の値を部分上書きする方式です。そのため、スマートテーブルインボイスファイル内でsample 系の列を何も指定しないと、元の invoice.json に入っている試料情報がそのまま残ります。

この挙動を踏まえ、スマートテーブルインボイスモードでは sample/names 列が指定され、かつ sample/sampleId 列に有効な値がない場合、その行は 新規試料として登録されます。このとき、元の invoice.json から継承されたダミー試料の各フィールドが自動的にクリアされます。

本挙動の詳細については、以下を参照ください。

* https://github.com/nims-mdpf/rdetoolkit/blob/main/docs/usage/mode/mode_smarttableinvoice.ja.md (日本語)
* https://github.com/nims-mdpf/rdetoolkit/blob/main/docs/usage/mode/mode_smarttableinvoice.en.md (英語)

### スマートテーブルインボイスモード専用機能

構造化処理プログラムを、通常のインボイスモードを使わずに、スマートテーブルインボイスモード限定にする場合以下の変数を利用することができます。

* paths.output_paths.smarttable_rawfile
* paths.output_paths.smarttable_row_data

(引数を`paths: RdeDatasetPaths`とした場合)

#### smarttable_rawfile

smarttable_rawfileには、当該行のみ取り出したCSVファイルのPosixPathオブジェクトが格納されます。

```python
smarttable_rowfile=PosixPath('data/temp/fsmarttable_test_0002.csv'),
```

CSVファイルを直接利用する場合は、こちらを利用します。

#### smarttable_row_data

smarttable_row_dataには、当該行の情報をセットしたdictオブジェクトが格納されます。

```python
smarttable_row_data={'basic/dataName': 'データ2',
    'custom/key1': 'あ3',
    'custom/key2': 'い3',
    'inputdata1': 'data0002.dat',
    'inputdata2': 'common.dat',
    'sample/names': nan},
```

上記の例では、'sample/names'欄に値が入っていないため、"nan"と表示されています。

> 構造化処理プログラム内では、WebUIでの入力値が利用されます。
