---
layout: post
title: "Puppet Learning Notes #1"
excerpt: "Puppet Learning Notes #1."
tags: [Puppet]
comments: false
---

## 認識Puppet ##

在動手之前先看看下面文章

* [Introduction to Puppet](http://docs.puppetlabs.com/guides/introduction.html)

## 嘗試單機(Apply)模式

有了大致的概念後不囉嗦直接動手玩玩看, 參考[官方文件](http://docs.puppetlabs.com/puppet/4.0/reference/install_linux.html)直上Puppet 4.0在RHEL 6.x環境 首先安裝repo

{% highlight shell linenos %}
rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-pc1-el-6.noarch.rpm
{% endhighlight %}

完成後搜尋看看repository 是不是找的到puppet

{% highlight shell linenos %}
yum search puppet
{% endhighlight %}

應該會看到以下的package

{% highlight text linenos %}
puppet-agent.x86_64 : The Puppet Agent package contains all of the elements needed to run puppet, including ruby, facter, hiera and mcollective.
puppetdb.noarch : Puppet Centralized Storage Daemon
puppetdb-terminus.noarch : Puppet terminus files to connect to PuppetDB
puppetlabs-release-pc1.noarch : Release packages for the Puppet Labs PC1 repository
puppetserver.noarch : Puppet Labs - puppetserver
{% endhighlight %}

接著安裝puppet agent

{% highlight shell linenos %}
yum -y install puppet-agent
{% endhighlight %}

裝完後檢查看看版本是否為4.0

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet --version
{% endhighlight %}

接下來就可以建立第一個設定檔

{% highlight shell linenos %}
cat >/etc/puppetlabs/code/environments/production/manifests/hellopuppet.pp

node "foo.bar" {

file { '/root/hellopuppet.txt':
ensure => "file",
owner => "root",
group => "root",
mode => "600",
content => "Congratulations!
Puppet has created this file.
",}

}
^D
{% endhighlight %}

用apply命令套用設定檔

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet apply /etc/puppetlabs/code/environments/production/manifests/hellopuppet.pp
{% endhighlight %}

這時候puppet應該會在家目錄下產生hellopuppet.txt

{% highlight shell linenos %}
cat /root/hellopuppet.txt
{% endhighlight %}

接下來可以任意改改看那個檔案的內容或是權限然後再執行apply的指令,檢查puppet是否正確地把檔案修復

## References

* [https://www.digitalocean.com/community/tutorials/how-to-install-puppet-in-standalone-mode-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-install-puppet-in-standalone-mode-on-centos-7)
* [https://www.digitalocean.com/community/tutorials/how-to-install-puppet-to-manage-your-server-infrastructure](https://www.digitalocean.com/community/tutorials/how-to-install-puppet-to-manage-your-server-infrastructure)
