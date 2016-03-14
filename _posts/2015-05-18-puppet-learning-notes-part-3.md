---
layout: post
title: "Puppet Learning Notes #3"
excerpt: "Puppet Learning Notes #3."
tags: [Puppet]
comments: true
---

## 安裝Puppet Server

玩過單機模式的apply之後接著就參考[官方文件](https://docs.puppetlabs.com/puppet/4.0/reference/install_linux.html)來試試看Daemon與Agent的架構

首先透過yum來安裝Puppet Server

{% highlight shell linenos %}
yum install -y puppetserver
{% endhighlight %}

接著產生新的憑證

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet master --verbose --no-daemonize
Notice: foo.bar has a waiting certificate request
Notice: Signed certificate request for foo.bar
Notice: Removing file Puppet::SSL::CertificateRequest foo.bar at '/etc/puppetlabs/puppet/ssl/ca/requests/foo.bar.pem'
Notice: Removing file Puppet::SSL::CertificateRequest foo.bar at '/etc/puppetlabs/puppet/ssl/certificate_requests/foo.bar.pem'
Notice: Starting Puppet master version 4.0.0
^C
{% endhighlight %}

看到`Starting Puppet master version 4.0.0`之後就可以按`Ctrl+C`跳出

接著可以檢查看看產生的憑證是否正確

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet cert list -all
{% endhighlight %}

如果沒有問題就可以啟動Pupppet Server服務

{% highlight shell linenos %}
service puppetserver start
{% endhighlight %}

Server端完成之後再來裝Agent端,首先將下面兩行加到/etc/puppetlabs/puppet/puppet.conf

{% highlight puppet linenos %}
[agent]
server = foo.bar
{% endhighlight %}

然後測試看看有沒有問題,記得先刪除之前所有的測試manifests

{% highlight shell linenos %}
rm -f /etc/puppetlabs/code/environments/production/manifests/*.pp
/opt/puppetlabs/bin/puppet agent --test
{% endhighlight %}

沒有問題的話就試試看放一些manifests,首先建立一個yumrepo的pp檔

{% highlight shell linenos %}
cat >/etc/puppetlabs/code/environments/production/manifests/yumrepo.pp 
class yumrepo_ol6 {
  file { "/etc/yum.repos.d/ol6.repo":
    ensure => file,
    mode   => "0644",
    owner  => root,
    group  => root,
    source => "puppet:///modules/yumrepo/ol6.repo"
  }
}
^D
cat >/etc/puppetlabs/code/environments/production/manifests/site.pp
node "foo.bar" {
  include yumrepo_ol6
}
^D
{% endhighlight %}

再次用test指令看看有沒有問題

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet agent --test
{% endhighlight %}

最後啟動Puppet Agent服務

{% highlight shell linenos %}
service puppet start
{% endhighlight %}

到這邊puppet就會幫忙維護上面所定義的資源,可以試著去更動該檔案puppet應當會自動修復

以上是在還是在同一台主機跑Server跟Agent,接著試試看在第二台主機也安裝Agent

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet agent --test
Info: Creating a new SSL key for foo2.bar
Info: Caching certificate for ca
Info: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for foo2.bar
Info: Certificate Request fingerprint (SHA256): 88:77:09:1B:9E:14:69:BE:6D:4C:82:6A:42:17:2F:5F:D1:93:6E:9F:F3:4D:53:9E:F2:39:59:A2:76:C4:BF:99
Info: Caching certificate for ca
Exiting; no certificate found and waitforcert is disabled
{% endhighlight %}

這時候回到Server檢視憑證會發現第二台主機沒被簽章,少了個"+"號

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet cert list -all
  "foo2.bar" (SHA256) 88:77:09:1B:9E:14:69:BE:6D:4C:82:6A:42:17:2F:5F:D1:93:6E:9F:F3:4D:53:9E:F2:39:59:A2:76:C4:BF:99
+ "foo.bar" (SHA256) F2:DA:19:EF:E6:30:07:AF:44:07:65:92:6E:AD:68:CA:C0:9B:2A:78:1C:BE:D2:07:82:53:C7:8D:C1:38:A3:9F (alt names: "DNS:puppet", "DNS:puppet.foo.baar", "DNS:foo.bar")
{% endhighlight %}

此時就必須手動簽發這個憑證,再次檢視就可以發現前面多了個"+"號

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet cert sign foo2.bar
Notice: Signed certificate request for foo2.bar
Notice: Removing file Puppet::SSL::CertificateRequest foo2.bar at '/etc/puppetlabs/puppet/ssl/ca/requests/foo2.bar.pem'
/opt/puppetlabs/bin/puppet cert list -all
+ "foo.bar" (SHA256) F2:DA:19:EF:E6:30:07:AF:44:07:65:92:6E:AD:68:CA:C0:9B:2A:78:1C:BE:D2:07:82:53:C7:8D:C1:38:A3:9F (alt names: "DNS:puppet", "DNS:puppet.foo.bar", "DNS:foo.bar")
+ "foo2.bar" (SHA256) 9D:38:22:6C:29:BC:7E:9D:5F:64:90:07:18:50:EC:DA:FF:F7:85:83:BF:41:40:30:E2:5A:0B:60:8D:A9:D5:A2
{% endhighlight %}

再次回到第二台主機測試前先把設定檔加入site.pp

{% highlight shell linenos %}
cat >>/etc/puppetlabs/code/environments/production/manifests/site.pp
node "foo2.bar" {
  include yumrepo_ol6
}
^D
{% endhighlight %}

然後回到第二台主機測試一下,沒問題就一樣開啟Agent服務

{% highlight shell linenos %}
/opt/puppetlabs/bin/puppet agent --test
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for foo2.bar
Info: Applying configuration version '1432038031'
Notice: /Stage[main]/Yumrepo_ol6/File[/etc/yum.repos.d/ol6.repo]/ensure: defined content as '{md5}9376e7492d65ccbfb331d7b7d0d80419'
Info: Creating state file /opt/puppetlabs/puppet/cache/state/state.yaml
Notice: Applied catalog in 0.10 seconds
service puppet start
{% endhighlight %}

如果每次都要手動sign那豈不崩潰,當然事情不是這樣的,可以開啟autosign

{% highlight shell linenos %}
cat >/etc/puppetlabs/puppet/autosign.conf
*.foo.bar
^D
{% endhighlight %}

這樣下次再有同個網域的Agent加入就不需要再到Server端操作了
