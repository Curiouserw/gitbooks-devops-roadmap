# Dockerfile扫描工具Hadolint

# 一、简介

Hadolint是一个用Haskell 编写的开源Dockerfiles语法检查、优化工具。

Github地址：https://github.com/hadolint/hadolint

在线扫描：https://hadolint.github.io/hadolint/

# 二、安装与配置

## 1、安装

- **OSX brew** 

  ```bash
  brew install hadolint
  ```

- **Windows scoop**

  ```bash
  scoop install hadolint
  ```

- **docker**

  ```bash
  docker pull hadolint/hadolint:latest-debian
  docker pull hadolint/hadolint:latest-alpine
  ```

## 2、配置

### hadolint会按照以下顺序读取配置文件

- `$PWD/.hadolint.yaml`
- `$XDG_CONFIG_HOME/hadolint.yaml`
- `~/.config/hadolint.yaml`

### 扫描时指定配置文件

```bash
hadolint --config /path/to/config.yaml Dockerfile
```

### 配置忽略规则

```bash
echo "export XDG_CONFIG_HOME=~/.hadolint" >> /etc/profile && \
mkdir ~/.hadolint && \
source /etc/profile && \
```

创建并编辑配置`vi ~/.hadolint/hadolint.yaml`

```yaml
ignored:
  - DL3000
  - SC1010

trustedRegistries:
  - docker.io
  - my-company.com:5000
```

## 3、命令参数

```bash
hadolint [-v|--version] [-c|--config FILENAME] [-f|--format ARG] [DOCKERFILE...] 
         [--ignore RULECODE] [--trusted-registry REGISTRY (e.g. docker.io)]

可选参数:
  -h,--help                Show this help text
  -v,--version             Show version
  -c,--config FILENAME     Path to the configuration file
  -f,--format ARG          The output format for the results [tty | json |
                           checkstyle | codeclimate | codacy] (default: tty)
  --ignore RULECODE        A rule to ignore. If present, the ignore list in the
                           config file is ignored
  --trusted-registry REGISTRY (e.g. docker.io)
                           A docker registry to allow to appear in FROM instructions
```

# 三、扫描Dockerfile

```bash
hadolint Dockerfile 
hadolint --ignore DL3003 --ignore DL3006 Dockerfile
# 或者
docker run --rm -i hadolint/hadolint < Dockerfile
```

输出扫描结果

```bash
Dockerfile:2 DL3020 Use COPY instead of ADD for files and folders
Dockerfile:3 DL3025 Use arguments JSON notation for CMD and ENTRYPOINT arguments
```

# 四、与CI流程与编辑器的集成

官方集成示例文档：https://github.com/hadolint/hadolint/blob/master/docs/INTEGRATION.md

## 编辑器

- **VSCode**
- **Sublime Text 3**
- **Vim and NeoVim**
- **Atom**

## CI流程

### 1、Travis CI

```bash
# Use container-based infrastructure for quicker build start-up
sudo: false
# Use generic image to cut start-up time
language: generic
env:
  # Path to 'hadolint' binary
  HADOLINT: "${HOME}/hadolint"
install:
  # Download hadolint binary and set it as executable
  - curl -sL -o ${HADOLINT} "https://github.com/hadolint/hadolint/releases/download/v1.17.5/hadolint-$(uname -s)-$(uname -m)"
    && chmod 700 ${HADOLINT}
script:
  # List files which name starts with 'Dockerfile'
  # eg. Dockerfile, Dockerfile.build, etc.
  - git ls-files --exclude='Dockerfile*' --ignored | xargs --max-lines=1 ${HADOLINT}
```

### 2、GitHub Actions

```bash
name: Lint Dockerfile
on: push
jobs:
  linter:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Lint Dockerfile
        uses: brpaz/hadolint-action@master
        with:
          dockerfile: "Dockerfile"
```

### 3、Gitlab CI

```bash
lint_dockerfile:
  image: hadolint/hadolint:latest-debian
  script:
    - hadolint Dockerfile
```

### 4、Jenkins declarative pipeline

```bash
stage ("lint dockerfile") {
    agent {
        docker {
            image 'hadolint/hadolint:latest-debian'
        }
    }
    steps {
        sh 'hadolint dockerfiles/* | tee -a hadolint_lint.txt'
    }
    post {
        always {
            archiveArtifacts 'hadolint_lint.txt'
        }
    }
}
```

### 5、Jenkins K8S plugin

声明hadolint pod

```bash
- name: hadolint
  image: hadolint/hadolint:latest-debian
  imagePullPolicy: Always
  command:
    - cat
  tty: true
```

```bash
stage('lint dockerfile') {
    steps {
        container('hadolint') {
            sh 'hadolint dockerfiles/* | tee -a hadolint_lint.txt'
        }
    }
    post {
        always {
            archiveArtifacts 'hadolint_lint.txt'
        }
    }
}
```

### 6、Bitbucket Pipelines

```bash
pipelines:
  default:
    - step:
        image: hadolint/hadolint:latest-debian
        script:
          - hadolint Dockerfile
```



# 五、扫描规则

`DL开头`的规则是**hadolint**的

`SC开头`的规则是**ShellChecke**的

| Rule                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [DL3000](https://github.com/hadolint/hadolint/wiki/DL3000)   | Use absolute WORKDIR.                                        |
| [DL3001](https://github.com/hadolint/hadolint/wiki/DL3001)   | For some bash commands it makes no sense running them in a Docker container like ssh, vim, shutdown, service, ps, free, top, kill, mount, ifconfig. |
| [DL3002](https://github.com/hadolint/hadolint/wiki/DL3002)   | Last user should not be root.                                |
| [DL3003](https://github.com/hadolint/hadolint/wiki/DL3003)   | Use WORKDIR to switch to a directory.                        |
| [DL3004](https://github.com/hadolint/hadolint/wiki/DL3004)   | Do not use sudo as it leads to unpredictable behavior. Use a tool like gosu to enforce root. |
| [DL3005](https://github.com/hadolint/hadolint/wiki/DL3005)   | Do not use apt-get upgrade or dist-upgrade.                  |
| [DL3006](https://github.com/hadolint/hadolint/wiki/DL3006)   | Always tag the version of an image explicitly.               |
| [DL3007](https://github.com/hadolint/hadolint/wiki/DL3007)   | Using latest is prone to errors if the image will ever update. Pin the version explicitly to a release tag. |
| [DL3008](https://github.com/hadolint/hadolint/wiki/DL3008)   | Pin versions in apt-get install.                             |
| [DL3009](https://github.com/hadolint/hadolint/wiki/DL3009)   | Delete the apt-get lists after installing something.         |
| [DL3010](https://github.com/hadolint/hadolint/wiki/DL3010)   | Use ADD for extracting archives into an image.               |
| [DL3011](https://github.com/hadolint/hadolint/wiki/DL3011)   | Valid UNIX ports range from 0 to 65535.                      |
| [DL3012](https://github.com/hadolint/hadolint/wiki/DL3012)   | Provide an email address or URL as maintainer.               |
| [DL3013](https://github.com/hadolint/hadolint/wiki/DL3013)   | Pin versions in pip.                                         |
| [DL3014](https://github.com/hadolint/hadolint/wiki/DL3014)   | Use the `-y` switch.                                         |
| [DL3015](https://github.com/hadolint/hadolint/wiki/DL3015)   | Avoid additional packages by specifying --no-install-recommends. |
| [DL3016](https://github.com/hadolint/hadolint/wiki/DL3016)   | Pin versions in `npm`.                                       |
| [DL3017](https://github.com/hadolint/hadolint/wiki/DL3017)   | Do not use `apk upgrade`.                                    |
| [DL3018](https://github.com/hadolint/hadolint/wiki/DL3018)   | Pin versions in apk add. Instead of `apk add ` use `apk add =`. |
| [DL3019](https://github.com/hadolint/hadolint/wiki/DL3019)   | Use the `--no-cache` switch to avoid the need to use `--update` and remove `/var/cache/apk/*` when done installing packages. |
| [DL3020](https://github.com/hadolint/hadolint/wiki/DL3020)   | Use `COPY` instead of `ADD` for files and folders.           |
| [DL3021](https://github.com/hadolint/hadolint/wiki/DL3021)   | `COPY` with more than 2 arguments requires the last argument to end with `/` |
| [DL3022](https://github.com/hadolint/hadolint/wiki/DL3022)   | `COPY --from` should reference a previously defined `FROM` alias |
| [DL3023](https://github.com/hadolint/hadolint/wiki/DL3023)   | `COPY --from` cannot reference its own `FROM` alias          |
| [DL3024](https://github.com/hadolint/hadolint/wiki/DL3024)   | `FROM` aliases (stage names) must be unique                  |
| [DL3025](https://github.com/hadolint/hadolint/wiki/DL3025)   | Use arguments JSON notation for CMD and ENTRYPOINT arguments |
| [DL3026](https://github.com/hadolint/hadolint/wiki/DL3026)   | Use only an allowed registry in the FROM image               |
| [DL3027](https://github.com/hadolint/hadolint/wiki/DL3027)   | Do not use `apt` as it is meant to be a end-user tool, use `apt-get` or `apt-cache` instead |
| [DL3028](https://github.com/hadolint/hadolint/wiki/DL3028)   | Pin versions in gem install. Instead of `gem install ` use `gem install :` |
| [DL4000](https://github.com/hadolint/hadolint/wiki/DL4000)   | MAINTAINER is deprecated.                                    |
| [DL4001](https://github.com/hadolint/hadolint/wiki/DL4001)   | Either use Wget or Curl but not both.                        |
| [DL4003](https://github.com/hadolint/hadolint/wiki/DL4003)   | Multiple `CMD` instructions found.                           |
| [DL4004](https://github.com/hadolint/hadolint/wiki/DL4004)   | Multiple `ENTRYPOINT` instructions found.                    |
| [DL4005](https://github.com/hadolint/hadolint/wiki/DL4005)   | Use `SHELL` to change the default shell.                     |
| [DL4006](https://github.com/hadolint/hadolint/wiki/DL4006)   | Set the `SHELL` option -o pipefail before `RUN` with a pipe in it |
| [SC1000](https://github.com/koalaman/shellcheck/wiki/SC1000) | `$` is not used specially and should therefore be escaped.   |
| [SC1001](https://github.com/koalaman/shellcheck/wiki/SC1001) | This `\c` will be a regular `'c'` in this context.           |
| [SC1007](https://github.com/koalaman/shellcheck/wiki/SC1007) | Remove space after `=` if trying to assign a value (or for empty string, use `var='' ...`). |
| [SC1010](https://github.com/koalaman/shellcheck/wiki/SC1010) | Use semicolon or linefeed before `done` (or quote to make it literal). |
| [SC1018](https://github.com/koalaman/shellcheck/wiki/SC1018) | This is a unicode non-breaking space. Delete it and retype as space. |
| [SC1035](https://github.com/koalaman/shellcheck/wiki/SC1035) | You need a space here                                        |
| [SC1045](https://github.com/koalaman/shellcheck/wiki/SC1045) | It's not `foo &; bar`, just `foo & bar`.                     |
| [SC1065](https://github.com/koalaman/shellcheck/wiki/SC1065) | Trying to declare parameters? Don't. Use `()` and refer to params as `$1`, `$2` etc. |
| [SC1066](https://github.com/koalaman/shellcheck/wiki/SC1066) | Don't use $ on the left side of assignments.                 |
| [SC1068](https://github.com/koalaman/shellcheck/wiki/SC1068) | Don't put spaces around the `=` in assignments.              |
| [SC1077](https://github.com/koalaman/shellcheck/wiki/SC1077) | For command expansion, the tick should slant left (` vs ´).  |
| [SC1078](https://github.com/koalaman/shellcheck/wiki/SC1078) | Did you forget to close this double-quoted string?           |
| [SC1079](https://github.com/koalaman/shellcheck/wiki/SC1079) | This is actually an end quote, but due to next char, it looks suspect. |
| [SC1081](https://github.com/koalaman/shellcheck/wiki/SC1081) | Scripts are case sensitive. Use `if`, not `If`.              |
| [SC1083](https://github.com/koalaman/shellcheck/wiki/SC1083) | This `{/}` is literal. Check expression (missing `;/\n`?) or quote it. |
| [SC1086](https://github.com/koalaman/shellcheck/wiki/SC1086) | Don't use `$` on the iterator name in for loops.             |
| [SC1087](https://github.com/koalaman/shellcheck/wiki/SC1087) | Braces are required when expanding arrays, as in `${array[idx]}`. |
| [SC1095](https://github.com/koalaman/shellcheck/wiki/SC1095) | You need a space or linefeed between the function name and body. |
| [SC1097](https://github.com/koalaman/shellcheck/wiki/SC1097) | Unexpected `==`. For assignment, use `=`. For comparison, use `[ .. ]` or `[[ .. ]]`. |
| [SC1098](https://github.com/koalaman/shellcheck/wiki/SC1098) | Quote/escape special characters when using `eval`, e.g. `eval "a=(b)"`. |
| [SC1099](https://github.com/koalaman/shellcheck/wiki/SC1099) | You need a space before the `#`.                             |
| [SC2002](https://github.com/koalaman/shellcheck/wiki/SC2002) | Useless cat. Consider `cmd < file | ..` or `cmd file | ..` instead. |
| [SC2015](https://github.com/koalaman/shellcheck/wiki/SC2015) | Note that `A && B || C` is not if-then-else. C may run when A is true. |
| [SC2026](https://github.com/koalaman/shellcheck/wiki/SC2026) | This word is outside of quotes. Did you intend to 'nest '"'single quotes'"' instead'? |
| [SC2028](https://github.com/koalaman/shellcheck/wiki/SC2028) | `echo` won't expand escape sequences. Consider `printf`.     |
| [SC2035](https://github.com/koalaman/shellcheck/wiki/SC2035) | Use `./*glob*` or `-- *glob*` so names with dashes won't become options. |
| [SC2039](https://github.com/koalaman/shellcheck/wiki/SC2039) | In POSIX sh, something is undefined.                         |
| [SC2046](https://github.com/koalaman/shellcheck/wiki/SC2046) | Quote this to prevent word splitting                         |
| [SC2086](https://github.com/koalaman/shellcheck/wiki/SC2086) | Double quote to prevent globbing and word splitting.         |
| [SC2140](https://github.com/koalaman/shellcheck/wiki/SC2140) | Word is in the form `"A"B"C"` (B indicated). Did you mean `"ABC"` or `"A\"B\"C"`? |
| [SC2154](https://github.com/koalaman/shellcheck/wiki/SC2154) | var is referenced but not assigned.                          |
| [SC2155](https://github.com/koalaman/shellcheck/wiki/SC2155) | Declare and assign separately to avoid masking return values. |
| [SC2164](https://github.com/koalaman/shellcheck/wiki/SC2164) | Use `cd ... || exit` in case `cd` fails.                     |