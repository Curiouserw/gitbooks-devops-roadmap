# aliyun-cli CLI工具

# 一、简介

官方文档：https://help.aliyun.com/product/29991.html

Github地址：https://github.com/aliyun/aliyun-cli

# 二、安装配置

## 1、二进制安装

**下载地址**：https://github.com/aliyun/aliyun-cli/releases

**下载解压至系统环境变量路径下即可**

## 2、配置

在使用阿里云CLI之前，您需要配置调用阿里云资源所需的凭证信息、地域、语言等。

### 凭证类型

| 验证方式   | 说明                                      | 交互式配置凭证（快速）                                       | 非交互式配置凭证                                             |
| :--------- | :---------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| AK         | 使用aliyun-cli-clicessKey ID/Secret访问。 | [配置aliyun-cli-clicessKey凭证](https://help.aliyun-cli.com/document_detail/121258.html#section-5pj-p7j-06z) | [配置aliyun-cli-clicessKey凭证](https://help.aliyun-cli.com/document_detail/121259.html#section-hhx-jpx-95g) |
| StsToken   | 使用STS Token访问。                       | [配置STS Token凭证](https://help.aliyun-cli.com/document_detail/121258.html#section-bdk-377-tnm) | [配置STS Token凭证](https://help.aliyun-cli.com/document_detail/121259.html#section-mek-l1j-xib) |
| RamRoleArn | 使用RAM子账号的AssumeRole方式访问。       | [配置RamRoleArn凭证](https://help.aliyun-cli.com/document_detail/121258.html#section-h4x-fnh-5yj) | [配置RamRoleArn凭证](https://help.aliyun-cli.com/document_detail/121259.html#section-uyo-8pk-uow) |
| EcsRamRole | 在ECS实例上通过EcsRamRole实现免密验证。   | [配置EcsRamRole凭证](https://help.aliyun-cli.com/document_detail/121258.html#section-pq4-04b-7an) | [配置EcsRamRole凭证](https://help.aliyun-cli.com/document_detail/121259.html#section-874-dbh-9k0) |

### 配置凭据

**交互式地配置凭据**


```bash
aliyun-cli configure --mode <authenticationMethod>
    Configuring profile 'akProfile' in '' authenticate mode...
    aliyun-cli-clicess Key Id []: # aliyun-cli-clicessKey ID
    aliyun-cli-clicess Key Secret []: # aliyun-cli-clicessKey Secret
    Default Region Id []: # cn-hangzhou
    Default Output Format [json]: json (Only support json))
    Default Language [zh|en] en:
    Saving profile[akProfile] ...Done.
# --profile：指定配置名称。如果指定的配置存在，则修改配置。若不存在，则创建配置。
# --mode：指定凭证类型。分别为AK、StsToken、RamRoleArn和EcsRamRole。
```

**非交互式地配置凭据**

```bash
aliyun-cli configure set \
  --profile akProfile \
  --mode AK \
  --region cn-hangzhou \
  --aliyun-cli-clicess-key-id aliyun-cli-clicessKeyId \
  --aliyun-cli-clicess-key-secret aliyun-cli-clicessKeySecret
  
# --profile（必选）：指定配置名称。如果指定的配置存在，则修改配置。若不存在，则创建配置。
# --region（必选）：指定默认区域的RegionId。阿里云支持的RegionId，请参见地域和可用区。
# --language：指定阿里云CLI显示的语言，默认为英语。
# --mode：指定配置的凭证类型，默认为AK。

#其他验证方式的设置省略  
```



### 凭据管理

```bash
aliyun-cli configure --help

子命令:
  get    显示配置项
  set    非交互式地配置凭据
  list   显示所有凭据
  delete 删除凭据
```

### 命令自动补全功能

仅支持zsh/bash

```bash
# 启用自动补全功能
aliyun-cli auto-completion
# 也可以手动在当前shell配置文件(例如~/.zshrc)中追加“complete -o nospaliyun-cli-clie -F /usr/local/bin/aliyun-cli-cli aliyun-cli-cli”

# 关闭自动补全功能
aliyun-cli auto-completion --uninstall
```



# 三、命令详解

## 命令格式

```bash
aliyun-cli <command> <subcommand> [options and parameters]

# command：指定一个顶级命令。通常表示命令行工具中支持的阿里云产品基础服务，例如ecs、rds等。也表示命令行工具本身的功能命令，例如help、configure等。

# subcommand：指定要执行操作的附加子命令，即具体的某一项操作。
# options and parameters：指定用于控制阿里云CLI行为的选项或者API参数选项，其选项值可以是数字、字符串和json结构字符串等。
```

调用产品接口时，首先需要判断API类型，选择标准的命令结构发起调用。

**通过以下特点判断API类型：**

- API参数中包含**aliyun-cli-clition**字段的是RPC API，需要**PathPattern**参数的是RESTful API。
- 一般情况下，每个产品内所有API的调用风格是统一的。
- 每个API仅支持特定的一种风格，传入错误的标识，可能会调用到其他API，或收到`ApiNotFound`的错误信息。

## 调用RPC API的命令格式

```bash
aliyun-cli <product> <ApiName> [--parameter1 value1 --parameter2 value2 ...]
```

## 调用RESTful API的命令格式

```bash
 aliyun-cli 云产品code [GET|PUT|POST|DELETE] <PathPattern> --body "$(cat input.json)"
```

## 格式化输出参数：--output

选项字段说明

| 字段名   | 描述                                                     | 补充说明                                                     |
| :------- | :------------------------------------------------------- | :----------------------------------------------------------- |
| **cols** | 表格的列名，需要与json数据中的字段相对应。               | 例如，ECS DescribeInstances接口返回结果中的字段**InstanceId**以及**Status** 。 |
| **rows** | 指定过滤字段所在的[JMESPath](http://jmespath.org/)路径。 | 通过[jmespath](http://jmespath.org/)查询语句来指定表格行在json结果中的数据来源。 |
| **num**  | 指定`num=true`，开启行号列，行号以数字0开始。            | 默认`num=false`。                                            |

### 示例

```bash
aliyun-cli ecs DescribeInstances --output cols=InstanceId,Status rows=Instances.Instance[] num=true

Num | InstanceId             | Status
--- | ----------             | ------
0   | i-12345678912345678123 | Stopped
1   | i-abcdefghijklmnopqrst | Running
```

## 分页类接口结果聚合输出参数：--pager

使用阿里云CLI调用各云产品的分页类接口时，默认情况下仅返回第一页的结果。当需要获取所有的结果时，您可以使用阿里云CLI聚合结果的功能。

| 字段名         | 描述                                                         |
| :------------- | :----------------------------------------------------------- |
| **PageNumber** | 字段值对应API返回结果中描述列表当前页码的字段，默认值：PageNumber。 |
| **PageSize**   | 字段值对应API返回结果中描述每页返回的最大结果数量的字段，默认值：PageSize。 |
| **TotalCount** | 字段值对应API返回结果中描述列表总行数的字段，默认值：TotalCount。 |
| **path**       | 由于API返回结果的多样性，您可以手动指定需要聚合的数组类型所在的JMESPath路径。**说明** **--pager**选项默认可以自动识别结果中的数组类型数据。 |

### 示例

```bash
aliyun-cli ecs DescribeInstances --pager PageNumber=PageNumber PageSize=PageSize TotalCount=TotalCount path=Instances.Instance
```

## 模拟调用参数：--dryrun

```bash
aliyun-cli ecs DescribeInstances --dryrun
```

## 结果轮询参数：--waiter

在阿里云API中，某些API返回的结果会随时间的推移而变化。您可以通过结果轮询，直到某个值出现特定状态时停止轮询，并返回数据

| 字段名   | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| **expr** | 表示通过[jmespath](http://jmespath.org/)查询语句指定的json结果中的被轮询字段。 |
| **to**   | 表示被轮询字段的目标值。                                     |

### **示例**

执行创建ECS实例的命令后，调用DescribeInstances接口查询一台或多台实例的详细信息。由于实例创建需要时间，将不断的查询实例的运行状态，直到处于Running状态，DescribeInstances接口成功返回数据。

```bash
aliyun-cli ecs DescribeInstances --InstanceIds '["i-12345678912345678123"]' --waiter expr='Instances.Instance[0].Status' to=Running
```

## 强制调用接口参数：--force

阿里云CLI集成了部分云产品的元数据，调用时会检查参数的合法性。由于API具有不同的版本，导致内置的产品和接口信息并不能满足所有的需求。您可以强制调用元数据列表以外的接口，并自行检查该接口相关信息的准确性。

在阿里云CLI中，如果调用了一个元数据中未包含的API或参数，会导致`unknown api`或`unknown parameter`错误。您可以通过使用**--force**选项，强制调用元数据列表以外的API和参数。调用时，您需要确保以下信息的准确性：

- 云产品code
- 接口名称及参数
- API版本
- endpoint信息

当使用**--force**选项强制调用接口时，必须指定**--version**选项，用以指定API版本。例如，ECS的版本号是2014-05-26。还可以指定**--endpoint**选项，用以指定产品的接入地址。若不指定，则从阿里云CLI内置数据中获取。

### **示例**：

在CMS产品中，有一个接口用于描述MetricList。在阿里云CLI 3.0.16版本中，CMS的API版本为2019-01-01，接口名称为DescribeMetricList。但在2017-03-01版本中，该接口名称为QueryMetricList。

```bash
aliyun-cli cms QueryMetricList [api参数] --force --version 2017-03-01
```



# 四、使用示例

## 1、查看域名的子域名解析记录

```bash
aliyun-cli-cli alidns DescribeDomainRecords --DomainName *.top
```

## 2、更新子域名的主机记录

```bash
aliyun-cli alidns UpdateDomainRecord --RecordId * --RR home --Type A --Value 1.1.1.1 --Line Default
```

