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
```
# インストール
curl -fsSL https://pixi.sh/install.sh | bash
```
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
```
powershell -ExecutionPolicy ByPass -c "irm -useb https://pixi.sh/install.ps1 | iex"

powershell -c "irm -useb https://pixi.sh/install.ps1 | more"
```
Microsoft.PowerShell_profile.ps1 の末尾に以下を追加してください。このファイルの場所は、PowerShell で $PROFILE 変数をクエリすることで確認できます。通常、パスは Windows では ~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1、Unix 系では ~/.config/powershell/Microsoft.PowerShell_profile.ps1 です。
```
(& pixi completion --shell powershell) | Out-String | Invoke-Expression
```

## アップデート
こまめにアップデートしましょう。
```
pixi self-update
```

