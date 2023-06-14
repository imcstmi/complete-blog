# complete-blog
快速部署一个完整的博客网站。

基于 Docker Compose ，封装了一个完整的博客网站至少需要的组件：反向代理（Nginx Proxy  Manager）、博客（Typecho）、数据库（MySQL）、监控面板（Ward）。

![architecture](/img/architecture.png "architecture")

## 准备

- 服务器
    - CPU ≥ 1 Core
    - 内存 ≥ 1 GB
    - 操作系统：理论上不限操作系统，本文以Ubuntu 20.04为例
    - 运行环境：[Docker](https://docs.docker.com/engine/install/ubuntu/)
    - 公网IP * 1
- 域名
    - 注册商：以 [Cloudflare](https://www.cloudflare.com/) 为例
    - 可用的域名： 以 `test.com` 为例

## 安装

1. 下载

    ```
    cd ~
    git clone https://github.com/imcstmi/complete-blog.git
    ```

2. 修改数据库密码

    修改 `docker-compose.yml` 文件中的 *MYSQL_ROOT_PASSWORD* 和 *MYSQL_PASSWORD* 的值为自己的密码并保存。

    ![change mysql password](/img/change-mysql-password.png "change mysql password")

3. 运行

    ```
    cd ~/complete-blog
    docker compose up -d
    ```

## 配置

### 域名解析

1. 在域名设置页面，进入 **DNS** > **Records**。
2. 选择 **Add record**。
3. 在 *Name* 中输入"*"，在 *IPv4 address* 中输入服务器的公网IP。
    > 可勾选 **Proxy status** 开启 CDN 加速，同时在 Cloudflare 中设置域名的 *SSL/TLS encryption* 模式为 *Full (strict)*。

    ![add dns record](/img/add-dns-record.png "add dns record")

4. 选择 **Save** 保存设置。

### 反向代理

#### 初始设置

> 确保服务器防火墙已经放行80、81和443端口。

1. 在浏览器地址栏中输入 [http://公网IP:81](http://公网IP:81) 访问 Ngingx Proxy Manager 的后台登录界面。
2. 使用默认邮箱和密码登录：

    ```
    Email address:    admin@example.com
    Password:         changeme
    ```

3. 登录成功后，会强制要求更改邮箱、密码和用户信息。
4. 修改完成后，成功进入 Ngingx Proxy Manager 的后台管理界面。

    ![npm init](/img/npm-init.png "npm init")

#### 申请泛域名SSL证书

> 申请一个 *.test.com 证书，这样所有二级域名都可以使用这个证书，不需要每个二级域名申请一个证书。

1. 在 Nginx Proxy Manager 后台管理界面，进入 **SSL Certificates** > **Add SSL Certificate**。
2. 在 *Domain Names* 中输入 `*.test.com` ，在 *Email Address for Let's Encrypt* 中输入你的邮箱，勾选 **Use a DNS Challenge***， 在 *DNS Provider* 中选择 `Cloudflare` ，在 *Credentials File Content* 中将 `dns_cloudflare_api_token = ` 后的字符串替换为你的 Cloudflare API token （ [如何获取？](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)），勾选 **I Agree to ... Service** 。
    
    ![add certificate](/img/add-ssl-certificate.png "add certificate")

3. 选择 **Save** 保存设置。

#### 代理 Nginx Proxy Manager

1. 在 Nginx Proxy Manager 后台管理界面，进入 **Dashboard** > **Proxy Hosts** > **Add Proxy Host**。
2. 选择 **Details** ，在 *Domain Names* 中输入 `npm.test.com` ，在 *Forward Hostname* 中输入 `localhost` ，在 *Forward Port* 中输入 `81` 。

    ![add proxy host details](/img/add-proxy-host-details.png "add proxy host details")

3. 选择 **SSL** ，在 *SSL Certificate* 中选择 `*.test.com` 。

    ![add proxy host ssl](/img/add-proxy-host-ssl.png "add proxy host ssl")

4. 选择 **Save** 保存设置。
5. 在浏览器地址栏中输入 [https://npm.test.com](https://npm.test.com) 即可访问 Ngingx Proxy Manager 的后台登录界面。
    > 此时可以在服务器防火墙上禁用81端口

#### 代理 Ward

方法同上，步骤2在 *Domain Names* 中输入 `dash.test.com` ，在 *Forward Hostname* 中输入 `dash` ，在 *Forward Port* 中输入 `4000`。

#### 代理 Typecho

方法同上，步骤2在 *Domain Names* 中输入 `blog.test.com` ，在 *Forward Hostname* 中输入 `blog` ，在 *Forward Port* 中输入 `80`。

### Typecho 设置

1. 在浏览器地址栏中输入 [https://blog.test.com](https://blog.test.com) 访问 typecho 安装界面，按照界面引导进行安装。
2. **初始化配置** 阶段，在 *数据库地址* 中输入 `db` ，在 *数据库用户名* 中输入 `complete-blog` ，在 *数据库密码* 中输入 `docker-compose.yml` 文件中 *MYSQL_PASSWORD* 设置的数据库密码，在 *数据库名* 中输入 `complete-blog` 。

    ![typecho init config](/img/typecho-init-config.png "typecho init config")

3. **创建您的管理员账号** 阶段，在 *网站地址* 中输入 `https://blog.test.com` ，在 *用户名* 中输入博客用户名 ，在 *登录密码* 中输入博客登录密码，在 *邮件地址* 中输入你的邮箱。

    ![typecho create root account](/img/typecho-create-root-account.png "typecho create root account")

4. 选择 **继续安装** ，即可安装成功，开启你的博客之旅吧！

    ![typecho install success](/img/typecho-install-success.png "typecho install success")

## 常见问题

- Typecho 无法上传图片
    ```
    cd ~/complete-blog/blog_db/uploads
    sudo chmod a+w uploads
    ```
