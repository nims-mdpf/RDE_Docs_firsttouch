<div class="page" />

# サムネイル画像の作成

RDEシステムでの`サムネイル画像`は、登録されたデータの一覧がタイル状に並んだときに小さく表示される画像を指します。

標準で"286 * 200 に収まるサイズ"の画像をサムネイル画像として想定していますが、RDEでは、それより大きい画像であっても自動的に縮小して表示されるので問題ありません。従って厳密にはサムネイル画像を作成する必要は、必ずしもありません。

また、RDEToolKitではイメージフォルダ(main_image、other_image)にある画像データを、そのままサムネイル画像としてコピーする機能をオプションとして提供しています。

本書では、前章にて作成している"全シリーズを重ねた画像"を元に、サムネイル画像を作成します。

## 想定

* メインイメージフォルダ("data/main_image"にあるファイル、1個に限定される)の画像ファイルを読み込み、サイズを"286 * 200"以下に縮小し、サムネイル画像保存フォルダ("data/thumbnail")に保存する。
* 保存するファイルは、(メインイメージと同様)PNG形式とし、ファイル名は"thumbnail.png"とする。

## custom_module()にロジックを実装

"main_image"にある画像をもとに、表示サイズを縮小した"thumbnail.png"として作成する。

modules/datasets_process.py
```python
:
from PIL import Image
:
def custom_module(srcpaths: RdeInputDirPaths, resource_paths: RdeOutputResourcePath) -> None:
    :
    # Write thumbnail images
    src_img_file_path = resource_paths.main_image / "all_series.png"
    out_img_file_path = resource_paths.thumbnail / "thumbnail.png"
    closure_w = 286
    closure_h = 200

    Image.MAX_IMAGE_PIXELS = None   # オープンする画像のピクセルサイズ制限を解除する
    img_org = Image.open(src_img_file_path)
    ratio_w = closure_w / img_org.width
    ratio_h = closure_h / img_org.height
    ratio = min(ratio_w, ratio_h)
    img_re = img_org.resize((int(img_org.width * ratio), int(img_org.height * ratio)), Image.BILINEAR)  # image magickの>デフォルトに合わせてバイリニアとしている
    img_re.save(out_img_file_path)
```

実行して、確認してみます。

```bash
(tenv) $ python main.py

(tenv) $ tree data/thumbnail
data/thumbnail
└── thumbnail.png

1 directory, 1 file
```

> thumbnail.pngが出力されていることが確認できます。RDEシステムでは、サムネイル画像のファイル名の規定はないので、上記以外のファイル名でも問題ありません。
>
> ただし、現時点(2025.5.9)では、画像ファイルの形式に制限があります。GIF形式(.gif)、JPEG形式(.jpg、.jpeg)、あるいはPNG形式(.png)のいずれかである必要があります。SVG形式といった上記想定以外のファイル形式を使った場合、データセット利用ができない(登録は出来るがダウンロードできない、等)場合があります。想定される画像ファイル形式で保存するようにしてください。

## (参考)画像データをそのままサムネイル画像として使う

前述の通り、RDEToolKitはイメージフォルダにある画像ファイルをそのままthumbnail/フォルダにコピーする機能があります。その機能を利用する場合は、以下の様にします。

最初に、当該機能を使うように設定します。

> RDEToolKitのコンフィグ設定は、下記の様にmain.pyで指定する方法のほか、data/tasksupportフォルダにrdeconfig.yamlまたはrdeconfig.ymlを設置し、指定する方法もあります。詳しくはRDEToolKitのマニュアル等を参照ください。
>
> なお、どちらの方法でコンフィグ設定を行っても同じ結果となりますが、両方設定した場合は、main.py内の設定のみが反映されますので注意してください。

最初にRDEToolKitの画像をサムネイル画像として自動コピーする機能を有効にします。

main.py

```python
import rdetoolkit
from modules import datasets_process


def main():
    config = rdetoolkit.config.Config()
    config.system.save_thumbnail_image = True
    rdetoolkit.workflows.run(
        config=config,
        custom_dataset_function=datasets_process.dataset
    )

if __name__ == '__main__':
    main()
```

次に、modules/datasets_process.py内に追加した"サムネイル画像生成部分"をコメントアウトします。

この状態で実行してみます。それまでの開発で出力されたファイルを消してから実行します。

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

(tenv) $ ls -l data/main_image/
合計 28
-rw-rw-r-- 1 enami enami 28473  5月  8 06:52 all_series.png

(tenv) $ ls -l data/thumbnail/
合計 28
-rw-rw-r-- 1 enami enami 28473  5月  8 06:52 all_series.png
```

> `main_image/`フォルダにある"all_series.png"が、ファイル名も含め、そのまま`data/thumbnail/`フォルダにコピーされていることが分かります。

