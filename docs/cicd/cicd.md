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

#### 自研产品
> 在实际搭建中，由于跟`rancher`对接并不是很友好，然后开发自研产品，作为`ci/cd`的支撑，该产品负责将`ci/cd`中涉及的工具都连接起来，是中间产品，并不是具体流程的发起点。

1. **ci申请：** 当用户在jira中发起ci申请时，会自动调用自研产品api，自研产品初始化涉及到的工具链
2. **开发代码发布：** jenkins调用自研产品api接口，该产品部署将该版本发布到rancher环境中
3. **发布申请：** jira调用自研产品第一个接口，自研产品发布到stg环境，并在stg环境测试成功后，通过第二个接口调用部署生产环境的信息，但并没有重启服务

#### rancher
> 最终的应用都是部署在rancher环境，由于catalog具有一次部署，多次使用的特性，最终我们选择将其以商店的方式部署。

而我们在进行设计的时候，需要以`rancher`最终部署app需要哪些因素来反推我们自研产品需要做什么。

比如，应用商店的信息是用`helm chart库(一组相关的kubnernets资源集合)`来描述的，所以我们自研程序就是要将需要部署的应用渲染成`helm chart`需要的文件模板，不同的版本是需要不同的`chart`模板的(升级版本变化的是镜像，而具体拉取哪个镜像是在chart中体现的)，**于是流程变成**：

自研产品渲染新的chart文件---->**推送到git**<-----rancher刷新git----upgrade最新版本

所以自研产品**对接rancher要做的事情就变很简单**：

自动发布流程：生成新版本的chart文件模块-----推送到git-----`call rancher refresh chart api`----`call rancher upgrade api`

chart模板格式如下：
```
charts/<APPLICATION>/<APP_VERSION>/
| --charts /            # 包含依赖的 Chart 的应用商店。
| --templates/          # 包含应用商店的模板，当与 values.yml 结合使用时，将生成 Kubernetes YAML。
| --app-readme.md       # 文本为显示在 Rancher UI 的 Chart 标题中。*
| --Chart.yml           # 必需的 Helm Chart 信息文件。
| --questions.yml       # 用于生成在 Rancher UI 中显示的应答表单。它们将显示在配置选项中。*
| --README.md           # 可选：在 Rancher UI 中显示的 Helm 自述文件。该文本显示在“详细描述”中。
| --requirements.yml    # 可选：YAML 文件列出了 Chart 的依赖关系。
| --values.yml          # Chart 的默认配置值。
```
核心是`templates`模板(存放的是k8s的yaml定义文件)，如`deployment.yaml`,`service.yaml` 

#### 自研产品关键流程
> 功能支持ansible(非容器化)，容器化项目(rancher,k8s)

1. appid申请(根据用户在jira中填写的信息完成初始化配置)
   - 配置git webhook，配置`develop`和`release`分支跟踪;
   - 配置git deploy key
   - 根据template在jenkins中同时创建`develop`和`release`的模板
   - 生成consult路径(服务注册，配置文件中心)
   - 生成rancher2 chart文件并提交到git(生成helm-templates文件)
2. 测试环境自动发布(开发提交代码到develop分支，自动触发jenkins build通过后调用接口)
   - 接收appmodel和version信息
   - 根据上述信息生成对应的chart文件，并提交git
   - 通知rancher去git中refresh chart文件，并调upgrade升级接口升级

#### 发布流程
1.push代码到release----2.自动触发jenkins release job-----3.自动触发jenkins sonar job-----------4.jira中创建发布申请-----ci测试通过-------5.部署到stg环境------部署成功-----6.stg测试/uat测试-----测试通过-----7.安全测试-----通过----8.将发布版本从stg拷贝到生产
