# Android 查看各个 app 的 UID 

UID：一般理解为 User Identifier 。 UID 在 Linux 中就是用户的 ID （还有 Group ID），表明是哪个用户运行了这个程序，主要用于权限的管理。而在android中有所不同，Android 中每个程序都有一个 UID 。默认情况下，Android 会给每个程序分配一个普通级别互不相同的 UID，如果用互相调用，只能是 UID 相同才行，这就使得共享数据具有一定安全性，每个软件之间是不能随意获得数据的。
 
Android 的应用的 UID 是从 10000 开始，可以在 Process.java 中查看到（FIRST_APPLICATION_UID和LAST_APPLICATION_UID），由于 UID 是应用安装时确认的。我们可以从源码看到 UID 的产生（Settings.java）









///////////////////////////////////////////////

Pid是进程ID，Uid是用户ID，只是Android和计算机不一样，计算机每个用户都具有一个Uid，哪个用户start的程序，这个程序的Uid就是那个用户，而Android中每个程序都有一个Uid，默认情况下，Android会给每个程序分配一个普通级别互不相同的 Uid，如果用互相调用，只能是Uid相同才行，这就使得共享数据具有了一定安全性，每个软件之间是不能随意获得数据的。而同一个application 只有一个Uid，所以application下的Activity之间不存在访问权限的问题。

    Android系统中修改了Linux的UID的含义：用来唯一确定某个用户的身份。由于Android是单用户系统，不需要支持多用户登陆。Android的UID的含义：每个APP对应一个UID——用UID对应用程序进行管理。

 Android中查看UID的方式：

               data/system/packages.list


代码：

ActivityManager am = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
		ApplicationInfo appinfo = getApplicationInfo();
		List<RunningAppProcessInfo> run = am.getRunningAppProcesses();
		for (RunningAppProcessInfo runningProcess : run) {
			if ((runningProcess.processName != null) && runningProcess.processName.equals(appinfo.processName)) {
				uid = String.valueOf(runningProcess.uid);
				break;
			}
		}
    PID即进程ID。

     查看： ps|grep XXX

    每一个不同的程序都能有一个UId，但是一个应用里面可以有多个PId