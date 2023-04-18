# 常用资源对象操作

## 1、登录

    oc project <project_name>
    oc login -u 用户名 集群master的URL 
    oc whoami #查看当前登录的用户，加-t参数可查看当前用户的token

## 2、切换Project

    oc project <project_name>

## 3、查看集群节点

    oc get node/no
    oc get node/no node1.test.openshift.com

## 4、查看集群节点的详细信息

    oc describe node node1.test.openshift.com

## 5、查看某个节点上的所有Pods

    oc adm manage-node node1.test.openshift.com --list-pods

## 6、使节点禁止参与调度

    oc adm manage-node router1.test.openshift.com --schedulable=false

## 7、疏散某个节点上的所有POD

    oc adm drain router1.test.openshift.com --ignore-daemonsets

## 8、清除旧的Build和Deployments历史版（所有namespace）

    统计要清除的资源个数
    #oc adm prune deployments --keep-younger-than=24h --keep-complete=5 --keep-failed=5|wc -l
    确认清除动作
    # oc adm prune [deployments|builds|images] --confirm --keep-younger-than=24h --keep-complete=5 --keep-failed=5

    参数详解
    --confirm  确认操作
    --keep-younger-than=1h0m0s   Specify the minimum age of a Build for it to be considered a candidate for pruning.
    --keep-complete=5            Per BuildConfig, specify the number of builds whose status is complete that will be preserved.
    --keep-failed=1              Per BuildConfig, specify the number of builds whose status is failed, error, or cancelled that will be preserved.
    --orphans=false              If true, prune all builds whose associated BuildConfig no longer exists and whose status is complete, failed, error, or cancelled.

    示例：
    清理images（在admin用户下执行）
    # oc adm prune images --keep-younger-than=400m --keep-tag-revisions=10 --registry-url=docker-registry.default.svc:5000 --certificate-authority=/etc/origin/master/registry.crt --confirm

## 9、删除所有Namespace中非Running的pods

    for i in `oc get po --all-namespaces|grep -v "Running"|grep -v "NAMESPACE"|awk '{print $1}'|sort -u` ;
    do
        echo "===================Namespace $i===================";
        oc -n $i delete po `oc get po -n $i |grep -v "Running"|grep -v "NAME"|awk 'BEGIN{ORS=" "}{print $1}'`;
    done

## 10、强制删除POD

    oc delete po gitlab-ce-16-ntzst --force --grace-period=0

## 11、资源的查看

    #查看当前项目的所有资源
    oc get all 
    #查看当前项目的所有资源，外加输出label信息
    oc get all --show-labels
    # 查看指定资源
    oc get pod/po
    oc get service/svc
    oc get persistentvolumes/pv

## 12、通过label选择器删除namespace下所有的资源

    #如果namespace下所有的资源都打上了“name=test”标签
    oc delete all -l name=test 

## 13、项目的管理

    #创建项目
    oc new-project --display-name=显示的项目名 --description=项目描述 project_name
    #删除项目
    oc delete project 项目名
    #查看当前处于哪个项目下
    oc project
    #查看所有项目
    oc projects

## 14、模板的管理

    #创建模板(模板文件格式为YAML/JSON.也可以在Openshift的web页面上直接导入)
    oc create -f <TemplateFile_Path>
    #查看模板
    oc get templates
    #编辑模板
    oc edit template <template_name>
    #删除模板
    oc delete template <template_name>

# 附录

```bash
buildconfigs (aka 'bc')                               #构建配置
builds                                                #构建版本
certificatesigningrequests (aka 'csr')
clusters (valid only for federation apiservers)
clusterrolebindings
clusterroles
componentstatuses (aka 'cs') 
configmaps (aka 'cm') 
daemonsets (aka 'ds') 
deployments (aka 'deploy')
deploymentconfigs (aka 'dc') 
endpoints (aka 'ep') 
events (aka 'ev') 
horizontalpodautoscalers (aka 'hpa')
imagestreamimages (aka 'isimage')
imagestreams (aka 'is') 
imagestreamtags (aka 'istag')
ingresses (aka 'ing')
groups
jobs
limitranges (aka 'limits')
namespaces (aka 'ns') 
networkpolicies
nodes (aka 'no') 
persistentvolumeclaims (aka 'pvc')
persistentvolumes (aka 'pv') 
poddisruptionbudgets (aka 'pdb')
podpreset
pods (aka 'po') 
podsecuritypolicies (aka 'psp')
podtemplates
policies
projects
replicasets (aka 'rs') 
replicationcontrollers (aka 'rc') 
resourcequotas (aka 'quota')
rolebindings
roles
routes
secrets
serviceaccounts (aka 'sa')
services (aka 'svc')
statefulsets
users
storageclasses
thirdpartyresources
```