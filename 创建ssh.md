### 二、在mac创建ssh公钥

首先在mac下进入~/.ssh，查看是否已经有ssh公钥了。

```
cd ~/.ssh
ls
# known_hosts
```

发现只有一个known_hosts，那我们就建一个ssh公钥，继续输入以下指令创建ssh公钥：

```
ssh-keygen
```

![image.png](https://ucc.alicdn.com/pic/developer-ecology/48f9f6287aac45da83634f0e7eff6384.png)

首先会让你输入公钥存储文件，输入默认的就行（/Users/你的电脑用户名/.ssh/id_rsa）。然后会输入公钥使用密码，输入两次，可以直接回车不设置。（建议直接回车，设置后后面提交代码也很麻烦，每次都要验证输入以下）。创建完成后，再次输入`ls`查看目录下是否已经生成了公钥，确认有后，使用命令cat id_rsa.pub查看公钥，

```
ls
cat id_rsa.pub
```

![image.png](https://ucc.alicdn.com/pic/developer-ecology/e8f560253b8f476e8c7b2841af24a46b.png)

从ssh-rsa开始一直到.local都是ssh公钥，复制出来，一会要到github中创建shh连接使用。

### 三、在github新建一个ssh连接，并配置ssh公钥

接下来进行github ssh连接配置，首先进入github，然后点击个人头像后，选择Settings。

![image.png](https://ucc.alicdn.com/pic/developer-ecology/f7f4da30e1cf40339eda60ed37282384.png)

点击SSH and GPG，再点击 New SSH key。

![image.png](https://ucc.alicdn.com/pic/developer-ecology/e636de72422945c5854502ca38dafc91.png)

有两个参数设置，title可以随便写，key就是我们前面复制的SSH公钥（id_rsa.pub文件内容）。

![image.png](https://ucc.alicdn.com/pic/developer-ecology/f8a5c2f41f2943ccb3694d017f24e894.png)

粘贴好后，点击 Add SSH Key即可。

### 四、验证测试ssh公钥配置是否成功

接下来我们验证下是否设置成功，终端输入下面指令进行测试

```
ssh -T git@github.com
```

**![image.png](https://ucc.alicdn.com/pic/developer-ecology/36d94251ac954b86b3f86393e99c0ed5.png)**

如果你和我一样之前创建ssh时设置了密码，需要先输入密码，然后根据提示输入yes，同意连接，显示结果为下面这样则表示连接成功。

```
Hi XksA-me! You've successfully authenticated, but GitHub does not provide shell access.
```

在进行push前你还需要改下上传模式（之前是http），进入对应项目目录，执行下面语句即可。

```
cd Desktop/Project/web\ _project/brief_blog
git remote set-url origin git@github.com:XksA-me/brief_blog.git
```