#Jinja2

Jinja2 是一个现代的，设计者友好的，仿照 Django 模板的 Python 模板语言。 它速度快，被广泛使用，并且提供了可选的沙箱模板执行环境保证安全。模板在Python的web开发中广泛使用，它能够有效的将业务逻辑和页面逻辑分开，使代码可读性增强、并且更加容易理解和维护。

html测试文件
```
<!doctype html>
<title>Hello Sample</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello World!</h1>
{% endif %}
```

会被解析为：

```

from jinja2.runtime import LoopContext, TemplateReference, Macro, Markup, TemplateRuntimeError, missing, concat, escape, markup_join, unicode_join, to_string, identity, TemplateNotFound, Namespace
name = 'index.html'

def root(context, missing=missing, environment=environment):
    resolve = context.resolve_or_missing
    undefined = environment.undefined
    if 0: yield None
    l_0_name = resolve('name')
    pass
    yield '<!doctype html>\n<title>Hello Sample</title>\n'
    if (undefined(name='name') if l_0_name is missing else l_0_name):
        pass
        yield '\n  <h1>Hello %s!</h1>\n' % (
            escape((undefined(name='name') if l_0_name is missing else l_0_name)), 
        )
    else:
        pass
        yield '\n  <h1>Hello World!</h1>\n'

blocks = {}
debug_info = '3=12&4=15'

```

最终解析为：

```
<!doctype html>
<title>Hello Sample</title>

  <h1>Hello 123!</h1>
```