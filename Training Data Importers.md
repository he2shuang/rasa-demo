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


### リンク
[https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa](https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa)

[https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa](https://medium.com/rasa-blog/customizing-training-data-importing-bc26ab1efbaa)  
[https://gist.github.com/wochinge/1ea8e9ac9a3231fb3dbe463cda20278e#file-git_importer-py](https://gist.github.com/wochinge/1ea8e9ac9a3231fb3dbe463cda20278e#file-git_importer-py)