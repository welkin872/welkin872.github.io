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
