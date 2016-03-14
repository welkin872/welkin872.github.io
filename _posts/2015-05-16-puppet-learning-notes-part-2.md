---
layout: post
title: "Puppet Learning Notes #2"
excerpt: "Puppet Learning Notes #2."
tags: [Puppet]
comments: true
---

## 嘗試不同的 manifests

接著試著用apply模式用puppet安裝及維護既有的系統設定

### yumrepo

先看看系統內原有的repo設定檔

{% highlight shell linenos %}
cat /etc/report.d/ol6.conf
[ol6_latest]
name=Oracle Linux $releasever Latest ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/latest/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=1
 
[ol6_addons]
name=Oracle Linux $releasever Add ons ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/addons/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=0
 
[ol6_ga_base]
name=Oracle Linux $releasever GA installation media copy ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/0/base/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=0
 
[ol6_u1_base]
name=Oracle Linux $releasever Update 1 installation media copy ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/1/base/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=0
 
[ol6_u2_base]
name=Oracle Linux $releasever Update 2 installation media copy ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/2/base/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=0
 
[ol6_u3_base]
name=Oracle Linux $releasever Update 3 installation media copy ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/3/base/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=0
 
[ol6_UEK_latest]
name=Latest Unbreakable Enterprise Kernel for Oracle Linux $releasever ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/UEK/latest/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=1
 
[ol6_UEK_base]
name=Unbreakable Enterprise Kernel for Oracle Linux $releasever ($basearch)
baseurl=http://public-yum.oracle.com/repo/OracleLinux/OL6/UEK/base/$basearch/
gpgkey=http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6
gpgcheck=1
enabled=0
{% endhighlight %}

接著就參考[官方文件](https://docs.puppetlabs.com/references/latest/type.html#yumrepo)改寫成puppet的manifests,要注意用雙引號的時候$要escape

{% highlight shell linenos %}
cat >/etc/puppetlabs/code/environments/production/manifests/yumrepo_ol6.pp
node "foo.bar" {
 
yumrepo {
"ol6_latest":
descr => "Oracle Linux $releasever Latest ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/latest/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "1";
"ol6_addons":
descr => "Oracle Linux $releasever Add ons ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/addons/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
"ol6_ga_base":
descr => "Oracle Linux $releasever GA installation media copy ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/0/base/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
"ol6_u1_base":
descr => "Oracle Linux $releasever Update 1 installation media copy ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/1/base/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
"ol6_u2_base":
descr => "Oracle Linux $releasever Update 2 installation media copy ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/2/base/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
"ol6_u3_base":
descr => "Oracle Linux $releasever Update 3 installation media copy ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/3/base/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
"ol6_UEK_latest":
descr => "Latest Unbreakable Enterprise Kernel for Oracle Linux $releasever ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/UEK/latest/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
"ol6_UEK_base":
descr => "Unbreakable Enterprise Kernel for Oracle Linux $releasever ($basearch)",
baseurl => "http://public-yum.oracle.com/repo/OracleLinux/OL6/UEK/base/$basearch/",
gpgkey => "http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6",
gpgcheck => "1",
enabled => "0";
}
 
}
^D
{% endhighlight %}

然後就可以執行apply

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet apply /etc/puppetlabs/code/environments/production/manifests/yumrepo_ol6.pp
{% endhighlight %}

因為原本的repo檔案還在所以puppet很聰明的不會做任何事情,接著刪除原來的repo讓puppet重新產生

{% highlight shell linenos %}
rm -f /etc/yum.repos.d/ol6.repo
/opt/puppetlabs/bin/puppet apply /etc/puppetlabs/code/environments/production/manifests/yumrepo_ol6.pp
Notice: Compiled catalog for foo.bar in environment production in 0.58 seconds
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_latest]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_addons]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_ga_base]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_u1_base]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_u2_base]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_u3_base]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_UEK_latest]/ensure: created
Notice: /Stage[main]/Main/Node[foo.bar]/Yumrepo[ol6_UEK_base]/ensure: created
Notice: Applied catalog in 0.07 seconds
{% endhighlight %}

這時候會發現puppet竟然是一個repo產生一個檔案,目前還查不到怎麼解到目前為止看起來挺OK的,但是要把既有的設定用手工的方式轉實在太蠢了這一定不是這樣的阿,果然找了一下puppet有指令就可以把既有的設定轉換為manifest格式

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet resource yumrepo
yumrepo { 'ol6_UEK_base':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/UEK/base/$basearch/',
descr => 'Unbreakable Enterprise Kernel for Oracle Linux $releasever ($basearch)',
enabled => '0',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_UEK_latest':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/UEK/latest/$basearch/',
descr => 'Latest Unbreakable Enterprise Kernel for Oracle Linux $releasever ($basearch)',
enabled => '1',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_addons':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/addons/$basearch/',
descr => 'Oracle Linux $releasever Add ons ($basearch)',
enabled => '0',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_ga_base':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/0/base/$basearch/',
descr => 'Oracle Linux $releasever GA installation media copy ($basearch)',
enabled => '0',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_latest':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/latest/$basearch/',
descr => 'Oracle Linux $releasever Latest ($basearch)',
enabled => '1',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_u1_base':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/1/base/$basearch/',
descr => 'Oracle Linux $releasever Update 1 installation media copy ($basearch)',
enabled => '0',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_u2_base':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/2/base/$basearch/',
descr => 'Oracle Linux $releasever Update 2 installation media copy ($basearch)',
enabled => '0',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
yumrepo { 'ol6_u3_base':
ensure => 'present',
baseurl => 'http://public-yum.oracle.com/repo/OracleLinux/OL6/3/base/$basearch/',
descr => 'Oracle Linux $releasever Update 3 installation media copy ($basearch)',
enabled => '0',
gpgcheck => '1',
gpgkey => 'http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6',
}
{% endhighlight %}

好吧,所以以上耍蠢了,乖乖copy&paste然後apply,收工!

等等,好像還有更簡單的方法,把原始的repo設定直接放到puppet的modules目錄,讓puppet去產生該檔

{% highlight shell linenos %}
mkdir -p /etc/puppetlabs/code/environments/production/modules/yumrepo/files/
cp /etc/yum.repos.d/ol6.repo /etc/puppetlabs/code/environments/production/modules/yumrepo/files/
{% endhighlight %}

然後修改manifest利用file這個type來安裝檔案,並用puppet:///來指定來源位置

{% highlight shell linenos %}
cat >/etc/puppetlabs/code/environments/production/manifests/yumrepo_ol6.pp
node "foo.bar" {
  file { "/etc/yum.repos.d/ol6.repo":
    ensure => file,
    mode   => "0644",
    owner  => root,
    group  => root,
    source => "puppet:///modules/yumrepo/ol6.repo"
  }
}
^D
/opt/puppetlabs/bin/puppet apply /etc/puppetlabs/code/environments/production/manifests/yumrepo_ol6.pp
{% endhighlight %}

這樣就可以很快速的把原本的設定放到puppet裡面去維護,不過好像就有點失去puppet的本意了XD

### package

搞定了repo接下來就可以裝些package了,很簡單puppet使用package這個type來設定

比說如我們需要安裝mc跟git這兩個package只要這麼寫即可

{% highlight shell linenos %}
cat >/etc/puppetlabs/code/environments/production/manifests/package.pp
node "foo.bar" {
  package { "mc":
    ensure => installed
  }
  package { "git":
    ensure => installed
  }
}
^D
/opt/puppetlabs/bin/puppet apply /etc/puppetlabs/code/environments/production/manifests/package.pp
{% endhighlight %}

到此大致了解puppet的運作了,更多的應用範例可以參考[Puppet Cookbook](http://www.puppetcookbook.com/)

## References ##
[yum repo and package dependencies with puppet](https://getpocket.com/a/read/470522464)
