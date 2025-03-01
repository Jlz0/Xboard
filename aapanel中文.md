# Xboard部署指南Aapanel环境

## 目录
1. [要求](#要求)
2. [快速部署](#快速部署)
3. [详细的配置](#详细的配置)
4. [维护指南](#维护指南)
5. [故障排除](#故障排除)

## 要求

### 硬件要求
- CPU：1核或更高
- 内存：2GB或以上
- 存储：10GB+可用空间

### 软件要求
- 操作系统：Ubuntu 20.04+ / Debian 10+（不建议使用⚠️Centos7）
- Aapanel的最新版本
- PHP 8.2
- MySQL 5.7+
- Redis
- Nginx (任何版本)

## 快速部署

### 1. 安装Aapanel
```bash
URL=https://www.aapanel.com/script/install_6.0_en.sh && \
if [ -f /usr/bin/curl ];then curl -ksSO "$URL" ;else wget --no-check-certificate -O install_6.0_en.sh "$URL";fi && \
bash install_6.0_en.sh aapanel
```

### 2. 基本环境设置

#### 2.1 安装LNMP环境
在Aapanel仪表板中，安装：
-Nginx（任何版本）
-MySQL 5.7
-PHP 8.2

#### 2.2 安装PHP扩展
> aaPanel 面板 > App Store > 找到PHP 8.2点击Setting > Install extentions选择以下扩展进行安装
所需的PHP扩展：
- redis
- fileinfo
- swoole
- readline
- event

#### 2.3 解除被禁止函数
> aaPanel 面板 > App Store > 找到PHP 8.2点击Setting > Disabled functions 将以下函数从列表中删除
需要启用的功能：
- putenv
- proc_open
- pcntl_alarm
- pcntl_signal

### 3. 站点配置

#### 3.1 创建网站
>aaPanel 面板 > Website > Add site。  
>>在 Domain 填入你指向服务器的域名  
>>在 Database 选择MySQL  
>>在 PHP Verison 选择PHP-81 

#### 3.2 部署Xboard
>通过SSH登录到服务器后访问站点路径如：/www/wwwroot/你的站点目录。
>以下命令都需要在站点目录进行执行。

##### 输入站点目录
```bash
cd /www/wwwroot/你的站点目录
```
##### 删除目录下文件
```bash
chattr -i .user.ini
rm -rf .htaccess 404.html 502.html index.html .user.ini
```
##### 执行命令从 Github 克隆到当前目录
```bash
git clone https://github.com/cedar2025/Xboard.git ./
```
##### 执行命令安装依赖包以及xboard
```bash
sh init.sh
```

#### 3.3 配置站点目录及伪静态
1. > 添加完成后编辑添加的站点 > Site directory > Running directory 选择 /public 保存。  
2. 添加完成后编辑添加的站点 > URL rewrite 填入伪静态信息。
```nginx
location /downloads {
}

location / {  
    try_files $uri $uri/ /index.php$is_args$query_string;  
}

location ~ .*\.(js|css)?$
{
    expires      1h;
    error_log off;
    access_log /dev/null; 
}
```

## 详细配置

### 1. 配置守护进程
>Xboard的系统强依赖队列服务，正常使用XBoard必须启动队列服务。下面以aaPanel中supervisor服务来守护队列服务作为演示。
1。安装 > aaPanel 面板 > App Store > Tools 找到Supervisor进行安装，安装完成后点击设置 > Add Daemon按照如下填写。
2。添加队列守护程序过程：
   - 在 Name 填写: `Xboard`
   - 在 Run User 选择: `www`
   - 在 Run Dir 选择: 站点目录
   - 在 Start Command 填写: `php artisan horizon`
   - 在 Processes 填写: 1
   - >填写后点击Confirm添加即可运行。

### 2. 配置定时任务
>aaPanel 面板 > Cron
- 在 Type of Task 选择: Shell Script
- 在 Name of Task 填写: Xboard
- 在 Run User 选择: www
- 在 Period 选择:N Minutes 填写 1 分钟
- 在 Script content 填写: `php /www/wwwroot/你的站点目录/artisan schedule:run`

### 3. 开启webman
> 在上述安装的基础上开启webman提高性能
#### 3.1 添加守护进程
>下面以aaPanel中supervisor服务来守护队列服务作为演示。
- 1.>aaPanel 面板 > App Store > Tools
- 2.找到Supervisor进行安装，安装完成后点击设置 > Add Daemon按照如下填写
- 在 Name 填写: webman
- 在 Run User 选择: www
- 在 Run Dir 选择: 站点目录
- 在 Start Command 填写: `/www/server/php/82/bin/php artisan octane:start --port 7010`
- 在 Processes 填写: 1
- >填写后点击Confirm添加即可运行。

#### 3.2 修改伪静态
> 站点设置 > URL Rewrite(伪静态) 填入一下内容<span style="color:red">(覆盖前伪静态配置)</span>
```nginx
location ~* \.(jpg|jpeg|png|gif|js|css|svg|woff2|woff|ttf|eot|wasm|json|ico)$ {
}

location ~ .* {
    proxy_pass http://127.0.0.1:7010;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Real-PORT $remote_port;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header Scheme $scheme;
    proxy_set_header Server-Protocol $server_protocol;
    proxy_set_header Server-Name $server_name;
    proxy_set_header Server-Addr $server_addr;
    proxy_set_header Server-Port $server_port;
}
```
> 在此你的webman已经成功部署了

## 维护指南

### 版本更新

#### 输入站点目录
```bash
cd /www/wwwroot/你的站点目录
```
#### 执行更新脚本
```bash
git fetch --all && git reset --hard origin/master && git pull origin master
sh update.sh
```
#### 如果启用了 webman ，请重新启动守护程序
```bash
aaPanel > App Store > Tools > Supervisor > Restart Octane
```

### 例行维护
- 常规日志检查
- 监视系统资源使用情况
- 数据库和配置文件的常规备份

## 故障排除

### 常见问题
1. 管理路径的更改要求重新启动服务才能生效。
2. 启用 webman 后任何代码更改都需要重新启动才能生效。
3. 当PHP扩展安装失败时，检查PHP版本是否正确。
4. 对于数据库连接故障，请检查数据库配置和权限。
