
### NLUモデルのみをトレーニングする

NLUモデルのみをトレーニングするには、以下を実行します：

```
rasa train nlu
```

これにより、`data/` ディレクトリ内にあるNLUトレーニングデータファイルが検索され、トレーニング済みのモデルが `models/` ディレクトリに保存されます。モデル名は `nlu-` で始まります。

### NLUモデルを試す

コマンドライン上でNLUモデルを試すには、以下のコマンドを実行します：

```
rasa shell nlu
```

または、`nlu` パラメータを省略し、NLUのみのモデルを直接渡すこともできます：

```
rasa shell -m models/nlu-20250925-184220-sophisticated-jaguar.tar.gz
```

### NLUサーバーの起動

NLUモデルを使用してサーバーを起動するには、実行時にモデル名を渡します：

```
rasa run --enable-api -m models/nlu-20190515-144445.tar.gz
```

その後、`/model/parse` エンドポイントを使用してモデルに予測をリクエストできます。そのためには、以下を実行します：

```
curl localhost:5005/model/parse -d '{"text":"hello"}'
```

### NLUサーバーへの接続

[Rasa NLUのみのサーバー](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-only#running-an-nlu-server)を、別途実行されているRasaの対話管理のみのサーバーに接続するには、対話管理サーバーのエンドポイント設定ファイルに接続情報を追加します：

```
nlu:
    url: "http://<your nlu host>:<your nlu port>"
    token: <token>  # [optional]
    token_name: <name of the token> # [optional] (default: token)
```

`token` と `token_name` は、オプションの[認証パラメータ](https://legacy-docs-oss.rasa.com/docs/rasa/http-api#token-based-auth)を指します。

##### 対話管理サーバー

対話管理サーバーは、NLUモデルを含まないモデルを提供（サーブ）する必要があります。対話管理のみを含むモデルを取得するには、`rasa train core` を使用してモデルをトレーニングするか、`rasa train` を使用しつつすべてのNLUデータを除外します。

対話管理サーバーがメッセージを受信すると、`http://<your nlu host>:<your nlu port>/model/parse` へ[リクエストを送信](https://rasa.com/docs/rasa/pages/http-api#operation/parseModelMessage)し、返された解析情報を使用します。

### リンク

[Using NLU Only](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-only)

[NLU-Only Server](https://legacy-docs-oss.rasa.com/docs/rasa/nlu-only-server)