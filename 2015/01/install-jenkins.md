# Install Jenkins #


## 安装Jenkins
```
JENKINS_HOME
--logfile=$LOG_PATH/jenkins_`date +"%Y%m-%d_%H-%M"`.log

update-rc.d jenkins defaults
```

## 配置ActiveDirectory认证

配置ActiveDirectory认证后，用户登录非常慢，最后报错如下：
```
javax.naming.TimeLimitExceededException: [LDAP: error code 3 - Timelimit Exceeded]; remaining name ...
```
在网上搜索后确定是因为用户和群组太多导致，调整“Group Membership Lookup Strategy”的配置为“Recursive group queries”，问题解决。

这个版本的ActiveDirectory是通过登录的用户和密码连接LDAP服务的，不需要单独设置连接LDAP服务的用户和密码。

## 按钮被遮盖的问题

使用的Jenkins 1.596版本遇到一个提交按钮被遮盖的问题，自己参照修改没有解决，最后从jenkins的持续集成网站下了最新的构建包部署解决。

https://ci.jenkins-ci.org/job/jenkins_main_trunk/

## 配置apache proxy

参考：

- https://wiki.jenkins-ci.org/display/JENKINS/Running+Jenkins+behind+Apache
- http://wiki.uniformserver.com/index.php/Reverse_Proxy_Server_2:_mod_proxy_html_2
- http://www.apachetutor.org/admin/reverseproxies

```
a2enmod proxy
a2enmod proxy_http
```

配置是需要注意``libapache2-mod-proxy-html``的版本，某些配置名称有变动。

基本上用下面的配置就基本可行，jenkins会报proxy配置有问题，但解决不了，不影响使用，可以忽略。

```
    ProxyPass         /jenkins  http://localhost:18080/jenkins nocanon
    ProxyPassReverse  /jenkins  http://localhost:18080/jenkins
    ProxyRequests     Off
    AllowEncodedSlashes On

    <Proxy http://localhost:18080/jenkins*>
        Order deny,allow
        Allow from all
    </Proxy>
```

验证proxy用的命令：
```
curl -iL -e http://10.163.233.74/jenkins/manage http://10.163.233.74/jenkins/administrativeMonitor/hudson.diagnosis.ReverseProxySetupMonitor/test
```
