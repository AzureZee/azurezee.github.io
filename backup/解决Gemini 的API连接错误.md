### 本想试试Gemini CLI 的 Flutter 扩展
于是把几个月前安装但是一直没用过的Gemini CLI更新后,
满怀期待地跟它打个招呼,等来的却是 [API Error: exception TypeError: fetch failed sending request] .

### 在相关issue里找到了解决方法
得先用命令`$env:http_proxy = "http://xxx.xxx.xxx.xxx:8080/"`
设置终端使用的代理的环境变量, 再进入Gemini CLI, 这次终于可以正常对话了.
不过用命令设置的变量是临时的, 在~/.gemini/.env文件配置HTTP_PROXY变量才能持久化,
顺便把apikey也加进去了, 之前是放在系统环境变量里的.
```
GEMINI_API_KEY="API-KEY"
HTTP_PROXY="http://xxx.xxx.xxx.xxx:8080/"
http_proxy="http://xxx.xxx.xxx.xxx:8080/"
HTTPS_PROXY="http://xxx.xxx.xxx.xxx:8080/"
https_proxy="http://xxx.xxx.xxx.xxx:8080/"
```