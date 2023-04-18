ES自带的有插件管理脚本命令

以RPM方式安装的ES，插件管理脚本在/usr/share/elasticsearch/bin/elasticsearch-plugin。该脚本能安装，列出，移除插件

    $> cd /usr/share/elasticsearch/bin/
    $> ./elasticsearch-plugin list   #列出所有插件
    $> ./elasticsearch-plugin install plugin_name #安装插件
    $> ./elasticsearch-plugin remove plugin_name  #卸载插件
    
    #该脚本的参数
    #-v 输出详细信息
    #-s 输出最简信息

脚本返回状态码
* **0** : everything was OK   
* **64** : unknown command or incorrect option parameter    
* **74** : IO error  
* **70** : any other error


设置代理来安装插件

    $ sudo ES_JAVA_OPTS="-Dhttp.proxyHost=代理服务器IP地址 -Dhttp.proxyPort=代理服务器端口 -Dhttps.proxyHost=代理服务器IP地址 -Dhttps.proxyPort=代理服务器端口" bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.0/elasticsearch-analysis-ik-6.3.0.zip