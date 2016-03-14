---
layout: post
title: "Install Redmine on AWS EC2"
excerpt: "Install Redmine on AWS EC2."
tags: [EC2, Redmine]
comments: false
---

最近因為~~心血來潮~~工作需要起一個Redmine來做專案管理,順便練習最近學的puppet, 於是就開始了這一路X到底的經驗, 這真是近期遇到最難搞的東西!

不廢話趕快開始不然天都要黑了,首先我選用的是Amazon Linux AMI 2015.03 (HVM)作為基本的OS, 開機後可參考先前的文章安裝puppet agent, 這邊就不多描述.

接著就是安裝Redmine 3.0.3, 這次安裝採用的方式是Apache+Fcgi, 編輯以下的pp檔

{% highlight puppet linenos %}
node "redmine" {
  package { "httpd": ensure => installed }
  package { "mod_fcgid": ensure => installed }
  package { "fcgi-devel": ensure => installed }
  package { "ImageMagick": ensure => installed }
  package { "ImageMagick-devel": ensure => installed }
  package { "ruby-devel": ensure => installed }
  package { "gcc": ensure => installed }
  package { "mysql-devel": ensure => installed }
  package { "patch": ensure => installed }
  package { "bundler": ensure  => installed, provider => "gem" }
  exec { "download_redmine_release":
    command => "/usr/bin/wget -q http://www.redmine.org/releases/redmine-3.0.3.tar.gz -O /var/www/redmine-3.0.3.tar.gz",
    creates => "/var/www/redmine-3.0.3.tar.gz",
  }
  exec { "unpack_redmine_release":
    command => "/bin/tar xzf /var/www/redmine-3.0.3.tar.gz -C /var/www/html",
    creates => "/var/www/html/redmine-3.0.3",
    require => Exec["download_redmine_release"]
  }
  file { "/var/www/html/redmine-3.0.3/Gemfile.local":
    ensure => "file",
    owner => "apache",
    group => "apache",
    mode => "664",
    content => "gem "fcgi"",
  }
  exec { "bundle-install":
    cwd => "/var/www/html/redmine-3.0.3",
    path => "/bin:/usr/bin:/usr/local/bin:/sbin:/usr/sbin:/usr/local/sbin",
    command => "bundle install --path vendor/bundle --without development test",
    user => "apache",
    logoutput => true,
  }
  file { "/var/www/html/redmine-3.0.3":
    ensure => directory,
    owner => "apache",
    group => "apache",
    recurse => true,
  }
  file { "/var/www/html/redmine-3.0.3/public/dispatch.fcgi":
    ensure => "file",
    owner => "apache",
    group => "apache",
    mode => "775",
    source => "/var/www/html/redmine-3.0.3/public/dispatch.fcgi.example",
  }
  file { "/var/www/html/redmine-3.0.3/public/.htaccess":
    ensure => "file",
    owner => "apache",
    group => "apache",
    mode => "664",
    source => "/var/www/html/redmine-3.0.3/public/htaccess.fcgi.example",
  }
  file { "/var/www/html/redmine-3.0.3/config/database.yml":
    ensure => "file",
    owner => "apache",
    group => "apache",
    mode => "664",
    content => "production:
  adapter: mysql2
  database: redmine
  host: #MYSQL_FQDN#
  username: #MYSQL_USERNAME#
  password: #MYSQL_PASSWORD#
  encoding: utf8
"
  }
  file { "/etc/httpd/conf.d/redmine.conf":
    ensure => "file",
    owner  => "root",
    group  => "root",
    mode   => "644",
    content => "<VirtualHost *:80>
  ServerName #SERVER_FQDN#
  ServerAdmin #SERVER_ADMIN#
  DocumentRoot /var/www/html/redmine-3.0.3/public/
  ErrorLog logs/redmine_error_log
  MaxRequestLen 52428800
  FcgidInitialEnv RAILS_ENV "production"

  <Directory "/var/www/redmine-3.0.3/public/">
    Options Indexes ExecCGI FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
  </Directory>
</VirtualHost>"
  }
  service { "httpd": ensure=> running, enable=> true }
}
{% endhighlight %}

完成後執行apply就完成了絕大部分, 咦, 這樣不是很簡單嗎? 不, 一點都不, 上面這個檔案花了我將近一天看了數十篇中外寫的都不一樣的安裝文才拚出來的, 不是我太弱就是這東西肯定有什麼問題, 幾個重點提示一下:

* 使用fcgi就不需要安裝Passenger, 不知道為什麼很多教學兩個都裝的用意是什麼
* 千萬不要用root跑bundle install, 不然會遇到最後要跑rake的時候一直說找不到mysql2的鬼問題
* 萬一真的很不幸這樣幹了用`bundle list | tail -n +2 | awk '{print $2}' | xargs gem uninstall`清乾淨重來
* 承上用apache跑bundle install要加上參數 --path vendor/bundle
* 還有跑之前記得要產一個Gemfile.local並且把fcgi加進去
* 最後fcgi要傳變數是要把FcgidInitialEnv RAILS_ENV加在vhost內,這個也搞了我很久

以為到這邊就結束了, 事情並沒有憨人想的這麼簡單, 照上面這樣搞會一直出現403因為vhost裡的AllowOverride根本沒生效, 原因在於httpd.conf預設值為None, 所以必須去修改成為All, 這段還不知道怎麼用puppet去改

{% highlight ApacheConf linenos %}
<Directory "/var/www/html">

#
# Possible values for the Options directive are "None", "All",
# or any combination of:
#   Indexes Includes FollowSymLinks SymLinksifOwnerMatch ExecCGI MultiViews
#
# Note that "MultiViews" must be named *explicitly* --- "Options All"
# doesn't give it to you.
#
# The Options directive is both complicated and important.  Please see
# http://httpd.apache.org/docs/2.2/mod/core.html#options
# for more information.
#
    Options Indexes FollowSymLinks

#
# AllowOverride controls what directives may be placed in .htaccess files.
# It can be "All", "None", or any combination of the keywords:
#   Options FileInfo AuthConfig Limit
#
    AllowOverride All
{% endhighlight %}

到這邊總行了吧, 好吧是的, 如果這樣還不行我也要崩潰了, 剩下的就是比較簡單的事情了, 因為我是用RDS所以細節我就不說了, 記得把正確的DB資訊寫到database.yml就可以執行起始DB的動作

{% highlight shell linenos %}
cd /var/www/html/redmine-3.0.3
bundle exec rake generate_secret_token
RAILS_ENV=production bundle exec rake db:migrate
{% endhighlight %}

如果一切順利重啟httpd之後應該就可以看到redmine的畫面, 我都要痛哭流涕了…

## References ##

* [Installing Redmine](http://www.redmine.org/projects/redmine/wiki/redmineinstall)
* [Install Redmine on Centos 6.5 - 64 bit](http://www.redmine.org/projects/redmine/wiki/Install_Redmine_25x_on_Centos_65_complete)
* [Redmine on CentOS installation HOWTO](http://www.redmine.org/projects/redmine/wiki/Redmine_on_CentOS_installation_HOWTO)
* [HowTo configure Apache to run Redmine](http://www.redmine.org/projects/redmine/wiki/HowTo_configure_Apache_to_run_Redmine)
* [安裝 Redmine (CentOS + Apache + Ruby on Rails + Redmine)](http://blog.tonycube.com/2013/11/redmine-centos-apache-ruby-on-rails.html)
