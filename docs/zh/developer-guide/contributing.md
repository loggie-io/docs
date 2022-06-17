# 代码贡献

!!! note

    如果你：
    
    - 有新的想法
    - 提交Feature
    - Bug report/fix
    - 贡献文档
    - Help wanted
    
    建议先提[Issues](https://github.com/loggie-io/loggie/issues)，描述你的目的或者问题。  

## 代码/文档贡献流程

### 1. 创建本地工程
#### fork代码
访问[https://github.com/loggie-io/loggie](https://github.com/loggie-io/loggie)，点击右上角的Fork，fork到自己的github仓库中。

#### clone到本地
将fork后的工程clone到本地：
```bash
cd ${go-workspace}

git clone git@github.com:${yourAccountId}/loggie.git
```

### 2. 本地开发

#### add upstream
在本地工程中，添加upstream remote：

```bash
git remote add upstream https://github.com/loggie-io/loggie.git
git remote set-url --push upstream no_push
```

可以使用`git remote -v`查看，origin为你个人账号fork的git地址，upstream为Loggie官方仓库地址，uptream仅为本地同步代码使用，正常无push权限。

#### create branch
基于最新的main分支，创建出你的分支，通常以feat/fix/chore等开头。

```bash
git checkout -b feat-mywork
```

在该分支开发完后，待提交前，建议保持代码和上游一致：

```bash
git pull upstream -r
```

#### 提交代码
```bash
git commit

git push origin feat-mywork
```

!!! caution "Commit规范"
    通常我们使用`<type>(<scope>): <subject>`的形式：

    **`<type>:`**（必须）

    - feat：表示一个新的功能feature
    - fix：bug的修复
    - chore：构建过程或者辅助工具的变动
    - test：测试相关 
    - docs：修改了文档
    - perf：性能优化相关改动
    - refactor：代码重构

    **`<scope>:`**（可选）

    - core：Loggie框架的核心代码改动
    - source：Loggie source部分功能新增或改动
    - sink：Loggie sink部分功能新增或改动
    - interceptor：Loggie interceptor部分功能新增或改动
    - discovery：服务发现和配置中心部分
    - monitor：monitor eventbus部分

    **`<subject>:`** （必须）即commit的内容描述


### 3. 创建PR

#### 提交至Github
访问你的个人github账号里fork的Loggie项目，并`Open pull request`。  
尽量描述清楚PR的背景、目的和改动点。  
如果有相关Issues，需关联。  

提交前请检查：

- [ ] 是否需要添加/修改相关unit test/e2e/性能测试
- [ ] 是否需要添加/修改相关文档

#### 寻求Merge
你可以在OWNERS找人，或者默认添加loggie-robot账号作为你PR的Reviewers或Assigneees。