＃X板部署指南Aapanel环境

＃＃ 目录
1。[要求]（＃要求）
2。[快速部署]（＃快速部署）
3。[详细配置]（＃详细构造）
4。[维护指南]（＃维护指南）
5。[故障排除]（＃故障排除）

＃＃ 要求

###硬件要求
-CPU：1核或更高
- 内存：2GB或以上
- 存储：10GB+可用空间

###软件要求
- 操作系统：Ubuntu 20.04+ / Debian 10+（不建议使用⚠️Centos7）
-  Aapanel的最新版本
-PHP 8.2
-MySQL 5.7+
-  redis
-Nginx（任何版本）

##快速部署

### 1。安装Aapanel
``bash
url = https：//www.aapanel.com/script/install_6.0_en.sh && \ \
如果[-f/usr/bin/curl];然后curl -ksso“ $ url”; else wget -no -check -certificate -o install_6.0_en.sh“ $ url” $ url“; fi && \
bash install_6.0_en.sh aapanel
````````

### 2。基本环境设置

#### 2.1安装LNMP环境
在Aapanel仪表板中，安装：
-Nginx（任何版本）
-MySQL 5.7
-PHP 8.2

#### 2.2安装PHP扩展
所需的PHP扩展：
-  redis
-  FileInfo
- 吞咽
- 读取线
- 事件

#### 2.3启用所需的PHP功能
需要启用的功能：
-  Putenv
-  proc_open
-cntl_alarm
-cntl_signal

### 3。站点配置

#### 3.1创建网站
1。导航到：Aapanel>网站>添加网站
2。填写信息：
   - 域：输入您的站点域
   - 数据库：选择MySQL
   -PHP版本：选择8.2

#### 3.2部署Xboard
``bash
＃输入站点目录
CD/www/wwwroot/your tomain

＃清洁目录
chattr -i .user.ini
rm -rf .htaccess 404.html 502.html index.html .user.ini

＃克隆存储库
git克隆https://github.com/cedar2025/xboard.git ./

＃安装依赖项
sh init.sh
````````

#### 3.3配置网站
1。将运行目录设置为“/public”
2。添加重写规则：
````nginx
位置 /下载{
}

地点 / {  
    try_files $ uri $ uri / / index.php$iis_args$ query_string;  
}

位置〜。*\。（JS | CSS）？$
{
    到期1H；
    error_log off;
    access_log /dev /null;
}
````````

##详细配置

### 1。配置守护程序过程
1。安装主管
2。添加队列守护程序过程：
   - 名称：`Xboard`
   - 运行用户：`www`
   - 运行目录：站点目录
   - 开始命令：`php工匠地平线
   - 过程计数：1

### 2。配置计划的任务
- 类型：Shell脚本
- 任务名称：V2板
- 运行用户：www
- 频率：1分钟
- 脚本内容：`php/www/wwwroot/site-directory/artisan时间表：运行“

### 3。辛烷值配置（可选）
#### 3.1添加辛烷守护程序
- 名称：辛烷值
- 运行用户：www
- 运行目录：站点目录
- 启动命令：`/www/server/php/82/bin/php工匠octane：start -port 7010`
- 过程计数：1

#### 3.2特定于辛烷值的重写规则
````nginx
位置〜** \。
}

位置〜。* {
    proxy_pass http://127.0.0.1:7010;
    proxy_http_version 1.1;
    proxy_set_header连接“”;
    PROXY_SET_HEADER X-REAL -IP $ remote_addr;
    PROXY_SET_HEADER X-REAL-PORT $ REMEND_PORT;
    proxy_set_header x-forwarded-for $ proxy_add_x_forwarded_for;
    proxy_set_header主机$ http_host;
    proxy_set_header方案$方案;
    PROXY_SET_HEADER SERVER-PROTOCOL $ SERVER_PROTOCOL;
    PROXY_SET_HEADER SERVER-NAME $ SERVER_NAME;
    proxy_set_header server-addr $ server_addr;
    proxy_set_header server-port $ server_port;
}
````````

##维护指南

###版本更新
``bash
＃输入站点目录
CD/www/wwwroot/your tomain

＃执行更新脚本
git提取 -  all && git reset- hard orargion/master && git pull origin Master
sh Update.sh

＃如果启用了辛烷，请重新启动守护程序
＃Aapanel> App Store>工具>主管>重新启动辛烷值
````````

###例行维护
- 常规日志检查
- 监视系统资源使用情况
- 数据库和配置文件的常规备份

##故障排除

###常见问题
1。管理路径的更改要求重新启动服务才能生效
2。启用辛烷后任何代码更改都需要重新启动才能生效
3。当PHP扩展安装失败时，检查PHP版本是否正确
4。对于数据库连接故障，请检查数据库配置和权限