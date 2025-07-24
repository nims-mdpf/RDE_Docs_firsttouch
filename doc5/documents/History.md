<div class="page" />

<!--
* h1ヘッダにidを付与するために、markdown-extentionsに"attr_list"を指定する必要があります。

e.g.

markdown_extensions:
  - attr_list

* 下記h1ヘッダの"{: #history}"部分は変更しないようにしてください。
  * 変更する場合は、下記"excludes_children:"部分の変更も必要となります。

e.g. mkdocs.yml

plugins:
    - with-pdf:
        excludes_children:
          - './:history'

* h1ヘッダへの章番号付与は、除外できません。(2025.02.28現在)
* excludes_childrenで指定されたh1ヘッダの下のh2ヘッダ、h3ヘッダには、章番号が
  付きません。

* 履歴は新しいものを一番最初に記入し、下に行くに従い古いバージョンとなるように記述してください。

-->

# 変更履歴    {: #history }

## 1.3.0 (2025.05.26)

* 対応RDEToolKit : v1.2.0
* サンプルのPythonバージョンを3.11から3.12に変更

## 1.2.0 (2025.02.28)

* 対応RDEToolKit : v1.1.1
* 開発の章が4章と5章に分かれていた部分を1つの章に統合

## 1.1.0 (2025.01.15)

* 対応RDEToolKit : v1.0.2

