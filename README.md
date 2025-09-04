# Exastro IT Automation in Docker Compose
## 概要
Docker Compose を利用することで、Exastro IT Automation を簡単に起動することが可能です。
  - (based on [exastro-it-automation](https://github.com/exastro-suite/exastro-it-automation))
  - (based on [exastro-platform](https://github.com/exastro-suite/exastro-platform))

## 前提条件

### ハードウェア要件(最小構成)

Ansible Automation Platfrom と連携しない場合（GitLabを起動しない場合）のハードウェア要件は下記の通りとなります。

|  種別 | 要求リソース |
| ------------ | ------ |
| CPU          | 2Cores |
| メモリ       | 8GB    |
| ディスク容量 | 40GB   |

### OSとソフトウェアの動作確認済みのバージョン

| OS            | podman                 | Docker Compose         | Docker                 |
| ------------- | ---------------------- | ---------------------- | ---------------------- |
| RHEL 9.4      | podman version 4.9.4-rhel | Docker Compose version v2.20.3 | -                  |
| RHEL 8.9      | podman version 4.9.4-rhel | Docker Compose version v2.20.3 | -                  |
| Almalinux 8.9 | -                         | -                              | Docker version 26.1.3 |
| Almalinux 9.5 | -                         | -                              | Docker version 28.1.1 |


## 環境構築

### Git インストール

[Getting Started - Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) の手順に従ってインストールをしてください。

### （Docker利用時）Docker Engine のインストール

[Install Docker Engine](https://docs.docker.com/engine/install/) に従ってインストールをしてください。

### （Podman利用時）Podman Engine のインストール


```
sudo dnf module enable -y container-tools:rhel8

sudo dnf module install -y container-tools:rhel8

sudo dnf -y install podman-docker

podman version
```

### （Podman利用時）Docker Compose のインストール

[Install Compose standalone](https://docs.docker.com/compose/install/standalone/#on-linux) に従ってインストールをしてください。

## 起動準備
はじめに、各種構成ファイルを取得します。
docker-compose.yml などの起動に必要なファイル群を取得します。

```
git clone https://github.com/exastro-suite/exastro-docker-compose.git
```

以降は、*exastro-docker-compose* ディレクトリで作業をします。

```shell
cd exastro-docker-compose
```

環境変数のサンプルファイルからコピーします。

```shell
cp .env.sample .env  # 値を変更することなく起動が可能ですが、変更を行いたい場合は .envファイルを編集してください。
```

末尾の[パラメータ一覧](#パラメータ一覧)を参考に、起動に必要な環境情報を登録します。

```
# ENCRYPT_KEY の作成は以下のコマンドを参考にしてください。
head -c 32 /dev/urandom | base64
```


## コンテナ起動


### exastroコンテナの起動方法
*docker* もしくは *docker-compose* コマンドを使いコンテナを起動します。


```shell
# docker コマンドを利用する場合(Docker環境)
docker compose --profile all up -d  --wait

# docker-compose コマンドを利用する場合(Podman環境)
docker-compose --profile all up -d  --wait
```

### （オプション）起動するコンテナを限定する場合の起動方法
起動するコンテナを限定する場合は、プロファイル (*--profile*) で対象を指定することで、起動するコンテナを選択することが可能です。

| プロファイル名                 | 対象となるコンテナ                                 | スケーリング                 |
| ------------------------------ | -------------------------------------------------- | ---------------------------- |
| *all*                          | すべてのコンテナ(batchを除く)                      |                              |
| *minimal*                      | Exastro IT Automation、Exastro Platform、Keycloak コンテナ(batchを除く) |                    |
| *base*                         | Exastro IT Automation、Exastro Platform、Keycloak、MariaDB コンテナ(batchを除く)  |               |
| *keycloak*                     | Keycloak コンテナ                                  | 不可 (対応予定)              |
| *mariadb*                      | MariaDB コンテナ                                   | 不可 (対応予定)              |
| *gitlab*                       | GitLab コンテナ                                    | 不可 (対応予定)              |
| *mongo*                        | MongoDB コンテナ                                   | 不可 (対応予定)              |
| *migration*                    | インストール・アップグレード用コンテナ             | 不可 (必ず同時に1つのみ起動) |
| *web*                          | Web 系のコンテナ                                   | 可能                         |
| *backyard*                     | Backyard 関連のコンテナ                            | 不可 (対応予定)              |
| *ita*                          | Exastro IT Automation 関連のコンテナ(batchを除く)   | 一部可能                     |
| *oase*                         | Exastro OASE 関連のコンテナ(MongoDB コンテナを除く)  | 不可 (対応予定)              |
| *platform*                     | Exastro Platform 関連のコンテナ                    | 可能                         |
| *batch*                        | バッチ処理関連のコンテナ(Crontabに登録が必要)      | 不可 (不要)                  |

以下の例では、**base** プロファイルを指定することで、Exastro IT Automationを利用する上で、最低限必要なコンテナのみを起動する方法をご紹介します。

```shell
# docker コマンドを利用する場合(Docker環境)
docker compose --profile base up -d  --wait

# docker-compose コマンドを利用する場合(Podman環境)
docker-compose --profile base up -d  --wait
```

## Crontabの設定例
exastro-suite/exastro-docker-composeを/home/test_user配下にgit cronしている前提で、
ita-by-file-autocleanを毎日00時01分、ita-by-file-autocleanを毎日00時02分に実行する場合のcrontabに設定する例です。

```shell
# docker コマンドを利用する場合(Docker環境)
01 00 * * * cd /home/test_user; /usr/bin/docker compose --profile batch run ita-by-file-autoclean > /dev/null 2>&1
02 00 * * * cd /home/test_user; /usr/bin/docker compose --profile batch run ita-by-execinstance-dataautoclean > /dev/null 2>&1

# docker-compose コマンドを利用する場合(Podman環境)
01 00 * * * cd /home/test_user; /usr/bin/podman unshare docker-compose --profile batch run ita-by-file-autoclean > /dev/null 2>&1
02 00 * * * cd /home/test_user; /usr/bin/podman unshare docker-compose --profile batch run ita-by-execinstance-dataautoclean > /dev/null 2>&1
```

## Organization作成とアクセス

### 設定例

| 設定項目                      | 設定値                               |
| ----------------------------- | ------------------------------------ |
| システム管理者                | admin                                |
| システム管理者パスワード      | password                             |
| Organization ID               | sample-org                           |
| Organization 管理者           | admin                                |
| Organization 管理者パスワード | password                             |
| EXASTRO_EXTERNAL_URL          | http://exastro.example.com:30080     |
| EXASTRO_MNG_EXTERNAL_URL      | http://exastro-mng.example.com:30081 |
| GITLAB_PROTOCOL               | http                                 |
| GITLAB_HOST                   | gitlab.example.com                   |
| GITLAB_PORT                   | 40080                                |


### Organization 作成
システム管理者用コンソールから作成可能です。

### 各ページのURL
#### システム管理者用コンソール
http://exastro-mng.example.com:30081/

#### Organization ページ
http://exastro.example.com:30080/sample-org/platform/

#### Gitlab
http://gitlab.example.com:40080


## パラメータ一覧

| パラメータ                                        | 説明                                                                                                                                                    | 変更                          | デフォルト値・選択可能な設定値                                                                                      |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| COMPOSE_PROJECT_NAME                              | Docker Compose におけるプロジェクト名                                                                                                                   | 可                            | exastro                                                                                                             |
| COMPOSE_PROFILES                                  | Docker Compose におけるプロファイル名<br>指定するプロファイルは上記を参照。                                                                             | 可                            | all                                                                                                                 |
| NETWORK_ID                                        | Exastro で利用する Docker ネットワークのID                                                                                                              | 可                            | 20230101                                                                                                            |
| LOGGING_MAX_SIZE                                  | コンテナ毎のログファイルの1ファイルあたりのファイルサイズ                                                                                               | 可                            | 10m                                                                                                                 |
| LOGGING_MAX_FILE                                  | コンテナ毎のログファイルの世代数                                                                                                                        | 可                            | 10                                                                                                                  |
| TZ                                                | Exastro システムで使用するタイムゾーン                                                                                                                  | 可                            | Asia/Tokyo                                                                                                          |
| DEFAULT_LANGUAGE                                  | Exastro システムで使用する規定の言語                                                                                                                    | 可                            | ja                                                                                                                  |
| LANGUAGE                                          | Exastro システムで使用する言語                                                                                                                          | 可                            | en                                                                                                                  |
| DB_VENDOR                                         | 起動するデータベースコンテナ                                                                                                                            | 可                            | **"mariadb"** (デフォルト): MariaDB を利用<br>**"mysql"**: MySQL を利用                                             |
| DB_VERSION                                        | 起動するデータベースコンテナのバージョン                                                                                                                | 可                            | 10.11.4                                                                                                             |
| DB_PORT                                           | 起動するデータベースコンテナが公開するポート番号(TCP)                                                                                                   | 可                            | 3306                                                                                                                |
| DB_ADMIN_USER                                     | 起動するデータベースコンテナの管理ユーザー名                                                                                                            | 可                            | root                                                                                                                |
| DB_ADMIN_PASSWORD                                 | 起動するデータベースコンテナの管理ユーザーのパスワード                                                                                                  | **必須**                      | Ch@ngeMeDBAdm                                                                                                       |
| GITLAB_VERSION                                    | 起動する Gitalb コンテナのバージョン                                                                                                                    | 可                            | latest                                                                                                              |
| GITLAB_PROTOCOL                                   | 起動する Gitalb コンテナの公開時のプロトコル                                                                                                            | 可                            | http                                                                                                                |
| GITLAB_HOST                                       | 起動する Gitalb コンテナの公開時のURL<br>AAPから接続できる必要がある。                                                                                  | **必須**                      | **"gitlab"**: デフォルト<br>*"空白"*: Git連携をしない場合                                                           |
| GITLAB_PORT                                       | 起動する Gitalb コンテナの公開時のポート番号                                                                                                            | 可                            | 40080                                                                                                               |
| GITLAB_ROOT_PASSWORD                              | 起動する Gitalb コンテナの root アカウントの初期パスワード                                                                                              | **必須**                      | Ch@ngeMeGL                                                                                                          |
| GITLAB_ROOT_TOKEN                                 | 起動する Gitalb コンテナの root トークン                                                                                                                | **必須**                      | change-this-token                                                                                                   |
| API_KEYCLOAK_HOST                                 | Keycloak API エンドポイントのホスト名、もしくは、FQDN                                                                                                   | 不要                          | keycloak                                                                                                            |
| API_KEYCLOAK_PORT                                 | Keycloak API エンドポイントのポート番号                                                                                                                 | 不要                          | 8080                                                                                                                |
| API_KEYCLOAK_PROTOCOL                             | Keycloak エンドポイントのプロトコル                                                                                                                     | 不要                          | http                                                                                                                |
| KEYCLOAK_HOST                                     | Keycloak エンドポイントのホスト名、もしくは、FQDN                                                                                                       | 不要                          | keycloak                                                                                                            |
| KEYCLOAK_PORT                                     | Keycloak API エンドポイントのポート番号                                                                                                                 | 不要                          | 8080                                                                                                                |
| KEYCLOAK_PROTOCOL                                 | Keycloak エンドポイントのプロトコル                                                                                                                     | 不要                          | http                                                                                                                |
| KEYCLOAK_MASTER_REALM                             | Keycloak のマスターレルム名                                                                                                                             | 不要                          | Master                                                                                                              |
| SYSTEM_ADMIN                                      | システム管理者のユーザ名                                                                                                                                | 不要                          | admin                                                                                                               |
| SYSTEM_ADMIN_PASSWORD                             | システム管理者のパスワード                                                                                                                              | **必須**                      | Ch@ngeMeKCAdm                                                                                                       |
| KEYCLOAK_DB_VENDOR                                | Keycloak が利用するデータベースエンジン                                                                                                                 | 可 (外部のデータベース利用時) | mariadb                                                                                                             |
| KEYCLOAK_DB_HOST                                  | Keycloak が利用するデータベースのホスト名                                                                                                               | 可 (外部のデータベース利用時) | mariadb                                                                                                             |
| KEYCLOAK_DB_PORT                                  | Keycloak が利用するデータベースのポート番号                                                                                                             | 可 (外部のデータベース利用時) | 3306                                                                                                                |
| KEYCLOAK_DB_USER                                  | Keycloak が利用するデータベースのユーザ名                                                                                                               | 可 (外部のデータベース利用時) | keycloak                                                                                                            |
| KEYCLOAK_DB_PASSWORD                              | Keycloak が利用するデータベースのパスワード                                                                                                             | **必須**                      | Ch@ngeMeKCADB                                                                                                       |
| KEYCLOAK_DB_DATABASE                              | Keycloak が利用するデータベース名                                                                                                                       | 可                            | keycloak                                                                                                            |
| EXTERNAL_URL_PORT                                 | Exastro Platform エンドポイントの公開ポート番号                                                                                                         | 可                            | 80                                                                                                                  |
| EXASTRO_EXTERNAL_URL                              | Exastro Platform エンドポイントの公開URL                                                                                                                | **必須**                      | https://localhost:80                                                                                                |
| EXTERNAL_URL_MNG_PORT                             | Exastro Platform 管理コンソールのエンドポイントの公開ポート番号                                                                                         | 可                            | 81                                                                                                                  |
| EXASTRO_MNG_EXTERNAL_URL                          | Exastro Platform 管理コンソールのエンドポイントの公開URL                                                                                                | **必須**                      | https://localhost:81                                                                                                |
| EXASTRO_HTTPS                                     | Exastro Platform にSSL証明書を設定有無                                                                                                                  | 可                            | false                                                                                                               |
| CERTIFICATE_FILE                                  | Exastro Platform サーバー証明書のファイル名                                                                                                             | 可                            | localhost.crt                                                                                                       |
| PRIVATE_KEY_FILE                                  | Exastro Platform 秘密鍵のファイル名                                                                                                                     | 可                            | localhost.key                                                                                                       |
| ENCRYPT_KEY                                       | Exastro Platform 内で保管するデータの暗号化と復号のための AES キー                                                                                      | **必須**                      | 'Q2hhbmdlTWUxMjM0NTY3ODkwMTIzNDU2Nzg5MDEyMzQ='                                                                      |
| PLATFORM_VERSION                                  | Exastro Platform のバージョン                                                                                                                           | 可                            | 1.10.0                                                                                                               |
| PLATFORM_DB_VENDOR                                | Exastro Platform が利用するデータベースエンジン                                                                                                         | 可 (外部のデータベース利用時) | **"mariadb"** (デフォルト): MariaDB を利用<br>**"mysql"**: MySQL を利用                                             |
| PLATFORM_DB_HOST                                  | Exastro Platform が利用するデータベースのホスト名                                                                                                       | 可 (外部のデータベース利用時) | mariadb                                                                                                             |
| PLATFORM_DB_PORT                                  | Exastro Platform が利用するデータベースのポート番号                                                                                                     | 可 (外部のデータベース利用時) | 3306                                                                                                                |
| PLATFORM_DB_USER                                  | Exastro Platform が利用するデータベースのユーザ名                                                                                                       | 可 (外部のデータベース利用時) | app_user                                                                                                            |
| PLATFORM_DB_PASSWORD                              | Exastro Platform が利用するデータベースのパスワード                                                                                                     | **必須**                      | Ch@ngeMePFDB                                                                                                        |
| PLATFORM_DB_ADMIN_USER                            | Exastro Platform が利用するデータベースの管理者ユーザ名                                                                                                 | 可 (外部のデータベース利用時) | app_user                                                                                                            |
| PLATFORM_DB_ADMIN_PASSWORD                        | Exastro Platform が利用するデータベースの管理者パスワード                                                                                               | **必須**                      | Ch@ngeMeDBAdm                                                                                                       |
| PLATFORM_DB_DATABASE                              | Exastro Platform が利用するデータベース名                                                                                                               | 可                            | platform                                                                                                            |
| AUDIT_LOG_ENABLED                                 | 監査ログの出力有無                                                                                                                                      | 可                            | "True" (デフォルト): 出力する<br>"False": 出力しない                                                                |
| AUDIT_LOG_PATH                                    | 監査ログのファイル名（ファイルパス）                                                                                                                    | 可                            | exastro-audit.log (デフォルト)                                                                                      |
| AUDIT_LOG_FILE_MAX_BYTE                           | 監査ログファイルの最大サイズ(Byte)を指定できます                                                                                                        | 可                            | 100000000 (デフォルト)                                                                                              |
| AUDIT_LOG_BACKUP_COUNT                            | 監査ログファイルのバックアップカウント数<br>監査ログファイルの最大サイズ(Byte)を超えた際、ファイル名に"."＋数値で指定された値の分、バックアップされます | 可                            | 30 (デフォルト)                                                                                                     |
| ITA_VERSION                                       | Exastro IT Automation のバージョン                                                                                                                      | 可                            | 2.6.1                                                                                                               |
| ITA_DB_VENDOR                                     | Exastro IT Automation が利用するデータベースエンジン                                                                                                    | 可 (外部のデータベース利用時) | **"mariadb"** (デフォルト): MariaDB を利用<br>**"mysql"**: MySQL を利用                                             |
| ITA_DB_HOST                                       | Exastro IT Automation が利用するデータベースのホスト名                                                                                                  | 可 (外部のデータベース利用時) | mariadb                                                                                                             |
| ITA_DB_PORT                                       | Exastro IT Automation が利用するデータベースのポート番号                                                                                                | 可 (外部のデータベース利用時) | 3306                                                                                                                |
| ITA_DB_USER                                       | Exastro IT Automation が利用するデータベースのユーザ名                                                                                                  | 可 (外部のデータベース利用時) | ITA_USER                                                                                                            |
| ITA_DB_PASSWORD                                   | Exastro IT Automation が利用するデータベースのパスワード                                                                                                | **必須**                      | Ch@ngeMeITADB                                                                                                       |
| ITA_DB_USER                                       | Exastro IT Automation が利用するデータベースのユーザ名                                                                                                  | 可 (外部のデータベース利用時) | ITA_USER                                                                                                            |
| ITA_DB_PASSWORD                                   | Exastro IT Automation が利用するデータベースのパスワード                                                                                                | **必須**                      | Ch@ngeMeDBAdm                                                                                                       |
| ITA_DB_DATABASE                                   | Exastro IT Automation が利用するデータベース名                                                                                                          | 可                            | ITA_DB                                                                                                              |
| UID                                               | Exastro IT Automation の実行ユーザ                                                                                                                      | 不要                          | **1000** (デフォルト): Docker 利用の場合<br>**0**: Podman 利用の場合                                                |
| HOST_DOCKER_GID                                   | ホスト上の Docker のグループID                                                                                                                          | 不要                          | **999**: Docker 利用の場合<br>**0**: Podman 利用の場合                                                              |
| HOST_DOCKER_SOCKET_PATH                           | ホストの Docker もしくは Podman のソケットファイルのパス                                                                                                | 可                            | **/var/run/docker.sock**: Docker 利用の場合<br>**/run/user/1000/podman/podman.sock**: Podman 利用の場合             |
| PWD                                               | Exastro IT Automation が利用する共有フォルダのパス                                                                                                      | 可                            | カレントディレクトリー                                                                                              |
| ANSIBLE_AGENT_IMAGE                               | Ansible Agent のコンテナイメージのリポジトリ名                                                                                                          | 不要                          | exastro/exastro-it-automation-by-ansible-agent                                                                      |
| ANSIBLE_AGENT_IMAGE_TAG                           | Ansible Agent のコンテナイメージのタグ                                                                                                                  | 不要                          | 2.6.1                                                                                                               |
| SYSTEM_ANSIBLE_EXECUTION_LIMIT                    | Exastro システム全体の Movement 最大実行数                                                                                                              | 可                            | 25                                                                                                                  |
| ORG_ANSIBLE_EXECUTION_LIMIT_DEFAULT               | オーガナイゼーションごとの Movement デフォルト実行数                                                                                                    | 可                            | 25                                                                                                                  |
| ORG_ANSIBLE_EXECUTION_LIMIT_MAX                   | オーガナイゼーションごとの Movement 最大実行数                                                                                                          | 可                            | 1000                                                                                                                |
| ORG_ANSIBLE_EXECUTION_LIMIT_DESCRIPTION           | Movement 最大実行数の説明文表記                                                                                                                         | 不要                          | Maximum number of movement executions for organization default                                                      |
| ORG_COMMON_UPLOAD_FILE_LIMIT_DEFAULT              | オーガナイゼーションごとのアップロード可能なファイルサイズのデフォルト上限（バイト）                                                                    | 可                            | 104857600                                                                                                           |
| ORG_COMMON_UPLOAD_FILE_LIMIT_MAX                  | オーガナイゼーションごとのアップロード可能なファイルサイズの上限（バイト）                                                                              | 可                            | 107374182400                                                                                                        |
| ORG_COMMON_UPLOAD_FILE_LIMIT_DESCRIPTION          | アップロード可能なファイルサイズ上限の説明文表記                                                                                                        | 不要                          | Maximum byte size of upload file for organization default                                                           |
| SYSTEM_MENU_EXPORT_IMPORT_BUFFER_SIZE             | Exastro システム全体の メニューエクスポート/インポートのバッファサイズ                                                                                  | 可                            | 10000                                                                                                               |
| SYSTEM_MENU_EXPORT_IMPORT_BUFFER_SIZE_DESCRIPTION | メニューエクスポート/インポートのバッファサイズの説明文表記                                                                                             | 不要                          | Maximum buffer size of menu Export/import for System default (Used for DB fetch size, file stream read size)        |
| ORG_MENU_EXPORT_IMPORT_BUFFER_SIZE_DEFAULT        | オーガナイゼーションごとの メニューエクスポート/インポートのバッファサイズ                                                                              | 可                            | 1000                                                                                                                |
| ORG_MENU_EXPORT_IMPORT_BUFFER_SIZE_MAX            | オーガナイゼーションごとの メニューエクスポート/インポートのバッファサイズ 最大値                                                                       | 可                            | 10000                                                                                                               |
| ORG_MENU_EXPORT_IMPORT_BUFFER_SIZE_DESCRIPTION    | メニューエクスポート/インポートのバッファサイズの説明文表記                                                                                             | 不要                          | Maximum buffer size of menu Export/import for Organization default (Used for DB fetch size, file stream read size) |
| ORG_COMMON_MAINTENANCE_RECORDS_LIMIT_DEFAULT      | オーガナイゼーションごとの 一度に処理できるレコード数                                                                              | 可                            | 10000                                                                                                                |
| ORG_COMMON_MAINTENANCE_RECORDS_LIMIT_MAX          | オーガナイゼーションごとの 一度に処理できるレコード数 最大値                                                                       | 可                            | 100000                                                                                                               |
| ORG_COMMON_MAINTENANCE_RECORDS_LIMIT_DESCRIPTION  | オーガナイゼーションごとの 一度に処理できるレコード数の説明文表記                                                                                             | 不要                          | Maximum number of maintenance records processing for organization default |
| MONGO_VERSION                                     | 起動するMongoDBのバージョン                                                                                                                             | 可                            | 6.0.7                                                                                                               |
| MONGO_INITDB_ROOT_USERNAME                        | 起動するMongoDBコンテナの管理ユーザー名                                                                                                                 | 可                            | adminer                                                                                                             |
| MONGO_INITDB_ROOT_PASSWORD                        | 起動するMongoDBコンテナの管理ユーザーのパスワード                                                                                                       | **必須**                      | Ch@ngeMeDBAdm                                                                                                       |
| MONGO_OPTION_SSL                                  | Exastro OASE 利用時のMongoDBへのSSL接続の使用                                                                                                           | 可                            | FALSE                                                                                                               |
| MONGO_SCHEME                                      | Exastro OASE 利用時のMongoDBのスキーム                                                                                                                  | 可                            | mongodb                                                                                                             |
| MONGO_HOST                                        | Exastro OASE 利用時のMongoDBのホスト名                                                                                                                  | 可                            | mongodb                                                                                                             |
| MONGO_PORT                                        | Exastro OASE 利用時のMongoDBのポート番号                                                                                                                | 可                            | 27017                                                                                                               |
| MONGO_ADMIN_USER                                  | Exastro OASE 利用時のMongoDBコンテナの管理ユーザー名                                                                                                    | 可                            | adminer                                                                                                             |
| MONGO_ADMIN_PASSWORD                              | Exastro OASE 利用時のMongoDBコンテナの管理ユーザーのパスワード                                                                                          | **必須**                      | Ch@ngeMeDBAdm                                                                                                       |
