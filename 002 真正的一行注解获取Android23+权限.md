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

最后，欢迎测试使用该库，给个 star 就更好了^_^