# 如何将 Java代码打包成 EXE 可安装执行软件

# 一、简介

Java构建产物Jar包在类Unix系统中可以快速部署运行。但是当服务器环境是 Windows ，该如何快速、图形化、傻瓜式地部署？

# 二、构建最小运行环境JRE

- SpringBoot使用了3.0或者3.0以上，开始最低支持JDK17。
- 使用 JRE 运行 java jar程序，减少最终安装包的体积。
- 下载 Windows架构的JDK压缩文件

- 使用 MacOS 下的Jlink构建Windows下可运行的 JRE

```bash
export JAVA_HOME=~/Downloads/jdk-17.0.7
export PATH=$JAVA_HOME/bin:$PATH

# 使用jlink构建适用于Windows的JRE
jlink --module-path $JAVA_HOME/jmods:~/Downloads/jdk-17.0.7/jmods \
      --add-modules java.base,java.logging,java.naming,java.desktop,java.management,java.instrument,java.sql,java.security.jgss \
      --output ./windows-jre17-0-7
```

- 其他详细构建参数参考：https://gitbook.curiouser.top/origin/jlink-jre.md

# 三、构建包含所有依赖的 Jar

使用 Maven 的构建插件 maven-assembly-plugin将所有依赖都解压打包到构建产物中；

```xml
<project>
		 .....
     <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.7.1</version>
                <configuration>
                    <archive>
                        <manifest>
                          	<!-- 填写主类 -->
                            <mainClass>org.example.Application</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <!-- 有内置的DescriptorRef:
																bin: 类似于默认打包，会将bin目录下的文件打到包中
																jar-with-dependencies: 会将所有依赖都解压打包到构建产物中
																src: 只将源码目录下的文件打包
																project: 将整个project资源打包
												-->
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <!-- 绑定到package生命周期阶段上 -->
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

```bash
mvn clean package
```

**bulid完成后, 在target文件夹下可以找到带 jar-with-dependencies后缀的包。**

查看构建好的Jar包中文件：`tar -tvf jar包`

# 四、exe4j构建可执行exe

下载地址：https://www.ej-technologies.com/download/exe4j/files

**exe4j安装配置**

- exe4有`Windows、MacOS、Linux`环境的安装包，故安装过程省略。本章节默认在 MacOS环境下操作，exe4j安装在`/opt/exe4j9/`路径下
- exe4j配置可通过图形化界面进行设置，也可以直接通过配置文件进行操作。新手建议直接通过图形化界面(`/opt/exe4j9/bin/exe4j9.app`)进行配置

**exe4j操作关键点**

- exe4j构建的Jave EXE可运行文件，点击 exe文件即可运行。由于是 Java应用程序，需要JDK或 JRE 运行环境。故而需要第五步骤的 Inno Setup将 JRE 集成到最终的exe安装运行文件中

**exe4j配置文件释义**

- 配置文件默认在Java项目根目录

>  demoj2e.exe4j

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- version指定使用的 exe4j 版本。transformSequenceNumber指定转换序列号，通常用于标识配置的版本或更新。 -->
<exe4j version="9.0" transformSequenceNumber="3">
  
  <!-- 设置当前目录为配置的基础目录。 -->
  <directoryPresets config="." />
  
  <!-- name指定应用程序名称。 distributionSourceDir 指定工作路径为当前目录。 -->
  <application name="demoj2e" distributionSourceDir="." />
  
	<!-- name指定可执行文件名称
       executableDir指定可执行文件所在目录，当前目录
			 wrapperType指定嵌入类型，表示将 JVM 嵌入到可执行文件中
       redirectStderr、redirectStdout指定是否将标准输出、标准错误输出到文件
       stderrFile、stdoutFile指定标准输出、标准错误输出写入文件的路径。可使用相对路径，相对路径为可执行程序文件路径。
       stderrMode、stdoutMode指定标准输出、标准错误输出写入文件的方式。append表示追加，overwrite表示追加覆盖
       executableMode指定可执行文件的执行方式，可选 console 
       singleInstance指定运行模式为单实例模式、
       globalSingleInstance指定运行模式为全局单实例模式，确保系统中只有一个实例运行。 -->
  <executable name="demoj2e" 
    wrapperType="embed"
    executableMode="gui"
    singleInstance="true"
    executableDir="."
    globalSingleInstance="true"
    redirectStderr="true" 
    redirectStdout="true"
    stderrFile=".\logs\demoj2e_stderr.log" 
    stdoutFile=".\logs\demoj2e_stdout.log" 
    stderrMode="append" 
    stdoutMode="append"
  />
  <!-- mainClass指定Java程序的主类，
       vmParameters设置传递给JVM的参数，其中参数设置文件编码为 UTF-8。
       minVersion指定最低 JVM 版本要求为1.8 -->
  <java mainClass="org.example.Application" vmParameters="-Dfile.encoding=utf-8" minVersion="1.8">
    <!-- searchSequence指定JVM搜索顺序 -->
    <searchSequence>
      <directory location="./windows-jre17-0-7" />
    </searchSequence>
    <!-- classPath指定类路径 -->
    <classPath>
      <!--  location指定 jar 文件的路径，failOnError设置如果找不到指定的 jar 文件，不会失败。-->
      <archive location="./target/demoj2e-1.0-SNAPSHOT-jar-with-dependencies.jar" failOnError="false" />
    </classPath>
  </java>
</exe4j>
```

```bash
/opt/exe4j9/bin/exe4jc demoj2e.exe4j
```

此时只要将 demoj2e.exe和windows-jre17-0-7复制Windows的同级目录下，点击 exe文件即可运行。

**但是预设实际情况目标 Windows 主机不会预装Java JDK或 JRE运行环境。同时希望用户点击 exe 后一步一步地在图形化界面下将整个程序安装设置完。这就需要第五步，将 JRE、exe文件集成到一起，再套一个有图形化安装步骤的 EXE 程序壳。**

# 五、Inno Setup整合套壳

官网地址：https://jrsoftware.org

- **Inno Setup安装配置**
  - Inno Setup只有 Windows 安装包。没有类unix安装包。所以只能在 Windows 环境下操作exe集成 jre及套壳动作
  - 在 Windows 下的安装过程省略，安装路径默认`C:\"Program Files (x86)"\"Inno Setup 6"`
  - Inno Setup使用`Inno Setup compiler`图形化编辑器操作。但是也可以通过命令行直接操作。新手推荐在图形化编辑器中操作
  - 示例配置文件：`C:\Program Files (x86)\Inno Setup 6\Examples`
  - 配置指令说明书文件路径：`C:\Program Files (x86)\Inno Setup 6\ISetup.chm`
- **Inno Setup操作关键点**
    - **给 exe 可执行程序套一个通用的图形化安装程序**
    - **通过添加`Files`指令块将JRE集成到图形化安装程序过程中**
- 以下操作及资源文件默认为在 Windows 当前用户桌面下完成，所需文件：
  - 第二章节构建的 Windows 下最小化JRE
  - 第四章节exe4j步骤产生的exe可执行文件
  - Inno Setup配置文件

> 使用`Inno Setup compiler`加载配置文件： demoj2e.iss

```ini
; 设置变量

#define MyAppName "demoj2e"
#define MyAppVersion "1.5"
#define MyAppExeName "demoj2e.exe"
#define MyJreName "windows-jre17-0-7" 
#define MyAppPublisher "My Company, Inc."
#define MyAppURL "https://www.example.com/"
#define MyAppAssocName MyAppName + " File"
#define MyAppAssocExt ".myp"
#define MyAppAssocKey StringChange(MyAppAssocName, " ", "") + MyAppAssocExt

[Setup]			 ; 设置安装程序的基本信息
AppId={{F8D6C745-BAC5-4B86-82F8-0D95FBE28B68}}		; 应用程序的唯一标识符（通常是 GUID）	
AppName={#MyAppName}															; 应用程序名称
AppVersion={#MyAppVersion}												; 应用程序版本
AppPublisher={#MyAppPublisher}										; 应用程序的发布者名称
AppPublisherURL={#MyAppURL}												; 应用程序发布者的URL
AppSupportURL={#MyAppURL}													; 应用程序支持的URL
AppUpdatesURL={#MyAppURL}													; 应用程序更新的URL
DefaultDirName={autopf}\Demoj2e                   ; 默认安装目录。autopf内置变量路径C:\Program Files (x86)
DisableProgramGroupPage=yes												; 是否禁用程序组选择页面（默认是 no）。
OutputDir=.\output																; 编译后安装程序的输出目录
OutputBaseFilename={#MyAppName}-installer 				; 编译后输出文件的基本文件名
Compression=lzma																	; 压缩算法，常见值有lzma、zip
SolidCompression=yes															; 是否启用固实压缩（可以提高压缩比）
WizardStyle=modern																; 安装向导风格（classic、modern）
PrivilegesRequired=admin													; 安装所需的权限级别（none、poweruser、admin、lowest）。
; AllowNoIcons：是否允许不创建快捷方式（默认是 yes）。
; AlwaysRestart：是否始终在安装结束后重新启动（默认是 no）。
; PrivilegesRequiredOverridesAllowed：允许覆盖所需权限级别的条件。
; DisableDirPage：是否禁用目录选择页面（默认是 no）。
; DisableProgramGroupPage：是否禁用程序组选择页面（默认是 no）。
; CreateAppDir：是否创建应用程序目录（默认是 yes）。
; 用户界面和体验
; WizardStyle：向导风格（classic、modern）。
; WindowVisible：安装时窗口是否可见（默认是 yes）。
; ShowLanguageDialog：是否显示语言选择对话框（默认是 no）。
; ShowTasksTreeLines：是否显示任务树的线条（默认是 yes）。
; ShowUndisplayableLanguages：是否显示不可用的语言（默认是 no）。
; LanguageDetectionMethod：语言检测方法（uilanguage、locale）。
; 安全和压缩
; SignTool：签名工具的路径。
; SignToolParameters：签名工具的参数。
; SignedUninstaller：卸载程序是否签名（默认是 no）。
; UsePreviousAppDir：是否使用之前的应用程序目录（默认是 yes）。
; UsePreviousGroup：是否使用之前的程序组（默认是 yes）。
; UsePreviousTasks：是否使用之前的任务（默认是 yes）。
; UsePreviousSetupType：是否使用之前的安装类型（默认是 yes）。
; 日志和错误处理
; CreateUninstallRegKey：是否创建卸载注册表键（默认是 yes）。
; UninstallDisplayName：卸载程序在控制面板中的显示名称。
; UninstallFilesDir：卸载程序文件的目录。
; UninstallDisplayIcon：卸载程序显示的图标。
; UninstallRestartComputer：卸载后是否重新启动计算机（默认是 no）。
; OutputManifestFile：输出清单文件的路径。
; LogMode：日志模式（append、overwrite、new）。
[Languages] ;定义安装程序支持的语言。
Name: "english"; MessagesFile: "compiler:Default.isl"

[Dirs]
Name: "{app}\logs"

[Files]			;定义需要安装复制的文件。
; Source指源文件路径。DestDir指目标目录，{app}表示安装目录
; Flags设置文件复制选项：ignoreversion(忽略版本，NFS类似共享文件系统不能设置)，recursesubdirs(递归子目录)，createallsubdirs(0创建所有子目录)
Source: ".\{#MyAppExeName}"; DestDir: "{app}"; Flags: ignoreversion
Source: ".\{#MyJreName}\*"; DestDir: "{app}\{#MyJreName}"; Flags: ignoreversion recursesubdirs createallsubdirs

[Registry]		;定义需要添加的注册表项
; Root：根注册表项
; Subkey：子项。
; ValueType：值类型（如 string、dword)
; ValueName：值名称
; ValueData：值数据
; Flags：标志, uninsdeletekey 表示卸载时删除键。
Root: HKA; Subkey: "Software\Classes\{#MyAppAssocExt}\OpenWithProgids"; ValueType: string; ValueName: "{#MyAppAssocKey}"; ValueData: ""; Flags: uninsdeletevalue
Root: HKA; Subkey: "Software\Classes\{#MyAppAssocKey}"; ValueType: string; ValueName: ""; ValueData: "{#MyAppAssocName}"; Flags: uninsdeletekey
Root: HKA; Subkey: "Software\Classes\{#MyAppAssocKey}\DefaultIcon"; ValueType: string; ValueName: ""; ValueData: "{app}\{#MyAppExeName},0"
Root: HKA; Subkey: "Software\Classes\{#MyAppAssocKey}\shell\open\command"; ValueType: string; ValueName: ""; ValueData: """{app}\{#MyAppExeName}"" ""%1"""
Root: HKA; Subkey: "Software\Classes\Applications\{#MyAppExeName}\SupportedTypes"; ValueType: string; ValueName: ".myp"; ValueData: ""

[Icons]			;定义创建的快捷方式。
; Name设置快捷方式名称, {group} 表示开始菜单组， {commondesktop} 表示公共桌面。
; Filename设置快捷方式指向的文件
; Tasks设置用于条件创建快捷方式的任务名称。
Name: "{autoprograms}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"
Name: "{autodesktop}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"; Tasks: desktopicon

[Tasks] ;定义用户可以选择的任务。
Name: "desktopicon"; Description: "{cm:CreateDesktopIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags: unchecked

[Run]			;定义安装过程中和安装后的运行命令
; Filename设置要运行的文件路径。Description设置描述信息。
; Flags设置运行选项: 
;     runhidden: 隐藏运行程序
;     runminimized: 最小化运行程序
;     runmaximized: 最大化运行程序
;     waituntilterminated: 等待程序终止后再继续安装或卸载过程
;     nowait: 不等待程序终止，立即继续安装或卸载过程
;     postinstall: 仅在安装成功后运行程序
;     skipifdoesntexist: 如果程序文件不存在，则跳过运行
;     shellexec: 使用 ShellExecute API 运行程序（通常用于运行外部文件或 URL）
;     unchecked: 如果程序运行失败，不会显示错误消息
;     32bit: 在 32 位模式下运行程序（仅在 64 位 Windows 上可用）
;     64bit: 在 64 位模式下运行程序（仅在 64 位 Windows 上可用）
;     runascurrentuser: 用于在提升权限的安装程序中以当前用户身份运行程序，而不是以提升的管理员权限运行。
;     runasoriginaluser: 以原始用户身份运行程序（在提升权限的安装程序中常用）
Filename: "{app}\{#MyAppExeName}"; Description: "{cm:LaunchProgram,{#StringChange(MyAppName, '&', '&&')}}"; Flags: runascurrentuser nowait postinstall skipifsilent

[UninstallDelete]    ;指定在卸载时的操作
; Type为filesandordirs时则删除目录及其所有内容
; Type为dirifempty时则仅在目录为空时删除
Type: filesandordirs; Name: "{app}\logs"
```

命令行构建 exe 安装包

```bash
C:\"Program Files (x86)"\"Inno Setup 6"\ISCC.exe ~\Desktop\demoj2e.iss
```

# 附录一：Inno Setup配置内置变量

- **`{app}：`**目标安装目录
- **`{win}：`**Windows 目录（通常是 `C:\Windows`）
- **`{sys}：`**系统目录（通常是 `C:\Windows\System32`）
- **`{src}：`**当前安装源目录
- **`{pf}：`**程序文件目录（通常是 `C:\Program Files`）
- **`{autopf}:`**常量会自动根据安装程序的位数选择正确的路径。对于 32 位安装程序,将解析为 `C:\Program Files (x86)`。对于 64 位安装程序,将解析为 `C:\Program Files`
- **`{cf}：`**公共文件目录（通常是 `C:\Program Files\Common Files`）
- **`{sd}：`**系统驱动器（通常是 `C:\`）
- **`{tmp}：`**临时目录
- **`{localappdata}：`**本地应用数据目录（通常是 `C:\Users\<username>\AppData\Local`）
- **`{commonappdata}：`**所有用户的公共应用数据目录（通常是 `C:\ProgramData`）
- **`{userappdata}：`**当前用户的应用数据目录（通常是 `C:\Users\<username>\AppData\Roaming`）
- **`{userdocs}：`**当前用户的文档目录（通常是 `C:\Users\<username>\Documents`）
- **`{userdesktop}：`**当前用户的桌面目录（通常是 `C:\Users\<username>\Desktop`）
- **`{commondesktop}：`**所有用户的公共桌面目录（通常是 `C:\Users\Public\Desktop`）
- **`{userprograms}：`**当前用户的程序组目录（通常是 `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs`）
- **`{commonprograms}：`**所有用户的程序组目录（通常是 `C:\ProgramData\Microsoft\Windows\Start Menu\Programs`）
- **`{userstartmenu}：`**当前用户的开始菜单目录（通常是 `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu`）
- **`{commonstartmenu}：`**所有用户的开始菜单目录（通常是 `C:\ProgramData\Microsoft\Windows\Start Menu`）
- **`{userstartup}：`**当前用户的启动目录（通常是 `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`）
- **`{commonstartup}：`**所有用户的启动目录（通常是 `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup`）
- **`{fonts}：`**系统字体目录（通常是 `C:\Windows\Fonts`）
- **`{param}：`**安装命令行参数

# 参考

- https://blog.csdn.net/wangpaiblog/article/details/119658741
- https://blog.csdn.net/orangeTop/article/details/120287684
- https://www.cnblogs.com/diysoul/p/14778834.html
- https://www.ngui.cc/el/2852791.html?action=onClick
- https://github.com/spring-projects/spring-boot/issues/24889
- https://www.eolink.com/news/post/47095.html