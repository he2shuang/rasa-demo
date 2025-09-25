
### ローカルディスクからの読み込み

デフォルトでは、モデルはローカルディスクから読み込まれます。`--model` パラメータを使用して、モデルのパスを指定できます：

```
rasa shell --model models-change/20250924-181607-felt-modem.tar.gz
```

ディレクトリ内の最新のモデルを読み込む場合は、ファイルの代わりにディレクトリを指定できます：

```
rasa shell --model models-change/
```

`--model` パラメータが指定されていない場合、Rasaは `models/` ディレクトリ内でモデルを検索します。以下の2つのコマンドは、同じモデルを読み込みます：

```
# このコマンドは同じモデルを読み込みます
rasa run --model models/
# ... こちらのコマンド（デフォルト設定を使用）と同様です
rasa run
```

### AWS S3からの読み込み

#### 1. boto3 パッケージのインストール

Amazon S3は `boto3` パッケージを使用しており、追加の依存関係として `pip3` を使ってこのパッケージをインストールする必要があります。

```
pip3 install boto3
```

#### 2. IAMユーザーの作成と認証情報の取得

###### ステップ1: 新しいIAMユーザーを作成する

###### ステップ2: 権限を設定する (Set permissions)

これは最も重要なステップであり、ここでは「最小権限の原則」を適用します。

1. **ポリシーを直接アタッチします (Attach policies directly)** を選択します。
2. **ポリシーを作成 (Create policy)** ボタンをクリックします。これにより、新しいブラウザのタブが開きます。
3. 新しい「ポリシーの作成」ページで、**JSON** タブを選択します。
4. テキストボックス内の既存のサンプルコードを削除し、以下のJSONコンテンツを**貼り付け**ます。**必ず `your-s3-bucket-name` をご自身のS3バケット名に置き換えてください！**

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListBucketForRasa",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::your-s3-bucket-name"
        },
        {
            "Sid": "ReadWriteObjectsForRasa",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::your-s3-bucket-name/*"
        }
    ]
}
```

> [!note] **このポリシーの機能**
> 
> - `s3:ListBucket`: Rasaがバケット内のオブジェクトを一覧表示することを許可します。Rasaが最新のモデルファイルを見つけるために必要です。
> - `s3:GetObject`: Rasaがバケットからモデルをダウンロード（プル）することを許可します。
> - `s3:PutObject`: Rasaが新しくトレーニングされたモデルをバケットにアップロード（プッシュ）することを許可します。
> - `Resource`: これらの操作を `your-s3-bucket-name` バケットのみに厳密に制限します。`/*` がバケット内のすべてのオブジェクトを意味する点にご注意ください。

###### ステップ3: 認証情報を取得し、安全に保存する

作成に成功すると、成功メッセージが表示されたページに移動します。

1. 先ほど作成したユーザー名（例：`rasa-s3-user`）をクリックします。
2. ユーザー詳細ページに移動したら、**セキュリティ認証情報 (Security credentials)** タブをクリックします。
3. **アクセスキー (Access keys)** セクションまで下にスクロールします。
4. **アクセスキーを作成 (Create access key)** をクリックします。
5. 「アクセスキーのベストプラクティスと代替案」ページで：
    - **コマンドラインインターフェイス (CLI)** を選択します。
    - 下の確認ボックスにチェックを入れます：「上記の推奨事項を理解し、アクセスキーの作成を続行することに同意します。」
    - **次へ (Next)** をクリックします。
6. (オプション) 説明タグを設定します（例：`Rasa S3 Access`）。
7. **アクセスキーを作成 (Create access key)** をクリックします。

これで、画面に **アクセスキーID (Access key ID)** と **シークレットアクセスキー (Secret access key)** が表示されます。

#### 3. 環境変数の設定

Rasaがモデルを認証してダウンロードできるようにするために、ストレージを必要とするコマンドを実行する前に、以下の環境変数を設定する必要があります。

```
export AWS_SECRET_ACCESS_KEY="**********"
export AWS_ACCESS_KEY_ID="********"
export AWS_DEFAULT_REGION="us-east-1"
export BUCKET_NAME="bucket-for-model-storage-250924"
```

#### 4. Rasaの起動

すべての環境変数を設定した後、 `remote-storage` オプションを `aws` に設定して、以下のコマンドでRasaサーバーを起動できます。

```
rasa shell --remote-storage aws --model 20250924-181607-felt-modem.tar.gz
```

または

```
rasa run --remote-storage aws --model 20250924-181607-felt-modem.tar.gz
```

> [!note]  
> コマンド`rasa shell --remote-storage aws --model 20250924-181607-felt-modem.tar.gz` を使用するには、プロジェクト内に`models`フォルダが存在し、そのフォルダ内にトレーニング済みのモデルが含まれている必要があります。そうでない場合、Rasaのチェックに失敗します。Rasaの起動ロジックには欠陥があります：**リモートストレージを使用するよう明示的に指示しても、まずローカルファイルシステムを頑なにチェックしようとします。** これは「Rasa 3.6.21」で確認された問題であり、他のバージョンでこの問題が存在するかどうかは不明です。

### リンク

[Model Storage](https://legacy-docs-oss.rasa.com/docs/rasa/model-storage)  
[Command Line Interface](https://legacy-docs-oss.rasa.com/docs/rasa/command-line-interface)