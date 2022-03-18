# Drone CICD
簡易架設方式
先到 Github 建立 Oauth App Homepage URL & Authorization callback URL 分別填入 https://your.droneapp.com https://your.droneapp.com/login 建立完成後會取得 Key1 : ID , Key2 : Secret , Key3 : 隨便建一組Key (drone環境變數會用到)

建立 docker-compose & .env .env要填入5個值 3個Key和2個Domain

DRONE_GITHUB_CLIENT_ID='Key1'
DRONE_GITHUB_CLIENT_SECRET='Key2'
DRONE_RPC_SECRET='Key3'
DRONE_SERVER_HOST='your.droneapp.com'
DRONE_SERVER_PROTO=https
DRONE_RPC_SERVER='rpc.your.droneapp.com'
DRONE_RPC_PROTO=https
## 環境變數填完後
sh drone.sh
## 執行完腳本還要做的事
開 TCP 4500 port
綁DNS
SSL
綁Github
放rsa key
這定專案的screct name screct value

授權需要CICD的專案 並 建立screte screte的name&value 分別填上 Key3 , your.droneapp.com(這台主機的 private key)

drone 會讀取CICD專案上的 .drone.yml 在被授權CICD的專案上 只要有push的動作 drone會先幫該專案做測試 然後用 ssh-deploy套件連到 10.140.0.2 , 10.140.0.5 其中 username = your.droneapp.com(這台主機的 public key 的名稱) key = Key3 target = your.droneapp.com(這台主機的 private key)

pipeline:
  build:
    image: node:8.11.3-alpine
    commands:
      - npm install
      - npm test
  ssh-deploy:
    image: appleboy/drone-ssh
    pull: true  # always pull the latest version of the `drone-ssh` plugin        
    host: 
      - 10.140.0.2
      - 10.140.0.5
    username: lin
    key:
      from_secret: Inward-Jump-Mechanism-Destructive-7
    target: /root/.ssh/id_rsa
    script:
      - sudo ls /var  # or whereever you put your `deploy.sh`       
    when:
      event: [push, tag, deployment]
  # slack:
  #   image: plugins/slack
  #   webhook: https://hooks.slack.com/services/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
  #   channel: drone
  #   template: > 
  #     {{#success build.status}}
  #       build {{build.number}} succeeded. Good job.
  #     {{else}}
  #       build {{build.number}} failed. Fix me please.
  #     {{/success}}