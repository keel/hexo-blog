title: 坑001:Hexo的配置和ftpsync插件的坑
date: 2015-11-08 15:15:00
tags:
- hexo
- nodejs
categories: nodejs
---
# 缘起
入Hexo之门才一天,这就踩坑了! 正是应了"凡悟道,必踩坑"之理.

# 小坑
先说说头一天踩的小坑吧:
1. _config.yml中的配置,这个配置文件对于字段键值的空白是尤其讲究啊!键名的冒号后面必须跟一个空格才还跟键值,否则报错.
  如: `language:zh-Hans`错误, `language: zh-Hans`正确. 另外键名之前的空格也必须对齐,否则也报错!如:
    ```json
    deploy:
    type: git
      repo: https://github.com/xyz/xyz.github.io.git
      branch: master
      message:
    ```
  以上错误,
    ```json
    deploy:
      type: git
      repo: https://github.com/xyz/xyz.github.io.git
      branch: master
      message:
    ```
  以上正确.
  总之,这个yml文件特别地敏感脆弱,就不要欺负她了.
2. hexo配置在项目根目录下的_config.yml文件中,但别忘记了主题里面还有个配置是在thems/themName/_config.yml,这里可控制主题和显示相关的配置,嗯,是的,她们都一样敏感脆弱.
3. 关于主题,这里注意有的主题会引入外部的字体之类,这些资源会拖慢整个页面的响应,建议找出他们并去除掉,另外配置里面一些默认为true的项目也看下是否真的需要,不需要就关掉.

<!-- more -->
# 大坑
ftpsync,这个必须是叫大坑了!
1. 插件看上去特别"简单",但工作原理肯定与你想的不一样,它是真的就是"同步"!是把ftp的服务端目录和文件调整成本地一模一样,也就是原来服务器上的文件如果本地没有话是会被**删除**,**删除**,**删除**(重要的事说三遍)掉的!而不是一般deploy只负责上传的理解.我去查了hexo-deploy-ftpsync插件,它使用的是[node-ftpsync](https://github.com/evanplaice/node-ftpsync),人家的说明里明确说了:
  > This is not ftp-deploy. This application will delete files and directories on the remote server to match the local machine. Use this application in production at your own risk.

  好吧,我这下损失大了,服务器的文件还不知道去哪里找...
2. 好吧,那我配置ignore总可以避免服务端文件被删除吧,但这个ignore简单的配置一个目录名居然不生效?那到底如何配置呢?经过多方试验,发现这里必须按如下配置:
    ```json
    port: 21
    ignore: ['dir1','.DS_Store','file1.htm']
    ```
  注意中括号不可少.
3. 下面是最大的坑了,话说最坑的必然留到最后...
配置好后,执行一次`hexo g -d`,好,成功生成并上传了! 慢着,趁兴致我再来个小改或者再写篇美文post一下吧,好,再次`hexo g -d`,咦?怎么报错了?
```
Error: 550 ./wwwroot/2015: Cannot create a file when that file already exists.
    at Ftp.parse (/Users/keel/dev/hexo/node_modules/jsftp/lib/jsftp.js:257:11)
    at Ftp.parseResponse (/Users/keel/dev/hexo/node_modules/jsftp/lib/jsftp.js:174:8)
    at Stream.<anonymous> (/Users/keel/dev/hexo/node_modules/jsftp/lib/jsftp.js:146:24)
    at emitOne (events.js:77:13)
    at Stream.emit (events.js:169:7)
    at ResponseParser.reemit (/Users/keel/dev/hexo/node_modules/duplexer/index.js:70:25)
    at emitOne (events.js:77:13)
    at ResponseParser.emit (events.js:169:7)
    at readableAddChunk (_stream_readable.js:146:16)
    at ResponseParser.Readable.push (_stream_readable.js:110:10)
    at ResponseParser.Transform.push (_stream_transform.js:128:32)
    at ResponseParser.<anonymous> (/Users/keel/dev/hexo/node_modules/ftp-response-parser/index.js:65:12)
    at Array.forEach (native)
    at ResponseParser._transform (/Users/keel/dev/hexo/node_modules/ftp-response-parser/index.js:48:9)
    at ResponseParser.Transform._read (_stream_transform.js:167:10)
    at ResponseParser.Transform._write (_stream_transform.js:155:12)
    at doWrite (_stream_writable.js:292:12)
    at writeOrBuffer (_stream_writable.js:278:5)
    at ResponseParser.Writable.write (_stream_writable.js:207:11)
    at Socket.ondata (_stream_readable.js:528:20)
    at emitOne (events.js:77:13)
    at Socket.emit (events.js:169:7)
```
难道是心神不聚所致? 此时捏个 **"凝神自省决"** :
* 网络--ok
* 配置--ok
* ftpServer--ok, **注意这里ftp是windows服务器,很重要!** ,直接用ftp工具上传是ok的

再次出手!还是报错!

好吧,看错误应该是远端的目录已经存在了无法再创建,开启 **"填坑大法第一式"** :查log.
开启ftpsync的verbose模式看看,果然够verbose,输出了一大版,原来这货是先取出远端的目录文件,再与本地的目录文件对比,然后将差异计算出来,最后执行相关的同步操作的.

仔细看了好多次,终于看出门道了,remote部分列出的目录不对!本来远端有个2015的目录,但remote的log中却只有一个5的目录,因此在计算差异时有了再次创建2015的目录的任务,但远端已经有了,所以执行时创建会失败.问题根源应该是在获取remote目录列表上了.

这是个大麻烦,很可能是个bug了,二话不说,先祭个 **"寻字符"** 吧,google,baidu,github狂搜一通,居然无解?那再祭起 **"请神符"** ,各论坛发问不提,还在github上提了相关的issue.

本想着就此等结果了,但身为踩坑道人,为平坑劫,求人先求已,心想nodejs有个好处,这些modules实际上是在自己手上,不像jar包没法动手脚,它是可以就着log追过去尝试寻下报错的根源.嗯,可以试试开启 **"填坑大法第二式"** :debug:
这个hexo-deploy-ftpsync是依赖于node-ftpsync的,而它是依赖于jsftp的,在node_modules里面找到jsftp猛下一顿console.log,尝试看下问题,
...历经3小时...
终于摸清了jsftp的ftp请求发送与接受处理流程,看来问题在接收上,jsftp还依赖一个作者的parse-listing项目,这个项目里对ftp返回内容做了解析,而这个解析是一个正则,经过验证,当目录为 ***纯数字*** 时,它会以为这是一个文件的大小,把它视作文件处理!

好吧,本道可以填坑了,先结一个 **"混元补天阵"** :fix. 阵眼分别是fork,git clone,patch,mocha,待到阵法达成之时,再`hexo g -d`,终于没问题了. 为了积功德,结个好的因果,遵天理,进行 ** [pull request](https://github.com/keel/parse-listing/commit/0b130bc927afec17c593f5b276a434d0a11e6e68) ** ,再对之前提的各个issue(是的,从hexo开始到jsftp所有依赖的项目里我都提了issue),做了补完.

至此,心愿已了,虽然费时甚多,但有了善果,可以愉快的睡了.