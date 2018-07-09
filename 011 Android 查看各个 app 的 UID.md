# Android 查看各个 app 的 UID 及 sharedUserId 的使用

## 一、UID 的概念及查看方式 ##
### 概念 ###
UID：一般理解为 User Identifier 。 UID 在 Linux 中就是用户的 ID （还有 Group ID），表明是哪个用户运行了这个程序，主要用于权限的管理。而在android中有所不同，Android 中每个程序都有一个 UID 。默认情况下，Android 会给每个程序分配一个普通级别互不相同的 UID，如果用互相调用，只能是 UID 相同才行，这就使得共享数据具有一定安全性，每个软件之间是不能随意获得数据的，而同一个 application 只有一个 UID ，所以 application 下的 Activity 之间不存在访问权限的问题。
 
Android 的应用的 UID 是从 10000 开始，可以在 Process.java 中查看到 **FIRST\_APPLICATION\_UID** 和 **LAST\_APPLICATION\_UID** ：

	// android.os.Process

    /**
     * Defines the start of a range of UIDs (and GIDs), going from this
     * number to {@link #LAST_APPLICATION_UID} that are reserved for assigning
     * to applications.
     */
    public static final int FIRST_APPLICATION_UID = 10000;

    /**
     * Last of application-specific UIDs starting at
     * {@link #FIRST_APPLICATION_UID}.
     */
    public static final int LAST_APPLICATION_UID = 19999;


由于 UID 是应用安装时确认的，下面我们看一下 UID 的生成逻辑:

	// com.android.server.pm.Settings

    // Returns -1 if we could not find an available UserId to assign
    private int newUserIdLPw(Object obj) {
        // Let's be stupidly inefficient for now...
        final int N = mUserIds.size();
        for (int i = mFirstAvailableUid; i < N; i++) {
            if (mUserIds.get(i) == null) {
                mUserIds.set(i, obj);
                return Process.FIRST_APPLICATION_UID + i;
            }
        }

        // None left?
        if (N > (Process.LAST_APPLICATION_UID-Process.FIRST_APPLICATION_UID)) {
            return -1;
        }

        mUserIds.add(obj);
        return Process.FIRST_APPLICATION_UID + N;
    }

既然我们知道了 UID 的生成逻辑，那么如何查看一个 app 的 UID 呢？

### 查看方式 ###
#### 1.通过 shell 查看 ####
终端输入 adb shell 然后输入 ps ，可以看到如下图所示的进程列表（为了方便展示，省略了很多）：

	USER     PID   PPID  VSIZE  RSS     WCHAN    PC        NAME
	root      1     0     9240   668   c01b950c 0806daa0 S /init
	root      2     0     0      0     c013aac6 00000000 S kthreadd
	root      3     2     0      0     c0128fe4 00000000 S ksoftirqd/0
									.
									.
									.
	root      63    1     11964  1380  c01d9a68 b76c1435 S /system/bin/lmkd
	system    64    1     10120  720   c0401967 b7760196 S /system/bin/servicemanager
	drm       72    1     28296  4456  ffffffff b7701196 S /system/bin/drmserver
	media     73    1     106416 18424 ffffffff b75c0196 S /system/bin/mediaserver
	install   74    1     10136  744   c04dc88e b7685eb6 S /system/bin/installd
	keystore  75    1     14108  1980  c0401967 b7615196 S /system/bin/keystore
	media_rw  77    1     16020  740   ffffffff b76bfeb6 S /system/bin/sdcard
	wifi      544   1     14864  2924  c01b950c b7558bb5 S /system/bin/wpa_supplicant
	radio     632   76    1535440 40916 ffffffff b7560435 S com.android.phone
	dhcp      791   1     10048  972   c01b950c b770e0c0 S /system/bin/dhcpcd
	system    994   76    1521900 28416 ffffffff b7560435 S com.android.tools
	u0_a45    1016  76    1553492 54048 ffffffff b7560435 S com.tencent.mtt.x86:service
	u0_a45    1118  76    1548416 53588 ffffffff b7560435 S com.tencent.mtt.x86

可以看到进程列表中有很多类型的 USER ，其中 u0\_axxx 代表着应用程序的用户，且每个应用程序的 u0\_axxx 都不一样，由于应用程序的 UID 是从 10000 开始的，所以 u0\_a 后面的数字加上 10000 所得的值就是 UID 了，例如最后一行 QQ 浏览器的 UID 就是 10045 。

#### 2.通过 app 获得所有已安装应用的 UID ####

        PackageManager packageManager = context.getPackageManager();
        List<PackageInfo> packageInfoList = packageManager.getInstalledPackages(PackageManager.GET_PERMISSIONS);
        for (PackageInfo info : packageInfoList) {
            Log.d(TAG, "app:" + info.applicationInfo.loadLabel(packageManager).toString()
                    + " uid:" + info.applicationInfo.uid
                    + " className:" + info.applicationInfo.className
            );
        }

#### 3.通过应用 PID ，查看对应 app 的 UID ####
PID 是进程 ID，每一个不同的程序都能有一个 UID ，但是一个应用里面可以有多个 PID

	终端中输入 adb shell ，然后输入 cat /proc/<pid>/status 

#### 4.通过 packages.xml ，查看需要查询的 app 的 UID ####

	// 这个命令会输出很多信息，需要再次筛选
	终端中输入 adb shell，然后输入cat /data/system/packages.xml
	// 例如查看 QQ 浏览器的信息：
	// cat /data/system/packages.xml|grep tenc

从中我们可以找到 QQ 浏览器 的信息：

	 // userId 为 10045 ，所以 方法1 的猜测时对的
	 <package name="com.tencent.mtt.x86" codePath="/data/app/com.tencent.mtt.x86-1" 
		nativeLibraryPath="/data/app/com.tencent.mtt.x86-1/lib" primaryCpuAbi="x86" 
		flags="1588804" ft="160077eb2c0" it="160077eb6c8" ut="160077eb6c8" version="611740" 
		userId="10045">
	......
    
## 二、sharedUserId 的使用 ##
    