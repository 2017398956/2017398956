# 真正的一行注解处理Android23+权限
关于 Android 23 及其以上版本的权限获取库，github 上有很多比较优秀的开源项目，如：RxPermission，AndPermission，PermissionGen 等。似乎我们不需要再纠结 23+ 上的权限问题了，但是（开始转折）这些开源项目的权限处理都可以用下面的流程图表示：

![](/images/002_01.png)

图一

正如上图所示这些开源项目无论用什么技术封装的都需要我们实现红色框中的逻辑，然后才能执行我们的“业务3”。而我们更想要的流程是下图：

![](/images/002_02.png)

图二

即将图一红框中的权限处理逻辑内化为 业务3 的一种能力而不需要我们对其处理，为了更直观的表现这一点，图二可以简化为下图：

![](/images/002_03.png)

图三

这样权限请求框架就不会侵入我们的业务逻辑，编写代码的时候我们只需要关心 业务3 怎么实现就行，而不需要处理权限问题，将权限申请和业务逻辑解耦。所以，本文的重点来了：[https://github.com/2017398956/AbcPermission](https://github.com/2017398956/AbcPermission "AbcPermission") 这个项目就实现了图三的逻辑，让我们把重心放在业务的实现上而不用管理权限。下面以一个读取联系人的权限为例来简单说明下该库的使用方式：

	@GetPermissions({Manifest.permission.READ_CONTACTS})
    private void readContacts() {
        Toast.makeText(this, "readContacts", Toast.LENGTH_SHORT).show();
    }

    public void onClick(View view) {
        readContacts();
    }

正如你看到的只需要加在方法前加一行注解即可，不需要再进行其他处理（像打开设置界面的操作都可以由项目自己完成并可自行配置），是不是非常简便，而且对业务无侵入。关于该项目的具体使用方法这里不再赘述，可以查看该项目的 readme 文件。

**兼容性**

关于兼用性由于没有相应手机可供测试，这里以 [AndPermission](https://github.com/yanzhenjie/AndPermission/blob/master/README-CN.md#%E5%9B%BD%E4%BA%A7%E6%89%8B%E6%9C%BA%E9%80%82%E9%85%8D%E6%96%B9%E6%A1%88) 提到的几种情况作为适配依据，以下粗体字为我的适配方法。

1. 部分中国厂商生产手机（例如小米某型号）的Rationale功能，在第一次拒绝后，第二次申请时不会返回true，并且会回调申请失败，也就是说在第一次决绝后默认勾选了不再提示。**AbcPermission 有自己的判断逻辑，所以这一点已经和正常的逻辑统一了，只需要在permissionListener.cannotRequestAgain(...); 中处理即可。**

2. 部分中国厂商生产手机（例如小米、华为某型号）在申请权限时，用户点击确定授权后，还是回调我们申请失败，这个时候其实我们是拥有权限的。**由于 AbcPermission 没有使用回调机制，所以这一点不会造成影响。**

3. 部分中国厂商生产手机在系统Setting中设置**[禁用/询问]**某权限，但是在申请此权限时却直接提示有权限，这可能是厂商故意这样设计的，当我们真正执行需要这个权限代码的时候系统会自动申请权限。**既然需要权限的代码可以自动申请权限，而 AbcPermission 在得知有权限时会直接执行相应代码，所以对我们没影响**

4. 部分中国厂商生产手机（例如vivo、pppo某型号）在用户允许权限，并且回调了权限授权成功的方法，但是实际执行代码时并没有这个权限。**由于没权限，当用户执行到这块代码的时候 app 会在 permissionListener.exeException(...) 抛出异常；此时我们在这里判断异常信息为缺权限时，提示用户打开设置界面即可，并且 permissionListener 是全局的，只需要维护一份代码即可。**

5. 部分开发者反馈，在某些手机的Setting中授权后，检查时还是没有权限，执行响应的代码时应用崩溃（错误提示是没有权限），这种手机真的兼容不到了，我也觉得没必要兼容了，建议直接放弃这种平台。**在 Nexus6P Android 8.0.0 版本发现在使用 "android.permission.SYSTEM_ALERT_WINDOW" 和或"android.permission.WRITE_SETTINGS" 时出现了该问题，估计是新系统 bug （2017-12-08），执行到需要这些权限的方法，AbcPermission 会一直执行你在 AbcPermission.permissionListener.cannotRequestAgain(...) 设定的逻辑（如弹框让用户在设置界面授予权限），考虑到 8.0.0 在国内占有量不大，用户体验还可以接受**

**AbcPermission 相较于其它权限库的优势**

1. **权限和业务分离**
1. **23 以下的版本适配到 23+ 以上非常方便：**当你的项目时间比较长，现在要从 Android 23 以下的版本适配到 Android 23 以上时，为了防止因为权限的问题而 crash 往往在 app 启动后立刻申请一系列的权限，如果用户不授予权限则不让用户使用（很多 app 都这么做的）；这样处理虽然简便，但十分粗暴，严重影响用户体验。如果对使用到特殊权限的代码进行逐一修改来适配 Android 23+ 工作量大不说，还要小心处理业务逻辑，代码改动越大越容易出错，升级成本过大。如果使用 AbcPermission 则可以直接在这些方法上加一句权限申请的注解即可，不用动以前的逻辑；如果原业务逻辑不在你自己写的方法中，可以对其抽离成一个方法，也十分方便。


最后，欢迎测试使用该库，给个 star 就更好了^_^