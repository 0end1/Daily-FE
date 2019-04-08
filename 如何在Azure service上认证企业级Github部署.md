## 如何在Azure service上认证企业级Github部署

在聊如何认证企业级Github部署前，我们先聊一聊如何认证私人Github部署：

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