# 利用 gradle 上传本地库到MAVEN服务器脚本 #

1. 上传到本地MAVEN仓库的gradle命令，在project层的build.gradle中添加如下代码：

		apply plugin: 'maven'		
		group='com.xxx'
		version='1.0.0'
		
		uploadArchives {
		    repositories {
		        mavenDeployer {
		            repository(url: uri('../repo'))
		        }
		    }
		}

然后在 android studio 右侧栏找到upload/uploadArchives 点击就可以上传成功了。上传成功后，会在当前project目录下生产一个repo目录，里面就是maven仓库的配置信息。

2.上传到本地私服MAVEN仓库： 需要使用nexus 先在本地搭建maven服务器

	apply plugin: 'maven'
	
	uploadArchives {
	    repositories {
	        mavenDeployer {
	            repository(url: 'http://localhost:8081/repository/maven-my-hosted/') {
	                authentication(userName: 'admin', password: 'admin123')
	            }
	            // 以下可以省略，会使用默认配置
	            // pom.version  = "0.54.2"  //版本号
	            // pom.artifactId = "react-native"  //"maven的坐标号"   可以和包名相同
	            // pom.groupId = "com.facebook.react"  //"maven的分组"
	            // pom.name = "react-native"    //"包名称"
	            // pom.packaging = "aar"
	        }
	    }
	}

同样在在project层的build.gradle中添加如上代码，也会生成gradle命令，直接点击运行就好了。