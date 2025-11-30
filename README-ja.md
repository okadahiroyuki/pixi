# Pixi: パッケージ管理を簡単に
Pixiは、condaエコシステムを基盤として構築されたクロスプラットフォーム対応の多言語パッケージマネージャー兼ワークフローツールです。

## Pixiの特徴
- Condaパッケージを使用してPython、C++、Rを含む複数言語をサポート。利用可能なパッケージは[prefix.dev](https://prefix.dev/)で確認可能。
- 主要なオペレーティングシステム全てに対応：Linux、Windows、macOS（Apple Siliconを含む）。
- 常に最新の[ロックファイル](https://pixi.sh/latest/workspace/lockfile/)を含む。
- クリーンでシンプルなCargo風のコマンドラインインターフェースを提供。
- プロジェクト単位またはシステム全体でのツールインストールが可能。
- 完全にRustで記述され、rattlerライブラリを基盤として構築。

## Pixiを知る
### 公式
[公式ドキュメント](https://pixi.sh/latest/)

[prefix.dev](https://prefix.dev/)

[prefix.dev](https://github.com/prefix-dev)

### 先人の知恵
[Anacondaはもう古い？次世代パッケージ管理ツール【Pixi】を徹底解剖](https://qiita.com/Shigurex/items/e5763324a282f0568866)

[【Pixi】新世代のCondaパッケージマネージャー](https://qiita.com/_masa_u/items/8d262ab3484be1b9a5ec)

[pixiで始めるmacOSにおけるROS 2開発環境構築（Gazebo Fortressとnav2を添えて)](https://qiita.com/dandelion1124/items/754a6d2d0ebd7bc827b9)

## インストールと設定
インストールはとても簡単です。
詳しくは[公式ドキュメント](https://pixi.sh/latest/)を参照。

### UbuntuやmacOS
#### インストール
```
curl -fsSL https://pixi.sh/install.sh | bash
```
#### 設定
~/.bashrc
```
# 自動補完
eval "$(pixi completion --shell bash)"
```
~/.zshrc
```
autoload -Uz compinit && compinit  # redundant with Oh My Zsh
eval "$(pixi completion --shell zsh)"
```

### Windows
#### インストール
```
powershell -ExecutionPolicy ByPass -c "irm -useb https://pixi.sh/install.ps1 | iex"

powershell -c "irm -useb https://pixi.sh/install.ps1 | more"
```
#### 設定
Microsoft.PowerShell_profile.ps1 の末尾に以下を追加してください。このファイルの場所は、PowerShell で $PROFILE 変数をクエリすることで確認できます。通常、パスは Windows では ~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1、Unix 系では ~/.config/powershell/Microsoft.PowerShell_profile.ps1 です。
```
(& pixi completion --shell powershell) | Out-String | Invoke-Expression
```

### アップデート
こまめにアップデートしましょう。
```
pixi self-update
```

## 始めてみよう
### 新しいワークスペースの作成
新しいPixiワークスペースを作成するには、`pixi init`コマンドを使用します。
```
pixi init my_workspace
```
コマンドが成功すると以下のような構造を持つディレクトリが作成されます。
```
my_workspace
├── .gitattributes
├── .gitignore
└── pixi.toml
```
pixi.tomlファイルは、Pixiワークスペースのマニフェストです。
ワークスペースに関するすべての情報（チャンネル、プラットフォーム、依存関係、タスクなど）が含まれています。

pixi initで作成されるファイルは、次のような最小限のマニフェストです。
```
pixi.toml

[workspace]
authors = ["YOUR-NAME <YOUR-EMAIL>"]
channels = ["conda-forge"]
name = "my_workspace"
platforms = ["osx-arm64"]
version = "0.1.0"

[tasks]

[dependencies]
```
`platforms`にはお使いのPCのアーキテクチャに従って、　"win-64", "linux-64", "osx-64", "osx-arm64" などが自動的に設定されます。


`pixi init` 時に --format オプションで pyproject を指定することで、 pixi.toml ではなく pyproject.toml でパッケージ管理を行うことが可能です。
```
pixi init my_workspace --format pyproject
```
```
pyproject.toml

[project]
authors = [{name = "YOUR-NAME", email = "YOUR-EMAIL"}]
dependencies = []
name = "my_workspace"
requires-python = ">= 3.11"
version = "0.1.0"

[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"]

[tool.pixi.pypi-dependencies]
my_workspace = { path = ".", editable = true }

[tool.pixi.tasks]
```

### pixi.toml と　pyproject.toml の違い
pixi.toml はCondaエコシステムを基盤とする次世代パッケージマネージャー Pixi の設定ファイルであり、pyproject.toml はPythonの標準的なパッケージングと依存関係の定義に使われるファイルです。

主な違いは、pixi.toml はクロスプラットフォームでの環境管理と、PythonだけでなくC/C++、Rなど他の言語の依存関係も管理できるのに対し、pyproject.toml は主にPythonのプロジェクト設定を記述する点です。 

#### pixi.toml
- 目的: Pixi という次世代パッケージマネージャーで、クロスプラットフォームな環境（Windows, macOS, Linux）を管理するための設定ファイルです。
- 特徴:
  - Condaエコシステムを基盤にしています。
  - Python、C/C++、Rなど、マルチランゲージの依存関係を管理できます。
  - プロジェクトの依存関係と環境の管理を統合的に行います。 

#### pyproject.toml
- 目的: PEP 518 で標準化された、Pythonプロジェクトのビルドシステムや依存関係、パッケージングに関する設定を定義するファイルです。
- 特徴:
  - PyPI（Python Package Index）で広く使われています。
  - Poetry や uv といったモダンなパッケージマネージャーで利用されます。
  - ビルドツール（例: setuptools）や、パッケージのメタデータ（名前、バージョンなど）を記述します。
  - Pythonパッケージの依存関係管理に特化しています。 


### 依存関係の管理
ワークスペースを作成したら、依存関係を追加できます。
Pixiでは、ワークスペースに依存関係を追加するために`pixi add`コマンドを使用します。このコマンドはデフォルトで、conda依存関係を`pixi.toml`に追加し、依存関係を解決し、ロックファイルを書き込み、環境内にパッケージをインストールします。
例えば、numpyとpytestをワークスペースに追加してみましょう。
```
pixi add numpy pytest
```
これにより、`pixi.toml`に以下の行が追加されます。
```
pixi.toml

[dependencies]
numpy = ">=2.2.6,<3"
pytest = ">=8.3.5,<9"
```
追加したい依存関係のバージョンを指定することもできます。
```
pixi add numpy==2.2.6 pytest==8.3.5
```

### PyPI依存関係
Pixiは通常、依存関係にcondaパッケージを使用しますが、PyPIからの依存関係を追加することも可能です。
Pixiは両方のソースから同じパッケージをインストールしようとせず、両者の競合を回避します。

ワークスペースに追加したい場合は、--pypiフラグを使用して行うことができます:
```
pixi add --pypi httpx
```
これにより、PyPIからhttpxパッケージがワークスペースに追加されます。
```
pixi.toml

[pypi-dependencies]
httpx = ">=0.28.1,<0.29"
```

### ロックファイル#
Pixiは依存関係を解決する際に常にロックファイルを作成します。
このファイルにはワークスペースの依存関係（およびそれらの依存関係）の正確なバージョンがすべて含まれます。
これにより再現可能な環境が実現され、他者と共有したり、テストやデプロイに使用したりできます。

ロックファイルは pixi.lock と呼ばれ、ワークスペースのルートディレクトリに作成されます。
```
pixi.lock

version: 6
environments:
  default:
    channels:
    - url: https://prefix.dev/conda-forge/
    indexes:
    - https://pypi.org/simple
    packages:
      osx-arm64:
      - conda: https://prefix.dev/conda-forge/osx-arm64/bzip2-1.0.8-h99b78c6_7.conda
      - pypi: ...
packages:
- conda: https://prefix.dev/conda-forge/osx-arm64/bzip2-1.0.8-h99b78c6_7.conda
  sha256: adfa71f158cbd872a36394c56c3568e6034aa55c623634b37a4836bd036e6b91
  md5: fc6948412dbbbe9a4c9ddbbcfe0a79ab
  depends:
  - __osx >=11.0
  license: bzip2-1.0.6
  license_family: BSD
  size: 122909
  timestamp: 1720974522888
- pypi: ...
```

### タスクの管理
Pixiには組み込みのクロスプラットフォームタスクランナーが搭載されており、マニフェスト内でタスクを定義できます。
タスクは、プロジェクト開発中に何度も繰り返す必要があるコマンド（またはコマンドの連鎖）と考えてください（例：テストの実行）。

これはタスクを他者と共有し、異なるマシン上で同じ環境において同一のタスクが確実に実行されるようにする優れた方法です。
タスクはpixi.tomlファイルの[tasks]セクションで定義されます。

pixi task addコマンドを実行することで、ワークスペースにタスクを追加できます。
```
pixi task add hello "echo Hello, World!"
```
これにより、pixi.tomlファイルに以下の行が追加されます。
```
pixi.toml

[tasks]
hello = "echo Hello, World!"
```
その後、pixi run コマンドを使用してタスクを実行できます。
```
pixi run hello
```

### 環境
Pixiは常にワークスペース用の環境（「デフォルト」環境）を作成します。
この環境には依存関係が含まれ、タスクが実行されます。
また、1つのワークスペースに複数の環境を含めることも可能です。
これらの環境はワークスペースのルートにある.pixi/envsディレクトリに配置されます。

これらの環境の使用は、pixi run または pixi shell コマンドを実行するだけで簡単です。
pixi run は残りの入力を環境内でコマンド（入力が定義済みタスク名と一致する場合はタスク）として実行し、pixi shell は環境内で新しいシェルセッションを生成します。
どちらのコマンドも環境を「アクティブ化」します。

```
pixi run python -VV
```
あるいは
```
pixi shell
python -VV
exit
```

## Pixiの基本的な使い方
Pixiは多くのことを行えますが、シンプルに使えるように設計されています。
Pixiの基本的な使い方を確認していきましょう。

### ワークスペースの管理
- pixi init -  現在のディレクトリに新しいPixiマニフェストを作成
- pixi add - マニフェストに依存関係を追加
- pixi remove - マニフェストから依存関係を削除
- pixi update - マニフェスト内の依存関係を更新
- pixi upgrade - 特定のバージョンに固定している場合でも、マニフェスト内の依存関係を最新版にアップグレード
- pixi lock - マニフェストのロックファイルを作成または更新
- pixi info - ワークスペースに関する情報を表示
- pixi run - マニフェストで定義されたタスク、または現在の環境内の任意のコマンドを実行
- pixi shell - 現在の環境でシェルを起動
- pixi list - 現在の環境内の全依存関係を表示
- pixi tree - 現在の環境内の依存関係ツリーを表示
- pixi clean - マシンから環境を削除

### グローバル環境の管理
Pixiはツールや環境のグローバルインストールを管理できます。
環境を共用の場所にインストールするため、どこからでも利用可能です。
- pixi global install - パッケージをグローバル空間内の独自の環境にインストールします。
- pixi global uninstall - グローバル空間から環境をアンインストールします。
- pixi global add - 既存のグローバルインストール済み環境にパッケージを追加します。
- pixi global sync - インストールしたいすべての環境を記述したグローバルマニフェストと、グローバルにインストールされた環境を同期する。
- pixi global edit - グローバルマニフェストを編集する。
- pixi global update - グローバル環境を更新する。
- pixi global list - インストール済みのすべての環境を一覧表示する。

### 単発コマンドの実行
Pixiは特定の環境で単発コマンドを実行できます。
```
pixi exec - 一時環境でコマンドを実行します。
pixi exec --spec - 特定の仕様で一時環境を指定しコマンドを実行します。
```
例えば、
```
> pixi exec python -VV
Python 3.13.5 | packaged by conda-forge | (main, Jun 16 2025, 08:24:05) [Clang 18.1.8 ]
> pixi exec --spec "python=3.12" python -VV
Python 3.12.11 | packaged by conda-forge | (main, Jun  4 2025, 14:38:53) [Clang 18.1.8 ]
```

### 複数の環境
Pixiワークスペースでは複数の環境を管理できます。
環境は1つ以上の機能で構成されます。
```
pixi add --feature - 機能にパッケージを追加
pixi task add --feature - 特定の機能にタスクを追加
pixi workspace environment add - ワークスペースに環境を追加
pixi run --environment - 特定の環境でコマンドを実行
pixi shell --environment - 特定の環境をアクティブ化
pixi list --environment - 特定の環境の依存関係を表示
```

### タスク
Pixiは組み込みのタスクランナーを使用してクロスプラットフォームタスクを実行できます。
これは事前定義されたタスクでも、通常の実行可能ファイルでも構いません。
```
pixi run - タスクまたはコマンドを実行
pixi task add - マニフェストに新しいタスクを追加
```
タスクは他のタスクを依存関係として持つことができます。
以下はより複雑なタスクのユースケース例です
```
pixi.toml

[tasks]
build = "make build"
# using the toml table view
[tasks.test]
cmd = "pytest"
depends-on = ["build"]
```

### マルチプラットフォームサポート
Pixiは最初から複数のプラットフォームをサポートしています。
ワークスペースがサポートするプラットフォームを指定すると、Pixiがそれらのプラットフォームと互換性のある依存関係を確保します。
```
pixi add --platform - 特定のプラットフォームのみにパッケージを追加
pixi workspace platform add - サポートしたいプラットフォームをワークスペースに追加
```

### ユーティリティ
Pixiには、デバッグや設定管理を支援する一連のユーティリティが付属しています。
```
pixi info - 現在のワークスペースとグローバル設定に関する情報を表示します。
pixi config - Pixiの設定を表示または編集します。
pixi tree - 現在の環境における依存関係のツリーを表示します。
pixi list - 現在の環境にあるすべての依存関係を一覧表示します。
pixi clean - マシンからワークスペース環境を削除します。
pixi help - Pixi コマンドのヘルプを表示します。
pixi help <サブコマンド> - 特定の Pixi コマンドのヘルプを表示します。
pixi auth - conda チャネルの認証を管理します。
pixi search - 設定済みのチャネル内でパッケージを検索します。
pixi completion - Pixi コマンド用のシェル補完スクリプトを生成します。
```


## チュートリアル
### Python
[Pixi を使用したPython プロジェクト](https://github.com/okadahiroyuki/pixi/blob/main/python/README-j.md)

### ROS2


[ROS2でVOICEVOXを使う環境をpixiで構築する](https://github.com/okadahiroyuki/voicevox_ros2_pixi)


