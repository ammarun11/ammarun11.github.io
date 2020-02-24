---

title : "[Fluentd] Logging NodeJS Lab Fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan kali ini saya akan melanjutkan lab fluentd.

### Logging NodeJS

**Prerequisites**
* Basic knowledge of Node.js and NPM
* Basic knowledge of Fluentd
* Node.js 6.0 or higher

### Install NodeJS
```shell
sudo apt install npm
nodejs -v
```
### Make dependencies
```javascript
vim package.json

---
{
  "name": "node-example",
  "version": "0.0.1",
  "dependencies": {
    "express": "^4.16.0",
    "fluent-logger": "^3.2.0"
  }
}
---
```
**Install dependecies dengan perintah**
```shell
npm install
```

```shell
sudo vim /etc/td-agent/td-agent.conf
---
<source>
   @type forward
   port 50003
</source>
<match fluentd.test.**>
   @type stdout
</match>
---

sudo systemctl restart td-agent
```
### deploy aplikasi NodeJS

```javascript
vim index.js
---
const express = require('express');
const logger = require('fluent-logger');
const app = express();

logger.configure('fluentd.test', {
  host: 'localhost',
  port: 50003,
  timeout: 3.0,
  reconnectInterval: 600000 // 10 minutes
});

app.get('/', function(request, response) {
  logger.emit('follow', {from: 'userA', to: 'userB'});
  response.send('Hello World!');
});
const port = process.env.PORT || 3000;
app.listen(port, function() {
  console.log("Listening on " + port);
});
```

jalankan script inde.js
```shell
node index.js
```

akses `http://10.0.0.10:3000/` menggunakan browser atau dengan curl ke port tersebut,
lalu log dengan perintah.
```shell
tail -f /var/log/td-agent/td-agent.log
```
Hasil
```shell
root@pod03-node0:~# curl http://10.0.0.10:3000/
Hello World!# 

root@pod03-node0:~# tail -f /var/log/td-agent/td-agent.log
2019-11-09 14:03:41 +0000 [warn]: #0 this parameter is highly recommended to save the position to resume tailing.
2019-11-09 14:03:41 +0000 [info]: adding source type="forward"
2019-11-09 14:03:41 +0000 [info]: #0 starting fluentd worker pid=32640 ppid=32632 worker=0
2019-11-09 14:03:41 +0000 [info]: #0 listening port port=50003 bind="0.0.0.0"
2019-11-09 14:03:41 +0000 [info]: #0 following tail of /var/log/apache2/access.log
2019-11-09 14:03:41 +0000 [info]: #0 [input_debug_agent] listening dRuby uri="druby://127.0.0.1:24230" object="Fluent::Engine" worker=0
2019-11-09 14:03:41 +0000 [info]: #0 [input_forward] listening port port=24224 bind="0.0.0.0"
2019-11-09 14:03:41 +0000 [info]: #0 fluentd worker is now running worker=0
2019-11-09 14:15:12 +0000 [warn]: #0 no patterns matched tag="fluent.test.follow"
2019-11-09 14:15:17 +0000 [warn]: #0 no patterns matched tag="fluent.test.follow"
^C
```

**Referensi**
* [https://docs.fluentd.org/language-bindings/nodejs](https://docs.fluentd.org/language-bindings/nodejs)
* [https://www.npmjs.com/package/fluent-logger](https://www.npmjs.com/package/fluent-logger)
# Happy,  Enjoy ngoprek~
