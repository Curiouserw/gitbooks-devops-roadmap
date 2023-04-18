# Gitlab CI/CD Pipeline

# 一、简介

- **Stages**
  - 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始。
  - 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
  - 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败
  - 有`.pre`、`.post`和`default`
- **Jobs**：构建任务，表示某个 Stage 里面执行的工作
  - 相同 Stage 中的 Jobs 会并行执行
  - 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
  - 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 失败

# 二、Stage定义及语法



# 三、Job定义及语法



## 变量variable

## 脚本script

## image





