# 1.NodeJS的版本问题
  ```
  package.json --- 人类 写的是项目需要的 版本 react": "^17.0.2"  当你install的时候，如果网上有 17.0.2⬆️的版本，但不超过17，就会自动更新， 比如 17.8.8 ，他其实会按照17.8.8去安装 对应的 环境。
  
  然后 npm会自动生成一个package-lock ， 把17.8.8锁死。如果存在 该packagpackagee-lock，下次安装的话 就不会变动 ，只会package-lock里的按照17.8.8来装。
  
  
  
  1.15.2对应就是MAJOR,MINOR.PATCH：1是marjor version；15是minor version；2是patch version。

MAJOR：这个版本号变化了表示有了一个不可以和上个版本兼容的大更改。

MINOR：这个版本号变化了表示有了增加了新的功能，并且可以向后兼容。

PATCH：这个版本号变化了表示修复了bug，并且可以向后兼容。

因为major version变化表示可能会影响之前版本的兼容性，所以无论是波浪符号还是插入符号都不会自动去修改major version，因为这可能导致程序crush，可能需要手动修改代码。
————————————————
版权声明：本文为CSDN博主「nj物是人非」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/njweiyukun/article/details/70309066
  
  
  ```
