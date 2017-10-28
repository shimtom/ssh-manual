# SSH Manual

## Usage
### 基本

```bash
$ ssh -p [ポート番号] user@host
```
* Note: `-p`を使用しない場合はポート番号として`22`が使用される.

初回接続時の質問には`yes`と答える.
```
Are you sure you want to continue connecting (yes/no)? yes
```
パスワードを入力する.
```
user@host's password:
```
#### 鍵認証を使用する場合
```
$ ssh -i ~/.ssh/id_rsa user@host -p [ポート番号]
```
事前に使用する秘密鍵と対となる公開鍵をホストに登録する必要がある.

## SSH Key の使用
### SSH Key の作成
1. RSA暗号化方式で鍵長4096bitの鍵を作成
    ```
    $ ssh-keygen -t rsa -b 4096
    ```
    SSH Keysの保存場所を入力.デフォルトの場所で問題ない場合は何も入力しない.  
    ```
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/you/.ssh/id_rsa):
    ```
    SSH Key自体に使用するパスワードを入力する.セキュリティ上は入力した方が好ましい.  
    ```
    Created directory '/Users/you/.ssh'.
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    ```
    指定したディレクトリに`id_rsa`(秘密鍵)と`id_rsa.pub`(公開鍵)が作成される.  
2. SSH Keyを作成したディレクトリにアクセス制限をかける  
    ```
    $ chmod 700 ~/.ssh
    ```

### SSH 公開鍵の登録
#### `ssh-copy-id`を使用する場合
```bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub user@remote_host
```
#### `ssh-copy-id`を使用しない場合
```
$ cat ~/.ssh/id_rsa.pub | ssh user@remote_host "cat >> ~/.ssh/authorized_keys"
```

## 設定ファイルの使用
ssh接続時のポート番号やユーザー名,使用する秘密鍵などの設定を設定ファイルにまとめることができる.
1. `~/.ssh/config`の作成
    ```
    $ touch ~/.ssh/config
    ```
2. `~/.ssh/config`に設定を書き込む
    ```dat:~/.ssh/config
    # サーバーA
    Host serverA
        HostName serverA.com
        User user
        IdentityFile ~/.ssh/id_rsa.serverA
        Port 22
        TCPKeepAlive yes
        IdentitiesOnly yes

    # サーバーB
    Host serverB
        HostName serverB.com
        User user
        IdentityFile ~/.ssh/id_rsa.serverB
        Port 2002
        TCPKeepAlive yes
        IdentitiesOnly yes
        ServerAliveInterval 120
    ```
    * `Host`: ホスト名.`HostName`が存在する場合には好きな名前を使用できる.
    * `HostName`: ホストのアドレスかipアドレス.
    * `User`: ログインユーザー名
    * `IdentityFile`: 秘密鍵のパス.秘密鍵を使用しない場合はなくてよい.
    * `Port`: 接続に使用するポート番号
    * `TCPKeepAlive`: `yes` or `no`. 接続状態を継続したい場合`yes`.
    * `IdentitiesOnly`: `yes` or `no`.秘密鍵を明示する場合は`yes`.
    * `ServerAliveInterval`: 一定期間サーバからデータが送られてこないときにタイムアウトする秒数.


### 参考
https://euske.github.io/openssh-jman/ssh_config.html
