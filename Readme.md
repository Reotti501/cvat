## [Computer Vision Annotation Tool (CVAT)](https://github.com/cvat-ai/cvat) のインストール、AIツール導入までの環境構築
  wsl2でcvatのインストールからAIツールの導入までの環境構築
  
  Ubuntuインストールしたら基本的にUbuntuコンソールしか使わない

  ## 環境について
  * windows 11 or 10
  * Windows Terminal

### WSL2 / linux　(こっちを推奨)
#### 1.	Docker Desktop for Windows のインストール
  [Docker Desktop for Windows](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)からダウンロード

  ダウンロードした`.exe`をダブルクリックで実行

  完了するとcloesかlogoutとか出てくるから指示に従う
		
#### 2.	wsl2 のインストール
  powershellを管理者起動して下のコマンドを実行　この時に`Ubuntu`のユーザー名を決める

    wsl --install

#### 3.	Docker と Docker Compose のインストール
  ターミナルアプリで`Ubuntu`を開いて下記コマンドコピペ

    sudo apt-get update
    sudo apt-get --no-install-recommends install -y \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg-agent \
      software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
    sudo apt-get update
    sudo apt-get --no-install-recommends install -y \
      docker-ce docker-ce-cli containerd.io docker-compose-plugin

#### 4.	dockerコマンドを打つときにsudoをつけなくてよくする設定
    sudo groupadd docker
    sudo usermod -aG docker $USER

  念のため$USERの部分は念のためご自身のユーザーネームに置き換えること　置き換えない場合は現在ログインしているユーザーに適用される

  ユーザー名はbashまたはshellとかに表示されているはず `aaaaaa@bbbbb-MacBook-fugafuga:$`　の`aaaaaa`の部分

  > [!IMPORTANT]
  > <b>この変更を適用するためには再ログインが必要なので一度閉じて再度Ubuntuを起動</b>

#### 5.	gitインストール
    sudo apt-get install git

#### 6.	cvatクローン作製
    git clone https://github.com/cvat-ai/cvat  
    cd cvat

#### 7.	dockerコンテナ起動
    docker compose up -d

#### 8.	cvatスーパーユーザー作成
    docker exec -it cvat_server bash -ic 'python3 ~/manage.py createsuperuser'

  この際に下記の項目について入力を求められる
  |               |                   |
  | :-----------: | :---------------: |
  | username      | お好みで           |
  | email         | enterで省略可      |
  | password      | お好みで           |
  | againpassword | passwordと同じもの |

#### 9.	google chromeインストール　WSL2で動かす場合windowsのchromeを使うのでいらない
    curl https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
    sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
    sudo apt-get update
    sudo apt-get --no-install-recommends install -y google-chrome-stable

#### 10.	とりあえず確認　この段階ではAIツールは使えない
  http://localhost:8080/

  コネクトできなかった場合、どこかでミスがある

#### 11.	AIツールを入れるために一旦落とす
    docker compose down

#### 12.	docker-compose.serverless.ymlの確認
  エクスプローラを開いて左下の`Linux`から

  `\Ubuntu\home\＜Ubuntuのユーザ名＞\cvat\components\serverless￥docker-compose.serverless.yml`の4行目を確認

    services:
      nuclio:
        container_name: nuclio
        image: quay.io/nuclio/dashboard:1.13.0-amd64
        restart: always
    （以下省略）

  となっているので、<バージョン>のところに`1.13.0`と入力し、バージョン`1.13.0`のnuctlをインストール   （※2024/06時点では`1.13.0`でした）
  
    sudo mv nuctl-<バージョン>-linux-amd64 /usr/local/bin/nuctl
    sudo chmod +x /usr/local/bin/nuctl

#### 13.	このコマンドでエラーでなければOK
    nuctl

#### 14.	Dockerのaptリポジトリを設定してる(らしい)

    # Add Docker's official GPG key:
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc

    # Add the repository to Apt sources:
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
      $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update

#### 15.	最新バージョンのDocker パッケージをインストールしてるらしい
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#### 16.	Docker Engine のインストールが成功したことを確認
    sudo docker run hello-world

#### 17.	このページのリンクから欲しいAIツールを選択
[Automated labeling](https://docs.cvat.ai/docs/getting_started/overview/#automated-labeling)

#### 18.	URLのservelessからnuclioの前までを切り出す
YOLOv7の場合：

    serverless/onnx/WongKinYiu/yolov7

#### 19.	切り出したURLを下のコマンドに入れて実行
CPUを活用する場合

    ./serverless/deploy_cpu.sh <切り出したURL>

GPUを活用する場合	コマンドは通ってるみたいだけどよくわ駆らない

    nuctl create project cvat
    nuctl deploy --project-name cvat \
      --path <切り出したURL> \
      --triggers '{"myHttpTrigger": {"maxWorkers": 1}}' \
      --resource-limit nvidia.com/gpu=1

### 20.	入れたいAIツール入れたらコンテナ起動コマンドで起動
コンテナ起動コマンド

    docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml up -d

コンテナ終了コマンド（終了させないとwindows環境が重くなる）

    docker compose -f docker-compose.yml -f components/serverless/docker-compose.serverless.yml down	

#### トラブルシューティング

  7-1以降でコマンドを打った時に

  `permission denied while trying to connect to the Docker daemon soket at unix ...`  （省略）

  が帰ってきてエラーになった場合、dockerで始まるコマンドに対してsudoをつける必要がある（再起動せずにそのまま行うとこの表示が出る）

    sudo docker compose up -d

#### コマンドプロンプトのコマンド
  wslで実行されているものを表示する		
  
      wsl -l -v
  
  wsl初期化ミスったらこれで最初から
  
    wsl --unregister Ubuntu



### どうしてもwsl2を使いたくない人用
  現状AIツールの導入方法がわからなかったので手動アノテーションしかできません　詳しい人教えて

#### 1.PowerShellの起動
  インストールしたいディレクトリでエクスプローラのアドレス欄に`powershell`と入力

#### 2.gitコマンド入力	
  powershellにて下記のコマンドを実行

  この時、今いるディレクトリにcvatというファイルが自動で作られ、その中にcvatのファイルが入る形になる
  
      git clone https://github.com/cvat-ai/cvat

#### 3.	cloneしたcvatファイルに移動
    cd cvat

#### 4. dockerコンテナ起動
    docker compose up -d

#### 5.	起動したコンテナ内に入りBash シェルを開く
    docker exec -it cvat_server /bin/bash

#### 6.	スーパーユーザー作成
    python3 ~/manage.py createsuperuser

|               |                   |
| :-----------: | :---------------: |
| username      | お好みで           |
| email         | enterで省略可      |
| password      | お好みで           |
| againpassword | passwordと同じもの |

#### 7. powershellを閉じる

#### 8.	完成
chromeで http://localhost:8080/
にアクセスして、[6.	スーパーユーザー作成](#5起動したコンテナ内に入りbash-シェルを開く)
 で作ったユーザー名とパスワードを入れて通れば完了


