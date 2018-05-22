# 常用 Maven 仓库地址 #

  在使用 Nexus 搭建自己的 Maven 仓库时，我们需要为一些公共的 Maven 仓库做镜像，这就需要这些仓库的真实地址了，下面是这些仓库的地址：


| 仓库名称 | 仓库地址 | gradle 引用方式 |
|---------|---------|------------    |
| jcenter | https://jcenter.bintray.com | jcenter() |
| mavenCentral | https://repo1.maven.org/maven2 或 http://central.maven.org/maven2/ （比较慢） | mavenCentral() |
| google | https://dl.google.com/dl/android/maven2/ | google() |
| 阿里云 | http://maven.aliyun.com/nexus/content/repositories/jcenter/ 或 http://maven.aliyun.com/nexus/content/groups/public/ | maven {url '阿里地址'} |

注： 
1. mavenCentral 官网地址 http://mvnrepository.com/ 在这里可以根据名称搜索相应的库和查看热门的库。
2. google 的 maven 仓库是不支持浏览的，如果你需要下载特定一个库，可以自行拼接这样的地址 https://dl.google.com/dl/android/maven2/com/android/databinding/compiler/3.0.1/compiler-3.0.1-sources.jar 来进行下载，其它 maven 仓库同理。
3. 当我们通过 gradle 下载过第三方库时，可以在 **gradle 的缓存目录** 找到它们，

		windows: C:\Users\用户名\.gradle\caches\modules-2\files-2.1
		OSX: /Users/用户名/.gradle/caches/modules-2/files-2.1

   如果你使用了 Android Studio 自带的 gradle 那么，xxx\android-studio\gradle\gradle-版本号\caches\modules-2\files-2.1 下也有大量库文件。
4. 由于国内网络环境的原因，我们可以通过下面方法选择最优 Maven 仓库：

		repositories {
    		// maven库
    		def aliMaven = "http://maven.aliyun.com/nexus/content/groups/public/"
    		def cMaven = "https://repo1.maven.org/maven2"
    		// 先从url中下载jar若没有找到，则在artifactUrls中寻找
		    maven {
		        url aliMaven
		        artifactUrls cMaven
		    }
		}