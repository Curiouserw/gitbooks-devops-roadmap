# Git Hook钩子

# 一、简介

官方文档：https://git-scm.com/docs/githooks/en

官方中文文档：https://git-scm.com/book/zh/v2

Hooks(钩子),是一些存放于`$GIT_DIR/hooks`文件夹的小脚本,在特定条件下触发动作。当执行`git init`，几个示例hook将复制到新资源库的hooks文件夹, 但默认情况下他们都是禁用状态，要启用一个hook(钩子),请移除其`.sample`后缀.

# 二、Hook类型



https://imweb.io/topic/5b13aa38d4c96b9b1b4c4e9d

http://www.360doc.com/content/16/1014/18/10058718_598431211.shtml

https://blog.51cto.com/u_13187574/2083938

- **applypatch-msg**
  - 触发时机：由`git am`触发
  - 接受的参数：
    - 提交的commit msg临时文件路径
  - 可通过`--no-verify` 略过
  - 如果以非0状态退出，那么`git am`将在patch(补丁)应用之前取消.
- **pre-commit**
  - 触发时机：由`git commit`触发, 在`commit msg`被创建之前执行. 
  - 不接受参数
  - 可通过`--no-verify` 略过
  - 如果以非0状态退出，将导致`git commit`被取消
- **post-commit**
  - 触发时机：由`git commit`触发, 当commit完成后执行
  - 不接受参数



| 触发端   | 类型               | 触发时机                                                     | 接受参数                                                     |      |
| -------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| Client端 | pre-merge-commit   |                                                              |                                                              |      |
| Client端 | prepare-commit-msg | 由'git commit',在准备好默认log信息后触发,但此时,编辑器尚未启动 | 它可能接受1到3个参数. 第一个参数是包含commit msg的文件路径. 第二个参数是commit msg的来源, 可能的值有:   `message` (当使用`-m` 或`-F` 选项);  `template` (当使用`-t` 选项,或`commit.template`配置项已经被设置);  `merge` (当commit是一个merge或者`.git/MERGE_MSG`存在);   `squash`(当`.git/SQUASH_MSG`文件存在);  `commit`, 且附带该commit的SHA1 (当使用`-c`, `-C` 或 `--amend`). | 是   |
| Client端 | commit-msg         |                                                              |                                                              | 是   |
| Client端 | post-commit        | 由`git commit`触发.  当commit完成后执行                      | 不接受参数                                                   |      |
| Client端 |                    |                                                              |                                                              |      |
| Server端 | pre-receive        |                                                              |                                                              |      |
| Server端 | update             |                                                              |                                                              |      |
| Server端 | post-receive       |                                                              | 不接受参数                                                   |      |
| Server端 |                    |                                                              |                                                              |      |
| Server端 |                    |                                                              |                                                              |      |
| Server端 |                    |                                                              |                                                              |      |
| Server端 |                    |                                                              |                                                              |      |
| Server端 |                    |                                                              |                                                              |      |
| Server端 |                    |                                                              |                                                              |      |
| Server端 |                    |                                                              |                                                              |      |



# 三、编译工具集成

## 1、NPM  Husky

```bash
npm install -g husky 
```



## 2、Maven

文档：https://blog.csdn.net/leandzgc/article/details/107974276

## 3、Composer composer-git-hooks

文档：https://packagist.org/packages/brainmaestro/composer-git-hooks

```bash
composer global require brainmaestro/composer-git-hooks
```

`composer.json`

```json
{
  	".....",
    "extra": {
        "hooks": {
						"config": {
              	// 配置执行失败时停止（当钩子是一系列命令时，当命令失败时停止执行会很有用）
                "stop-on-failure": ["pre-push"],
              	// 配置自定义hook
               	"custom-hooks": ["pre-flow-feature-start"]
            },
						"pre-commit": [
                "echo committing as $(git config user.name)",
                "php-cs-fixer fix ." // fix style
            ],
          	"pre-flow-feature-start": [
                "echo 'Starting a new feature...'"
            ],
            // verify commit message. ex: ABC-123: Fix everything
            "commit-msg": "grep -q '[A-Z]+-[0-9]+.*' $1",
            "pre-push": [
                "php-cs-fixer fix --dry-run ." // check style
                "phpunit"
            ],
            "post-merge": "composer install"
            "...": "..."
        }
    }
}
```

