---
layout: article
title:  "配置.ssh"
toc: true
disqus: true
categories: basic
---

>简介：本文主要介绍如何生成key，配置key

# 步骤一：查看本地是否已经生成了ssh
---

An SSH key allows you to establish a secure connection between your computer and GitLab. Before generating an SSH key in your shell, check if your system already has one by running the following command:
GNU/Linux/Mac/PowerShell:

{% highlight bash %}
cat ~/.ssh/id_rsa.pub
{% endhighlight %}

如果你看到一串ssh-rsa的字符串，可以忽略ssh-keygen这一步
If you see a long string starting with ssh-rsa, you can skip the ssh-keygen step.
To generate a new SSH key, use the following command:

2,使用下面命令生成key


{% highlight bash %}
ssh-keygen -t rsa -C “weifang.zeng@happylifeplat.com"
{% endhighlight %}

This command will prompt you for a location and filename to store the key pair and for a password. When prompted for the location and filename, just press enter to use the default. If you use a different name, the key will not be used automatically.
Note: It is a best practice to use a password for an SSH key, but it is not required and you can skip creating a password by pressing enter.
If you want to change the password of your key later, you can use the following command: ssh-keygen -p <keyname>
GNU/Linux/Mac/PowerShell:

{% highlight bash %}
cat ~/.ssh/id_rsa.pub
{% endhighlight %}


3，复制公钥到github

Copy-paste the key to the 'My SSH Keys' section under the 'SSH' tab in your user profile. Please copy the complete key starting with ssh-rsa and ending with your username and host.
To copy your public key to the clipboard, use the code below. Depending on your OS you'll need to use a different command:

{% highlight bash %}
pbcopy < ~/.ssh/id_rsa.pub
{% endhighlight %}


4,测试本机是否可以连接github


{% highlight bash %}
$ ssh -T git@github.com  
{% endhighlight %}













