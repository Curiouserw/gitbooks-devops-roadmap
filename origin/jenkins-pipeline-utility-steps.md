# Jenkins Pipeline-Utility-steps插件 

# 一、Pipeline-Utility-steps插件简介

Jekins中的Pipeline-Utility-steps插件能让你在pipeline的Step中直接使用它的API方法进行某些操作，例如查找文件，读取YAML/JSON/Properties文件、读取Maven工程POM文件等。这些方法有一个前提，任何文件都需要放在jenkins的workspace下，执行的job才能去找到文件。

Github地址：https://github.com/jenkinsci/pipeline-utility-steps-plugin

相关文档：https://jenkins.io/doc/pipeline/steps/pipeline-utility-steps/

# 二、Pipeline-Utility-steps插件的方法

## 1、文件操作

### findFiles

根据一些字符串规则去查找文件，如果有匹配的查找，返回是一个fille数组对象。（[文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/fs/FindFilesStep/help.html)）

- 参数

  - excludes(可选，参数类型为String)
  - glob(可选，参数类型为String)

- 示例

  ```groovy
  def files = findFiles(glob: '**/TEST-*.xml')

  echo """${files[0].name} ${files[0].path} ${files[0].directory} ${files[0].length} ${files[0].lastModified}"""
  ```

### touch

创建文件（如果文件不存在的话）并设置时间戳. Returns a FileWrapper representing the file that was touched. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/fs/TouchStep/help.html))

- 参数

  - file(参数类型为String)：The path to the file to touch.
  - timestamp(可选，参数类型为long)：The timestamp to set (number of ms since the epoc), leave empty for current system time.

### sha1

计算指定文件的SHA1 ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/fs/FileSha1Step/help.html))

- 参数

  - file(参数类型为String): The path to the file to hash.

### tee

将输出重定向到文件

- 参数

  - file(参数类型为String)

### Zip Files

- `zip`：创建Zip文件. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/zip/ZipStep/help.html))
- 参数
  
  - zipFile(参数类型为String): The name/path of the zip file to create.
    - archive(可选，参数类型为boolean): If the zip file should be archived as an artifact of the current build. The file will still be kept in the workspace after archiving.
    - dir(可选，参数类型为String): The path of the base directory to create the zip from. Leave empty to create from the current working directory.
    - glob(可选，参数类型为String): Ant style pattern of files to include in the zip. Leave empty to include all files and directories.
  
- `unzip`：解压或读取Zip文件 ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/zip/UnZipStep/help.html))
- 参数
    - zipFile(参数类型为String): The name/path of the zip file to extract.
    - charset(可选，参数类型为String): Specify which Charset you wish to use eg. UTF-8
    - dir(可选，参数类型为String): The path of the base directory to extract the zip to. Leave empty to extract in the current working directory.
    - glob(可选，参数类型为String): Ant style pattern of files to extract from the zip. Leave empty to include all files and directories.
    - quiet(可选，参数类型为boolean): Suppress the verbose output that logs every single file that is dealt with. E.g. unzip zipFile: 'example.zip', quiet: true
    - read(可选，参数类型为boolean): Read the content of the files into a Map instead of writing them to the workspace. The keys of the map will be the path of the files read. E.g. def v = unzip zipFile: 'example.zip', glob: '*.txt', read: true String version = v['version.txt']
    - test(可选，参数类型为boolean): Test the integrity of the archive instead of extracting it. When this parameter is enabled, all other parameters (except for zipFile) will be ignored. The step will return true or false depending on the result instead of throwing an exception.

## 2、配置文件操作

### readProperties

Reads a file in the current working directory or a String as a plain text [Java Properties](https://docs.oracle.com/javase/7/docs/api/java/util/Properties.html) file. The returned object is a normal Map with String keys. The map can also be pre loaded with default values before reading/parsing the data. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/conf/ReadPropertiesStep/help.html))

- 参数

  - defaults (可选，Nested Choice of Objects): An Map containing default key/values. These are added to the resulting map first.
  - file(可选，参数类型为String): path to a file in the workspace to read the properties from. These are added to the resulting map after the defaults and so will overwrite any key/value pairs already present.
  - interpolate (可选，参数类型为boolean): Flag to indicate if the properties should be interpolated or not. In case of error or cycling dependencies, the original properties will be returned.
  - text (可选，参数类型为String): An String containing properties formatted data. These are added to the resulting map after file and so will overwrite any key/value pairs already present.

- 示例

    ```groovy
    def d = [test: 'Default', something: 'Default', other: 'Default']
    def props = readProperties defaults: d, file: 'dir/my.properties', text: 'other=Override'
    assert props['test'] == 'One'
    assert props['something'] == 'Default'
    assert props.something == 'Default'
    assert props.other == 'Override'
    ```

    ```groovy
    def props = readProperties interpolate: true, file: 'test.properties'
    assert props.url = 'http://localhost'
    assert props.resource = 'README.txt'
    // if fullUrl is defined to ${url}/${resource} then it should evaluate to http://localhost/README.txt
    assert props.fullUrl = 'http://localhost/README.txt'
    ```

### readManifest

Reads a Jar Manifest file or text and parses it into a set of Maps. The returned data structure has two properties: main for the main attributes, and entries containing each individual section (except for main). ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/conf/mf/ReadManifestStep/help.html))

- 参数

  -  file(可选，参数类型为String。值只能是file或text，两者不能同时设置): path to a file to read. It could be a plain text, .jar, .war or .ear. In the latter cases the manifest will be extracted from the archive and then read.
  -  text(可选，参数类型为String。值只能是file或text，两者不能同时设置):  text containing the manifest data.

- 示例
    ```groovy
    def man = readManifest file: 'target/my.jar'
    
    assert man.main['Version'] == '6.15.8'
    assert man.main['Application-Name'] == 'My App'
    assert man.entries['Section1']['Key1'] == 'value1-1'
    assert man.entries['Section2']['Key2'] == 'value2-2'
    ```

### readYaml

Reads a file in the current working directory or a String as a plain text [YAML](http://yaml.org/) file. It uses [SnakeYAML](https://bitbucket.org/asomov/snakeyaml) as YAML processor. The returned objects are standard Java objects like List, Long, String, ...: bool: [true, false, on, off] int: 42 float: 3.14159 list: ['LITE', 'RES_ACID', 'SUS_DEXT'] map: {hp: 13, sp: 5}. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/conf/ReadYamlStep/help.html))

- 参数

  - file(可选，参数类型为String)
  - text(可选，参数类型为String)

- 示例

    ```groovy
    // 读取单个YAML文件
    def datas = readYaml text: """
    something: 'my datas'
    size: 3
    isEmpty: false
    """
    assert datas.something == 'my datas'
    assert datas.size == 3
    assert datas.isEmpty == false
    ```

    ```groovy
    // 读取多个YAML文件
    def datas = readYaml text: """
    ---
    something: 'my first document'
    ---
    something: 'my second document'
    """
    assert datas.size() == 2
    assert datas[0].something == 'my first document'
    assert datas[1].something == 'my second document'
    ```

    ```groovy
    // With file dir/my.yml containing something: 'my datas' :
    def datas = readYaml file: 'dir/my.yml', text: "something: 'Override'"
    assert datas.something == 'Override'
    ```

### writeYaml

Write a YAML file from an object. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/conf/WriteYamlStep/help.html))

- 参数
  - file(参数类型为String): Mandatory path to a file in the workspace to write the YAML datas to.
  - data: A Mandatory Object containing the data to be serialized.
  - charset: Optionally specify the charset to use when writing the file. Defaults to UTF-8 if nothing else is specified. What charsets that are available depends on your Jenkins master system. The java specification tells us though that at least the following should be available:
     [ US-ASCII、ISO-8859-1、UTF-8、UTF-16BE、UTF-16LE、UTF-16]
- 示例

    ```groovy
    def amap = ['something': 'my datas', 'size': 3, 'isEmpty': false]
    writeYaml file: 'datas.yaml', data: amap
    def read = readYaml file: 'datas.yaml'
        
    assert read.something == 'my datas'
    assert read.size == 3
    assert read.isEmpty == false
    ```

### readJSON

Reads a file in the current working directory or a String as a plain text [JSON](http://www.json.org/json-it.html) file. The returned object is a normal Map with String keys or a List of primitives or Map. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/json/ReadJSONStep/help.html))

- 参数

  - file(可选，参数类型为String。值只能是file或text，两者不能同时设置): Path to a file in the workspace from which to read the JSON data. Data could be access as an array or a map.
- text(可选，参数类型为String。值只能是file或text，两者不能同时设置): A string containing the JSON formatted data. Data could be access as an array or a map.

- 示例

    ```groovy
    def props = readJSON file: 'dir/input.json'
    assert props['attr1'] == 'One'
    assert props.attr1 == 'One'
    def props = readJSON text: '{ "key": "value" }'
      assert props['key'] == 'value'
      assert props.key == 'value'
            
    def props = readJSON text: '[ "a", "b"]'
      assert props[0] == 'a'
      assert props[1] == 'b'
    ```
- `writeJSON`：Write a [JSON](http://www.json.org/json-it.html) file in the current working directory. That for example was previously read by `readJSON`. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/json/WriteJSONStep/help.html))

  - 参数
  
    - file(可选，参数类型为String): Path to a file in the workspace to write to.
    - json(Nested Choice of Objects): The JSON object to write.
    - pretty (可选，参数类型为int): Prettify the output with this number of spaces added to each level of indentation.
  
  - 示例

```groovy
  def input = readJSON file: 'myfile.json'
  //Do some manipulation
  writeJSON file: 'output.json', json: input
  //or pretty print it, indented with a configurable number of spaces
  writeJSON file: 'output.json', json: input, pretty: 4
```

### readCSV

Reads a file in the current working directory or a String as a plain text. A List of [CSVRecord](https://commons.apache.org/proper/commons-csv/apidocs/org/apache/commons/csv/CSVRecord.html) instances is returned. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/csv/ReadCSVStep/help.html))

- 参数
  
    - file(可选，参数类型为String。值只能是file或text，两者不能同时设置): Path to a file in the workspace from which to read the CSV data. Data is accessed as a List of String Array.
    - text(可选，参数类型为String。值只能是file或text，两者不能同时设置): A string containing the CSV formatted data. Data is accessed as a List of String Arrays.
    - format(可选，org.apache.commons.csv.CSVFormat)
    
  - 示例

```groovy
def records = readCSV file: 'dir/input.csv'
assert records[0][0] == 'key'
assert records[1][1] == 'b'
def content = readCSV text: 'key,value\na,b'
    assert records[0][0] == 'key'
    assert records[1][1] == 'b'
      
// 进阶示例
    def excelFormat = CSVFormat.EXCEL
    def records = readCSV file: 'dir/input.csv', format: excelFormat
    assert records[0][0] == 'key'
    assert records[1][1] == 'b'
      
def content = readCSV text: 'key,value\na,b', format: CSVFormat.DEFAULT.withHeader()
    assert records[1].get('key') == 'a'
    assert records[1].get('value') == 'b'
```
### writeCSV

Write a CSV file in the current working directory. That for example was previously read by `readCSV`. See [CSVPrinter](https://commons.apache.org/proper/commons-csv/apidocs/org/apache/commons/csv/CSVPrinter.html) for details.([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/csv/WriteCSVStep/help.html))

- 参数
  
    - file(参数类型为String): Path to a file in the workspace to write to.
    - records(java.lang.Iterable): The list of CSVRecord instances to write.
- format(可选，org.apache.commons.csv.CSVFormat):See CSVFormat for details.

- 示例

  ```groovy
    def records = [['key', 'value'], ['a', 'b']]
    writeCSV file: 'output.csv', records: records, format: CSVFormat.EXCEL
  ```

## 3、Maven项目

### readMavenPom

读取Maven POM文件到一个[Model](http://maven.apache.org/components/ref/3.3.9/maven-model/apidocs/org/apache/maven/model/Model.html)数据结构中. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/maven/ReadMavenPomStep/help.html))

- 参数
  
  - file(可选，参数类型为String)：默认读取目前工作区下的POM.xml文件
  
- 示例
  
    ```groovy
    stage ('上传制品') {
    steps {
        script{
        def pomfile = readMavenPom file: 'pom.xml'
        nexusPublisher nexusInstanceId: 'curiouser-okd-nexus', \
        nexusRepositoryId: 'Maven-Releases', \
            packages: [[$class: 'MavenPackage', \
            mavenAssetList: [[classifier: '', extension: '', \
            filePath: "target/${pomfile.artifactId}-${pomfile.version}.${pomfile.packaging}"]], \
            mavenCoordinate: [artifactId: "${pomfile.artifactId}", \
            groupId: "${pomfile.groupId}", \
            packaging: "${pomfile.packaging}", \
            version: "${pomfile.version}"]]]
        }
    }
    }
    ```

### writeMavenPom

Writes a [Maven project](https://maven.apache.org/pom.html) file. That for example was previously read by `readMavenPom`. ([文档](https://github.com/jenkinsci/pipeline-utility-steps-plugin/blob/master/src/main/resources/org/jenkinsci/plugins/pipeline/utility/steps/maven/WriteMavenPomStep/help.html))

- 参数
  
  - model(参数类型为org.apache.maven.model.Model): The Model object to write.
  - file(可选，参数类型为String):  Optional path to a file in the workspace to write to. If left empty the step will write to pom.xml in the current working directory.
  
- 示例
  
```groovy
  def pom = readMavenPom file: 'pom.xml'
  //Do some manipulation
  writeMavenPom model: pom
```

