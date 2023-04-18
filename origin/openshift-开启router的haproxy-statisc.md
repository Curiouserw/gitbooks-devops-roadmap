1. 设置router POD 所在节点的iptables对1936端口的放行

    ```bash
    iptables -I OS_FIREWALL_ALLOW -p tcp -m tcp --dport 1936 -j ACCEPT
    ```

2. 获取访问router haproxy statics 页面的用户名密码。

    ![](../assets/OCP3-11开启router的haproxy-statisc-1.png)

3. 删除掉router dc中的环境变量”ROUTER_METRICS_TYPE“
    
    这个环境变量默认值为“haproxy”。不删除的话，访问的时候会报一下错误

    ```bash
    Forbidden: User "system:anonymous" cannot get routers/metrics.route.openshift.io at the cluster scope
    ```

4. 将健康检查readiness的HTTP GET URL由“/healthz/ready”改为"/healthz"。（不然router POD无法通过健康检查）

    ![](../assets/OCP3-11开启router的haproxy-statisc-2.png)

5. 验证监听端口80，443，1936

    ```bash
    ss -ntl|grep 80
    ss -ntl|grep 443
    ss -ntl|grep 1936
    ```

6. 访问router haproxy statistics 页面。
   
    访问方式是：http://<user>:<password>@router所在节点IP地址:1936 例如：http://admin:MJbJFvODhP@allinone.curiouser.com:1936

    ![](../assets/OCP3-11开启router的haproxy-statisc-3.png)

7. 相关链接
   
    https://docs.openshift.com/container-platform/3.11/install_config/router/default_haproxy_router.html#using-wildcard-routes
    https://bugzilla.redhat.com/show_bug.cgi?id=1579054
    https://github.com/openshift/origin/issues/17025
    https://blog.chmouel.com/2016/09/27/how-to-view-openshift-haproxy-stats/