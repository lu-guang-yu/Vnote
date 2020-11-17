# makefile规则

## 目录搜寻
### 一般搜索（变量VPATH）
通过变量VPATH可以指定依赖文件的搜索路径
### 选择性搜索（关键字vpath）
它的使用方法有三种：
1、vpath PATTERN DIRECTORIES
为所有符合模式“PATTERN”的文件指定搜索目录“DIRECTORIES”。
2、vpath PATTERN
清除之前为符合模式“PATTERN”的文件设置的搜索路径。
3、vpath
清除所有已被设置的文件搜索路径。
注：PATTERN的通配符为%