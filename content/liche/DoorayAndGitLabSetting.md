# Dooray & GitLab 关联设置

[Dooray](https://dooray.com)是一个与其它所有工具连接，且简单易懂的业务工具。

[GitLab](https://gitlab.p2shop.cn:8443)是代码托管的工具（公司自己创建了GitLab网站）。

Dooray也是一个聊天工具，有客户端和web端

​	-web端地址：https://dooray.com/messenger/orgs

​	-客户端下载地址：https://dooray.com

1. **注册Dooray,GitLab,（Dooray推荐下载载客户端）**

   ​

2. **登陆Dooray账户，并创建业务群。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray001.png)

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray002.png)

3. **登陆https://pangpang.dooray.com 网站,点击右上脚的纸轮->设置->联动服务。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray003.png)

   ​

4. **点击添加服务->选择GitLab添加服务。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray004.png)

5. **选择要关联的项目和群，复制URL地址，下一步再Gitlab设置中用到。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray005.png)

6. **登陆GitLab并创建项目，点击settings->Integrations，填写上面复制的URL，勾选需要的Trigger，并保存。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray006.png)

7. **点击test测试。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray007.png)

8. **Dooray客户端接收到消息，到这一步就配置成功了～。**

   ![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray008.png)

9. **当你你通过git，push，merge等操作时，[Dooray messenger](https://dooray.com/messenger/orgs)会把信息发给群，群用户可以看到响应信息。**

   ​

   ## 代码commit 时输入语句

- push commit时message内容输入`fix#my-dooray-project/1234` ，my-dooray-project业务的序号1234中会记录push内容。

- 除了fix还可以用以下的单词。

  ```
  해결, 완료, fix,fixed,  
  close, closes, closed  
  ressolve, resolves, resolved
  ```


- Dooray的业务内容中所记录内容如下。

![](http://web-static-files.oss-cn-shanghai.aliyuncs.com/Share_Images/dooray009.png)















