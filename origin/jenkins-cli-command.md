# Jenkins命令行接口

# 一、简介

## 1、下载命令行Jar包

下载CLI的Jar包：`http://Jenkins地址/jnlpJars/jenkins-cli.jar`

## 2、使用语法

```bash
java -jar jenkins-cli.jar [-s URL] 子命令 [opts...] args...
```

## 3、参数

```bash
-s URL                        : the server URL (defaults to the JENKINS_URL env var)
-http                         : use a plain CLI protocol over HTTP(S) (the default; mutually exclusive with -ssh)
-webSocket                    : like -http but using WebSocket (works better with most reverse proxies)
-ssh                          : use SSH protocol (requires -user; SSH port must be open on server, and user must have registered a public key)
-i KEY                        : SSH private key file used for authentication (for use with -ssh)
-noCertificateCheck           : bypass HTTPS certificate check entirely. Use with caution
-noKeyAuth                    : don't try to load the SSH authentication private key. Conflicts with -i
-user                         : specify user (for use with -ssh)
-strictHostKey                : request strict host key checking (for use with -ssh)
-logger FINE                  : enable detailed logging from the client
-auth [ USER:SECRET | @FILE ] : specify username and either password or API token (or load from them both from a file);
                                for use with -http.
                                Passing credentials by file is recommended.
                                See https://www.jenkins.io/redirect/cli-http-connection-mode for more info and options.
-bearer [ TOKEN | @FILE ]     : specify authentication using a bearer token (or load the token from file);
                                for use with -http. Mutually exclusive with -auth.
                                Passing credentials by file is recommended.
```

## 4、子命令

| 子命令                             | 功能描述                                                     |
| ---------------------------------- | ------------------------------------------------------------ |
| add-job-to-view                    | Adds jobs to view.                                           |
| apply-configuration                | Apply YAML configuration to instance                         |
| build                              | Builds a job, and optionally waits until its completion.     |
| cancel-quiet-down                  | Cancel the effect of the "quiet-down" command.               |
| check-configuration                | Check YAML configuration to instance                         |
| clear-queue                        | Clears the build queue.                                      |
| connect-node                       | Reconnect to a node(s)                                       |
| console                            | Retrieves console output of a build.                         |
| copy-job                           | Copies a job.                                                |
| create-credentials-by-xml          | Create Credential by XML                                     |
| create-credentials-domain-by-xml   | Create Credentials Domain by XML                             |
| create-job                         | Creates a new job by reading stdin as a configuration XML file. |
| create-node                        | Creates a new node by reading stdin as a XML configuration.  |
| create-view                        | Creates a new view by reading stdin as a XML configuration.  |
| declarative-linter                 | Validate a Jenkinsfile containing a Declarative Pipeline     |
| delete-builds                      | Deletes build record(s).                                     |
| delete-credentials                 | Delete a Credential                                          |
| delete-credentials-domain          | Delete a Credentials Domain                                  |
| delete-job                         | Deletes job(s).                                              |
| delete-node                        | Deletes node(s)                                              |
| delete-view                        | Deletes view(s).                                             |
| disable-job                        | 禁用任务                                                     |
| disable-plugin                     | Disable one or more installed plugins.                       |
| disconnect-node                    | Disconnects from a node.                                     |
| enable-job                         | 启用任务                                                     |
| enable-plugin                      | Enables one or more installed plugins transitively.          |
| export-configuration               | Export jenkins configuration as YAML                         |
| get-credentials-as-xml             | Get a Credentials as XML (secrets redacted)                  |
| get-credentials-domain-as-xml      | Get a Credentials Domain as XML                              |
| get-job                            | Dumps the job definition XML to stdout.                      |
| get-node                           | Dumps the node definition XML to stdout.                     |
| get-view                           | Dumps the view definition XML to stdout.                     |
| groovy                             | Executes the specified Groovy script.                        |
| groovysh                           | Runs an interactive groovy shell.                            |
| help                               | Lists all the available commands or a detailed description of single command. |
| import-credentials-as-xml          | Import credentials as XML. The output of "list-credentials-as-xml" can be used as input here as is, the only needed change is to set the actual Secrets which are redacted in the output. |
| install-plugin                     | Installs a plugin either from a file, an URL, or from update center. |
| keep-build                         | 永久保留这次构建。                                           |
| list-changes                       | Dumps the changelog for the specified build(s).              |
| list-credentials                   | Lists the Credentials in a specific Store                    |
| list-credentials-as-xml            | Export credentials as XML. The output of this command can be used as input for "import-credentials-as-xml" as is, the only needed change is to set the actual Secrets which are redacted in the output. |
| list-credentials-context-resolvers | List Credentials Context Resolvers                           |
| list-credentials-providers         | List Credentials Providers                                   |
| list-jobs                          | Lists all jobs in a specific view or item group.             |
| list-plugins                       | Outputs a list of installed plugins.                         |
| mail                               | Reads stdin and sends that out as an e-mail.                 |
| offline-node                       | Stop using a node for performing builds temporarily, until the next "online-node" command. |
| online-node                        | Resume using a node for performing builds, to cancel out the earlier "offline-node" command. |
| quiet-down                         | Quiet down Jenkins, in preparation for a restart. Don’t start any builds. |
| reload-configuration               | Discard all the loaded data in memory and reload everything from file system. Useful when you modified config files directly on disk. |
| reload-jcasc-configuration         | Reload JCasC YAML configuration                              |
| reload-job                         | Reload job(s)                                                |
| remove-job-from-view               | Removes jobs from view.                                      |
| replay-pipeline                    | 从标准输入中获取的脚本并回放流水线执行                       |
| restart                            | 重新启动Jenkins                                              |
| restart-from-stage                 | Restart a completed Declarative Pipeline build from a given stage. |
| safe-restart                       | 安全地重新启动Jenkins                                        |
| safe-shutdown                      | Puts Jenkins into the quiet mode, wait for existing builds to be completed, and then shut down Jenkins. |
| session-id                         | Outputs the session ID, which changes every time Jenkins restarts. |
| set-build-description              | Sets the description of a build.                             |
| set-build-display-name             | Sets the displayName of a build.                             |
| shutdown                           | 立刻关闭Jenkins                                              |
| stop-builds                        | Stop all running builds for job(s)                           |
| update-credentials-by-xml          | Update Credentials by XML                                    |
| update-credentials-domain-by-xml   | Update Credentials Domain by XML                             |
| update-job                         | Updates the job definition XML from stdin. The opposite of the get-job command. |
| update-node                        | Updates the node definition XML from stdin. The opposite of the get-node command. |
| update-view                        | Updates the view definition XML from stdin. The opposite of the get-view command. |
| version                            | Outputs the current version.                                 |
| wait-node-offline                  | Wait for a node to become offline.                           |
| wait-node-online                   | Wait for a node to become online.                            |
| who-am-i                           | Reports your credential and permissions.                     |

# 二、常用操作

## 1、导出Job为XML文件

```bash
java -jar jenkins-cli.jar -s http://jenkins地址/ -webSocket -auth 用户名:密码 get-job 任务名 > 任务名.xml
```

### **批量导出Job脚本**

```bash
jenkins_host_url=http://jenkins地址
jenkins_user=用户名
jenkins_password=密码
wget $jenkins_host_url/jnlpJars/jenkins-cli.jar
for i in `java -jar jenkins-cli.jar -s $jenkins_host_url -auth $jenkins_user:$jenkins_password list-jobs` ;do
  java -jar jenkins-cli.jar -s $jenkins_host_url -auth $jenkins_user:$jenkins_password get-job $i > $i.xml;
done
```

## 2、从XML文件导入Job

```bash
java -jar jenkins-cli.jar -s http://jenkins地址/ -webSocket -auth 用户名:密码 create-job 任务名 < 任务名.xml
```







