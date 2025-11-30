# Pixi を使用したPython プロジェクト

## 基本的な使用方法
このチュートリアルでは、Pixi を使用してシンプルな Python プロジェクトを作成する方法を紹介します。
pdm や poetry などには現在含まれていない、Pixi が提供する機能の一部を解説します。

### なぜ有用なのか？
Pixi は conda エコシステムを基盤としており、必要な依存関係をすべて含む Python 環境を作成できます。
これは特に、複数のPythonインタプリタやC/C++ライブラリへのバインディングを扱う際に有用です。
例えばPyPIのGDALにはCバイナリ依存関係がありませんが、condaパッケージには存在します。
一方、PyPI経由でしか入手できないパッケージも存在し、Pixiはそれらもインストール可能です。
両方の長所を兼ね備えたこの手法、早速試してみましょう！

### pixi.toml と pyproject.toml
このプロジェクトでは、2つのマニフェスト形式をサポートしています(pyproject.toml と pixi.toml) 。
このチュートリアルでは、Pythonプロジェクトで最も一般的な形式である pyproject.toml 形式を使用します。

### さあ始めよう
pyproject.tomlファイルを使用する新しいプロジェクトを作成することから始めましょう。
```
pixi init pixi-py --format pyproject
```
これにより、以下の構造を持つプロジェクトディレクトリが作成されます。
```
pixi-py
├── pyproject.toml
└── src
    └── pixi_py
        └── __init__.py
```
プロジェクトのpyproject.tomlは次のようになります。
```
[project]
dependencies = []
name = "pixi-py"
requires-python = ">= 3.11"
version = "0.1.0"

[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["osx-arm64"]

[tool.pixi.pypi-dependencies]
pixi_py = { path = ".", editable = true }

[tool.pixi.tasks]
```

### pyproject.tomlには何が含まれているのか？
追加されたセクションとpyproject.tomlの編集方法を見ていきましょう。

pyproject.tomlファイルに追加された最初のエントリは以下の通りです。
```
# Main pixi entry
[tool.pixi.workspace]
channels = ["conda-forge"]
# This is your machine platform by default
platforms = ["osx-arm64"]
```

チャンネルとプラットフォームは[tool.pixi.workspace]セクションに追加されます。conda-forgeのようなチャンネルはPyPIと同様にパッケージを管理しますが、言語を跨いだ異なるパッケージを扱えます。キーワードプラットフォームはワークスペースがサポートするプラットフォームを決定します。

pixi_pyパッケージ自体は編集可能な依存関係として追加されます。これはパッケージが編集モードでインストールされることを意味し、環境を再インストールすることなく、パッケージに変更を加えてその変更を環境に反映させることができます。


### CondaとPyPIの両方の依存関係を管理する
プロジェクトは通常、他のパッケージに依存しています。
```
cd pixi-py # Move into the project directory
pixi add black
```
これにより、blackパッケージがCondaパッケージとしてpyproject.tomlファイルに追加されます。
その結果、pyproject.tomlには以下の内容が付加されます：
```
[tool.pixi.dependencies]
black = ">=25.1.0,<26"
```
ただし、使用すべきバージョンについて厳密に指定することも可能です。
```
pixi add black=25
```
その結果、
```
[tool.pixi.dependencies]
black = "25.*"
```
condaチャネルでは利用できないがPyPIに公開されているパッケージが存在する場合があります。
```
pixi add black --pypi
```
その結果、pyproject.tomlのdependenciesキーに追加されます
```
dependencies = ["black"]
```
pypi依存関係を使用する際、他のパッケージが追加機能として提供するオプション依存関係を利用できます。
例えば、flaskはasync依存関係オプションを提供しており、--pypiキーワードで追加できます：
```
pixi add "flask[async]==3.1.0" --pypi
```
依存関係エントリを更新して
```
dependencies = ["black", "flask[async]==3.1.0"]
```

### インストール: pixi install
Pixiは環境を実行する際、常にpyproject.tomlファイルで環境が最新であることを保証します。手
動で実行したい場合は、以下を実行できます:
```
pixi install
```
ワークスペースのルートに.pixiという新しいディレクトリが作成されました。
この環境はConda環境であり、CondaおよびPyPIの依存関係がすべてインストールされています。

この環境は常にpixi.lockファイルの結果であり、このファイルはpyproject.tomlファイルから生成されます。
このファイルには、プラットフォームを問わず環境にインストールされた依存関係の正確なバージョンが記載されています。

### 環境には何が含まれているのか？
pixi list を使用すると、環境に含まれる内容を確認できます。
これは基本的にロックファイル（pixi.lock）をより見やすく表示したものです：
```
Package          Version     Build               Size       Kind   Source
asgiref          3.8.1                           68.5 KiB   pypi   asgiref-3.8.1-py3-none-any.whl
black            24.10.0     py313h8f79df9_0     388.7 KiB  conda  black
blinker          1.9.0                           23.9 KiB   pypi   blinker-1.9.0-py3-none-any.whl
bzip2            1.0.8       h99b78c6_7          120 KiB    conda  bzip2
ca-certificates  2024.12.14  hf0a4a13_0          153.4 KiB  conda  ca-certificates
click            8.1.8       pyh707e725_0        82.7 KiB   conda  click
flask            3.1.0                           335.9 KiB  pypi   flask-3.1.0-py3-none-any.whl
itsdangerous     2.2.0                           45.8 KiB   pypi   itsdangerous-2.2.0-py3-none-any.whl
...
...
tzdata           2025a       h78e105d_0          120 KiB    conda  tzdata
werkzeug         3.1.3                           743 KiB    pypi   werkzeug-3.1.3-py3-none-any.whl
```
ここでは、さまざまなcondaおよびPypiパッケージが一覧表示されています。
ご覧の通り、現在作業中のpixi-pyパッケージは編集可能な状態でインストールされています。
Pixiの環境はすべて分離されていますが、中央キャッシュディレクトリからハードリンクされたファイルを再利用します。
つまり、同じパッケージを持つ複数の環境を作成でき、個々のファイルはディスク上に一度だけ保存されることになります。








## pyproject.toml


## PyTorchのインストール
