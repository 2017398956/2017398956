# 一行注解处理Android23+权限 AbcPermission 重要更新

关于 [AbcPermission](https://github.com/2017398956/AbcPermission "AbcPermission") 的概念 [这里](http://blog.csdn.net/niuzhucedenglu/article/details/78707302 "AbcPermission") 有详细的讲解。但是在以前的版本中我们都不能对用户授权后进行回调，虽然用户只在初次授权的时候会多一次操作，但对用户体验影响不大。为了更加优化用户体检，从 V1.6 以后 [AbcPermission](https://github.com/2017398956/AbcPermission "AbcPermission") 支持对 **无返回值的方法** 进行回调，并且其回调流程已统一整合在  AbcPerpermission.permissionListener 。请放心，这次升级完全兼容 V1.6 之前的版本。

## ABCPermission V1.6 新特性的使用方法

1. 首先按照 [这里](http://blog.csdn.net/niuzhucedenglu/article/details/78707302 "AbcPermission") 配置好所需的环境；

2. V1.6 之前的版本获取权限时使用的是 @GetPermissions({...}) ,现在你可以放弃这种用法了，将其改为  @GetPermissionsAuto({...}) 这样就可以对 **无返回值的方法** 在进行申请权限时进行回调；对于 **有返回值的方法** ，会走以前的逻辑，请放心升级。
3. 回调分为以下三类：1.用户点击 **同意授权** ，这种不需要你处理；      2.用户点击 **拒绝授权** ， 用户点击后会触发 AbcPerpermission.permissionListener.showRequestPermissionRationale(final Permission23Fragment permission23Fragment, final String[] permissions) 你可以在这里根据权限信息弹出相应的提示，当用户点击确定后 通过 **permission23Fragment.requestPermissions(permissions);** 再次申请权限，注意，这里**不要**使用 Fragment#requestPermissions(String[], int) 申请权限；      3.用户点击 **拒绝授权且不再提示** ，*如果你已经接入了 V1.6 之前的版本，那么，不需要做任何处理，就是这么省心 ^_^，*如果是第一次接入你可以在 ABCPermission.permissionListener.cannotRequestAgain(...) 中进行弹窗提示和打开设置界面，详细内容请移步 [这里](http://blog.csdn.net/niuzhucedenglu/article/details/78707302 "AbcPermission") 。

## 使用示例
1.申请权限只需要一行代码
	
	@GetPermissionsAuto({Manifest.permission.WRITE_EXTERNAL_STORAGE})
    private void readFile() {
        ...
    }

2.回调的处理，这里只以 用户点击 **拒绝授权** 为例

	@RequiresApi(api = Build.VERSION_CODES.M)
        public void showRequestPermissionRationale(final Permission23Fragment permission23Fragment, final String[] permissions) {
            StringBuffer stringBuffer = new StringBuffer();
            for (String permission : permissions) {
                stringBuffer.append(permission);
                stringBuffer.append("\n");
            }
            AlertDialog.Builder builder = new AlertDialog.Builder(permission23Fragment.getActivity()).setTitle("权限申请")
                    .setMessage(stringBuffer.toString())
                    .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
							// 用户看到需要权限的理由，同意后再次申请
                            permission23Fragment.requestPermissions(permissions);
                        }
                    })
                    .setNegativeButton("取消", null);
            builder.create().show();

        }

最后，欢迎测试使用该库，如果你觉得还不错，请给个 [Star](https://github.com/2017398956/AbcPermission "AbcPermission") (。・`ω´・)