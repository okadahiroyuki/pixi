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

![エビフライトライアングル](https://github.com/user-attachments/assets/9424c44b-b7c0-48f4-8e7d-501131e9e9e5 "サンプル")


