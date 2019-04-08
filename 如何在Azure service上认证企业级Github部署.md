## 如何在Azure service上认证企业级Github部署

在聊如何认证企业级Github部署前，我们先聊一聊如何认证个人Github部署：

1. 在应用的内部版本菜单栏选择使用持续部署将更新自动发布到Azure:

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040801.png)

2. 进入部署中心，选择Github存储库(需要登录),点击继续：

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040802.png)

3. 选择应用服务Kudu生成服务器，点击继续：

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040803.png)

4. 选择好存储库及相应分支，点击继续：

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040804.png)

5. 部署重心的配置信息将被展示，点击完成部署设置：

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040805.png)

6. 部署成功后，相关信息将被展示，且每被触发一次将对多一条部署记录：

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040806.png)

好了，认证个人Github部署就完成了，基本就是将Azure与Github连通就可以了，不需要太多其他的步骤。不过，因为我司的SSO机制，当将以上步骤应用到部署时是不可行的（会在第二步登录不了）。下面我们来聊聊如何才能认证企业级Github部署：

1. 获取 Deployment Trigger URL 并设置成Github仓库的webhook(当代码有更新时用来通知外部服务):

    1. 点击应用All App service settings菜单，在Properties栏下找到Deployment Trigger URL的值，复制下来:
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040809.png)
    
    2. 在仓库的设置里找到添加webhook的地方，将Deployment Trigger URL添加到Payload URL栏，其他栏均为默认:
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040810.png)
    
    3. 点击Add webhooks,三个branch添加了三个hooks:
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019040811.png)

2. 生成Github access token并应用到部署中心的连接配置中：

    1. 在Github的账号Setting里选择Person access tokens，点击Generate new token：
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408113.png)
    
    2. 添加token的描述，并选择需要的权限，可全部勾选，点击Generate token后生成token，请记录下来：
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408114.png)
    
    3. 在应用的部署中心选择External，点击继续：
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408112.png)
    
    4. 代码存储库的连接地址需要添加token认证：
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408115.png)
    
    5. 填写分支并点击继续，完成部署。
    
3. 此时的部署依然会Fail，从Log里可以看出是认证问题。搜遍Google，在[Continuous Deployment From GitHub Enterprise Repository to Azure Web App](https://nsamteladze.wordpress.com/2015/07/19/continuous-deployment-from-github-enterprise-repository-to-azure-web-app/)文章里提到了如果我们想要Azure从Github拉取代码，还需要添加应用的SSH key作为Deploy key：

    1. Azure应用生成SSH key:
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408116.png)
    
    将上图中红框中部分替换为 `api/sshkey?ensurePublicKey=1` 则该应用的SSH key即会被展示到网页上：
     
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408117.png)
    
    2. 将SSH key应用到Github的Deploy keys项内，三个分支则添加了三个Deploy keys:
    
    ![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408118.png)
    
4. 尝试提交代码到Github，此时即可在应用的内部版本里看到我们构建成功的history:

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/20190408119.png)