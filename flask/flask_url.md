#URL

和URL链接有关的类大致由以下三个：
  * `Map`：存储所有的`URL`规则。
  * `Rule`：代表一个`URL`模式。
  * `MapAdapter`：执行`URL`匹配的具体工作。

####Rule类
一个`Rule`的实例代表一个`URL`模式的对象，通过这个对象可以解析并匹配请求对应的视图函数。
Flask应用中`Rule`的对象来源于两点
  * `statics`静态页面文件夹的路径
  * `@app.route`的装饰器

`Rule`会把这些模式通过`compile()`方法编译成一个正则表达式，与`URL`匹配时通过`match()`方法直接和正则表达式匹配。

####Map类
`Map`中存储了各种各样`Rule`。
<div align=center>
![](map_rule.png)
</div>

`Map`实例可以通过后续的调用和给定的`URL`进行匹配。主要由以下属性：
  * `_rules`：当前规则中所有的模式列表。
  * `_rules_by_endpoint`：视图函数对应的模式列表。


####MapAdapter类
一个`Map`实例的适配器。构造函数传入一个`map`对象和部分环境信息。
`MapAdapter`类中主要的方法为`match()`——该方法将会进行具体的URL匹配工作。它会将请求中的`url`和`Map`实例中的所有`Rule`进行匹配。