<div class="page"/>

# エクセルインボイスモード

"エクセルインボイスモード"は、以下のような処理を実行します。

* 送り状(invoice.json)の代わりに、同等の内容を含むExcel形式ファイルを使って、複数のデータを一度に登録する。
* 1つのデータタイルに登録されるファイルは、1個でも複数でも構わない。ただし、複数個の場合は、zip圧縮して1つのファイルにしておく必要がある。
* 構造化処理は、通常の"インボイスモード"の場合と同様に実装される。
* 1つのデータタイルに登録される送り状データは、Excelファイルの1行に記述される。
* 画面から入力された送り状データ(invoice.json)は、一部を除き**利用されない**。

## エクセルインボイスモード利用時の開発概要

エクセルインボイスモードを利用した構造化処理プログラムの開発は以下のような流れで実施します。

1. (通常の)インボイスモードとして構造化処理プログラムを開発する。
2. 上で作成した送り状設定ファイル(invoice.schema.json)の形式に合わせて、エクセルインボイスファイル(xlsx形式)を作成する。必要であればinvoice.jsonの内容を、このExcelファイルに転記する。
3. 上記エクセルインボイスで指定したファイル(入力ファイル)を作成する。
4. エクセルインボイスモードとしての動作確認

> 構造化処理プログラムとしては、"インボイスモード"と"エクセルインボイスモード"での違いはありません。RDEToolKitが自動的に判別し、適切に前処理を実行してくれるためです。

## 準備

### RDEToolKitのインストールと初期フォルダ構成作成

『共通処理』の章で示したように、RDEToolKitのインストールと、初期フォルダ構成の作成を実施します。

```bash
$ cd ${HOME}/rde-sample
$ mkdir excelinvoice
$ cd excelinvoice/

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

## インボイスモードの開発 (その1)

"その1"として、'Hello RDE!'を表示するだけの構造化処理プログラムを作成実行することにより、入力ファイルが非公開(または公開)フォルダにコピーされるところまでを実装、確認します。

### テスト用データ配置

テスト用のデータとして、以下を配置します。

* sample-data.txt (0KB)

0バイトのファイルですので、`touch`コマンドにより設置します。

```bash
(venv) $ touch data/inputdata/sample-data.txt
```

実際に格納されると、以下の様になります。

```bash
(venv) $ ls -l data/inputdata/
total 0
-rw-r--r-- 1 devel devel 0  5月 16 07:55 sample-data.txt
```

> 上記サンプルデータは"0バイト"のファイルです。この例ではファイルの読み込み処理を行わないダミー実装のためです。`nonshared_raw/`フォルダへのコピーのみ実行されます。

### 構造化処理作成

ここから、実際に構造化処理プログラムを作成していきます。

#### main.py

初期化にて作成されたmain.pyを以下のように変更します。

main.py

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

### modules/datasets_process.py

modules/datasets_process.py

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath


@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    print('Hello RDE!')
```

実行してみます。

```bash
(venv) $ python main.py
Hello RDE!
```

想定通りに表示されました。

出力データを確認します。

```bash
(venv) $ tree data
data
├── attachment
├── inputdata
│   └── sample-data.txt
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
├── main_image
├── meta
├── nonshared_raw
│   └── sample-data.txt
├── other_image
├── raw
├── structured
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
└── thumbnail

15 directories, 5 files
```

`sample-data.txt`が、想定通り`nonshared_raw/`フォルダにコピーされていることが確認できます。

## インボイスモードの開発 (その2)

次に、実働する構造化処理プログラムを作成し、送り状ファイル(invoice.json)などの関連するファイルを作成していきます。

先に関連するファイル群を作成していきます。

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
        "common_data_origin": "experiments",
        "common_technical_category": "measurement",
        "measurement_method_category": "散乱_回折",
        "measurement_method_sub_category": "X線回折",
        "measurement_analysis_field": null,
        "measurement_measurement_environment": null,
        "measurement_energy_level_transition_structure_etc_of_interst": null,
        "measurement_measured_date": "2017-11-21",
        "measurement_standardized_procedure": null,
        "measurement_instrumentation_site": null,
        "measurement_reference": null,
        "measurement_temperature": null,
        "sample_holder_name": null,
        "key2": "test2",
        "key3": "test3",
        "key4": "test4",
        "key5": "test5",
        "key6": "test6",
        "key7": "test7",
        "key8": "test8",
        "key9": "test9",
        "key10": "test10",
        "flood_gun_voltage": 1,
        "flood_gun_emission_current": 0,
        "ion_gun_voltage": 0,
        "analysis_field": "test_myfield",
        "sample_position": "test_#1",
        "sample_condition": "test_condition",
        "no3dimage": 0
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

一部の項目は、`invoice.schema.json`では**ない**ファイルにて定義されています。例えば`"ownerId": "test_owner"`のような指定をすると、バリデーションエラーとなります。


これは`(RDEToolKitのインストールフォルダ)/static/invoice_basic_and_sample.schema_.json`で指定している、以下の内容に合致していないために出力されます。

```json
：
                    "ownerId": {
                        "description": "sample owner id",
                        "type": "string",
                        "pattern": "^([0-9a-zA-Z]{56})$"
                    },
：
```

`invoice.schema.json`の指定と比べて問題がないのにエラーが発生する場合は、上記ファイルの設定と合致していない可能性がありますので、そちらも合わせて確認してください。

### invoice.schema.json

> 別途提供されている"Excelファイルからjson生成"するツールで作成した`invoice.schema.json`は"description"部分が入らない(→ "None"が指定される)ので、出来上がったjsonファイルを手動にて修正する必要があります(下記は修正済み)。

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
              "date": {
                  "label": {
                      "ja": "日付",
                      "en": "Date"
                  },
                  "type": "string"
              },
              "substrate": {
                  "label": {
                      "ja": "基板",
                      "en": "Substrate"
                  },
                  "type": "string"
              },
              "target": {
                  "label": {
                      "ja": "層構造",
                      "en": "Target"
                  },
                  "type": "string"
              },
              "tg": {
                  "label": {
                      "ja": "成膜温度",
                      "en": "Tg"
                  },
                  "type": "number",
                  "options": {
                      "unit": "C"
                  }
              },
              "po2": {
                  "label": {
                      "ja": "成膜酸素圧",
                      "en": "PO2"
                  },
                  "type": "number",
                  "options": {
                      "unit": "Torr"
                  }
              },
              "laser_fluence": {
                  "label": {
                      "ja": "レーザー強度",
                      "en": "Laser fluence"
                  },
                  "type": "number",
                  "options": {
                      "unit": "mJ/pulse"
                  }
              },
              "spot_size": {
                  "label": {
                      "ja": "レーザースポットサイズ",
                      "en": "Spot Size"
                  },
                  "type": "number",
                  "options": {
                      "unit": "mm^2"
                  }
              },
              "frequency": {
                  "label": {
                      "ja": "レーザー周波数",
                      "en": "Frequency"
                  },
                  "type": "number",
                  "options": {
                      "unit": "Hz"
                  }
              },
              "pulse_count": {
                  "label": {
                      "ja": "成膜パルス数",
                      "en": "Pulse Count"
                  },
                  "type": "string"
              },
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
              },
              "common_data_type": {
                  "label": {
                      "ja": "登録データタイプ",
                      "en": "Data Type"
                  },
                  "type": "string",
                  "default": "ELN"
              },
              "common_data_origin": {
                  "label": {
                      "ja": "データの起源",
                      "en": "Data Origin"
                  },
                  "type": "string",
                  "default": "experiments"
              },
              "common_technical_category": {
                  "label": {
                      "ja": "技術カテゴリー",
                      "en": "Technical Category"
                  },
                  "type": "string",
                  "default": "process"
              },
              "common_reference": {
                  "label": {
                      "ja": "参考文献",
                      "en": "Reference"
                  },
                  "type": "string"
              },
              "process_processed_date": {
                  "label": {
                      "ja": "処理年月日",
                      "en": "Processed date"
                  },
                  "type": "string",
                  "format": "date"
              },
              "process_process_category": {
                  "label": {
                      "ja": "プロセスカテゴリー",
                      "en": "Process category"
                  },
                  "type": "string",
                  "default": "蒸着_成膜"
              },
              "process_process_technique": {
                  "label": {
                      "ja": "プロセス手法",
                      "en": "Process technique"
                  },
                  "type": "string",
                  "default": "パルスレーザ堆積"
              },
              "process_processing_environment": {
                  "label": {
                      "ja": "処理環境",
                      "en": "Processing environment"
                  },
                  "type": "string",
                  "default": "真空中"
              },
              "process_key_temperature": {
                  "label": {
                      "ja": "主となる処理温度",
                      "en": "Key temperature"
                  },
                  "type": "number",
                  "options": {
                      "unit": "C"
                  }
              },
              "process_instrumentation_site": {
                  "label": {
                      "ja": "装置設置場所",
                      "en": "Instrumentation site"
                  },
                  "type": "string"
              },
              "process_standardized_procedure": {
                  "label": {
                      "ja": "標準手順",
                      "en": "Standardized procedure"
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

> "file_size"を繰り返し有りメタデータ項目として定義しています。入力ファイルは1つですので本来は通常のメタデータ項目ですが、ここでは例を示す意味で"繰り返し有り"としています。繰り返しのないメタデータ項目として扱う場合は"variable: 1"の行を削除します。

続いて、構造化処理を実行するようにプログラムを作成していきます。

> ここではサンプルを表示することを主題にしていますので、一部の処理が意味の無い処理のままとなっています。

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

最後に、上で記述した処理を使うようにdatasetメソッドを記述します。

> 元からある内容を、下記内容で上書きします。

```python
from rdetoolkit.errors import catch_exception_with_message
from rdetoolkit.models.rde2types import RdeInputDirPaths, RdeOutputResourcePath
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


def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    co = CustomProcessingCoordinator(
             FileReader(),
             MetaParser(),
             GraphPlotter(),
             StructuredDataProcessor()
         )

    # Read Input File
    inputfile = resource_paths.rawfiles[0] # Only one input file is expected
    meta, df_data = co.file_reader.read(inputfile)

    # Save csv
    co.structured_processer.to_csv(
        df_data,
        resource_paths.struct / "sample.csv"
    )

    # Meta
    co.meta_parser.parse(meta)
    co.meta_parser.save_meta(
        resource_paths.meta / "metadata.json",
        Meta(srcpaths.tasksupport / "metadata-def.json")
    )

    # Graph
    co.graph_plotter.plot(
        df_data,
        resource_paths.main_image / "sample.png"
    )


@catch_exception_with_message(error_message="ERROR: failed in data processing")
def dataset(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    custom_module(srcpaths, resource_paths)
```

> 上記例では、ファイル名をプログラム内で固定で記述するなど、実際の構築の際にはそうするべきでない実装が含まれています。

この状態で実行すると、以下の様になります。

```bash
(venv) $ python main.py 
(venv) $ 
```

> Excelインボイスファイルを入力としていませんので、この時点では"インボイスモード"となっています。

結果、以下の様に入力ファイルをrawフォルダにコピーする処理だけが実施されます。グラフ作成処理がダミーなので、イメージ系のファイル作成はありません。

```bash
(venv) $ tree data
data
├── attachment
├── inputdata
│   └── sample-data.txt
├── invoice
│   └── invoice.json
├── invoice_patch
├── logs
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

15 directories, 7 files
```

## エクセルインボイスモードの開発

> data/inputdata/フォルダに、条件を満たすExcel形式ファイルが存在する場合、自動的に"エクセルインボイスモード"となります。従って、`エクセルインボイスモード`を使うために、main.pyを修正してConfig()を使ったモード指定するといった**追加指定は不要**です。

### テスト用データ配置

通常のインボイスモードでの開発が済んでいるので、エクセルインボイスでの確認を進めます。

フォルダ構成を初期化します。

```bash
(venv) $ ./reinit.sh 
./data/divided was removed
./data/logs was removed
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

上で使用したデータファイル(`sample-data.txt`)は、不要ですので削除します。

```bash
rm data/inputdata/sample-data.txt
```

また、invoice.jsonファイルは、以下の構造化処理プログラム実行で上書きされますので、念のためバックアップを取っておきます。

```bash
cp data/invoice/invoice.json data/invoice/invoice.json.orig
```

エクセルインボイスモードのテスト用のデータを作成します。

```bash
(venv) $ echo '0000' > data0000.dat
(venv) $ echo '0001' > data0001.dat
(venv) $ echo '0002' > data0002.dat

(venv) $ zip data.zip ./data0000.dat ./data0001.dat ./data0002.dat
  adding: data0000.dat (stored 0%)
  adding: data0001.dat (stored 0%)
  adding: data0002.dat (stored 0%)

(venv) $ ls -l data.zip
-rw-rw-r-- 1 devel devel 493  5月 16 08:13 data.zip
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
-rw-rw-r-- 1 devel devel 493  5月 16 08:13 data.zip
```

構造化処理プログラムが、`インボイスモード`か`エクセルインボイスモード`かのいずで実行されるかを判断するポイントは、`data/inputdata`フォルダに、ファイル名の末尾が`_excel_invoice.xlsx`となっているExcelファイルが存在するかどうか、です。当該ファイルが存在すればエクセルインボイスモードとして稼働しますし、存在しなければ通常のインボイスモードとして稼働します。

先に、エクセルインボイスモードで使用するExcelファイルについて説明し、その後作成します。

### ファイルモードとフォルダモード

エクセルインボイスを使用する場合、入力ファイルはzipファイル(zip形式の圧縮ファイル)である必要があります。

zipファイルを展開した時に、ファイル1つ1つを別々のデータタイルとして登録するモードを"ファイルモード"、展開したときにデータタイルごとにフォルダが別になっているものを"フォルダモード"といいます。

ファイルモードとフォルダモードでは、登録シートを以下の様に設定します。
* ファイルモード ： A2セルに"data_file_names"、A3セルに"name"をそれぞれ指定
* フォルダモード ： A2セルに"data_folder"を指定(A3セルは任意)

5行目以降のA列に、ファイルモードの場合は展開後のファイル名を、フォルダモードの場合はフォルダ名をそれぞれ指定します。

5行目以降の、B列以降はインボイス(送り状)の内容を記述します。

> エクセルインボイスの登録シートの書き方については、"https://dice.nims.go.jp/services/RDE/service_menu/catalog/manuals/excel_invoice/excel_invoice/" を参照してください。
>
> 上で作成したdata.zipは、展開後ファイルとなりますので、`ファイルモード`を利用することになります。

### エクセルインボイスモードで使用するExcelファイルの条件

エクセルインボイスモードで使用するExcelファイルには、いくつかの条件があります。

* xslx形式のファイルであること。
* ファイル名の末尾が`_excel_invoice.xlsx`(拡張子を含む)であること
* シート名には制限はないが、登録に使用するシート(以下"登録シート"といいます)は、左上端 (A1セル)の値が"`invoiceList_format_id`"であること。
* 左上端 (A1セル)の値が"`invoiceList_format_id`"であるシートが1個だけ存在していること。

以下に、具体的に説明します。

#### xslx形式のファイルであること

拡張子が`xls`のExcelファイル(古い形式のExcelファイル)はサポートしていません。

`xsl`形式のExcelファイルをinputfileとして指定した場合、以下のようなエラーとなります。

```bash
(venv) $ ls -l data/inputdata/
:
-rwxrwxr-x 1 devel devel 94720  2月  3 17:54 sample_excel_invoice.xls

(venv) $ python main.py
：
Exception Type: ImportError
Error: Missing optional dependency 'xlrd'. Install xlrd >= 2.0.1 for xls Excel support Use pip or conda to install xlrd.
```

エラーメッセージのように、`xlrd`パッケージを導入すればうまく稼働する可能性はありますが、RDEToolKitのサポート外となります。

#### ファイル名の末尾が`_excel_invoice.xlsx`(拡張子を含む)であること

ファイル名の末尾が`_excel_invoice.xlsx`(拡張子を含む)でないExcelファイルを入力ファイルとした場合、エクセルインボイスモードでの実行はできません。

上記形式に則っていないファイル名のExcelファイルを`data/inputdata`フォルダに配置して、`python main.py`を実行した場合、実行自体はエラーにならない可能性があります(入力ファイルのチェック内容次第)。

しかし、構造化処理プログラムからは、当該Excelファイルが`単なる入力ファイルの1つ`として扱われます。つまり、`エクセルインボイスモード`ではなく、通常の`インボイスモード`での登録となります。

また、ファイル名の末尾が`_excel_invoice.xlsx`(拡張子を含む)のExcelファイルは、`data/inputdata/`フォルダに1個だけである必要があります。当該条件を満たすExcelファイルが2件(以上)ある場合は、以下のようなエラーとなります。

```bash
(venv) $ ls -l data/inputdata/
:
-rwxrwxr-x 1 devel devel 56883  2月  7 11:26 ample_excel_invoice.xlsx
-rwxr-xr-x 1 devel devel 56883  2月  7 11:31 sample_excel_invoice.xlsx

(venv) $ python main.py

(→ ここでエラーメッセージは表示されません)

(venv) $ cat data/job.failed
ErrorCode=1
ErrorMessage=ERROR: more than 1 excelinvoice file list. file num: 2
```

### シートの制限

Excelファイルが複数のシートを含んでいる場合、インボイスとして登録に使用するのは、`シート内のA1セルの値が"invoiceList_format_id"である`シート(以下"登録シート"と呼びます)となります。

登録シートが複数ある場合は、以下のようなエラーとなります。

```bash
(venv) $ python main.py

(venv) $ cat data/job.failed
ErrorCode=1
ErrorMessage=ERROR: multiple sheet in invoiceList files
```

> デフォルト設定では、ターミナルにはエラーメッセージが表示されません。`data/job.failed`ファイルや、`data/logs/rdesys.log`(存在している場合)などを確認し、必要な対処を実施してください。

登録シートの"シート名"には特に制限はありません。Excelが許容する範囲で自由にシート名を決定することが出来ます。新規ファイル作成時の`Sheet 1`や`Sheet1`でも問題ありません。

### Excelファイルの作成(編集)

上記を踏まえ、エクセルインボイスモードで使用するExcelファイルを作成していきます。

RDEToolKitには、エクセルインボイスモードで使用するExcelファイルのひな形を作成する機能がありますので、これを使います。

ひな形を作成するには、`make-excelinvoice`サブコマンドを使用します。

```bash
(venv) $ python -m rdetoolkit make-excelinvoice data/tasksupport/invoice.schema.json
📄 Generating ExcelInvoice template...
:
✨ ExcelInvoice template generated successfully! : /home/devel/rde-sample/excelinvoice/container/template_excel_invoice.xlsx
```

> カレントディレクトリに、`template_excel_invoice.xlsx`という名前で作成されました。
>
> フォルダモードを使用する場合は、(末尾に)`-m folder`を追加して実行してください。

このファイルを、MS Excelが稼働するPC(Windows、Mac)にコピー(もしくは移動)します。

![template_excel_invoice_tempalte.png](./images/template_excel_invoice_template.png)

> `generalTerm`および`specificTerm`のシートは、空のシートとしてでも必要なシートです。これら2つのシートは必要な情報がセットされているはずですので、修正不要です。
>
> `_version`のシートには、生成したRDEToolKitのバージョン番号がセットされています。当該シートは存在していても問題ありませんので、通常はそのまま保持してください。

#### A1セル

`invoice_form`のシートを選択します。

> 前述の様にシート名は任意です。ここでは変更せずにそのまま利用することにします。

A1セル(シートの左上端)の値が`invoiceList_format_id`であるのを確認します。

> そうなっているはずなので、確認だけでOKです。

#### A2セル

今回は、"ファイルモード"ですので、`data_file_names`であることを確認します。

"フォルダモード"を使う場合は、`data_folder`に変更してください。

#### 2行目(A2を除く)、3行目

インボイスの項目名が入りますので、基本的に**変更しない**でください。

#### 4行目

2、3行目で示した項目名の、日本語名が入ります。ここも**修正不要**です。

> この行は、入力する担当者向けにある行です。構造化処理プログラム実行には影響を与えません。
> 別の項目名に変更するなどしても影響はないですが、混乱のもととなりますので、修正しないでください。
> 
> またセルの中に改行を含むものがありますが、うまく改行されずに表示されることがあります。カーソルを持っていき、いったん編集モードにしてから確定すると正しく改行されます。気になる場合はお試しください。

### 5行目以降

送り状データを、1行ずつ書いていきます。

* 4行目の日本語項目名に`(必須)`と書かれた項目は、必ず入力してください。入力が漏れていた場合は、構造化処理プログラム実行時にエラーになります。

テストデータとして、以下をいれます。

> サンプル値ですので、変更して入力しても構いません。サンプルですので、多くの列で、まったく意味がないデータとなっています。
>
> 下記例では、"入力値"欄に3つ例示が有る場合は行毎の、1個しかない場合は、3行とも同じ値を使います。

| 列名 | 列番号 | 入力値 | 備考 |
| ---- | ---- | ---- | ---- |
| ファイル名 (拡張子も含め入力) | A列 | data0000.dat、data0001.dat、data0002.dat | 展開後のファイル名を拡張子込みでセット |
| データセット名 | B列 | Dummy | 必須だが、ダミーでよい (本当?) |
| データ所有者 | C列 | (空欄) | |
| NIMS user UUID | D列 | 97e05f8b9ed6b4b5dd6fd50411a9c163a2d4e38d6264623666383669 | 全部同じでよい |
| データ名 | E列 | data0000.dat、data0001.dat、data0002.dat | A列と同値 |
| 実験ID | F列 | (空欄) | |
| 説明 | G列 | (空欄) | |
| 試料名(ローカルID) | H列 | Inu、Neko、(空欄) | 7行目だけ空欄にします。 |
| 試料UUID | I列 | (空欄) | |
| 試料管理者UUID | J列 | de17c7b3f0ff5126831c2d519f481055ba466ddb6238666132316439 | 全部同じでよい |
| 化学式・組成式・分子式など | K列 | (空欄) | |
| 参考URL | L列 | (空欄) | |
| 試料の説明 | M列 | (空欄) | |
| 一般名称 | N列 | A1、B1、C1 | |
| CAS番号 | O列 | A2、B2、C2 | |
| 結晶構造 | P列 | A3、B3、C3 | |
| 試料形状 | Q列 | A4、B4、C4 | |
| 試料購入日 | R列 | (空欄) | |
| 購入元 | S列 | A6、B6、C6 | |
| ロット番号、製造番号など | T列 | A7、B7、C7 | |
| 日付 | U列 | (空欄) | |
| 基板 | V列 | (空欄) | |
| 層構造 | W列 | (空欄) | |
| 成膜温度 | X列 | (空欄) | |
| 成膜酸素圧 | Y列 | (空欄) | |
| レーザー強度 | Z列 | (空欄) | |
| レーザースポットサイズ | AA列 | (空欄) | |
| レーザー周波数 | AB列 | (空欄) | |
| 成膜パルス数 | AC列 | (空欄) | |
| キー1 | AD列 | あ1、あ2、あ3 | |
| キー2 | AE列 | い1、い2、い3 | |
| キー3 | AF列 | う1、う2、う3 | |
| キー4 | AG列 | え1、え2、え3 | |
| キー5 | AH列 | お1、お2、お3 | |
| 登録データタイプ | AI列 | (空欄) | |
| データの起源 | AJ列 | (空欄) | |
| 技術カテゴリー | AK列 | (空欄) | |
| 参考文献 | AL列 | (空欄) | |
| 処理年月日 | AM列 | (空欄) | |
| プロセスカテゴリー | AN列 | (空欄) | |
| プロセス手法 | AO列 | (空欄) | |
| 処理環境 | AP列 | (空欄) | |
| 主となる処理温度 | AQ列 | (空欄) | |
| 装置設置場所 | AR列 | (空欄) | |
| 標準手順 | AS列 | (空欄) | |

### その他の注意点

2列目以降に`空の列`を追加しても、項目名(2行目と3行目)が空の場合は影響ありません。

5行目以降は、途中の行に`空の行`があってはいけません。

途中に空の行のあるExcelファイルを用いて構造化処理プログラムを実行した場合は、以下のようなエラーとなります。

```bash
(venv) $ cat data/job.failed 
ErrorCode=1
ErrorMessage=Error! Blank lines exist between lines
```

## エクセルインボイスモードとして実行

上記内容を入力→保存したExcelファイルを、`data/inputdata`フォルダに配置します。

```bash
(venv) $ tree data/inputdata/
data/inputdata/
├── Template_excel_invoice.xlsx
└── data.zip

1 directory, 2 files
```

> 転送の仕方により、ファイル名の1文字目が大文字に変更される場合があります。

続いて、(通常の)インボイスモードでの実行結果をクリアします。

```bash
./reinit.sh
```

この状態でエクセルインボイスモードとして実行します。

```bash
python main.py
```

出来上がったフォルダ構成は以下の様になります。

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
│   │   │   └── data0001.dat
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
│       │   └── data0002.dat
│       ├── other_image
│       ├── raw
│       ├── structured
│       │   └── sample.csv
│       ├── temp
│       └── thumbnail
├── inputdata
│   ├── Template_excel_invoice.xlsx
│   └── data.zip
├── invoice
│   ├── invoice.json
│   └── invoice.json.orig
├── invoice_patch
├── logs
├── main_image
├── meta
│   └── metadata.json
├── nonshared_raw
│   └── data0000.dat
├── other_image
├── raw
├── structured
│   └── sample.csv
├── tasksupport
│   ├── invoice.schema.json
│   └── metadata-def.json
├── temp
│   ├── data0000.dat
│   ├── data0001.dat
│   ├── data0002.dat
│   └── invoice_org.json
└── thumbnail

42 directories, 21 files
```

前述のように、構造化処理プログラムは修正不要です。`data/inputdata`フォルダ上に、単独の入力ファイルを置くのではなく、zip圧縮された入力ファイルとExcelインボイスファイル(xlsxファイル)を配置するだけで、通常のインボイスモードがエクセルインボイスモードに変わります。

複数のデータタイルに対応するため、dividedフォルダが新規に作成され、さらに4桁数字連番のサブフォルダが作成されます。もとの`data`フォルダに1行目のデータ処理結果が、`data/divided/0001`に2行目のデータが、`data/divided/0002`に3行目のデータがそれぞれ格納されます。

例えば、生成されたinvoice.jsonのキー1～キー5までの部分を以下に示します。

data/invoice/invoice.json

```json
:
    "custom": {
        "key1": "あ1",
        "key2": "い1",
        "key3": "う1",
        "key4": "え1",
        "key5": "お1",    
:
```

data/divided/0001/invoice/invoice.json

```json
:
    "custom": {
        "key1": "あ2",
        "key2": "い2",
        "key3": "う2",
        "key4": "え2",
        "key5": "お2",    
:
```

data/divided/0002/invoice/invoice.json

```json
:
    "custom": {
        "key1": "あ3",
        "key2": "い3",
        "key3": "う3",
        "key4": "え3",
        "key5": "お3",    
:
```

> 上の様に、key1～key5が順番に並ぶとは限りません。

また上の例では、3つ目(シート上は7行目)の`試料名(ローカルID)`を空欄にしました。構造化処理プログラム実行の結果を確認すると以下の様になっています。

data/divided/0002/invoice/invoice.json

```json
:
    "sample": {
        "sampleId": null,
        "names": [
            "testdev01_1"
        ],
:
```

この"testdev01_1"は、もとのinvoice.jsonの`sample.names`の値が転記されています。

```bash
(venv) $ vi data/invoice/invoice.json.orig
:
:
    "sample": {
        "sampleId": "8ea50153-7a87-44b7-9d6e-10179c507039",
        "names": [
            "testdev01_1"  (★)
        ],
        "composition": "AB2C3",
        "referenceUrl": "test_ref",
:
```

> Excelファイルに指定が無かったので、上記の(★)の行のデータが利用された、ということです。

data.zipは`data/temp/`フォルダ上で展開され、それぞれの`nonshared_raw/`フォルダに展開されます。

```bash
(venv) $ tree -f data | grep \\.dat
│   │   │   └── data/divided/0001/nonshared_raw/data0001.dat
│       │   └── data/divided/0002/nonshared_raw/data0002.dat
│   └── data/nonshared_raw/data0000.dat
:
│   ├── data/temp/data0000.dat
│   ├── data/temp/data0001.dat
│   ├── data/temp/data0002.dat
```

