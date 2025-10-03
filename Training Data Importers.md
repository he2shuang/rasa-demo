### ローカルのトレーニングデータを指定

[`--data` コマンドライン引数](https://legacy-docs-oss.rasa.com/docs/rasa/command-line-interface)を使用すると、Rasaがディスク上でトレーニングデータを探す場所を指定できます。そうすると、Rasaは潜在的なすべてのトレーニングファイルをロードし、それらを使用してアシスタントをトレーニングします。

```
rasa train --data data-test
```

--data DATA : コアおよびNLUデータファイルへのパス（デフォルト: data）

### RasaFileImporter (default)

Rasaはデフォルトでインポーター `RasaFileImporter` を使用します。これを単独で使用する場合、設定ファイルで何も指定する必要はありません。他のインポーターと一緒に使用する場合は、次のように設定ファイルに追加します：

```
importers:
- name: "module.CustomImporter"
  parameter1: "value"
  parameter2: "value2"
- name: "RasaFileImporter"
```

### MultiProjectImporter

このインポーターを使用すると、再利用可能な複数のRasaプロジェクトを組み合わせてモデルをトレーニングできます。例えば、あるプロジェクトで雑談（chitchat）を処理し、別のプロジェクトでユーザーへの挨拶を処理する、といった使い方ができます。これらのプロジェクトは個別に開発し、アシスタントをトレーニングする際に組み合わせることが可能です。

```
.
├── config.yml
├── domain.yml
├── endpoint.yml
├── credentials.yml
├── data
|	├── nlu.yml
|	├── rules.yml
|   └── stories.yml
└── projects
    ├── GreetBot
    │   ├── data
    │   │   ├── nlu.yml
    │   │   └── stories.yml
    │   └── domain.yml
    └── ChitchatBot
        ├── config.yml
        ├── data
        │   ├── nlu.yml
        │   └── stories.yml
        └── domain.yml
```

Rasaに `MultiProjectImporter` モジュールを使用するよう指示するには、ルートの `config.yml` にある `importers` リストにそれを追加する必要があります。

```
importers:
- name: MultiProjectImporter
```

次に、同じファイル（`config.yml`）内で、インポートしたいプロジェクトを `imports` リストに追加して指定します。

```
imports:
- projects/ChitchatBot
```

### カスタムインポーターの作成

必要に応じて、Rasaがトレーニングデータをインポートする方法をカスタマイズすることもできます。その潜在的なユースケースとしては、以下のようなものが考えられます：

- カスタムパーサーを使用して、他のフォーマットのトレーニングデータをロードする
- 異なる方法でトレーニングデータを収集する（例：異なるリソースからロードする）

[カスタムインポーターを作成](https://legacy-docs-oss.rasa.com/docs/rasa/training-data-importers#writing-a-custom-importer)し、設定ファイルに `importers` セクションを追加してインポーターとその完全なクラスパスを指定することで、Rasaにそれを使用するよう指示できます：

```
importers:
- name: "module.CustomImporter"
  parameter1: "value"
  parameter2: "value2"
- name: "RasaFileImporter"
```

`name` キーは、どのインポーターをロードすべきかを決定するために使用されます。追加のパラメータはすべて、ロードされたインポーターにコンストラクタ引数として渡されます。

> [!tip]  
> 複数のインポーターを指定できます。Rasaはそれらの結果を自動的にマージします。

#### 例：AWS S3からトレーニングデータをインポートするカスタムインポーターの作成

必要なトレーニングデータ（domain、config、nlu、stories、rulesを含む）が、すべてS3の「rasa-training-data/」フォルダに配置されていると仮定します。

##### 1. boto3 パッケージのインストール

Amazon S3は `boto3` パッケージの使用をサポートしています。追加の依存関係として、`pip3` を使用してこのパッケージをインストールする必要があります。

```
pip3 install boto3
```

##### 2. IAMユーザーを作成し、認証情報を取得する

###### ステップ 1: 新しいIAMユーザーの作成

###### ステップ 2: 権限を設定する (Set permissions)

ここが最も重要なステップであり、「最小権限の原則」を適用します。

1. **ポリシーを直接アタッチする (Attach policies directly)** を選択します。
2. **ポリシーの作成 (Create policy)** ボタンをクリックします。新しいブラウザタブが開きます。
3. 新しい「ポリシーの作成」ページで、**JSON** タブを選択します。
4. テキストボックス内の既存のサンプルコードを削除し、以下のJSONコンテンツを**貼り付け**てください。**必ず `your-s3-bucket-name` をご自身のS3バケット名に置き換えてください！**

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

> [!note] **このポリシーが何をするか？**
> 
> - `s3:ListBucket`: Rasaがバケット内のオブジェクトを一覧表示することを許可します。Rasaが最新のモデルファイルを見つけるために必要です。
> - `s3:GetObject`: Rasaがバケットからモデルをダウンロード（プル）することを許可します。
> - `s3:PutObject`: Rasaが新しくトレーニングしたモデルをバケットにアップロード（プッシュ）することを許可します。
> - `Resource`: これらの操作を `your-s3-bucket-name` というバケットのみに厳密に制限します。`/*` の違いに注意してください。これはバケット内のすべてのオブジェクトを意味します。

###### ステップ 3: 認証情報を取得し、安全に保存する

作成に成功すると、成功メッセージが表示されたページに移動します。

1. 先ほど作成したユーザー名（例：`rasa-s3-user`）をクリックします。
2. ユーザー詳細ページに入ったら、**セキュリティ認証情報 (Security credentials)** タブをクリックします。
3. **アクセスキー (Access keys)** セクションまで下にスクロールします。
4. **アクセスキーの作成 (Create access key)** をクリックします。
5. 「アクセスキーのベストプラクティスと代替案」ページで：
    - **コマンドラインインターフェイス (CLI)** を選択します。
    - 下の確認ボックスにチェックを入れます：「上記の推奨事項を理解し、アクセスキーの作成を続行することに同意します」。
    - **次へ (Next)** をクリックします。
6. (オプション) 説明タグを設定します。例：`Rasa S3 Access`。
7. **アクセスキーの作成 (Create access key)** をクリックします。

これで、画面に **アクセスキーID** と **シークレットアクセスキー** が表示されます。

#### 3. 環境変数の設定

Rasaがモデルを認証してダウンロードできるようにするために、ストレージを必要とするコマンドを実行する前に、以下の環境変数を設定する必要があります。

```
export AWS_SECRET_ACCESS_KEY="********"
export AWS_ACCESS_KEY_ID="**********"
export AWS_DEFAULT_REGION="us-east-1"
export BUCKET_NAME="***********"
export S3_DATA_PREFIX=rasa-training-data/
```

#### 4. カスタムインポーター

プロジェクトのルートディレクトリに「cloud_importer.py」ファイルを追加し、そのファイルに以下のコードを記述すると仮定します：

```
import os
import boto3
import logging
import tempfile
import threading
from typing import Optional, Text, Dict, List, Union
  
from rasa.shared.core.domain import Domain
from rasa.shared.nlu.interpreter import RegexInterpreter
from rasa.shared.core.training_data.structures import StoryGraph
from rasa.shared.importers.importer import TrainingDataImporter
from rasa.shared.nlu.training_data.training_data import TrainingData
from rasa.shared.utils import io as shared_io_utils
from rasa.shared.nlu.training_data import loading as nlu_loading
from rasa.shared.core.training_data.story_reader.yaml_story_reader import (
    YAMLStoryReader,
)
  
logger = logging.getLogger(__name__)
  
class S3Importer(TrainingDataImporter):
    def __init__(
        self,
        config_file: Optional[Text] = None,
        domain_path: Optional[Text] = None,
        training_data_paths: Optional[Union[List[Text], Text]] = None,
        **kwargs: Dict
    ):
        self.s3_bucket = os.environ.get("BUCKET_NAME") or kwargs.get("bucket_name")
        env_prefix = os.environ.get("S3_DATA_PREFIX")
        if env_prefix is not None:
            self.s3_prefix = env_prefix
        else:
            self.s3_prefix = kwargs.get("s3_prefix", "")
  
        if not self.s3_bucket:
            raise ValueError(
                "S3Importerにはbucket_nameが必要です。環境変数 'BUCKET_NAME' を設定するか、"
                "config.ymlのimporter設定で 'bucket_name' を指定してください。"
            )

  
        # --- 遅延読み込み（lazy loading）モードの重要な変更点 ---
        # 1. 一時ディレクトリの参照のみを作成し、すぐにはダウンロードしない
        self.temp_dir = tempfile.mkdtemp()
        self._data_downloaded = False
        # 2. スレッドロックを使用し、マルチスレッド環境でも一度しかダウンロードされないようにする
        self._download_lock = threading.Lock()
        logger.debug(f"S3Importerが初期化されました。一時ディレクトリ: {self.temp_dir}")
  
    def _ensure_data_is_downloaded(self):
        """
        S3データが一度だけダウンロードされることを保証するヘルパー関数。
        """
        with self._download_lock:
            if not self._data_downloaded:
                logger.info(f"S3Importerがバケット '{self.s3_bucket}' のプレフィックス '{self.s3_prefix}' からデータをロードします。")
                self._download_data_from_s3()
                self._data_downloaded = True
  
    def _download_data_from_s3(self):
        s3 = boto3.client("s3")
        try:
            response = s3.list_objects_v2(Bucket=self.s3_bucket, Prefix=self.s3_prefix)
            if "Contents" not in response:
                logger.warning(f"S3バケット '{self.s3_bucket}' のプレフィックス '{self.s3_prefix}' にファイルが見つかりませんでした。")
                return
  
            for obj in response["Contents"]:
                s3_key = obj["Key"]
                if s3_key.endswith("/"): continue
                relative_path = os.path.relpath(s3_key, self.s3_prefix)
                local_path = os.path.join(self.temp_dir, relative_path)
                os.makedirs(os.path.dirname(local_path), exist_ok=True)
  
                logger.info(f"s3://{self.s3_bucket}/{s3_key} から {local_path} へダウンロードしています...")
                s3.download_file(self.s3_bucket, s3_key, local_path)
  
        except Exception as e:
            logger.error(f"S3からのデータダウンロード中にエラーが発生しました: {e}")
            raise e
  
    def _get_local_path_for(self, file_path: Text) -> Optional[Text]:
        """
        ファイルのローカルパスを返す。ファイルが存在しない場合はNoneを返す。
        """
        full_path = os.path.join(self.temp_dir, file_path)
        return full_path if os.path.exists(full_path) else None
  
    # --- 各 get_* メソッドで _ensure_data_is_downloaded を呼び出す ---
  
    def get_config_file_for_auto_config(self) -> Optional[Text]:
        self._ensure_data_is_downloaded()
        return self._get_local_path_for("config.yml")
  
    def get_config(self) -> Dict:
        self._ensure_data_is_downloaded()
        path = self._get_local_path_for("config.yml")
        if not path:
            # S3にconfig.ymlが存在しない場合、クラッシュを避けるために空の辞書を返す
            logger.warning("S3で 'config.yml' が見つかりませんでした。空の設定を使用します。")
            return {}
        return shared_io_utils.read_config_file(path)
  
    def get_domain(self) -> Domain:
        self._ensure_data_is_downloaded()
        return Domain.load(self._get_local_path_for("domain.yml"))
  
    def get_nlu_data(self, language: Optional[Text] = "en") -> TrainingData:
        self._ensure_data_is_downloaded()
        path_to_nlu = self._get_local_path_for("data/nlu.yml")
        return nlu_loading.load_data(path_to_nlu)
  
  
    # ======================== 決定的な修正（Rasaのソースコードを参照） ========================
    def get_stories(self, interpreter: "NaturalLanguageInterpreter") -> StoryGraph:
        """
        すべてのストーリーとルールファイルをロードしてマージする。
        YAMLStoryReaderは、ファイル内の 'stories:' と 'rules:' キーを同時に解析できる。
        """
        self._ensure_data_is_downloaded()
  
        # 1. ストーリーまたはルールを含む可能性のあるすべてのファイルを特定する
        story_like_files = [
            self._get_local_path_for("data/stories.yml"),
            self._get_local_path_for("data/rules.yml"),
        ]
        # 存在しないファイルを除外する (例：S3上にrules.ymlがない場合)
        files_to_process = [f for f in story_like_files if f is not None]
  
        if not files_to_process:
            logger.debug("S3でstoriesまたはrulesファイルが見つかりませんでした。")
            return StoryGraph([])
  
        # 2. YAMLStoryReaderを使用して統一的に読み込み、マージする
        all_steps = []
        reader = YAMLStoryReader()
        domain = self.get_domain()
  
        for file_path in files_to_process:
            logger.debug(f"YAMLStoryReaderを使用して '{file_path}' から読み込んでいます...")
            # read_from_file はストーリーのステップとルールのステップを含むリストを返す
            steps = reader.read_from_file(file_path, domain)
            all_steps.extend(steps)
  
        return StoryGraph(all_steps)
  
    # 3. get_rules メソッドの削除
    # get_stories() がすべてのロジックを処理するため、もはや get_rules() は不要です。
    # ======================================================================
```

> [!note]  
> Rasa 2.0以前は、StoriesとRulesは異なるファイル形式（`.md` と `.yml`）でした。Rasa 2.0からは、それらは同じYAML形式に統一されました。一つの`data/stories.yml`ファイルには、`stories:`と`rules:`の両方のトップレベルキーを含めることができます。Rasaは、**.ymlファイルが `stories:` または `rules:` キーを含む可能性がある**限り、それを「ストーリーファイル」として認識します。
> 
> **簡単に言えば、Rasaチームは設計上の決定を下しました：StoriesとRulesのためにほぼ同じメソッド（`get_stories` と `get_rules`）を2つ作成する代わりに、それらをより汎用的な一つのプロセスに統合しました。このプロセスは `get_stories()` によってトリガーされ、すべての情報を含む `StoryGraph` を返します。** `StoryGraph` は非常に重要なデータ構造です。これは単なるストーリーのリストではなく、**すべての対話パスを含むグラフ構造**であり、以下の要素から構成されるパスが含まれています：**Stories**からのパスと**Rules**からのパス。
> 
> したがって、`get_stories()` メソッドが `StoryGraph` オブジェクトを返すとき、そのオブジェクトは**内部にファイルから読み取られたすべてのルールデータを既に保持しています**。フォーマットが統一されたため、Rasaは統一されたリーダー `YAMLStoryReader` も設計しました。このリーダーの役割は、YAMLファイルを開き、その中の`stories`と`rules`をすべて読み取ることです。

#### 5. ルートディレクトリのconfigファイルを修正する

`config.yml` ファイルに `importer` を追加し、私たちが作成したカスタムデータインポーターを追加します。

```
importers:
- name: cloud_importer.S3Importer  # フォーマット：ファイル名.クラス名
```

> [!note]  
> テスト時には、カスタムデータインポーターをルートディレクトリに配置したため、ここでのインポート形式は「ファイル名.クラス名」となっています。特定のフォルダに配置することも可能で、その場合、configでのインポート形式は「パッケージ名.ファイル名.クラス名」となります。

#### 6. Rasaのトレーニング/起動

すべての環境変数を設定した後、以下のコマンドでRasaサーバーをトレーニング/起動できます。

```
rasa train

rasa shell

rasa run
```


### リンク
[https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa](https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa)

[https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa](https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa)  
[https://gist.github.com/wochinge/1ea8e9ac9a3231fb3dbe463cda20278e#file-git_importer-py](https://gist.github.com/wochinge/1ea8e9ac9a3231fb3dbe463cda20278e#file-git_importer-py)