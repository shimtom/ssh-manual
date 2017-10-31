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

### X転送
オプションに`-X`を使用することでX転送ができる.これにより,ホスト側でGUIアプリケーションを起動した場合,クライアント側にそのGUIを表示する.

```bash
$ ssh -X -p [ポート番号] user@host
```

### SSHを利用したファイル,ディレクトリの転送
`scp`もしくは`sftp`を使用すればファイル,ディレクトリの転送が行える.

#### scp
* クライアントからホストに転送

    - ファイルの転送

        ```bash
        $ scp /path/to/file -P [ポート番号] user@host:/path/to/destination/
        ```

    - ディレクトリの転送
    
        ```bash
        $ scp -r /path/to/directory -P [ポート番号] user@host:/path/to/destination/
        ```

* ホストからクライアントに転送

    - ファイルの転送

        ```bash
        $ scp -P [ポート番号] user@host:/path/to/file /path/to/destination/
        ```

    - ディレクトリの転送
    
        ```bash
        $ scp -r -P [ポート番号] user@host:/path/to/directory /path/to/destination/
        ```

* Note
    - ポート番号の指定はsshと異なり`-P`であることに注意
    - その他,基本的にsshとコマンドオプションなどは同じ
    - もちろん,鍵認証なども使用できる

#### sftp
省略.

#### scp, sftpの違い
* scp
    - 転送の再開ができない.
    - SFTPに比べて高速(と言われている)
    - sshが使用できるなら使用可能

* sftp
    - 転送を中断しても, 途中から再開できる
    - SCPよりも転送速度が遅い
    - 対話的
    - ホスト側で使用を許可する必要がある


## 鍵の事前登録
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

### 設定ファイルの使用
ssh接続時のポート番号やユーザー名,使用する秘密鍵などの設定を設定ファイルにまとめることができる.
1. `~/.ssh/config`の作成

    ```bash
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
    - `Host`: ホスト名.`HostName`が存在する場合には好きな名前を使用できる.
    - `HostName`: ホストのアドレスかipアドレス.
    - `User`: ログインユーザー名
    - `IdentityFile`: 秘密鍵のパス.秘密鍵を使用しない場合はなくてよい.
    - `Port`: 接続に使用するポート番号
    - `TCPKeepAlive`: `yes` or `no`. 接続状態を継続したい場合`yes`.
    - `IdentitiesOnly`: `yes` or `no`.秘密鍵を明示する場合は`yes`.
    - `ServerAliveInterval`: 一定期間サーバからデータが送られてこないときにタイムアウトする秒数.  

3. 設定の使用  

    デフォルトでは`~/.ssh/config`が自動で使用される.設定ファイルを指定したい場合は`-F`を使用する.
    
    ```bash
    $ ssh -F /path/to/configfile host
    ```

* 参考:
    https://koejima.com/archives/583/#i-3

## Troubleshooting
* ssh接続時に`WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`と警告され接続できない.  
    `~/.ssh/known_hosts`内に接続先のホスト名またはipアドレスなどから始まる行があるので削除すれば解決する.

* ssh接続時に`sign_and_send_pubkey: signing failed: agent refused operation`と警告される.

    ```bash
    # [Private key]は接続時に使用する秘密鍵に変更してください.
    $ ssh-add [Private Key]
    ```

## TODO
- `sftp`の使用方法を追加する.
- X転送での`-X`と`-Y`の違いを記述する.


