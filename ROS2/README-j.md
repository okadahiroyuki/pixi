# ROS2
このチュートリアルでは、pixi を使って ROS 2 パッケージを開発する方法を説明します。

チュートリアルは上から順に実行することを前提として書かれており、途中の手順を飛ばすとエラーになる場合があります。

対象読者は、
- ROS 2 に慣れている開発者で、
- 開発ワークフローに Pixi を試してみたい人
  
です。

## 前提条件
- 事前に pixi がインストールされている必要があります。
まだインストールしていない場合は、インストールガイドの手順に従ってください。
このチュートリアルのキモは、必要なのは pixi だけという点です。
- Windows の場合は、開発者モード (Developer mode) を有効にすることが推奨されます。
設定 → 更新とセキュリティ → 開発者向け → 開発者モード

## Pixi ワークスペースの作成
```
pixi init my_ros2_project -c robostack-humble -c conda-forge
cd my_ros2_project
```
これで次のようなディレクトリ構造が作成されているはずです：
```
my_ros2_project
├── .gitattributes
├── .gitignore
└── pixi.toml
```
pixi.toml は、このワークスペースのマニフェストファイルです。
内容は次のようになっているはずです：
```
[workspace]
name = "my_ros2_project"
version = "0.1.0"
description = "Add a short description here"
authors = ["User Name <user.name@email.url>"]
channels = ["robostack-humble", "conda-forge"]
# Your project can support multiple platforms, the current platform will be automatically added.
platforms = ["linux-64"]

[tasks]

[dependencies]
```
- channels に指定したものはパッケージのリポジトリで、prefix.dev の Web から検索することができます。
- platforms はサポートしたいシステムを表し、Pixi では複数プラットフォームをサポートできますが、
どのプラットフォームを対象とするかを明示することで、依存パッケージがそのプラットフォームに対応しているかを Pixi が検証できます。
- その他のフィールドは、プロジェクトに合わせて自由に編集できます。

## ROS 2 の依存パッケージを追加する

Pixi ワークスペースを使う場合、システム側に手動で依存パッケージをインストールする必要はありません。
必要なものはすべて pixi 経由で追加することで、他のユーザーも同じワークスペースを問題なく利用できるようになります。

まずは有名な turtlesim の例から始めましょう：
```
pixi add ros-humble-desktop ros-humble-turtlesim
```

これで ros-humble-desktop と ros-humble-turtlesim パッケージがマニフェストに追加されます。
ネットワーク速度にもよりますが、ROS がワークスペースフォルダ（.pixi）にインストールされるため、多少時間がかかる場合があります。

インストールが終わったら、turtlesim のサンプルを実行してみます：
```
pixi run ros2 run turtlesim turtlesim_node
```

あるいは、シェルを起動してアクティブ化された環境から実行することもできます：
```
pixi shell
ros2 run turtlesim turtlesim_node
```

これで、pixi 上で ROS 2 が動作しました。おめでとうございます 🎉

![カメ](https://github.com/user-attachments/assets/9424c44b-b7c0-48f4-8e7d-501131e9e9e5 "カメ")

## カスタム Python ノードを追加する

ROS はカスタムノードで機能を拡張していく仕組みなので、
ここではプロジェクトに 自作の Python ノード を追加してみます。
```
pixi run ros2 pkg create --build-type ament_python --destination-directory src --node-name my_node my_package
```

このパッケージをビルドするには、いくつか追加の依存パッケージが必要です：
```
pixi add colcon-common-extensions "setuptools<=58.2.0"
```

また、作成された ROS ワークスペース用の初期化スクリプトをマニフェストファイル（pixi.toml）に登録します。

その後、ビルドコマンドを実行します：
```
pixi run colcon build
```

これにより install フォルダ内に「source 可能なスクリプト」が生成されます。

このスクリプトを Pixi のアクティベーションスクリプトとして読み込むことで、自作ノードを利用できるようにします。

通常であれば .bashrc に書くような内容ですが、Pixi では代わりに pixi.toml に設定します。

### Linux & macOS の場合
```
[activation]
scripts = ["install/setup.sh"]
```
### Windows の場合
install/setup.bat 等を指定します

## 複数プラットフォーム対応
これで、次のコマンドで自作ノードを実行できるようになりました：
```
pixi run ros2 run my_package my_node
```

## ユーザー体験をシンプルにする（タスクの活用）
Pixi には tasks（タスク） 機能があり、
マニフェストファイルにタスクを定義しておくと シンプルなコマンドで複雑な処理を実行できます。

turtlesim と自作ノード用のタスクを追加してみましょう：
```
pixi task add sim "ros2 run turtlesim turtlesim_node"
pixi task add build "colcon build --symlink-install"
pixi task add hello "ros2 run my_package my_node"
```

これで次のように実行できます：
```
pixi run sim
pixi run build
pixi run hello
```

## タスクの高度な使い方（Advanced task usage）
タスクは Pixi における強力な機能です。

たとえば：

- depends-on を使って、タスク同士を依存関係でつなげる
- cwd を指定して、ワークスペースのルートとは異なるディレクトリでコマンドを実行する
- inputs / outputs を指定して、入力が変わったときのみ実行されるタスクにする
- target 構文を使って、特定のマシン上でのみ実行されるタスクを定義する

例：
```
[tasks]
sim = "ros2 run turtlesim turtlesim_node"
build = { cmd = "colcon build --symlink-install", inputs = ["src"] }
hello = { cmd = "ros2 run my_package my_node", depends-on = ["build"] }
```

ここでは：

- hello タスクは build に依存しており、
- pixi run hello を実行すると、先に build が実行されるようになります。

## C++ ノードをビルドする

C++ ノードをビルドするには、ament_cmake とその他いくつかのビルド依存をマニフェストに追加する必要があります。
```
pixi add ros-humble-ament-cmake-auto compilers pkg-config cmake ninja
```

次に、以下のコマンドで C++ ノードを作成します：
```
pixi run ros2 pkg create --build-type ament_cmake --destination-directory src --node-name my_cpp_node my_cpp_package
```

その後、以下のコマンドでビルドして実行します：
```
# Ninja でビルドするための引数を build タスクに渡している例
# デフォルトで Ninja を使いたい場合は、これを manifest に書いておくとよい
pixi run build --cmake-args -G Ninja

pixi run ros2 run my_cpp_package my_cpp_node
```
  
