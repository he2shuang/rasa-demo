
アシスタントの会話はトラッカーストア内に保存されます。Rasaは様々なストアタイプの実装を標準で提供していますが、独自のカスタムストアを作成することも可能です。`InMemoryTrackerStore` はデフォルトのトラッカーストアです。他のトラッカーストアが設定されていない場合に使用されます。会話履歴をメモリ内に保存します。このストアはすべての履歴をメモリに保持するため、Rasaサーバーを再起動すると履歴全体が失われます。`InMemoryTrackerStore`を使用するために設定は必要ありません。


### 種類

1. InMemoryTrackerStore (default)
	メモリを使用する

2. SQLTrackerStore
	SQLデータベースを使用する

3. RedisTrackerStore
	Redisデータベースを使用する

4. MongoTrackerStore
	MongoDBを使用する

5. DynamoTrackerStore
	DynamoDBを使用する



以下に、Redisデータベースを使用して会話記録を保存する方法について説明します。
## 設定手順（Redis）

1. Redis をインストールする

```
# システム更新
sudo apt update && sudo apt upgrade -y

# Redisサーバーのインストール
sudo apt install redis-server -y

# Redisサービスを起動する
sudo service redis-server start


# サービス状態を確認する
sudo service redis-server status

```

2. Rasa の endpoint.yml ファイルを設定し、tracker_store の設定を追加する

![[Pasted image 20250918095155.png]]

```yaml
tracker_store:
   type: redis
   url: localhost
   port: 6379
   db: 1
   key_prefix: "rasaTracker"
  #  password: <password used for authentication>
  #  use_ssl: <whether or not the communication is encrypted, default false>
```


以下のコマンドを使⽤して Rasa を 起動する

```
rasa run --endpoints endpoints.yml

```

### パラメータ説明


- `url`（デフォルト: `localhost`）： RedisインスタンスのURL。
    
- `port` (デフォルト: `6379`): Redisが実行されているポート番号。
    
- `db` (デフォルト: `0`): Redisデータベースの番号。
    
- `key_prefix` (デフォルト: `None`): トラッカーストアのキーに接頭辞として付加する文字列。英数字でなければならない。
    
- `username` (デフォルト: `None`): 認証に使用するユーザー名。
    
- `password` (デフォルト: `None`): 認証に使用するパスワード（`None` は認証なしを意味する）。
    
- record_exp（デフォルト: `None`）: レコードの有効期限（秒単位）。
    
- `use_ssl` (デフォルト: `False`): 転送時の暗号化にSSLを使用するかどうか。


## ## 設定手順（MySQL）

1. pymysql をインストールする
```
pip install pymysql
```

2. Rasa の endpoint.yml ファイルを設定し、tracker_store の設定を追加する

![[Pasted Image 20250918165353_246.png]]

```yaml
tracker_store:
   type: SQL
   dialect: "mysql+pymysql"
   url: "172.18.24.79"
   port: 3306
   db: rasa_tracker_db
   username: navicat_user
   password: 123456
```

 Rasa を 再起動する

### パラメータ説明

- domain (デフォルト: None): このトラッカーストアに関連付けられたドメインオブジェクト

- dialect (デフォルト: sqlite): SQLバックエンドとの通信に使用する方言。利用可能な方言については [SQLAlchemy docs](https://docs.sqlalchemy.org/en/latest/core/engines.html#database-urls) を参照してください。

- url (デフォルト: None): SQLサーバーのURL
  
- port (デフォルト: None): SQLサーバーのポート番号
  
- db (デフォルト: rasa.db): 使用するデータベースのパス
  
- username (デフォルト: None): 認証に使用するユーザー名
  
- password (デフォルト: None): 認証に使用するパスワード
  
- event_broker (デフォルト: None): イベントを発行するイベントブローカー
  
- login_db (デフォルト: None): 初期接続時に接続し、db で指定されたデータベースを作成する代替データベース名 (PostgreSQL のみ)
  
- query (デフォルト: None): 接続時にダイアレクトおよび/または DBAPI に渡すオプションの辞書


## リンク

[Tracker Stores](https://legacy-docs-oss.rasa.com/docs/rasa/tracker-stores/#redistrackerstore)

[Rasa中的tracker_store和event_broker - 扫地升 - 博客园](https://www.cnblogs.com/shengshengwang/p/17942219)

