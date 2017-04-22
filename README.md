# wechat-server-token-check
使用Nodejs接入并验证微信公众号和微信小程序服务器配置

# 接入步骤

### 1、微信后台配置
- [url] 写自己的已经绑定服务器的域名
- [Token] 随便写

截图如下：
![服务器基础配置选项](http://7xlmr2.com1.z0.glb.clouddn.com/wechat.png)


### 2、自己的服务器部署以下代码

check.js
```
'use strict';

const http = require('http');
const url = require('url');
const querystring = require('querystring');
const crypto = require('crypto');

const TOKEN = 'whatever';   // token可以是任何字符串，前提是必须和上图基础配置的一致

http.createServer((req, res) => {
  if (/\*\/\*/img.test(req.headers.accept)) {
    const _query = url.parse(req.url).query;
    const query = querystring.parse(_query);
    const signature = query.signature;
    const echostr = query.echostr;
    const timestamp = query.timestamp;
    const nonce = query.nonce;

    // 拼成数组，字典排序，再拼接
    const tmpStr = [TOKEN, timestamp, nonce].sort().reduce((prev, cur) => prev + cur);

    // sha1加密
    const sha1 = crypto.createHash('sha1');
    const sha1_result = sha1.update(tmpStr).digest('hex');

    // 如果是来自微信的请求就返回echostr
    if (sha1_result === signature) {
      res.end(echostr);
    }
  }
}).listen(80, () => {
  console.log(`server start at 80`);   // 一定要是80端口
});
```

```
node check.js   //启动服务，如果80端口被其他进程占用需要先停止其他进程
```

### 3、在基础配置中点击提交
提示配置成功即完成了校验
