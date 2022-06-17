# 版本和Release发布流程

## 版本发布涉及工程
- Loggie 主工程
- Loggie-installation 部署脚本工程
- Loggie docs 文档项目（暂时为最新的版本文档，后续会支持多版本）

## 代码分支策略
- 版本参考[Semantic语义](https://semver.org/)
- 使用main主分支为开发分支，日常所有代码PR均合入main分支
- 新版本使用release分支，格式为`release-vA.B`，例如`release-v1.1`，这里只使用版本前两位
- 基于release分支打上tag表示具体版本，例如`v1.1.0`
- bugfix 除了肯定会合入main分支外，根据严重情况cherry-pick至具体的release分支，同时打上tag，需要升级最小版本，例如`v1.1.1`

## 发布新版本步骤

#### 1. 创建新的release分支

确认版本需要的所有功能已经合入，main分支为最新提交：

``` bash
git checkout main
git pull upstream main -r
```

基于main分支创建新的release分支，例如`release-v1.1`：
```bash
git checkout -b release-v${A.B}
```

#### 2. 填写CHANGELOG
基于该分支填写CHANGELOG，提交至工程中合入

```bash
git add .
git commit -m'Release: add v${A.B.C} changelog'
git push upstream release-v${A.B.C}
```

#### 3. release Loggie-installation
release Loggie-installationation部署脚本工程，和上述1，2步骤相同

请注意：

- 请勿忘记修改Chart.yaml中的版本号

#### 4. 回归测试
基于release分支进行回归测试。  
release分支push至Github后，会触发action进行镜像构建，使用该镜像进行回归测试。  
这里的测试会使用loggie-installationation部署脚本里相应的release分支，同时回归验证部署脚本正确性。  
如果测试发现Bug，提交PR合入release分支，重新进行回归。  

#### 5. 打上版本tag

测试通过后，基于该release分支打上相应的版本tag，比如`v1.1.0`
```bash
git pull upstream release-v${A.B.C}
git tag v${A.B.C}
git push v${A.B.C}
```
注意Loggie工程和Loggie-installationation部署脚本均需要。

#### 6. 在Github上发布release

- Loggie工程复制项目中的CHANGELOG即可
- Loggie installation工程需要提供`loggie-linux-amd64`二进制和`loggie-v${A.B.C}.tgz` helm chart包。
  - 二进制需要找台linux amd64机器进行`make build`
  - helm chart包需要基于对应版本的loggie installation项目，执行`helm package ./helm-chart`即可。

#### 7. 更新Loggie docs文档

- 更新Loggie docs文档，并提PR合入
- 注意修改Loggie镜像和版本号等


#### 8. 将release分支合入main
提PR将release分支合入main，可等稳定一段时间后再合入，注意Loggie工程和Loggie-installation部署脚本均需要。


## 有BugFix时

- 如果是不重要的bug，基于main分支提交修改即可，可以不合入release分支
- 如果为重要的bug，需要修复，确定需要修复的版本release分支（可为最近一个或者多个），除了提交至main分支外，还需要cherry-pick至指定release分支，同时打上tag，新增最小版本号，同时需要确认Loggie-installation和docs是否同步修改。