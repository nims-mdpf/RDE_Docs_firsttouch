<div class="page" />

# 例外処理(エラー処理)とフォルダ初期化

## 例外処理(エラー処理)

RDE構造化処理プログラム中で、以降の処理を停止するような何らかのエラー処理をする場合は、"StructuredError([エラーメッセージ[,エラー番号]])"をraiseします。

> 第一引数のエラーメッセージも省略可能ですが、エラー内容を正しく伝えるためにエラーメッセージは必ずセットしてraiseするようにしてください。

例:

```python
    ：
    raise StructuredError('*** RDE ERROR ***', 200)
```

これを実行すると、以下の様になります。

```bash
(tenv) $ python main.py
Hello RDE!

Traceback (simplified message):
Call Path:
   File: /home/devel/tenv/lib/python3.11/site-packages/rdetoolkit/exceptions.py, Line: 151 in wrapper()
    └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 13 in dataset()
        └─ File: /home/devel/tutorial/container/modules/datasets_process.py, Line: 8 in custom_module()
            └─> L8: raise StructuredError('*** RDE ERROR ***', 200) 🔥

Exception Type: StructuredError
Error: *** RDE ERROR ***
```

同時に、"data/job.failed"ファイルが作成され、上記で設定した内容が書き込まれます。

```bash
(tenv) $ cat data/job.failed
ErrorCode=200
ErrorMessage=Error: *** RDE ERROR ***
```

RDEのジョブ実行結果はWebブラウザで確認することができますが、ジョブが失敗した場合、そのステータス情報は、この"data/job.failed"の内容をもとに表示されます。

データ登録するユーザが、「なぜ登録に失敗したのか?」を確認できるのはこの画面だけになります。そのため、"data/job.failed"にセットされる内容、つまり"StructuredError()"に与える引数は非常に重要です。

### 終了コード

構造化処理プログラムが正常に終了した場合は、シェルの終了コードは "0" で終了します。
なんらかの異常が発生して終了した場合は、上述の様に"data/job.failed"に内容をセットすると同時に、終了コード "0以外"で終了します。

> "raise StructuredError()"を実行した場合、特に指定しなくても終了コード "1"で終了しますので、通常は終了コードについて考慮する必要はありません。

```bash
(tenv) $ python main.py > /dev/null 2>&1 ; echo $?
1
```

## フォルダ初期化

前述の様に、ジョブ実行時にエラーが発生した場合は"data/job.failed"ファイルが自動で生成されます。

RDEの本番環境では、ジョブ実行の都度フォルダが新しく生成されますので問題ありませんが、ローカル開発においては、前の処理で作られた"data/job.failed"あるいはその他生成されたフォルダ/ファイルを削除しないと、修正がうまくいっているのわかりにくい場合があります。

そのため、必須ではありませんが、以下の様のスクリプトを利用することで、実行の都度生成されるフォルダ・ファイルをクリアすることも考慮してください。

例： reinit.sh

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
./data/logs
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
chmod a+x ./reinit.sh 
```

ローカル環境でのRDE構造化処理実行時、つまり"python main.py"の実行前に実行します。

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
```

> 表示が不要の場合は、echo文をコメントアウトするなどして調整してください。

なお、繰り返しになりますが、実際のRDEの環境では、RDE構造化処理が実行される都度、あたらしいフォルダ構成が生成されます。そのため、明示的に"前の処理で作成されたフォルダやファイルを削除する"処理は必要ありません。

> 本書では扱いませんが、RDEToolKitには"エクセルインボイスモード"や"マルチデータタイルモード"といったモードも存在し、それを使うと"data/divided"フォルダが作成されます。そういった場合には"rm -rf data/divided"の処理を追加する必要があります。
