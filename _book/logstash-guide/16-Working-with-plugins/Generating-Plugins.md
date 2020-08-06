## 生成插件

您现在可以在几秒钟内创建自己的Logstash插件！ `bin/logstash-plugin` 的generate子命令可以根据模版文件生成新的插件，它创建正确的目录结构、gemspec文件和依赖项，因此您可以开始添加自定义代码以使用Logstash处理数据。

示例用法

```sh
bin/logstash-plugin generate --type input --name xkcd --path ~/ws/elastic/plugins
```

- `--type`：插件类型 — 输入、过滤器、输出或编解码器
- `--name`：新插件的名称
- `--path`：将创建新插件结构的目录路径。 如果未指定，则将在当前目录中创建。