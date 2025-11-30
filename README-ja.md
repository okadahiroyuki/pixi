# Pixi: パッケージ管理を簡単に
pixiは、condaエコシステムを基盤として構築されたクロスプラットフォーム対応の多言語パッケージマネージャー兼ワークフローツールです。

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

## アップデート
こまめにアップデートしましょう。
```
pixi self-update
```

## Pixiワークスペースの作成
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

## 依存関係の管理
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

## PyPI依存関係
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

## ロックファイル#
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

## タスクの管理
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

## 環境
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

