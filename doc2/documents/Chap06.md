# データセットテンプレートのバリデーション

## RDEToolKitにおけるJSONファイルのバリデーション

　RDEToolKitはinvoice.jsonファイルなどのバリデーションを行い書式や値の不整合を確認します。invoice.jsonはinvoice.schema.jsonを用いてバリデーションできますが、すべてのinvoice項目を網羅していません。例えば、invoice.jsonで指定するdatasetId、 basic.dataOwnerI、basic.dateSubmittedなど必須項目がありますが、これらはinvoice.schema.jsonではバリデーションできません。そのため、RDEToolKitでは、ユーザ定義のinvoice項目以外のシステムが生成する項目について網羅したJSONスキーマファイルを用いてバリデーションを行っています。このバリデーションで不具合があるとデータの永続登録時に不具合が発生し登録処理が異常終了となるため、事前の確認が欠かせません。<br>
　構造化処理プログラム開発においてはinvoice.jsonファイルは自作する必要があり(テンプレート生成ツールを利用すれば作成できます)、テストにおいてJSONスキーマに則ったjsonファイルを作成することが必要です。そのための情報を下記にまとめました。

### invoice.jsonのbasic、sampleなどの定義

　以下はinvoice.jsonのbasicとsampleの出力事例です。

　datasetId、dataOwnerId(などのユーザID)、試料のclassId、termIdなどはUUIDやランダム文字列と定義されています。それぞれ長さが異なっています。試料のclassId、termIdはテンプレート様式ファイルで提供される値を利用すれば良いのですが、datasetId、sampleIdなどはシステムが生成するものであり、またdataOwnerIdなどのuserIdについては本人以外知り得ない情報となり自作する必要があります。

```JSON
{
  "datasetId" : "b84a220a-0be1-40f5-b46b-bde08a318523",
  "basic" : {
    "dateSubmitted" : "2024-03-13",
    "dataOwnerId" : "de17c7b3f0ff5126831c2d519f481055ba466ddb6238666132316439",
    "dataName" : "invoiceSample_TSK20240313-1",
    "instrumentId" : null,
    "experimentId" : null,
    "description" : ""
  },
  "sample" : {
    "sampleId" : "2ddf932e-fa80-40d7-a010-939ab486e309",
    "names" : [ "sample_name" ],
    "composition" : null,
    "referenceUrl" : null,
    "description" : null,
    "generalAttributes" : [ {
      "termId" : "3adf9874-7bcb-e5f8-99cb-3d6fd9d7b55e",
      "value" : null
    },
    "specificAttributes" : [ {
      "classId" : "52148afb-6759-23e8-c8b8-33912ec5bfcf",
      "termId" : "70c2c751-5404-19b7-4a5e-981e6cebbb15",
      "value" : null
    }],    
    "ownerId" : "de17c7b3f0ff5126831c2d519f481055ba466ddb6238666132316xxx"
  }
}
```


　invoice.jsonにおけるidの自作についてはそれぞれ以下の表のように作成してください。

| 項目 | 生成条件 | 必須 |
|---|---|---|
|datasetId|UUID形式|必須|
|basic.dateSubmitted|日付、YYYY-MM-DD|必須|
|basic.dataOwnerId|ランダム文字列、56文字|必須|
|basic.dataName|文字列、256文字|必須|
|basic.instrumentId|UUID形式|任意|
|basic.experimentId|任意文字列、256文字|任意|
|basic.description|任意文字列、8192文字|任意|
|sample.sampleId|UUID形式|利用時必須|
|sampleのtermId|UUID形式|利用時必須|
|sampleのclassId|UUID形式|利用時必須|

なお、invoice.jsonには含まれてはいない項目ですがdataのIDについては以下の通り

| 項目 | 生成条件 | 必須 |
|---|---|---|
|dataId|UUID形式|利用時必須|

### RDEToolKitでinvoice.jsonのバリデーションに用いられるJSONスキーマファイル

以下はRDEToolKitでinvoice.jsonのバリデーションに用いられるJSONスキーマファイル<br>
invoice_basic_and_sample.schema_.json<br>
の全出力です。invoice.schema.jsonで開発者が定義できるもの以外は下記のようにスキーマ定義されています。



```JSON
{
  "$schema": "https://json-schema.org/draft-07/schema",
  "description": "invoice_basic_schema/when adding samples",
  "type": "object",
  "required": [
    "basic",
    "datasetId"
  ],
  "properties": {
    "datasetId": {
      "type": "string"
    },
    "basic": {
      "type": "object",
      "label": {
        "ja": "送状基本情報",
        "en": "Invoice Basic Information"
      },
      "required": [
        "dateSubmitted",
        "dataOwnerId",
        "dataName"
      ],
      "properties": {
        "dateSubmitted": {
          "type": "string",
          "format": "date"
        },
        "dataOwnerId": {
          "type": "string",
          "pattern": "^([0-9a-zA-Z]{56})$"
        },
        "dateName": {
          "type": "string",
          "pattern": "^.*"
        },
        "instrumentId": {
          "type": [
            "string",
            "null"
          ],
          "pattern": "^$|^([0-9a-zA-Z]{8}-[0-9a-zA-Z]{4}-[0-9a-zA-Z]{4}-[0-9a-zA-Z]{4}-[0-9a-zA-Z]{12})$"
        },
        "experimentId": {
          "type": [
            "string",
            "null"
          ]
        },
        "description": {
          "type": [
            "string",
            "null"
          ]
        }
      }
    },
    "sample": {
      "anyOf": [
        {
          "$ref": "#/definitions/sample/sampleWhenAdding"
        },
        {
          "$ref": "#/definitions/sample/sampleWhenRef"
        },
        {
          "$ref": "#/definitions/sample/sampleWhenAddingExcelInvoice"
        }
      ]
    }
  },
  "definitions": {
    "sample": {
      "sampleWhenAdding": {
        "type": "object",
        "required": [
          "sampleId",
          "names",
          "ownerId"
        ],
        "properties": {
          "sampleId": {
            "type":  "string",
            "pattern": "^$"
          },
          "names": {
            "type": "array"
          },
          "ownerId": {
            "description": "sample ownere id",
            "type": "string",
            "pattern": "^([0-9a-zA-Z]{56})$"
          },
          "composition": {
            "type": [
              "string",
              "null"
            ]
          },
          "referenceUrl": {
            "type": [
              "string",
              "null"
            ]
          },
          "description": {
            "type": [
              "string",
              "null"
            ]
          },
          "generalAttributes": {
            "type": "array",
            "properties": {
              "termId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "value": {
                "type": [
                  "string",
                  "null"
                ]
              }
            }
          },
          "specificAttributes": {
            "type": "array",
            "properties": {
              "classId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "termId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "value": {
                "type": [
                  "string",
                  "null"
                ]
              }
            }
          }
        }
      },
      "sampleWhenRef": {
        "type": "object",
        "required": [
          "sampleId"
        ],
        "properties": {
          "sampleId": {
            "type": "string",
            "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
          },
          "names": {
            "type": "array"
          },
          "ownerId": {
            "description": "sample ownere id",
            "type": "string",
            "pattern": "^([0-9a-zA-Z]{56})$"
          },
          "composition": {
            "type": [
              "string",
              "null"
            ]
          },
          "referenceUrl": {
            "type": [
              "string",
              "null"
            ]
          },
          "description": {
            "type": [
              "string",
              "null"
            ]
          },
          "generalAttributes": {
            "type": "array",
            "properties": {
              "termId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "value": {
                "type": [
                  "string",
                  "null"
                ]
              }
            }
          },
          "specificAttributes": {
            "type": "array",
            "properties": {
              "classId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "termId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "value": {
                "type": [
                  "string",
                  "null"
                ]
              }
            }
          }
        }
      },
      "sampleWhenAddingExcelInvoice": {
        "type": "object",
        "required": [
          "names",
          "ownerId"
        ],
        "properties": {
          "sampleId": {
            "type":  "string",
            "pattern": "^$"
          },
          "names": {
            "type": "array"
          },
          "ownerId": {
            "description": "sample ownere id",
            "type": "string",
            "pattern": "^([0-9a-zA-Z]{56})$"
          },
          "composition": {
            "type": [
              "string",
              "null"
            ]
          },
          "referenceUrl": {
            "type": [
              "string",
              "null"
            ]
          },
          "description": {
            "type": [
              "string",
              "null"
            ]
          },
          "generalAttributes": {
            "type": "array",
            "properties": {
              "termId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "value": {
                "type": [
                  "string",
                  "null"
                ]
              }
            }
          },
          "specificAttributes": {
            "type": "array",
            "properties": {
              "classId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "termId": {
                "type": [
                  "string",
                  "null"
                ],
                "pattern": "^([0-9a-f]{8})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{4})-([0-9a-f]{12})$"
              },
              "value": {
                "type": [
                  "string",
                  "null"
                ]
              }
            }
          }
        }
      }
    }
  }
}
```

<div class="page" />