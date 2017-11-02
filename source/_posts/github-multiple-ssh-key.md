title: 坑002:在mac上使用不同的ssh key访问不同的git服务端
date: 2015-11-09 22:15:00
tags:
- github
- ssh
- git
categories: git
---
# 缘起
一个ssh key绑定了一个github账号之后,无法再次绑定到另一个github账号,但现在需要在同一台mac上访问两个github账号下的repo,比如说一个私人的,一个工作的,怎么办?

# 填坑
此坑本道祭了道 **"寻龙符"** 搞定! 也就是google了一下,嘿嘿,现总结如下:

## 总体思路:
1. 如果需要访问两个不同的github账号的repo,则需要两个ssh key分别绑定到这两个github账号上.
2. 需要调整ssh配置和本地git项目的remote配置,使之分别对应到不同的ssh key上去.

<!-- more -->
## 填坑法:
1. 生成一个新的ssh key,首先建议将~/.ssh目录备个份,`cp -r ~/.ssh ~/ssh_bak`,然后再执行以下指令,注意在提示保存位置时使用不同的文件名:
  ```json
  $ cd ~/.ssh
  $ ssh-keygen -t rsa -C "your_email@youremail.com"
  ```
  好了之后,.ssh目录下应该多了一对公钥和私钥,为了好举例,我再生成一个,就变成了这样:
  ```json
  -rw-------   1 abc  staff   179  5  2 3:06 id_rsa
  -rw-r--r--   1 abc  staff    10  5  2 3:06 id_rsa.pub
  -rw-------   1 abc  staff   326 11  9 6:12 id_rsa_abc
  -rw-r--r--   1 abc  staff    37 11  9 6:12 id_rsa_abc.pub
  -rw-------   1 abc  staff   326 11  9 6:12 id_rsa_xyz
  -rw-r--r--   1 abc  staff    37 11  9 6:12 id_rsa_xyz.pub
  ```
  生成之后,记得把id_rsa_abc.pub添加到对应的github账号中去.
2. 把这两个key连同原来的key进行添加:
  ```json
  $ ssh-add ~/.ssh/id_rsa
  $ ssh-add ~/.ssh/id_rsa_abc
  $ ssh-add ~/.ssh/id_rsa_xyz
  ```
  查看添加结果:
  ```json
  $ ssh-add -l
  ```
3. 配置ssh的config:
    ```json
    $ vim ~/.ssh/config
    ```
    内容如下:
    ```json
    Host abc-github
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa_abc
    Host xyz-github
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa_xyz
    ```
    注意这里第一行的Host实际上是一个别名,这个别名大有用处,第二行的HostName是真正的域名,第三行不解释,第四行就是ssh key的位置,那么如果还有更多的key,也是可以如法泡制的.
4. 修改git项目的remote配置,这里我就直接修改`项目dir/.git/config`这个文件的remote部分了.
  ```json
  [remote "origin"]
          url = git@abc-github:loyoo/testAAA.git
          fetch = +refs/heads/*:refs/remotes/origin/*
  ```
  看见了没,这里用到了之前提到的 **别名** ,替换了原来的`github.com`,这就是解决问题的关键! 其实这样就结束了,这个项目现在就能使用id_rsa_abc来进行fetch和push操作了.

# 总结
网上的帮助有不少,但都没直接讲清楚这个 **别名** 的意思,有的写成了abc.github.com,特别容易造成误会,子域名?难道我还要改下hosts?
其实就是别名而已.当然,如果你还有gitcafe或者oschina的账号,也是可以同上处理的.


