---
{"title":"使用pip下载时提示“You are using pip version 8.1.1, however version 22.1 is available.“","url":"https://blog.csdn.net/Cynthialearn/article/details/124754685","clipped_at":"2022-09-20 15:20:12","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/使用pip下载时提示-You-are-using-pip-version-8-1-1-however-version-22-1-is-available_1663658412/","dgPassFrontmatter":true}
---


# 使用pip下载时提示“You are using pip version 8.1.1, however version 22.1 is available.“

在使用pip install下载其他包时，报了错，如图：

![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-c52d74dcf6033f510979a09d5004e01e.png)  
提示：“**You are using pip version 8.1.1, however version 22.1 is available.  
You should consider upgrading via the ‘pip install --upgrade pip’ command.**”

根据提示，执行"pip install --upgrade pip"依旧无效。如图：

![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-f175420975ad9d9c0fafb4474ff1a8c3.png)  
然后，去网上查找广大网友的解决方案，并找到了**解决办法**：

```python
# 升级pip:
1.sudo wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
2.sudo python get-pip.py
3.pip -V

# 升级pip3:
1.sudo wget https://bootstrap.pypa.io/pip/3.5/get-pip.py
2.sudo python3 get-pip.py
3.pip -V
```

![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-46ca14a2eba9d4e7a5aafa3e3ae73642.png)

![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-861498dfc2c3eb93ac0adc873b198db0.png)  
期间，也遇到了2个问题：

### 问题一

如果在执行完第1、2步后，使用"pip -V"查看版本时，报了错：  
“**bash: /usr/bin/pip: No such file or directory**”

![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-281dfd8676d0ad9225c0554395212b17.png)  
解决方案，执行以下命令：

```python
# 清除缓存
hash -r

# 如果清除缓存不行，再执行
sudo apt-get update
sudo apt-get upgrade
```

### 问题二

我按顺序依次将pip和pip3升级后，执行"pip -V"和"pip3 -V", 发现pip也指向了python3.5包下，如下图：

![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-a109e670808bbd171294db234183590f.png)  
于是重新执行"**sudo python get-pip.py**"就正常了，如图：  
![在这里插入图片描述](/img/user/阅读库藏/assets/1663658412-17f5439965ea72936b64d9e403a088c9.png)  
升级完pip, 就可以继续下载其他包了。