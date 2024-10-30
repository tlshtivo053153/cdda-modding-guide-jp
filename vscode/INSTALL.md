# VSCodeのインストール

## Windows

### wingetでのインストール
管理者としてPowerShellを開いて次のコマンドを実行する。

```
winget install --id=Microsoft.VisualStudioCode  -e
```

### 手動でのインストール
以下の公式ページからダウンロードして、インストールする。
https://code.visualstudio.com/Download

#### インストーラーの違い
##### User Installer
このインストーラーは現在のユーザーでのみVSCodeを利用可能になるものである。
管理者権限を持たなくてもインストール可能。
特別な理由がなければこれをインストールすれば良い。

##### System Installer
このインストーラーは全てのユーザーでVSCodeを利用可能になるものである。
インストールに管理者権限が必要である。

##### .zip
インストール不要のものである。
コンピュータにインストールする権限をもっていない場合に利用する。

##### CLI
GUIが必要ない場合に利用するもの。
例えば、VSCodeをサーバーとして実行して、Web版のVSCodeからアクセスするなどである。

> [!NOTE]
> CLIについては、よくわからず適当に書いた。
> たぶん、あっているだろう。

## Mac
公式のダウンロードページからダウンロードして、インストールする。
https://code.visualstudio.com/Download

## Linux
公式のパッケージマネージャーを利用してインストールする。
SnapやFlatpakを用いてインストールしても良い。
