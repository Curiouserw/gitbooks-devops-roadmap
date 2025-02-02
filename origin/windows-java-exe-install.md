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

Github：https://github.com/jrsoftware/issrc

示例配置：https://github.com/jrsoftware/issrc/tree/main/Examples

文档：https://jrsoftware.org/ishelp/

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
; UninstallDisplayName：卸载程序在"程序和功能"中显示的名称。
; UninstallFilesDir：卸载程序文件的目录。
; UninstallDisplayIcon：卸载程序显示的图标。
; UninstallRestartComputer：卸载后是否重新启动计算机（默认是 no）。
; OutputManifestFile：输出清单文件的路径。
; LogMode：日志模式（append、overwrite、new）

[Languages] ;定义安装程序支持的语言。
Name: cn; MessagesFile: "compiler:Languages\ChineseSimplified.isl"
Name: en; MessagesFile: "compiler:Default.isl"

[Messages]
cn.BeveledLabel = 中文
en.BeveledLabel = 英文

[CustomMessages]
cn.msgInstallMySQL = 安装 MySQL
en.msgInstallMySQL = Install MySQL

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

# 附录一：Inno Setup安装MySQL示例

- 静默安装 MySQL 8
- 创建 MySQL 的系统服务
- 重置`root@localhost`密码

```ini
#define MyAppName                      "mysql-8.4.2"
#define MYSQL_SERVICE                  "MySQL842"
#define MYSQL_VERSION                  "8.4.2"
#define MYSQL_PORT                     "3306"
#define MYSQL_PASSWD                   "M1y2S3Q_4L"
#define MYSQL_PACKAGE                  "mysql-8.4.2-winx64"
#define MYSQL_INSTALL_DIR              "mysql-8.4.2"
#define MYSQL_DATA_DIR                 MYSQL_INSTALL_DIR+"Data"

[Setup]
AppId={{131E7842-****-****-****-0E00A8B6EB8E}}
AppName={#MyAppName}
AppVersion={#MYSQL_VERSION}
DefaultDirName={autopf}\{#MyAppName}
ChangesAssociations=yes
DisableProgramGroupPage=yes
OutputDir=.
OutputBaseFilename={#MyAppName}-installer
Compression=lzma
SolidCompression=yes
WizardStyle=modern
ShowTasksTreeLines=yes
PrivilegesRequired=admin

[Dirs]
Name: "{app}\{#MYSQL_INSTALL_DIR}\data"

[Languages]
Name: "english"; MessagesFile: "compiler:Default.isl"

[Tasks]
Name: "desktopicon"; Description: "{cm:CreateDesktopIcon}"; GroupDescription: "{cm:AdditionalIcons}"; Flags: unchecked

[Files]
Source: ".\{#MYSQL_PACKAGE}\*"; DestDir: "{app}\{#MYSQL_INSTALL_DIR}"; Flags: ignoreversion createallsubdirs recursesubdirs

[Run]
Filename: "{app}\{#MYSQL_INSTALL_DIR}\bin\mysqld.exe"; Parameters: " --initialize-insecure"; Flags: runascurrentuser runhidden
Filename: "{app}\{#MYSQL_INSTALL_DIR}\bin\mysqld.exe"; Parameters: " --install {#MYSQL_SERVICE}";  Flags: runascurrentuser runhidden
Filename: sc.exe; Parameters: " start {#MYSQL_SERVICE}"; AfterInstall: AfterInstallMySQL(); Flags: runascurrentuser runhidden

[UninstallDelete]
Type: filesandordirs; Name: "{app}\logs"

[Code]
function IsMySQLAppExists(): Boolean;
var
theMySQLApp: String;
begin
  theMySQLApp := ExpandConstant('{app}\{#MYSQL_INSTALL_DIR}') + '\bin\mysql.exe';
  Result := FileExists(theMySQLApp);
end;

procedure ResetMySQLPasswd();
var
theMySQLApp: String;
theMySQLParam: String;
theSQL: String;
theCommandParam: String;
theResultCode: Integer;
begin
  theMySQLApp := ExpandConstant('{app}\{#MYSQL_INSTALL_DIR}') + '\bin\mysql.exe';
  theMySQLParam := ' -P' + ExpandConstant('{#MYSQL_PORT}') + ' -uroot -e';
  theSQL := ' "alter user ''root''@''localhost'' identified by ''' + ExpandConstant('{#MYSQL_PASSWD}') + ''';"';
  theCommandParam := theMySQLParam + theSQL;
  if not Exec(theMySQLApp, theCommandParam, '', SW_HIDE, ewWaitUntilTerminated, theResultCode) then begin
    Log('reset passwd failed, result msg:' + SysErrorMessage(theResultCode));
  end;
end;

procedure AfterInstallMySQL();
begin
  if IsMySQLAppExists() then begin
    ResetMySQLPasswd();
  end;
end;
```



# 附录二：Inno Setup配置内置变量

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

# 附录三、简体中文语言文件

```ini
....
[Languages]
Name: cn; MessagesFile: "compiler:Languages\ChineseSimplified.isl"
Name: en; MessagesFile: "compiler:Default.isl"

[Messages]
cn.BeveledLabel = 中文
en.BeveledLabel = 英文

[CustomMessages]
cn.msgInstallMySQL = 安装 MySQL
en.msgInstallMySQL = Install MySQL

....
```

> Inno Setup安装目录\Languages\ChineseSimplified.isl

```ini
[LangOptions]
LanguageName=简体中文

LanguageID=$0804

LanguageCodePage=936
; If the language you are translating to requires special font faces or
; sizes, uncomment any of the following entries and change them accordingly.
;DialogFontName=
;DialogFontSize=8
;WelcomeFontName=Verdana
;WelcomeFontSize=12
;TitleFontName=Arial
;TitleFontSize=29
;CopyrightFontName=Arial
;CopyrightFontSize=8

[Messages]

; *** 应用程序标题
SetupAppTitle=安装
SetupWindowTitle=安装 - %1
UninstallAppTitle=卸载
UninstallAppFullTitle=%1 卸载

; *** Misc. common
InformationTitle=信息
ConfirmTitle=确认
ErrorTitle=错误

; *** SetupLdr messages
SetupLdrStartupMessage=现在将安装 %1。您想要继续吗？
LdrCannotCreateTemp=无法创建临时文件。安装程序已中止
LdrCannotExecTemp=无法执行临时目录中的文件。安装程序已中止
HelpTextNote=

; *** 启动错误消息
LastErrorMessage=%1。%n%n错误 %2: %3
SetupFileMissing=安装目录中缺少文件 %1。请修正这个问题或者获取程序的新副本。
SetupFileCorrupt=安装文件已损坏。请获取程序的新副本。
SetupFileCorruptOrWrongVer=安装文件已损坏，或是与这个安装程序的版本不兼容。请修正这个问题或获取新的程序副本。
InvalidParameter=无效的命令行参数：%n%n%1
SetupAlreadyRunning=安装程序正在运行。
WindowsVersionNotSupported=此程序不支持当前计算机运行的 Windows 版本。
WindowsServicePackRequired=此程序需要 %1 服务包 %2 或更高版本。
NotOnThisPlatform=此程序不能在 %1 上运行。
OnlyOnThisPlatform=此程序只能在 %1 上运行。
OnlyOnTheseArchitectures=此程序只能安装到为下列处理器架构设计的 Windows 版本中：%n%n%1
WinVersionTooLowError=此程序需要 %1 版本 %2 或更高。
WinVersionTooHighError=此程序不能安装于 %1 版本 %2 或更高。
AdminPrivilegesRequired=在安装此程序时您必须以管理员身份登录。
PowerUserPrivilegesRequired=在安装此程序时您必须以管理员身份或有权限的用户组身份登录。
SetupAppRunningError=安装程序发现 %1 当前正在运行。%n%n请先关闭正在运行的程序，然后点击“确定”继续，或点击“取消”退出。
UninstallAppRunningError=卸载程序发现 %1 当前正在运行。%n%n请先关闭正在运行的程序，然后点击“确定”继续，或点击“取消”退出。

; *** 启动问题
PrivilegesRequiredOverrideTitle=选择安装程序模式
PrivilegesRequiredOverrideInstruction=选择安装模式
PrivilegesRequiredOverrideText1=%1 可以为所有用户安装(需要管理员权限)，或仅为您安装。
PrivilegesRequiredOverrideText2=%1 只能为您安装，或为所有用户安装(需要管理员权限)。
PrivilegesRequiredOverrideAllUsers=为所有用户安装(&A)
PrivilegesRequiredOverrideAllUsersRecommended=为所有用户安装(&A) (建议选项)
PrivilegesRequiredOverrideCurrentUser=只为我安装(&M)
PrivilegesRequiredOverrideCurrentUserRecommended=只为我安装(&M) (建议选项)

; *** 其他错误
ErrorCreatingDir=安装程序无法创建目录“%1”
ErrorTooManyFilesInDir=无法在目录“%1”中创建文件，因为里面包含太多文件

; *** 安装程序公共消息
ExitSetupTitle=退出安装程序
ExitSetupMessage=安装程序尚未完成。如果现在退出，将不会安装该程序。%n%n您之后可以再次运行安装程序完成安装。%n%n现在退出安装程序吗？
AboutSetupMenuItem=关于安装程序(&A)...
AboutSetupTitle=关于安装程序
AboutSetupMessage=%1 版本 %2%n%3%n%n%1 主页：%n%4
AboutSetupNote=
TranslatorNote=简体中文翻译由Kira(847320916@qq.com)维护。项目地址：https://github.com/kira-96/Inno-Setup-Chinese-Simplified-Translation

; *** 按钮
ButtonBack=< 上一步(&B)
ButtonNext=下一步(&N) >
ButtonInstall=安装(&I)
ButtonOK=确定
ButtonCancel=取消
ButtonYes=是(&Y)
ButtonYesToAll=全是(&A)
ButtonNo=否(&N)
ButtonNoToAll=全否(&O)
ButtonFinish=完成(&F)
ButtonBrowse=浏览(&B)...
ButtonWizardBrowse=浏览(&R)...
ButtonNewFolder=新建文件夹(&M)

; *** “选择语言”对话框消息
SelectLanguageTitle=选择安装语言
SelectLanguageLabel=选择安装时使用的语言。

; *** 公共向导文字
ClickNext=点击“下一步”继续，或点击“取消”退出安装程序。
BeveledLabel=
BrowseDialogTitle=浏览文件夹
BrowseDialogLabel=在下面的列表中选择一个文件夹，然后点击“确定”。
NewFolderName=新建文件夹

; *** “欢迎”向导页
WelcomeLabel1=欢迎使用 [name] 安装向导
WelcomeLabel2=现在将安装 [name/ver] 到您的电脑中。%n%n建议您在继续安装前关闭所有其他应用程序。

; *** “密码”向导页
WizardPassword=密码
PasswordLabel1=这个安装程序有密码保护。
PasswordLabel3=请输入密码，然后点击“下一步”继续。密码区分大小写。
PasswordEditLabel=密码(&P)：
IncorrectPassword=您输入的密码不正确，请重新输入。

; *** “许可协议”向导页
WizardLicense=许可协议
LicenseLabel=请在继续安装前阅读以下重要信息。
LicenseLabel3=请仔细阅读下列许可协议。在继续安装前您必须同意这些协议条款。
LicenseAccepted=我同意此协议(&A)
LicenseNotAccepted=我不同意此协议(&D)

; *** “信息”向导页
WizardInfoBefore=信息
InfoBeforeLabel=请在继续安装前阅读以下重要信息。
InfoBeforeClickLabel=准备好继续安装后，点击“下一步”。
WizardInfoAfter=信息
InfoAfterLabel=请在继续安装前阅读以下重要信息。
InfoAfterClickLabel=准备好继续安装后，点击“下一步”。

; *** “用户信息”向导页
WizardUserInfo=用户信息
UserInfoDesc=请输入您的信息。
UserInfoName=用户名(&U)：
UserInfoOrg=组织(&O)：
UserInfoSerial=序列号(&S)：
UserInfoNameRequired=您必须输入用户名。

; *** “选择目标目录”向导页
WizardSelectDir=选择目标位置
SelectDirDesc=您想将 [name] 安装在哪里？
SelectDirLabel3=安装程序将安装 [name] 到下面的文件夹中。
SelectDirBrowseLabel=点击“下一步”继续。如果您想选择其他文件夹，点击“浏览”。
DiskSpaceGBLabel=至少需要有 [gb] GB 的可用磁盘空间。
DiskSpaceMBLabel=至少需要有 [mb] MB 的可用磁盘空间。
CannotInstallToNetworkDrive=安装程序无法安装到一个网络驱动器。
CannotInstallToUNCPath=安装程序无法安装到一个 UNC 路径。
InvalidPath=您必须输入一个带驱动器卷标的完整路径，例如：%n%nC:\APP%n%n或UNC路径：%n%n\\server\share
InvalidDrive=您选定的驱动器或 UNC 共享不存在或不能访问。请选择其他位置。
DiskSpaceWarningTitle=磁盘空间不足
DiskSpaceWarning=安装程序至少需要 %1 KB 的可用空间才能安装，但选定驱动器只有 %2 KB 的可用空间。%n%n您一定要继续吗？
DirNameTooLong=文件夹名称或路径太长。
InvalidDirName=文件夹名称无效。
BadDirName32=文件夹名称不能包含下列任何字符：%n%n%1
DirExistsTitle=文件夹已存在
DirExists=文件夹：%n%n%1%n%n已经存在。您一定要安装到这个文件夹中吗？
DirDoesntExistTitle=文件夹不存在
DirDoesntExist=文件夹：%n%n%1%n%n不存在。您想要创建此文件夹吗？

; *** “选择组件”向导页
WizardSelectComponents=选择组件
SelectComponentsDesc=您想安装哪些程序组件？
SelectComponentsLabel2=选中您想安装的组件；取消您不想安装的组件。然后点击“下一步”继续。
FullInstallation=完全安装
; if possible don't translate 'Compact' as 'Minimal' (I mean 'Minimal' in your language)
CompactInstallation=简洁安装
CustomInstallation=自定义安装
NoUninstallWarningTitle=组件已存在
NoUninstallWarning=安装程序检测到下列组件已安装在您的电脑中：%n%n%1%n%n取消选中这些组件不会卸载它们。%n%n确定要继续吗？
ComponentSize1=%1 KB
ComponentSize2=%1 MB
ComponentsDiskSpaceGBLabel=当前选择的组件需要至少 [gb] GB 的磁盘空间。
ComponentsDiskSpaceMBLabel=当前选择的组件需要至少 [mb] MB 的磁盘空间。

; *** “选择附加任务”向导页
WizardSelectTasks=选择附加任务
SelectTasksDesc=您想要安装程序执行哪些附加任务？
SelectTasksLabel2=选择您想要安装程序在安装 [name] 时执行的附加任务，然后点击“下一步”。

; *** “选择开始菜单文件夹”向导页
WizardSelectProgramGroup=选择开始菜单文件夹
SelectStartMenuFolderDesc=安装程序应该在哪里放置程序的快捷方式？
SelectStartMenuFolderLabel3=安装程序将在下列“开始”菜单文件夹中创建程序的快捷方式。
SelectStartMenuFolderBrowseLabel=点击“下一步”继续。如果您想选择其他文件夹，点击“浏览”。
MustEnterGroupName=您必须输入一个文件夹名。
GroupNameTooLong=文件夹名或路径太长。
InvalidGroupName=无效的文件夹名字。
BadGroupName=文件夹名不能包含下列任何字符：%n%n%1
NoProgramGroupCheck2=不创建开始菜单文件夹(&D)

; *** “准备安装”向导页
WizardReady=准备安装
ReadyLabel1=安装程序准备就绪，现在可以开始安装 [name] 到您的电脑。
ReadyLabel2a=点击“安装”继续此安装程序。如果您想重新考虑或修改任何设置，点击“上一步”。
ReadyLabel2b=点击“安装”继续此安装程序。
ReadyMemoUserInfo=用户信息：
ReadyMemoDir=目标位置：
ReadyMemoType=安装类型：
ReadyMemoComponents=已选择组件：
ReadyMemoGroup=开始菜单文件夹：
ReadyMemoTasks=附加任务：

; *** TDownloadWizardPage wizard page and DownloadTemporaryFile
DownloadingLabel=正在下载附加文件...
ButtonStopDownload=停止下载(&S)
StopDownload=您确定要停止下载吗？
ErrorDownloadAborted=下载已中止
ErrorDownloadFailed=下载失败：%1 %2
ErrorDownloadSizeFailed=获取下载大小失败：%1 %2
ErrorFileHash1=校验文件哈希失败：%1
ErrorFileHash2=无效的文件哈希：预期 %1，实际 %2
ErrorProgress=无效的进度：%1 / %2
ErrorFileSize=文件大小错误：预期 %1，实际 %2

; *** “正在准备安装”向导页
WizardPreparing=正在准备安装
PreparingDesc=安装程序正在准备安装 [name] 到您的电脑。
PreviousInstallNotCompleted=先前的程序安装或卸载未完成，您需要重启您的电脑以完成。%n%n在重启电脑后，再次运行安装程序以完成 [name] 的安装。
CannotContinue=安装程序不能继续。请点击“取消”退出。
ApplicationsFound=以下应用程序正在使用将由安装程序更新的文件。建议您允许安装程序自动关闭这些应用程序。
ApplicationsFound2=以下应用程序正在使用将由安装程序更新的文件。建议您允许安装程序自动关闭这些应用程序。安装完成后，安装程序将尝试重新启动这些应用程序。
CloseApplications=自动关闭应用程序(&A)
DontCloseApplications=不要关闭应用程序(&D)
ErrorCloseApplications=安装程序无法自动关闭所有应用程序。建议您在继续之前，关闭所有在使用需要由安装程序更新的文件的应用程序。
PrepareToInstallNeedsRestart=安装程序必须重启您的计算机。计算机重启后，请再次运行安装程序以完成 [name] 的安装。%n%n是否立即重新启动？

; *** “正在安装”向导页
WizardInstalling=正在安装
InstallingLabel=安装程序正在安装 [name] 到您的电脑，请稍候。

; *** “安装完成”向导页
FinishedHeadingLabel=[name] 安装完成
FinishedLabelNoIcons=安装程序已在您的电脑中安装了 [name]。
FinishedLabel=安装程序已在您的电脑中安装了 [name]。您可以通过已安装的快捷方式运行此应用程序。
ClickFinish=点击“完成”退出安装程序。
FinishedRestartLabel=为完成 [name] 的安装，安装程序必须重新启动您的电脑。要立即重启吗？
FinishedRestartMessage=为完成 [name] 的安装，安装程序必须重新启动您的电脑。%n%n要立即重启吗？
ShowReadmeCheck=是，我想查阅自述文件
YesRadio=是，立即重启电脑(&Y)
NoRadio=否，稍后重启电脑(&N)
; used for example as 'Run MyProg.exe'
RunEntryExec=运行 %1
; used for example as 'View Readme.txt'
RunEntryShellExec=查阅 %1

; *** “安装程序需要下一张磁盘”提示
ChangeDiskTitle=安装程序需要下一张磁盘
SelectDiskLabel2=请插入磁盘 %1 并点击“确定”。%n%n如果这个磁盘中的文件可以在下列文件夹之外的文件夹中找到，请输入正确的路径或点击“浏览”。
PathLabel=路径(&P)：
FileNotInDir2=“%2”中找不到文件“%1”。请插入正确的磁盘或选择其他文件夹。
SelectDirectoryLabel=请指定下一张磁盘的位置。

; *** 安装状态消息
SetupAborted=安装程序未完成安装。%n%n请修正这个问题并重新运行安装程序。
AbortRetryIgnoreSelectAction=选择操作
AbortRetryIgnoreRetry=重试(&T)
AbortRetryIgnoreIgnore=忽略错误并继续(&I)
AbortRetryIgnoreCancel=关闭安装程序

; *** 安装状态消息
StatusClosingApplications=正在关闭应用程序...
StatusCreateDirs=正在创建目录...
StatusExtractFiles=正在解压缩文件...
StatusCreateIcons=正在创建快捷方式...
StatusCreateIniEntries=正在创建 INI 条目...
StatusCreateRegistryEntries=正在创建注册表条目...
StatusRegisterFiles=正在注册文件...
StatusSavingUninstall=正在保存卸载信息...
StatusRunProgram=正在完成安装...
StatusRestartingApplications=正在重启应用程序...
StatusRollback=正在撤销更改...

; *** 其他错误
ErrorInternal2=内部错误：%1
ErrorFunctionFailedNoCode=%1 失败
ErrorFunctionFailed=%1 失败；错误代码 %2
ErrorFunctionFailedWithMessage=%1 失败；错误代码 %2.%n%3
ErrorExecutingProgram=无法执行文件：%n%1

; *** 注册表错误
ErrorRegOpenKey=打开注册表项时出错：%n%1\%2
ErrorRegCreateKey=创建注册表项时出错：%n%1\%2
ErrorRegWriteKey=写入注册表项时出错：%n%1\%2

; *** INI 错误
ErrorIniEntry=在文件“%1”中创建 INI 条目时出错。

; *** 文件复制错误
FileAbortRetryIgnoreSkipNotRecommended=跳过此文件(&S) (不推荐)
FileAbortRetryIgnoreIgnoreNotRecommended=忽略错误并继续(&I) (不推荐)
SourceIsCorrupted=源文件已损坏
SourceDoesntExist=源文件“%1”不存在
ExistingFileReadOnly2=无法替换现有文件，它是只读的。
ExistingFileReadOnlyRetry=移除只读属性并重试(&R)
ExistingFileReadOnlyKeepExisting=保留现有文件(&K)
ErrorReadingExistingDest=尝试读取现有文件时出错：
FileExistsSelectAction=选择操作
FileExists2=文件已经存在。
FileExistsOverwriteExisting=覆盖已存在的文件(&O)
FileExistsKeepExisting=保留现有的文件(&K)
FileExistsOverwriteOrKeepAll=为所有冲突文件执行此操作(&D)
ExistingFileNewerSelectAction=选择操作
ExistingFileNewer2=现有的文件比安装程序将要安装的文件还要新。
ExistingFileNewerOverwriteExisting=覆盖已存在的文件(&O)
ExistingFileNewerKeepExisting=保留现有的文件(&K) (推荐)
ExistingFileNewerOverwriteOrKeepAll=为所有冲突文件执行此操作(&D)
ErrorChangingAttr=尝试更改下列现有文件的属性时出错：
ErrorCreatingTemp=尝试在目标目录创建文件时出错：
ErrorReadingSource=尝试读取下列源文件时出错：
ErrorCopying=尝试复制下列文件时出错：
ErrorReplacingExistingFile=尝试替换现有文件时出错：
ErrorRestartReplace=重启并替换失败：
ErrorRenamingTemp=尝试重命名下列目标目录中的一个文件时出错：
ErrorRegisterServer=无法注册 DLL/OCX：%1
ErrorRegSvr32Failed=RegSvr32 失败；退出代码 %1
ErrorRegisterTypeLib=无法注册类库：%1

; *** 卸载显示名字标记
; used for example as 'My Program (32-bit)'
UninstallDisplayNameMark=%1 (%2)
; used for example as 'My Program (32-bit, All users)'
UninstallDisplayNameMarks=%1 (%2, %3)
UninstallDisplayNameMark32Bit=32 位
UninstallDisplayNameMark64Bit=64 位
UninstallDisplayNameMarkAllUsers=所有用户
UninstallDisplayNameMarkCurrentUser=当前用户

; *** 安装后错误
ErrorOpeningReadme=尝试打开自述文件时出错。
ErrorRestartingComputer=安装程序无法重启电脑，请手动重启。

; *** 卸载消息
UninstallNotFound=文件“%1”不存在。无法卸载。
UninstallOpenError=文件“%1”不能被打开。无法卸载。
UninstallUnsupportedVer=此版本的卸载程序无法识别卸载日志文件“%1”的格式。无法卸载
UninstallUnknownEntry=卸载日志中遇到一个未知条目 (%1)
ConfirmUninstall=您确认要完全移除 %1 及其所有组件吗？
UninstallOnlyOnWin64=仅允许在 64 位 Windows 中卸载此程序。
OnlyAdminCanUninstall=仅使用管理员权限的用户能完成此卸载。
UninstallStatusLabel=正在从您的电脑中移除 %1，请稍候。
UninstalledAll=已顺利从您的电脑中移除 %1。
UninstalledMost=%1 卸载完成。%n%n有部分内容未能被删除，但您可以手动删除它们。
UninstalledAndNeedsRestart=为完成 %1 的卸载，需要重启您的电脑。%n%n立即重启电脑吗？
UninstallDataCorrupted=文件“%1”已损坏。无法卸载

; *** 卸载状态消息
ConfirmDeleteSharedFileTitle=删除共享的文件吗？
ConfirmDeleteSharedFile2=系统表示下列共享的文件已不有其他程序使用。您希望卸载程序删除这些共享的文件吗？%n%n如果删除这些文件，但仍有程序在使用这些文件，则这些程序可能出现异常。如果您不能确定，请选择“否”，在系统中保留这些文件以免引发问题。
SharedFileNameLabel=文件名：
SharedFileLocationLabel=位置：
WizardUninstalling=卸载状态
StatusUninstalling=正在卸载 %1...

; *** Shutdown block reasons
ShutdownBlockReasonInstallingApp=正在安装 %1。
ShutdownBlockReasonUninstallingApp=正在卸载 %1。

; The custom messages below aren't used by Setup itself, but if you make
; use of them in your scripts, you'll want to translate them.

[CustomMessages]

NameAndVersion=%1 版本 %2
AdditionalIcons=附加快捷方式：
CreateDesktopIcon=创建桌面快捷方式(&D)
CreateQuickLaunchIcon=创建快速启动栏快捷方式(&Q)
ProgramOnTheWeb=%1 网站
UninstallProgram=卸载 %1
LaunchProgram=运行 %1
AssocFileExtension=将 %2 文件扩展名与 %1 建立关联(&A)
AssocingFileExtension=正在将 %2 文件扩展名与 %1 建立关联...
AutoStartProgramGroupDescription=启动：
AutoStartProgram=自动启动 %1
AddonHostProgramNotFound=您选择的文件夹中无法找到 %1。%n%n您要继续吗？
```



# 参考

- https://blog.csdn.net/wangpaiblog/article/details/119658741
- https://blog.csdn.net/orangeTop/article/details/120287684
- https://www.cnblogs.com/diysoul/p/14778834.html
- https://www.ngui.cc/el/2852791.html?action=onClick
- https://github.com/spring-projects/spring-boot/issues/24889
- https://www.eolink.com/news/post/47095.html
- https://www.cnblogs.com/diysoul/p/14778834.html
- https://github.com/linkease/Inno-Setup-Chinese-Simplified-Translation