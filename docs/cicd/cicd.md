#### 概述
> ci/cd通过构建持续发布、持续集成的工作流，实现自动化和敏捷开发部署，旨在提高研发效能。

#### 系统架构

这是CI/CD的一般流程：
![](https://files.mdnice.com/user/4251/1c5102f8-c827-41bc-8a0d-6074e7da8ab7.png)

假如整个流程发布到生产不需要审批的话，按如上流程部署就行。

**但实际上，很多企业发布都需要审批，只有通过之后才能上生产环境，这需要另外一个组件jira(流程管理)**

#### 功能
> 我们以开发的日常工作角度来设计这个流程，分为**开发代码流程**和**发布代码流程**
1. 开发代码流程，开发提交代码到dev分支就自动编译走流程并发布到CI。
   - `gitlab`、`webhook`、`jenkins`联动，参考链接[Gitlab+jenkins自动构建指定分支](https://www.rootop.org/pages/4140.html)，[CICD | Jenkins & Gitlab集成:WebHook触发构建](https://bbs.huaweicloud.com/blogs/173837)
   - `gitlab`、`webhook`、`jenkins`之间的关联的配置自动化完成，让用户在`jira`中申请相应的应用模块时（用户填写应用相关的元数据信息），就自动生成。
2. 发布代码流程，功能开发完毕后提交到**发布分支**后，自动触发jenkins流程但不发布到生产环境，开发需要在jira中提发布流程，批准后开始自动发布生产。
    - 将确定发布的代码，`push`到`release`分支，自动触发jenkins流程，打包镜像文件并上传到`docker hub`，这里并没有发布到生产环境；
    - 开发人员在`jira`创建**发布申请**，jira中构建发布工作流，最终调用k8s的api接口进行发布；
    - jira发布申请工作流，测试---发布stg环境---部署生产环境---等待再次审批(jira自定义工作流，并通过网络钩子的方式访问外部系统)

在该方案的设计中，**发起点只有两个**：
- 用户提交代码到具体分支
- 用户在jira中申请对应工单（ci申请、发布申请、上产线按钮）