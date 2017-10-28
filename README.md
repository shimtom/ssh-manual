# SSH Manual

## Usage
### 基本

```bash
$ ssh -p [ポート番号] user@host
```
```
Are you sure you want to continue connecting (yes/no)? # 初回接続時の質問には`yes`と答える.
user@host's password:                                  # userのパスワードを入力
```
* Note  
    `-p`を使用しない場合はポート番号として`22`が使用される.


### 鍵認証を使用した接続
```bash
$ ssh -i /path/to/private_key -p [ポート番号] user@host
```
* Note
    - 事前に使用する秘密鍵と対となる公開鍵をホストに登録する必要がある.
    - 鍵の指定を省略した場合`~/.ssh/id_rsa`が使用される.
    - 鍵が存在しない場合はパスワード認証が行われる.

#### 鍵の事前登録
1. 鍵の作成  

    RSA暗号化方式で鍵長4096bitの秘密鍵,公開鍵を作成する.

    ```bash
    $ ssh-keygen -t rsa -b 4096
    ```
    ```
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/you/.ssh/id_rsa): # 鍵の保存場所を入力.デフォルトで問題ない場合は何も入力せず`Enter`
    Created directory '/Users/you/.ssh'.
    Enter passphrase (empty for no passphrase):                   # 鍵の使用時に使用するパスワードを入力する.設定しなくても良いがセキュリティ上は設定した方が好ましい.
    Enter same passphrase again:                                  # パスワードを再入力
    ```
    指定したディレクトリに`id_rsa`(秘密鍵)と`id_rsa.pub`(公開鍵)が作成される.  
    * Note
        - 非対話で作成することも可能.

            ```bash
            $ ssh-keygen -t rsa -b 4096 -N password -f /path/to/ssh_keys/id_rsa
            ```
        - 鍵の名称は`id_rsa`でなくてもよい.ただし,秘密鍵に対し,公開鍵は`[秘密鍵名].pub`と定められている.

2. SSH Keyを作成したディレクトリにアクセス制限をかける  
    安全のため所有者以外には読み書き不可にする.
    ```bash
    $ chmod 700 ~/.ssh
    ```

3. 鍵の登録
    * `ssh-copy-id`を使用する場合

        ```bash
        $ ssh-copy-id -i /path/to/private_key -p [ポート番号] user@host
        ```
        ```
        Are you sure you want to continue connecting (yes/no)? # 初回接続時の質問には`yes`と答える.
        user@host's password:                                  # userのパスワードを入力
        ```

        * Note
            - 秘密鍵と公開鍵は同じディレクトリに存在しなければならない.

    * `ssh-copy-id`を使用しない場合

        ```bash
        $ cat ~/.ssh/id_rsa.pub | ssh -p [ポート番号] user@host "cat >> ~/.ssh/authorized_keys"
        ```
        ```
        Are you sure you want to continue connecting (yes/no)? # 初回接続時の質問には`yes`と答える.
        user@host's password:                                  # userのパスワードを入力
        ```

4. 接続の確認
    ```bash
    $ ssh -p [ポート番号] user@host
    ```
    `user`のパスワードを尋ねられることなく接続できる.ただし,鍵の作成時にパスワードを設定した場合は鍵のパスワードを尋ねられる.

## Troubleshooting
* ssh接続時に`WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`と警告され接続できない.  
    `~/.ssh/known_hosts`内に接続先のホスト名またはipアドレスなどから始まる行があるので削除すれば解決する.


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
