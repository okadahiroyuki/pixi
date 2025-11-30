# Pixi を使用したPython プロジェクト
このチュートリアルでは、Pixi を使ってシンプルな Python プロジェクトを作成する方法を紹介します。
Pixi が提供する、pdm や poetry にはまだない機能についてもいくつか取り上げます。

なぜこれは便利なのか？<br>
Pixi は conda エコシステムの上に構築されており、必要なすべての依存関係を含む Python 環境を作成できます。
特に、複数の Python インタプリタや C/C++ ライブラリへのバインディングを扱う場合に有用です。

## 基本的な使い方
### pixi.toml と pyproject.toml
Pixi は pyproject.toml と pixi.toml の2種類のマニフェスト形式をサポートします。
このチュートリアルでは、一般的な Python プロジェクトで最も普及している pyproject.toml を使用します。

#### pyproject.toml：Python 標準のプロジェクト定義ファイル
Python の公式仕様（PEP 518/621）に基づく 標準的なプロジェクト設定ファイル。
- 主な特徴
    - Python プロジェクトの メタデータを書く場所: 例：name / version / authors / requires-python
    - Python の **ビルドシステム（hatchling, setuptools など）**を指定
    - PyPI パッケージに関する情報を書く
    - 他のツール（ruff, black, pytest, mypy, etc.）の設定も含めやすい
- Pixi もこのファイルを読める
Pixi は tool.pixi.* セクションを追加して、環境管理のために使用する。


#### pixi.toml：Pixi 専用の設定ファイル（Pixi だけで使う）
Pixi が提供する 独自形式の設定ファイル。
- 主な特徴
    - Pixi の 環境管理と 依存関係管理だけを行う
    - PyPI ではなく Pixi の workspace だけで完結
    - Python 以外のプロジェクトにも使いやすい（Rust / C++ / Go など）
- Python プロジェクトでなくても使える
Pixi は "language-agnostic（言語に依存しない）" を目指しているため、pixi.toml を使えば Python 以外のプロジェクトでも Pixi 管理が可能。

Pixi は pyproject.toml を使うことを推奨しており、Python プロジェクトではほぼこちら一択になる。

### はじめよう
まず pyproject.toml を使う新しいプロジェクトを作成します：
```
pixi init pixi-py --format pyproject
```
これにより、次のようなプロジェクト構造ができます：
```
pixi-py
├── pyproject.toml
└── src
    └── pixi_py
        └── __init__.py
```
生成された pyproject.toml は次のとおりです：
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
このプロジェクトは src-layout ですが、Pixi は flat-layout と src-layout の両方をサポートします。

### pyproject.toml の内容は？
追加されたセクションを見てみましょう。

まず Pixi のメインエントリ：
```
[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["osx-arm64"]
```
- channels は conda-forge など、PyPI と似たパッケージ管理リポジトリです
- platforms はサポートするプラットフォームを指定します

次に、このプロジェクト自身のパッケージが editable に登録されています：
```
[tool.pixi.pypi-dependencies]
pixi-py = { path = ".", editable = true }
```
Pixi は editable インストールを pyproject.toml に明示的に記述します。
これにより、どの environment にパッケージを含めるかを柔軟に指定できます。

### Conda と PyPI 依存関係の管理
conda と PyPI(pip) は「どちらもパッケージ管理システム」だが、目的も仕組みも別物です。

#### conda
- パッケージは 完全にビルドされたバイナリ
- 依存関係（C ライブラリや OS 依存も含む）を一括で管理
- 「環境の再現性が高い」
例：
- numpy + OpenBLAS
- pytorch + CUDA
- opencv + FFmpeg
などが そのまま使える状態で配布される。

#### PyPI（pip）
- 主に Python コードのみ配布
- ネイティブ部分は「環境に合わせてビルド」が必要なことがある
- C/C++ 依存は OS のライブラリに依存する

例：
- GDAL（pip 版はビルドが大変）
- OpenCV（pip 版は機能制限あり）


他のパッケージに依存させたい場合：
```
cd pixi-py
pixi add black
```
これで conda の black パッケージが依存関係に追加されます：
```
[tool.pixi.dependencies]
black = ">=25.1.0,<26"
```
特定バージョンにしたい場合：
```
pixi add black=25
```
PyPI の black を使う場合：
```
pixi add black --pypi
```
結果：
```
dependencies = ["black"]
```

extras（追加オプション）も利用できます：
```
pixi add "flask[async]==3.1.0" --pypi
```
結果：
```
dependencies = ["black", "flask[async]==3.1.0"]
```

### インストール（pixi install）
Pixi は通常、環境を実行するときに pyproject.toml と同期を取ります。手動で実行したい場合は：
```
pixi install
```
.pixi ディレクトリが作成され、すべての conda / PyPI 依存関係がインストールされます。
環境は pixi.lock を元に構築されます。

### 環境に含まれているものを確認
```
pixi list
```
ここでは conda パッケージと PyPI パッケージを一覧できます。
pixi-py が editable として入っているのもわかります。
- Pixi の環境は isolated
- しかし central cache をハードリンクで再利用するため無駄が少ない

### 複数環境の作成
Pixi では 複数の environment を作成できます。
これは dependency-groups（PEP 735） と統合されています。

テスト環境用の feature を追加：
```
pixi add --pypi --feature test pytest
```
結果：
```
[dependency-groups]
test = ["pytest"]
```
環境追加：
```
pixi workspace environment add default --solve-group default --force
pixi workspace environment add test --feature test --solve-group default
```
結果：
```
[tool.pixi.environments]
default = { solve-group = "default" }
test = { features = ["test"], solve-group = "default" }
```
実行：
```
pixi install --environment test
pixi run --environment test pytest
```

### コードを動かす
src/pixi_py/__init__.py に関数を追加：
```
from rich import print

def hello():
    return "Hello, [bold magenta]World[/bold magenta]!", ":vampire:"

def say_hello():
    print(*hello())
```
rich を追加：
```
pixi add --pypi rich
```
動作確認：
```
pixi run python -c 'import pixi_py; pixi_py.say_hello()'
```

### テストを追加
tests/test_me.py
```
from pixi_py import hello

def test_pixi_py():
    assert hello() == ("Hello, [bold magenta]World[/bold magenta]!", ":vampire:")
```
テストタスク作成：
```
pixi task add --feature test test "pytest"
```
実行：
```
pixi run test
```

### test 環境と default 環境の違い
```
pixi list --explicit --environment test
pixi list --explicit
```
test 環境には pytest があり、default にはない。

環境を用途別に最適化できる。

### PyPI パッケージを conda パッケージに置き換える
rich 経由で入った pygments を確認：
```
pixi list pygments
```
PyPI 版（whl）で入っている。

conda に置き換えるには：
```
pixi add pygments
```

結果：
```
[tool.pixi.dependencies]
pygments = "=2.19.1,<3"
```
再確認：
```
pixi list pygments
```
→ conda 版に切替わった！

### 結論
このチュートリアルでは以下を学びました：
- pyproject.toml を使って Pixi の依存関係を管理する方法
- conda と PyPI の依存を同じ workspace で混在させる方法
- editable インストールの扱い
- 複数環境（test / default 等）の構築
- PyPI と conda パッケージの相互置換

Pixi は Python プロジェクト管理を柔軟で強力にします。


## pyproject.toml
Pixi では、pyproject.toml をマニフェストファイルとして使用することをサポートしています。
これにより、すべての設定を 1つのファイルにまとめることができます。

pyproject.toml は Python プロジェクトの標準です。
Python プロジェクト以外では pyproject.toml を使わないことを推奨します。
その他のタイプのプロジェクトでは、pixi.toml のほうが適しています。

### pyproject.toml の初期セットアップ
すでに pyproject.toml が存在する場合、そのフォルダで:
```
pixi init
```
を実行できます。
Pixi は自動的に以下を行います：
1. [tool.pixi.workspace] セクションの追加<br>
- Pixi に必要な platform と channel の情報が入る
2. 現在のプロジェクトを editable な PyPI 依存として追加
3. .gitignore と .gitattributes に Pixi 用のデフォルト設定を追加

pyproject.toml が存在しない場合は:
```
pixi init --format pyproject
```
を実行すると、Pixi がゼロから適切なデフォルトを含んだ pyproject.toml を生成します。


### Python のバージョン依存（requires-python）
pyproject.toml は requires-python フィールドをサポートしています。
Pixi はこのフィールドを読み取り、Python の依存関係として自動的に追加します。

例：
```
[project]
name = "my_project"
requires-python = ">=3.9"

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]
```
これは pixi.toml の次と等価です：
```
[workspace]
name = "my_project"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[dependencies]
python = ">=3.9"
```
### 依存関係（dependencies）セクション
pyproject.toml の dependencies フィールドは、
Pixi では [pypi-dependencies] として扱われます。

例：
```
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = [
    "numpy",
    "pandas",
    "matplotlib",
]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]
```

Pixi では次と等価：
```
[workspace]
name = "my_project"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-arm64", "osx-64", "win-64"]

[pypi-dependencies]
numpy = "*"
pandas = "*"
matplotlib = "*"

[dependencies]
python = ">=3.9"
```
### conda 依存で上書きも可能
pypi-dependencies を無視して conda を優先したい場合：
```
[tool.pixi.dependencies]
numpy = "*"
pandas = "*"
matplotlib = "*"
```
Pixi は conda の依存関係を優先するため、PyPI 依存は無視されます。

### オプショナル依存（optional-dependencies）
Python プロジェクトに optional dependencies がある場合、
Pixi はそれらを 同名の Pixi feature として解釈し、
該当する pypi-dependencies を設定します。

Pixi の環境に手動で追加することもできますし、
pixi init を使えば自動的に feature ごとに environment を作成します。

自己参照（他の optional group を含む）も対応しています。

例：
```
[project]
name = "my_project"
dependencies = ["package1"]

[project.optional-dependencies]
test = ["pytest"]
all = ["package2","my_project[test]"]
```

pixi init の結果：
```
[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64"]

[tool.pixi.environments]
default = {features = [], solve-group = "default"}
test = {features = ["test"], solve-group = "default"}
all = {features = ["all"], solve-group = "default"}
```

Pixi が作る環境：
```
環境	含まれる依存
default	package1
test	package1 + pytest
all	package1 + package2 + pytest
```

### Dependency Groups（PEP 735）
dependency-groups も optional-dependencies と同様に
Pixi の feature として扱われます。

例：
```
[project]
name = "my_project"
dependencies = ["package1"]

[dependency-groups]
test = ["pytest"]
docs = ["sphinx"]
dev = [{include-group = "test"}, {include-group = "docs"}]
```

pixi init 後：
```
[tool.pixi.environments]
default = {features = [], solve-group = "default"}
test = {features = ["test"], solve-group = "default"}
docs = {features = ["docs"], solve-group = "default"}
dev = {features = ["dev"], solve-group = "default"}
```

Pixi が作る環境：
```
環境	依存
default	package1
test	package1 + pytest
docs	package1 + sphinx
dev	package1 + pytest + sphinx
```

### Pixi 全機能を pyproject.toml で書く例
```
[project]
name = "my_project"
requires-python = ">=3.9"
dependencies = ["numpy","pandas","matplotlib","ruff"]

[tool.pixi.workspace]
channels = ["conda-forge"]
platforms = ["linux-64","osx-arm64","osx-64","win-64"]

[tool.pixi.dependencies]
compilers = "*"
cmake = "*"

[tool.pixi.tasks]
start = "python my_project/main.py"
lint = "ruff lint"

[tool.pixi.system-requirements]
cuda = "11.0"

[tool.pixi.feature.test.dependencies]
pytest = "*"

[tool.pixi.feature.test.tasks]
test = "pytest"

[tool.pixi.environments]
test = ["test"]

## Pytorch Installation
```

### [build-system] セクション

通常 pyproject.toml には [build-system] が存在します。
Pixi はこの設定を使用して、
PyPI パス依存（path dependency）のビルド & インストールを行います。

[build-system] が無い場合、
Pixi は uv のデフォルト（以下と同等）を使います：
```
[build-system]
requires = ["setuptools >= 40.8.0"]
build-backend = "setuptools.build_meta:__legacy__"
```

含めることが強く推奨されます。
迷う場合は以下を追加すれば安全：
```
[build-system]
build-backend = "hatchling.build"
requires = ["hatchling"]
```

Pixi は pyproject 初期化時に hatchling を採用します。

### [tool.uv.sources] による開発依存の扱い

Pixi は PyPI パッケージのビルドに uv を使うため、
[tool.uv.sources] を使って PyPI 依存のソース元を指定できます。

#### これは何に役立つ？

モノレポ構成などで、
複数の Python パッケージがお互いを参照する場合、
tool.uv.sources を使うと便利です。

例：

ディレクトリ構成：
```
.
├── main_project
│   └── pyproject.toml (a を参照)
├── a
│   └── pyproject.toml (b に依存)
└── b
    └── pyproject.toml
```

main_project の設定：
```
[tool.pixi.pypi-dependencies]
a = { path = "../a" }
```

a の pyproject.toml には：
```
[tool.uv.sources]
flask = { git = "github.com/pallets/flask", branch = "main" }
b = { path = "../b" }
```

## PyTorch のインストール
### 概要
このガイドでは、Pixi と PyTorch を統合する方法を説明します。
Pixi は複数の方法で PyTorch をインストールすることができます。
- conda-forge の Conda チャンネル経由で PyTorch をインストールする（推奨）
- uv との連携を用いて、PyPI から PyTorch をインストールする（利用可能なバージョンが最も多い）
- pytorch Conda チャンネルから PyTorch をインストールする（レガシー）

これらの選択肢によって、要件に応じて最適なインストール方法を選べます。

### システム要件
PyTorch の文脈において、システム要件は Pixi に対して「CUDA 関連パッケージをインストール・利用できるかどうか」を伝える役割を持ちます。
これにより、依存関係の解決時に互換性が保たれます。

ここでの鍵となるメカニズムが、__cuda のような「仮想パッケージ (virtual packages)」の利用です。
仮想パッケージは、利用可能なシステム機能（例：CUDA のバージョン）を表します。

system-requirements.cuda = "12" と指定することで、
「CUDA 12 が利用可能であり、解決時に使用できる」ことを Pixi に伝えます。

例：

- あるパッケージが __cuda >= 12 に依存している場合、Pixi は適切なバージョンを解決します。
- あるパッケージがバージョン指定なしに __cuda に依存している場合、利用可能な任意の CUDA バージョンが使われます。
- system-requirements.cuda を適切に指定しない場合、Pixi はデフォルトで CPU版のみ の PyTorch およびその依存パッケージをインストールします。

### conda-forge からのインストール
conda-forge チャンネルを使って PyTorch をインストールすることができます。
これらは conda-forge コミュニティによってメンテナンスされているビルドです。
NVIDIA 提供のパッケージを直接利用することで、各パッケージが互いに正しく動作するようにできます。

#### 最小構成の conda-forge + CUDA 版 PyTorch インストール例
```
[workspace]
channels = ["https://prefix.dev/conda-forge"]
name = "pytorch-conda-forge"
platforms = ["linux-64", "win-64"]

[system-requirements]
cuda = "12.0"

[dependencies]
pytorch-gpu = "*"
```

特定バージョンの CUDA パッケージを明示的にインストールしたい場合は、cuda-version パッケージに依存させます。
これにより、他のパッケージが解決時にこの指定を解釈します。
cuda-version パッケージは __cuda 仮想パッケージと cudatoolkit パッケージのバージョンを制約します。
これによって、cudatoolkit の正しいバージョンがインストールされ、依存ツリーが正しく解決されます。

#### conda-forge PyTorch インストールに CUDA バージョンを追加
```
[dependencies]
pytorch-gpu = "*"
cuda-version = "12.6.*"
```
conda-forge では PyTorch の CPU 版 をインストールすることもできます。
よくあるユースケースとしては、「CUDA 搭載マシン用」と「非 CUDA マシン用」で 2つの環境を用意するパターンです。

#### CPU 用環境の追加例
```
[workspace]
channels = ["https://prefix.dev/conda-forge"]
name = "pytorch-conda-forge"
platforms = ["linux-64"]

[feature.gpu.system-requirements]
cuda = "12.0"

[feature.gpu.dependencies]
cuda-version = "12.6.*"
pytorch-gpu = "*"

[feature.cpu.dependencies]
pytorch-cpu = "*"

[environments]
cpu = ["cpu"]
default = ["gpu"]
```

これらの環境は pixi run コマンドで実行できます。
```
pixi run --environment cpu python -c "import torch; print(torch.cuda.is_available())"
pixi run -e gpu python -c "import torch; print(torch.cuda.is_available())"
```

ここから先は、自分の依存パッケージやタスクを追加して拡張していけばよいでしょう。

代表的なパッケージへのリンク：
- pytorch
- pytorch-cpu
- pytorch-gpu
- torchvision
- torchaudio
- cuda-version

### PyPI からのインストール
uv との統合により、PyPI から PyTorch をインストールすることもできます。

[dependencies] と [pypi-dependencies] を混在させる場合の注意

この方法で torch パッケージを扱う場合、torch に依存するパッケージも PyPI からインストールすべきです。
つまり、
```
Conda パッケージが PyPI の torch に依存するような形にしない
```
ことが重要です。

その理由は、Pixi の依存解決が 2段階 になっているためです：

1. まず Conda パッケージを解決
2. その後 PyPI パッケージを解決

したがって、「Conda パッケージが PyPI パッケージに依存する」構造は解決不可能になります。

#### PyTorch インデックス（PyPI 側）
PyTorch のパッケージは、PyTorch チームが管理する専用インデックスから提供されます。
これは Conda チャンネルに似た仕組みです。
PyTorch をこのインデックスからインストールするには、依存関係ごとに index を明示するのが最善です（その index を必ず使わせるため）。

- CPU のみ: https://download.pytorch.org/whl/cpu
- CUDA 11.8: https://download.pytorch.org/whl/cu118
- CUDA 12.1: https://download.pytorch.org/whl/cu121
- CUDA 12.4: https://download.pytorch.org/whl/cu124
- ROCm 6: https://download.pytorch.org/whl/rocm6.2

#### PyPI から PyTorch をインストールする例
```
[workspace]
channels = ["https://prefix.dev/conda-forge"]
name = "pytorch-pypi"
platforms = ["osx-arm64", "linux-64", "win-64"]

[dependencies]
# pytorch と互換性のある python バージョンが必要
python = ">=3.11,<3.13"

[pypi-dependencies]
torch = { version = ">=2.5.1", index = "https://download.pytorch.org/whl/cu124" }
torchvision = { version = ">=0.20.1", index = "https://download.pytorch.org/whl/cu124" }

[target.osx.pypi-dependencies]
# macOS は CUDA 非対応なので CPU 版を使用
torch = { version = ">=2.5.1", index = "https://download.pytorch.org/whl/cpu" }
torchvision = { version = ">=0.20.1", index = "https://download.pytorch.org/whl/cpu" }
```

Pixi に対して、CPU 版 と GPU 版で別々の環境を使わせることもできます。

#### 複数の PyTorch バージョン（CPU / GPU）を環境で分ける例
```
[workspace]
channels = ["https://prefix.dev/conda-forge"]
name = "pytorch-pypi-envs"
platforms = ["linux-64", "win-64"]

[dependencies]
# pytorch と互換性のある python バージョンが必要
python = ">=3.11,<3.13"

[feature.gpu]
system-requirements = { cuda = "12.0" }
[feature.gpu.pypi-dependencies]
torch = { version = ">=2.5.1", index = "https://download.pytorch.org/whl/cu124" }
torchvision = { version = ">=0.20.1", index = "https://download.pytorch.org/whl/cu124" }

[feature.cpu.pypi-dependencies]
torch = { version = ">=2.5.1", index = "https://download.pytorch.org/whl/cpu" }
torchvision = { version = ">=0.20.1", index = "https://download.pytorch.org/whl/cpu" }

[environments]
gpu = { features = ["gpu"] }
# CPU 環境をデフォルトにする
default = { features = ["cpu"] }
```

実行例：
```
pixi run --environment cpu python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
pixi run -e gpu python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

#### macOS と CUDA を pypi-dependencies で混在させる場合
pypi-dependencies を使うとき、Pixi は PyPI 依存を解決するための「解決用環境 (solve environment)」を作成します。
これは、

1. Conda 依存をインストール
2. その環境内で PyPI 依存を解決

という流れで処理します。

このとき、macOS マシン上で Linux/Windows 向け CUDA 対応版の PyTorch を解決しようとすると問題になります。
macOS はこれらの CUDA 環境をサポートしていないため、CUDA 用の Conda 依存がインストールできず、解決が失敗します。

現状：<br>
Pixi のメンテナはこの制限を認識しており、
こうしたケースでのクロスプラットフォーム依存解決を可能にする解決策に取り組んでいます。

それまでは、Linux や Windows など CUDA 対応のマシン上で解決処理を行う必要があるかもしれません。

