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







## pyproject.toml


## PyTorchのインストール
