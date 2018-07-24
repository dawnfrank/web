#RequireHandler

<div align=center>
![](requirehandler类结构.png)
</div>

####BaseHTTPRequestHandler
HTTP请求处理的基类,该协议识别请求的三个部分：
  * 请求行：`<command> <path> <version>`
    * `command`：请求方法，一般为`GET`或`POST`
    * `path`：请求的URL，URL编码规范 (使用 %xx 来解释 ASCII 字符，其中xx为字符16进制).
    * `version`：HTTP版本，`HTTP/1.0`或者`HTTP/1.1`
  * RFC-822形式的请求头：每行通过CRLF分开
  * 消息主体（数据）
  * 请求头和消息中间由空行隔开