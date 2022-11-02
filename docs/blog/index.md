
<style>
  .md-sidebar--secondary:not([hidden]) {
    visibility: hidden;
  }
</style>

# Blog
## 网易数帆云原生日志平台架构实践

<aside style="font-size: .7rem" markdown>
<span>:octicons-person-16: [__ethfoo__] &nbsp;</span>
<span>
:octicons-calendar-24: 2022-03-11 &nbsp;
:octicons-clock-24: 8 min read 
</span>
</aside>

---

网易从2015年就开始了云原生的探索与实践，作为可观测性的重要一环，日志平台也经历了从主机到容器的演进，支撑了集团内各业务部门的大规模云原生化改造。  
本文会讲述在这个过程中我们遇到的问题，如何演进和改造，并从中沉淀了哪些经验与最佳实践。

主要内容包括：

- Operator化的⽇志采集
- ⼤规模场景下的困境与挑战
- 开源Loggie的现在与未来
  
  [:octicons-arrow-right-24: __Continue reading__](https://mp.weixin.qq.com/s/rtnBplSAdOzQmtYAtXrAqA)
  [__ethfoo__]: https://github.com/ethfoo
  [网易数帆云原生日志平台架构实践]: (https://mp.weixin.qq.com/s/rtnBplSAdOzQmtYAtXrAqA)


---
## 使用Loggie快速构建可扩展的云原生日志架构

<aside style="font-size: .7rem" markdown>
<span>:octicons-person-16: [__ethfoo__] &nbsp;</span>
<span>
:octicons-calendar-24: 2022-10-26 &nbsp;
:octicons-clock-24: 10 min read 
</span>
</aside>

---

主要内容包括：

- Loggie简介与核心设计思路
- Loggie在Kubernetes下采集日志的各种姿势
- 使用Loggie构建不同规模的日志系统架构
  
  [:octicons-arrow-right-24: __Continue reading__](https://github.com/loggie-io/assets/blob/main/ppt/%E4%B8%8B%E4%B8%80%E4%BB%A3%E4%BA%91%E5%8E%9F%E7%94%9F%E5%8F%AF%E8%A7%82%E6%B5%8B%E6%95%B0%E6%8D%AE%E9%87%87%E9%9B%86Agent%20Loggie.pdf)
  [__ethfoo__]: https://github.com/ethfoo
  [使用Loggie快速构建可扩展的云原生日志架构]: (https://github.com/loggie-io/assets/blob/main/ppt/%E4%B8%8B%E4%B8%80%E4%BB%A3%E4%BA%91%E5%8E%9F%E7%94%9F%E5%8F%AF%E8%A7%82%E6%B5%8B%E6%95%B0%E6%8D%AE%E9%87%87%E9%9B%86Agent%20Loggie.pdf)
