# java 文件重命名 renameTo(File dest) 失败原因探究 #

重命名文件的原理是什么？同一个盘下：只修改文件的引导路径？不同盘下会不会复制文件？



rename方法应该是异步的，线程不安全，所以上一个还没修改完成你就循环修改下一个了。要么就在循环里重新new一个file对象吧。当然这只是我的猜测。rename方法是不是异步的我也不知道。




从C盘到E盘失败了，从C盘到D盘成功了。因为我的电脑C、D两个盘是NTFS格式的，而E盘是FAT32格式的。所以从C到E就是上面文章所说的"file systems"不一样。从C到D由于同是NTFS分区，所以不存在这个问题，当然就成功了。

In the Unix'esque O/S's you cannot renameTo() across file systems. This behavior is different than the Unix "mv" command. When crossing file systems mv does a copy and delete which is what you'll have to do if this is the case. 
The same thing would happen on Windows if you tried to renameTo a different drive, i.e. C: -> D:


我还碰到过在大压力情况下在windows renameTo有一定概率失败的情况。
在linux操作系统上在不同盘符之间renameTo也会失败。典型的应用场景就是从本地硬盘renameTo到mount的硬盘上或文件系统上。


有时候要注意对文件的引用是否已经关闭了
假如原来的名字为a.txt,需要改名为b.txt,则有如下情况：

	1. a.txt存在且b.txt不存在的时候renameTo成功
	2. a.txt存在且b.txt存在的时候renameTo失败
	3. 当a.txt进行读的操作且未关闭输入流的时候renameTo失败
	4. 当a.txt进行写的操作且未关闭输出流的时候renameTo失败

下面的这个无法重命名：

         boolean renamed = false;
		 File renameToIndexPathFile;
		 String path = "D:/apache-tomcat-7.0.6/webapps/cc/index/advertset";
		 File indexPathFile = new File(path);
		 int count = 1;
		 renameToIndexPathFile = new File(path + "-copy" + (count++)); 
		 renamed = indexPathFile.renameTo(renameToIndexPathFile);		 
		 System.out.println(renamed);

如果把上面代码的path 间隔符改成下面就成功了：

	String path = "D:\\apache-tomcat-7.0.6\\webapps\\cc\\index\\advertset";  








