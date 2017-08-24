在Gralde中创建自定义插件，Gradle提供了三种方式:

1. 在build.gradle脚本中直接使用
2. 在buildSrc文件夹中使用
3. 在独立Module中使用

注意事项:

* 通过buildSrc方式自定义插件，使用apply plugin方法导入时候，不能加单引号，而且类要是完整的包路径