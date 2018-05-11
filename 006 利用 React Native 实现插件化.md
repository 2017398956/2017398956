# Android 项目中使用 React Native 实现插件化 #

  关于 Android 原生项目中混编 React Native 的方法，网上已经有很多文章了，这里推荐一篇 [React Native嵌入Android原生项目中](https://blog.csdn.net/u011965040/article/details/53331859?locationNum=15&fps=1 "React Native嵌入Android原生项目中") 。不过，这样的混编有些许缺陷：
1. 原生项目过度依赖 React Native ；
2. Android Studio 对 JavaScript 支持不友好（可以尝试配置插件来解决）；
3. 原生代码和 js 代码混合在一块，不利于 web 端同事开发；
4. 项目比较大时调试不够轻便；