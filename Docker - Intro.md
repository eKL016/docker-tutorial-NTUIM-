# Docker - Intro
# Docker - CLI

## 結構 
```shell=
    docker [選項] 指令 [參數1] [參數2] ...
    #如果使用者不在docker群組裡的話會需要superuser
```
## Docker
```shell=
    選項:
          --config [路徑]                    載入已經準備好的設定檔 (預設在 "~/.docker")
      -D, --debug                             啟動debug模式
          --help                                    列出所有指令
      -H, --host [目標主機(s)]       將這個指令發送到遠端docker deamon      
      -l, --log-level string              改變log的層級("debug"|"info"|"warn"|"error"|"fatal")
                               (預設為"info")
      -v, --version                             列出docker版本號
```
## 範例1 -  如何創建一個Ubuntu Container?
以下透過在本機上建立一個Ubuntu Container作為示範，來展現如何透過docker快速佈署已經打包好的服務：
1. 首先透過`docker search  ubuntu`來搜尋公用Registry裡的docker image
![](https://imgur.com/Wn3rUdY.png)
2. 接著我們直接從公用Registry中把第一個image透過`docker pull  ubuntu:latest`給載下來(如果沒有在後面用":"指定版本號的話會預設抓取最新版)
![](https://imgur.com/IASNFy4.png)
3. 再來用`docker images`列出所有已經存在本機上的image，在這裡可以看到image的詳細資訊，其中最重要的就是他的ID，後面對Image做操作都會透過這裡
![](https://imgur.com/DHTBFYv.png)
4. 最後我們就可以透過`docker run -it ubuntu:latest bash`創立一個掛載在此image上的container layer並直接進入Bash CLI
![](https://imgur.com/yWwATgm.png)

## Docker - 一些常用的管理指令
### docker ps
```shell=
Usage:	docker ps [選項]

列出所有運行中的Container

```
### docker logs
```shell=
Usage:	docker logs  [選項] Container名

顯示某個Container的Log
```
### docker inspect 
```shell=
Usage:	docker inspect [選項] NAME/ID

列出Image或是Container的詳細資料
```
### docker port
```shell=
Usage:	docker port Container名 [Container的port[/協定]]

列出某個Container（或他特定port）和外界的port對應
```
### docker kill
```shell=
Usage:	docker kill [選項] CONTAINER名 [CONTAINER名...]

停止一個或以上的Container
```
### docker attach
```shell=
Usage:	docker attach [選項] CONTAINER名

把你的STDIN、STDOUT、STDERR和某個container串接起來 ( 注意如果要離開而不想關閉container要detach )
```
## 範例二 - 如何直接Build出一個docker Image
有時候我們會有需要把自己的系統服務直接包成image，這樣下次就可以自動重新佈署，
抑或是有些時候載下來的docker服務並沒有包成image的時候，我們就必須透過（撰寫）dockerfile將其打包
接下來我將透過打包一個包含nodejs的ubuntu來示範 (Modified from: [kstaken@gihub](https://github.com/kstaken/dockerfile-examples/blob/master/nodejs/Dockerfile))：

1.  cd到含`Dockerfile`的目錄之後執行`docker build .`，便會在當前目錄下開始按照Dockerfile打包
```shell=
docker build .
Sending build context to Docker daemon  3.584kB
Step 1/8 : FROM ubuntu:15.04
 ---> d1b55fd07600
Step 2/8 : RUN ln -s /usr/bin/node /usr/local/bin/node
 ---> Running in d3f3c005496d
 ---> 3f8d85d73950
Removing intermediate container d3f3c005496d
Step 3/8 : RUN curl -fsSL https://test.docker.com | sh
 ---> Running in 6d3f73096d28
/bin/sh: 1: curl: not found
 ---> dc3cba67483a
Removing intermediate container 6d3f73096d28
Step 4/8 : RUN apt-get update
 ---> Running in 3e847d0ce3d7

......
Reading package lists...
 ---> 4766f9129031
Removing intermediate container 3e847d0ce3d7
Step 5/8 : RUN apt-get install -y nodejs

......
 ---> Running in 50f41d7ff934
Step 6/8 : RUN mkdir /var/www
 ---> Running in 65133ce62d58
 ---> f565edeeeeb9
Removing intermediate container 65133ce62d58
Step 7/8 : ADD app.js /var/www/app.js
 ---> 9252097aa46d
Removing intermediate container f3f31a909419
Step 8/8 : CMD /usr/bin/nodejs /var/www/app.js
 ---> Running in fe60c7b3eb81
 ---> 2ab1333319b9
Removing intermediate container fe60c7b3eb81
Successfully built 2ab1333319b9 #產生了hash為2ab13....的image

```
2. 接者就可以使用`docker run -it  2ab`直接開啟一個nodejs的docker container了！在docker路由設定好的狀況下應該可以從外部連上它
```shell=
#docker run -it 2ab
Server running at http://127.0.0.1:8080/
```
## Docker - Volume相關
一般VM Host中的共享資料夾在Docker的世界裡稱為Volume，以下是一些常見的Volume操作
### docker volume create
```shell=
Usage:	docker volume create [選項] [名字]

在本機上建立一個Volume
```
### docker volume ls
```shell=
Usage:	docker volume ls [選項]

顯示所有已建立的Volume
```

### docker volume rm
```shell=
Usage:	docker volume rm [選項] VOLUME [VOLUME...]

移除一或多個Volume
```
### docker run -v 某個Volume:/欲掛載的路徑
      就可以掛載Container到Image上時直接掛載Volume
## Docker - 快照
一般在VM Host中會有個功能可以儲存目前機器的State，有需要時再還原 － Docker當然也有這個功能，不過有兩種儲存機器狀態的方式，分別是儲存Container和Image
### docker export
```shell=
Usage:	docker export [選項] CONTAINER 

匯出完整Image+Container Layer到STDOUT，如果要儲存到某個檔案之中就得使用Pipeline(>)或是加上-o 選項（這種方式與我們一般認知上的快照比較類似）
```
### docker save
```shell=
Usage:	docker save [選項] IMAGE [IMAGE...]

純粹只是把當下版本Image變成壓縮檔而已，一樣導到STDOUT（這個並不會儲存你的State，注意）
```
### docker import
```shell=
Usage:	docker import [選項] 檔案|網址| - [來源庫[:版本]]

從檔案、網址或來源庫讀入Image和Container(相當於快照)並還原它
```
### docker load
```shell=
Usage:	docker load [選項]


從STDIN或檔案讀入Image還原它
```
# Dockerfile
## 用途
當你今天需要以別人Image為基底建造一個經過你定制的Docker container，或是想要讓自己的Image在佈署時有些自動化的功能時，這種時候我們就可以透過撰寫Dockerfile來達到此目的。
## 範例一 - IMCamp 2017_IMfinity
```dockerfile=
ARG MONGODB_URL
FROM google/nodejs

LABEL maintainer="ekl@ntu.im"

ADD . /var/www/
WORKDIR /var/www/
RUN npm install --silent

ENV MONGODB_URL=${MONGODB_URL}


CMD []
ENTRYPOINT ["node", "app.js"]
```
首先我們以資管系在2017年的宿營網站[](https://github.com/eKL016/imfinity_production)作為例子來示範Dockerfile此處能派上什麼用場，這一年的資管營網站是一個簡單的Express.js網站，最終佈署時是用Docker在系學會主機上達成佈署，接下來將以這個專案的Dockerfile作為範例帶大家對Dockerfile機制能有更多瞭解。
### ARG MONGODB_URL
```dockerfile=
ARG <參數名> [=<預設值>]
```
首先這行指令是整個Dockerfile唯一可以放在`FROM`前的指令，作用是讓執行`docker build`的時候可以代入參數進入dockerfile，在此處是讓晚點執行express應用程式時連接上指定的Mongodb。

***注意！*** 這個帶入參數作用的期間僅存在build期間，一旦建構完就會消滅，同時他也不適合用來帶入superuser的帳號密碼，因為ARG會被保存在紀錄檔中。
### FROM google/nodejs
```dockerfile=
FROM <image>[:<標籤>] [AS 別名>]
```
**整個Dockerfile最重要的指令**，指定這個docker image要基於哪個已知的docker image去做操作，以這次範例來舉例，我們從google的docker repository中把nodejs的基本image給pull下來操作。

***注意！*** 這個參數一定要放在dockerfile的最前面（ARG除外，這讓你有辦法讓使用者在build時有可能可以指定base image），不然build運行會出錯，因為docker不知道你要根據哪個image去操作。
### LABEL  maintainer="ekl/@ntu.im"
```dockerfile=
LABEL <鍵>=<值> <鍵>=<值> <鍵>=<值> ...
```
附加metadata到你的docker image上面，例如這邊所列的"maintainer"（這取代了以前的`MAINTAINER <name>`指令）
### ADD . /var/www/
```dockerfile=
ADD <來源>... <目標>
```
把來源位置的檔案複製到image之中，在這個dockerfile中就是把整個git repository複製到image中的`/var/www/`目錄之中。

***注意！*** 來源如果是一個目錄位置則該目錄不會被複製，而是底下之所有目錄與檔案，另外如果來源也可以是個網路上的位置甚至是個tar檔，docker也會自動處理。
### WORKDIR /var/www/
```dockerfile=
WORKDIR /切換/目標/目錄
```
把當前dockerfile操作的工作目錄切換到別處，和shell script中的cd雷同。
***注意！*** `RUN cd /var/www`並沒有辦法取代`WORKDIR /var/www/`，因為在dockerfile中每一個指令運行的container不同，用`RUN cd /var/www`只在該行有效果而已。
### RUN npm install --silent
```dockerfile=
RUN <指令>
```
在現在階段的container上再開一層，並透過Shell(預設是bash)執行指定CLI指令，在這個dockerfile中是把需要的node module給安裝起來。
### ENV MONGODB_URL=${MONGODB_URL}
```dockerfile=
ENV <鍵>=<值> ...
```
把此行以下的執行環境變數指定為特定的值，在這邊的用途是把當初用ARG傳進來的mongodb位置設定在環境變數中給nodejs程式讀取。
### CMD []
```dockerfile=
CMD 指令 參數1 參數2 //如果沒有指定ENTRYPOINT時，視同執行shell指令
or
CMD ["參數1","參數2"] //如果有指定ENTRYPOINT時，視同送入參數給ENTRYPOINT
```
指定build完成後的image被啟動時要往`ENTRYPOINT`送入的參數，沒有特別指定`ENTRYPOINT`時預設是往Bash送，但是在這邊的例子我們發現`ENTRYPOINT`被指定為nodejs程式本身，故我們不用送入任何參數維持空白。
### ENTRYPOINT ["node", "app.js"]
```dockerfile=
ENTRYPOINT ["可執行檔", "參數1", "參數2"] //使用情境不是執行shell指令
or
ENTRYPOINT 指令 參數1 參數2 //使用情境是執行shell指令
```
指定build完成之後的image被啟動時要執行的可執行檔/指令，預設是bash，在這裡指定為`node app.js`。
***注意！*** 一般來說不會特別在執行image時改變ENTRYPOINT的值（雖然可以），這是為了讓image原作者可以維持他想要image做的事，例如一個Database就不該有interactive功能，然而CMD卻是設計上來讓使用者可以自行變更的。
## 其他重要Dockerfile指令
### EXPOSE
```dockerfile=
EXPOSE [<連接埠>/<TCP/UDP>...]
```
這是一個聲明性的指令，聲明這個image運行container時要開啟哪些連接埠以供使用，實際上的host對container的port mapping還是要手動開啟。***不過如果啟動container時使用`-P`參數開放隨機對應時有被`EXPOSE`聲明的埠才會被映射。***
### COPY
```dockerfile=
COPY <來源>... <目標>
```
和ADD基本上相同的指令，但是不支援從網路上下載檔案或是自動解壓縮，dockerfile的編寫習慣上建議單純的複製就用`COPY`指令就好，不然`ADD`會發生的行為一般來講沒有辦法一眼就看透。
### VOLUME
```dockerfile=
VOLUME ["<路徑1>", "<路徑2>"...]
```
在運行Container時指定哪個目錄會被寫到自動創建的匿名Volume中，這是為了讓container維持stateless的手段，不過換句話說，沒有被掛載成Volume的目錄在Container終結的時候改動 ***不會*** 被儲存下來，所以在打包會有狀態儲存的image時要特別注意。
### USER
```dockerfile=
USER <使用者>[:<群組>] 
or
USER <UID>[:<GID>]
```
指定接下來執行的所有`RUN`、`CMD`和`ENTRYPOINT`指令會以什麼使用者身份執行。
***注意！*** 如果該使用者沒有primary group那接下來的指令會以`root`執行。
# Docker-Compose
## 概述
前面的部份我們提到了dockerfile的使用，讓我們可以將單一個容器的佈署給自動化，但是當我們要佈署一個真正的專案時，往往需要同時運行許多的服務（例如Dockerfile範例一的Express應用程式仍需要配合一個MongoDB實體，可以的話甚至還得配合一個nginx實體作為Reverse Proxy使用），因此這個時候我們就需要使用Docker Compose來達到同時佈署多個服務的效果。
## 範例一 - IMCamp 2017_IMfinity_FullProject

...待補

# Docker與Docker-Compose的網路機制
Docker Container的網路連線主要都是環繞著Linux系統的veth機制與網路命名空間分割來建立，以下將簡單整理Docker的網路運作原理與其相關設定。
## Docker網路概述
### Host
一般在我們啟動Docker服務時，Docker服務會在Host OS中建立一個虛擬的`docker0`網路介面並橋接到真實的網路介面，這個介面會被指派一個Private IP並設定好netmask以指定自己的子網路，以後創建Container的時候預設他們都會在該子網路下。
### Container
![](https://philipzheng.gitbooks.io/docker_practice/content/_images/network.png)
Ref:《Docker —— 從入門到實踐 正體中文版》

在Docker服務建立好`docker0` interface之後，便可以來進行Container方的網路設定了。

當我們建立一個Container時Host OS會產生一對veth pair，一端接上`docker0`一端接上一個和Host OS的網路介面中處於不同命名空間的新介面，在完成這操作之後Container中就會可見一個新的eth0介面，而從這個虛擬的eth0中所發送、接收的封包便會直接透過複製和`docker0`橋接器做交換，在邏輯上這個Container就等同於處於`docker0`的網域中且透過`docker0`作為Gateway，如此就可以和其他Container傳輸資訊與和外部連線，***而且Host OS和Container的兩個網路介面在邏輯上是完全無關的***。
## 基本參數設定
### 啟動Docker服務時
#### -b, --bridge 
```shell=
-b, --bridge string        Attach containers to a network bridge
```
把這個docker服務所產生的container自動接上指定的bridge網路，預設是docker0。
#### --bip
```shell=
--bip string               Specify network bridge IP
```
指定橋接器所要使用的IP含遮罩，以後Container自動加入的網段就會在這之中。
#### --icc
```shell=
--icc=true|false           Enable inter-container communication (default true)
```
是否准許Container間的通訊，預設是True
#### --ip-forward
```shell=
--ip-forward=true|false    Enable net.ipv4.ip_forward (default true)
```
剛才有提到Container eth0的Gateway都會被設定成為docker服務的橋接器，但是如果這個預設是true的選項被設定成False的時候就不能外連了。
#### --iptables
```shell=
--iptables=true|false      Enable addition of iptables rules (default true)
```
此選項是予不予許Docker修改iptables，預設是開啟的，這樣container才能在什麼事都沒設定的狀況下就能和外界聯絡而不會被Host OS給擋下。
### 啟動Docker Container時
#### -h, --hostname
```shell=
-h, --hostname string      Container host name
```
在啟動個別的Container時，設定Container所要使用的Hostname。
#### --link
```shell=
--link list                Add link to another container
```
讓被啟動的這個Container可以透過Hostname直連到所指定的單/多個Container。
#### --network 
```shell=
--network string           Connect a container to a network
```
指定該Container的連線方式，通常有`bridge|none|container:NAME_or_ID|host`這幾種選擇，分別是預設的橋接網路、不連線、直接連線某個Container和直接使用宿主網路。
#### -p, --publish
```shell=
-p, --publish list         Publish a container's port(s) to the host
```
指定要在Container上開啟哪些Port給外界透過Host OS連線進入。
#### -P, --publish-all
```shell=
-P, --publish-all          Publish all exposed ports to random ports
```
把當時在Build Container時透過`EXPOSE`指定的PORT給隨機映射到Host OS上讓外部連入。
### 啟動Docker / Docker Container時
#### --dns
```shell=
--dns list                 Set custom DNS servers
```
指定Container要使用的DNS Server，如果是在執行Docker服務時輸入此選項則為指定預設值。
#### --dns-search
```shell=
--dns-search list          Set custom DNS search domains
```
指定Container要使用的DNS搜索域，如果是在執行Docker服務時輸入此選項則為指定預設值。
## Docker-Compose中的網路設定
預設Docker-Compose執行`docker-compose build`時會預設將Project底下的所有Container自動加入到一個和Project同名的網路之中，且預設所有機器都可以被直接用Container name當作hostname連線並發現。
***注意！*** 每次修改Container後重新執行`docker-compose up`時，舊的Container會被關閉並斷開所有連線，而新的Container會用一個不一樣的IP重新連接上網路，而Hostname會維持不變。
### Links
雖然在預設的情境下不需要特別指定哪些Containers可以互相連線，但是在某些情況下我們會需要特別標明出他們之間的連結/依賴關係，這種時候我們就需要在Compose file中特別指定。
```dockerfile=
version: "3"
services:
  
  web:
    build: .
    links:
      - "db:database"
  db:
    image: postgres
```
以Docker官方document中的這個例子來看，特別指定了web會連線到db且為它指派一個別名為database，這樣明確指定的好處除了在非預設全連通的網路中能互連之外，也同時賦予了兩Container間一個依賴關係，以此Project為例，`web`這個container的啟動順序便會在`db`啟動之後。

***注意！*** 用`links`創立的依賴關係`depends_on`一樣，是指定**啟動順序** - 並不保證後啟動的container會等到前者**Ready**才啟動。
### 手動管理網路
在Compose file中我們可以使用最上級的`networks`和`services`層級的`networks`指令來細部指定我們Project中的網路組態。
```dockerfile=
version: "3"
services:
  
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```
一樣以Docker官方document的例子來看，在最上級的`networks`標籤中，我們分別定義了`frontend`和`backend`兩組docker網路，並且指定了他們要使用的外部驅動插件。
接著我們往`services`中的`networks`看下去，我們可以發現，在這個例子中透過了指定container所屬docker網路的關係，我們得以將這個project的內外網給切開來，隔離了database。

***注意！*** 你可以透過在最上級`networks`指定`default`網路的相關設定來設定預設網路，而如果你要將此project下的container加入一個已經存在的外部docker網路的話，可以在任一網路的下級加入`external:`參數並且指定再下一級的`name:`參數為該網路的名稱即可。

### Network - Service級
#### Aliases
為Container在同一網路下取其他別名(不同Hostname)，作用範圍僅限於該docker網路內。
```dockerfile=
version: '2'

services:
  web:
    build: ./web
    networks:
      - new

  worker:
    build: ./worker
    networks:
      - legacy

  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```
以上的docker官方範例將`db`這個service在兩個網路`new`和`lagacy`中分別配了`database`和`mysql`兩個別名。

#### ipv4_address, ipv6_address
為container指定在網路中要使用的ip位址
```dockerfile=
version: '2.1'

services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
      -
        subnet: 2001:3984:3989::/64
```
以上的docker官方範例將`app`這個service指定了`172.16.238.10`和`2001:3984:3989::10`這兩個ip位置。

***注意！*** 使用這兩個參數時，需要在最上級的`networks`參數內分別開啟與設定ipam（對應ipv4）或enable_ipv6（對應ipv6）。

#### ports
指定container與host的port映射關係。
```dockerfile=
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```
以上的docker官方範例將該container`80 port`上的`tcp`傳輸映射到了`host`的`8080 port`上，並且指定它為host模式（映射的模式與docker swarm有關，有興趣的人可以自行參考）。