# 真正的一行注解获取Android23+权限 重要更新 接入优化

## 前言

由于部分读者反映 [AbcPermission](https://github.com/2017398956/AbcPermission "AbcPermission") 的接入方式太麻烦，利用周末时间我写了个 gradle 插件用于简化接入流程。这里提供了 2 套接入方案：一为最简便的；另一个支持版本自定义（如果你一直用最新版，这个可以忽略）。[AbcPermission](https://github.com/2017398956/AbcPermission "AbcPermission") 的概念 [这里](http://blog.csdn.net/niuzhucedenglu/article/details/78707302 "AbcPermission") 有详细的讲解。[这里是一次重要的升级](http://blog.csdn.net/niuzhucedenglu/article/details/78842723)，也可以直接看 [AbcPermission](https://github.com/2017398956/AbcPermission "AbcPermission") 的 readme 文件。

## 方案一

### 1.在根目录下的 build 文件中添加如下代码

	buildscript {
	    repositories {
	       ...
	       maven { url 'https://jitpack.io' }
	    }
	    dependencies {
	        ...
	        classpath 'com.github.2017398956:abcpermission-plugin:1.3'
	    }
	}

	allprojects {
	    repositories {
	        ...
	        maven { url 'https://jitpack.io' }
	    }
	}

### 2.在需要申请权限的 module 中添加如下代码

	apply plugin: 'abcpermission.plugin'

## 方案二

### 1.在根目录下的 build 文件中添加如下代码

	buildscript {
	    repositories {
	       ...
	       maven { url 'https://jitpack.io' }
	    }
	    dependencies {
	        ...
	        classpath 'com.github.2017398956:AspectPlugin:1.0'
	    }
	}

	allprojects {
	    repositories {
	        ...
	        maven { url 'https://jitpack.io' }
	    }
	}

### 2.在需要申请权限的 module 中添加如下代码

	apply plugin: 'AspectPlugin'

	dependencies {
	    ...
	    api("com.github.2017398956:AbcPermission:1.6") {
	        exclude module: 'permissionAnnotation'
	        exclude module: 'permissionCompiler'
	    }
	    provided("com.github.2017398956:AbcPermission:1.6") {
	        exclude module: 'permissionSupport'
	        exclude module: 'permissionCompiler'
	    }
	    annotationProcessor("com.github.2017398956:AbcPermission:1.6") {
	        exclude module: 'permissionSupport'
	    }
	}

使用的具体方法可以参考本文中的其它链接。
