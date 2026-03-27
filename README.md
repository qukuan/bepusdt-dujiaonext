# 1Panel面板安装Bepusdt多链支付中间件

* **Bepusdt开源项目仓库地址：** `https://github.com/v03413/BEpusdt`
* **感谢Bepusdt项目作者：** `v03413`
* **本篇教程视频：** 手动下载作者编译好的releases包，通过守护进程启动

---

## 准备工作

1.  **服务器选择**
    * 一台2h2g服务器（2核2G运行内存最好，虽然1H1G也行但不推荐），SSD硬盘是最好的，具备高并发读写性能。
    * 普通 VPS 便宜 → 硬盘可能是 HDD → 读写性能差，体验感不太友好。
    * 高性能数据库或低延迟 VPS → NVMe SSD。
    * 选服务器（VPS）的时候都可以看一下配置，地区我们正常选美国或国外，香港也可以。
    * **为什么呢？** 因为我们只要是部署：和Tron区块链交互相关的程序/项目，以及Telegram机器人相关的程序和项目，如果你买国内的服务器是连接不上相对应的api的。所以这里我们选服务器地区：美国或其它国外地区，香港。
    * **服务器操作系统：** 本篇教程使用的是Ubuntu24.04系统，你也可以安装Debian12系统。

2.  **域名与证书**
    * 一个域名，解析到服务器ip，这里推荐先托管到cloudflare再登录cloudflare进行解析域名。
    * 然后在cloudflare给我们的域名生成15年的证书，把证书都复制下来，这样子我们在添加站点的时候就可以直接使用了。

3.  **安装1panel面板**
    * 连接服务器终端执行安装1Panel命令：
        ```bash
        curl -sSL [https://resource.1panel.pro/quick_start.sh](https://resource.1panel.pro/quick_start.sh) -o quick_start.sh && bash quick_start.sh
        ```
    * 安装好1Panel面板后先不要着急关闭终端，继续执行安装守护进程命令：
        ```bash
        sudo apt-get install supervisor
        ```
    * 你也可以参考1Panel官方文档：[https://1panel.cn](https://1panel.cn)

> 到这，准备工作完毕~

---

## 正片：开始安装

1.  登录1Panel面板 → 左侧菜单栏点击 → **应用商店** → 安装以下两个应用：
    * **OpenResty**
    * **PostgreSQL**
    * *注：安装的过程中注意：不要开启高级设置，不要开外部访问！！！*

2.  左侧菜单栏点击 → **网站** → **网站** → **创建** → 选择：**反向代理**：
    * 主域名我们填写我们解析的bepusdt域名（你给bepusdt程序解析什么域名就填什么）。
    * 代理地址填写：`127.0.0.1:8080` → 保存。
    * 创建好站点后我们再点击一下我们刚刚创建好的站点域名，**开启HTTPS**，选择我们cloudflare生成的证书，保存，一定要点保存！

3.  点击进入我们站点的根目录，例如我这里是：
    `/opt/1panel/apps/openresty/openresty/www/sites/bepusdt.vfaka.cn/index`

4.  点击**远程下载**，url填写Bepusdt项目相对应的Releases包链接：
    `https://github.com/v03413/BEpusdt/releases/download/v1.23.3/linux-amd64-BEpusdt.tar.gz`
    *注意：不要下载错了，我们Linux服务器ubuntu/Debian系统下载的都是linux-amd64-BEpusdt.tar.gz*

5.  解压我们下载好的 `linux-amd64-BEpusdt.tar.gz`，得到目录结构：
    ```text
    ├── bepusdt           # 可执行程序文件
    ├── bepusdt.service   # systemd 服务配置文件
    └── .env.example
    ```

6.  左侧菜单栏点击 → **数据库** → 选择 **PostgreSQL** → 点击 **创建数据库**：
    * 数据库用户跟名称都是一样的，然后设置数据库密码 → 保存。
    * 记录下来我们创建好的数据库用户/名称以及数据库密码。
    * 示例：数据库用户 `bepusdt` / 数据库名称 `bepusdt` / 数据库密码 `qq123456`

7.  点击打开 `.env.example` 填写自己的配置信息：
    * `# SQLITE=/var/lib/bepusdt/sqlite.db` 加个#注解掉。
    * `POSTGRESQL_DSN=postgres://bepusdt:qq123456@127.0.0.1:5432/bepusdt?sslmode=disable&connect_timeout=3` 删掉前面的#。
    * 保存后，将文件 `.env.example` 名称更改为 `.env`。

8.  确保 `bepusdt` 这个可执行程序文件权限为 **755**。

9.  左侧菜单栏点击 → **工具箱** → 点击：**进程守护**：
    * 第一次打开进程守护会提示我们初始化，点击初始化就可以 → **创建守护进程**。
    * **名称：** 随便填写，例如 `Bepusdt`。
    * **启动用户：** 默认 `root`。
    * **运行目录：** 也就是我们bepusdt站点根目录，例如我的：
        `/opt/1panel/apps/openresty/openresty/www/sites/bepusdt.vfaka.cn/index`
    * **启动命令：** `我们bepusdt站点根目录/bepusdt start`
        例如我的启动命令：`/opt/1panel/apps/openresty/openresty/www/sites/bepusdt.vfaka.cn/index/bepusdt start`
    * 点击：**确认**。
    * 如果显示已启动，则安装完成，点击日志抄下来登录入口，账号密码（**只会显示一次，注意保存**）。

10. **登录bepusdt后台，完成基本配置（必须）**
    * **系统管理 → 基本设置 → 交易通知：**
        * Bot Token：填写机器人token令牌。
        * Chat ID：填写你的telegram id，可以通过机器人 `@myidbot` 发送 `/getid` 查询自己的id。
        * 测试看到信息就是成功了，订单支付成功你的机器人就会通知你。
    * **系统管理 → 基本设置 → API设置：**
        * 应用URI：填你的bepusdt域名，例如我的：`https://bepusdt.vfaka.cn`
    * **系统管理 → 区块节点 → Tron 网络 → Tron Grid Api Key：**
        * 去 [https://www.trongrid.io/register](https://www.trongrid.io/register) 注册登录申请key，然后填写Tron Grid Api Key。
        * 这个正常使用邮箱注册就可以，因为我已经注册过了，Create a new API key 创建key。
        * 这个看不懂英文浏览器都有自动翻译功能。

> 这样就配置完成了，登录入口账号密码这些再自己更改，因为我也没怎么用过bepusdt，写这个教程比较长，感谢耐心观看。我们继续测试一下创建订单看看吧，这就是配置成功后的效果。

---

## 11. 我们继续完成接入新版独角吧

顺便补上前面我没有出接入usdt的视频，正好今天独角有更新了，补上更新教程：
我们登录面板或者连接服务器终端，怎么都行，反正只要我们在独角根目录下执行两条命令就能完成更新版本。

只需要执行这两条命令：
```bash
root@racknerd-c8095b4:/opt/dujiaonext# docker compose -f docker-compose.postgres.yml pull
[+] pull 12/12
 ✔ Image dujiaonext/admin:latest Pulled
 ✔ Image dujiaonext/user:latest  Pulled
 ✔ Image dujiaonext/api:latest   Pulled

root@racknerd-c8095b4:/opt/dujiaonext# docker compose --env-file .env -f docker-compose.postgres.yml up -d
[+] up 3/3
 ✔ Container dujiaonext-api    Healthy
 ✔ Container dujiaonext-user   Recreated
 ✔ Container dujiaonext-admin  Recreated

```
- 全绿就是更新成功了

### 登录我们的独角后台：
- 支付管理 → 支付渠道 → 新增渠道
- 然后看我怎么填写的

- 增加USDT支付方式：
```bash
网关地址：https://bepusdt.vfaka.cn  没有斜杠！！！注意了
API Token：在我们的bepusdt后台获取
支付币种：usdt.trc20
法币类型：CNY
异步通知地址：https://你独角的api后端域名/api/v1/payments/callback
同步回跳地址：https://你独角的api后端域名/pay
```


- 增加TRX支付方式：
```bash
网关地址：https://bepusdt.vfaka.cn  没有斜杠！！！注意了
API Token：在我们的bepusdt后台获取
支付币种：tron.trx
法币类型：CNY
异步通知地址：https://你独角的api后端域名/api/v1/payments/callback
同步回跳地址：https://你独角的api后端域名/pay
```

```bash
异步通知地址跟同步回跳地址  的后缀是固定的
异步通知地址固定后缀：/api/v1/payments/callback
同步回跳地址固定后缀：/pay
```

## 什么是api后端地址  跟用户前端地址 ？  
请看我上一个1panel部署新版独角dujiao-next的视频 https://www.youtube.com/watch?v=Mv946RQJmrY


成功，如果你在过程中遇到：支付网关请求错误  等问题，那你肯定是没有跟上我本篇视频的节奏，肯定是错误的地方，重新看仔细
到这就结束了，再见
