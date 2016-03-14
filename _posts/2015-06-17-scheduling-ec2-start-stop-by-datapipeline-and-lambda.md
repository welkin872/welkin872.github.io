---
layout: post
title: "Scheduling EC2 Start/Stop by Data Pipeline + Lambda"
excerpt: "Scheduling EC2 Start/Stop by Data Pipeline + Lambda."
tags: [EC2, Lambda, Start, Stop, Data Pipeline]
comments: true
---

有用過AWS的知道, EC2是依照時間收錢的, 所以不用的時候就該關掉, 所以很自然地會覺得應該有個功能是可以設定什麼時間開機什麼時間關機, 但很抱歉, 它沒有!

好吧, AWS沒有自然有人做出來, 果然搜尋一下就有Skeddly這種公司提供, 但是天下沒有白吃的午餐, 它要錢!而且還是照開關次數計費!

到頭來還是得自己想辦法, 開一台EC2跑cron去開關其他機器? 不如拿石頭砸自己的腳 !AWS服務這麼多, 一定有繞過EC2的方法, 於是有一天靈機一動就想到了, Data Pipeline + Lambda或許就是解法.

AWS Data Pipeline提供了cron的功能, 可以設定時間啟動AWS的服務但必須跑在EC2或EMR上(別著急,往下看), AWS Lambda則是提供跑NodeJS程式而不需要開EC2, 所以理論上應該可以用Data Pipeline設定時間然後想辦法觸發Lambda跑AWS SDK去執行開關機.

沒錯! 這理論確實可行, 而且不只可行還非常便宜, 以Data Pipeline來說, 每天開關一次一個月才美金$1.2, Lambda更不用說, 光是免費的就可以執行一百萬次, 根本就是免費, 興奮嗎? 以下就開始介紹這以其人之道還治其人方法:

首先, 最關鍵的部分就是如何用Data Pipeline設定開關機時間卻不用開啟EC2或EMR資源, 技巧就是利用Precondition讓這pipeline永遠都失敗然後去觸發SNS, 步驟如下:

1. 新增一個pipeline
2. 新增一個data node
3. Type選S3DataNode, File Path隨便打, 存在或不存在都無所謂
4. 新增一個Precondition
5. 設定Precondition的Type為S3KeyExists, S3 Key輸入一個不存在的檔案
6. 新增Precondition的On Fail Action, 設定Action的Type為SnsAlarm
7. 設定SNS的Subject為ec2-start-instancec或ec2-stop-instance
8. 設定SNS的Message為{"region":"ap-northeast-1","instanceId":"i-xxxxxxxx"}
9. 設定開機或關機的時間

接著就要開一隻Lambda, 然後把event source設定為上述的SNS, 根據Subject跟Message的內容用AWS SDK去執行開關EC2的instance並且發送SNS通知(這個跟上述的SNS必須不同, 不然嘿嘿嘿), 參考以下原始碼:

{% highlight js linenos %}
console.log('Loading function');
var AWS = require('aws-sdk');
var snsRegion = 'us-west-2';
var snsTopic = 'arn:aws:sns:us-west-2:xxxxxxxx:SNS_TOPIC';
 
exports.handler = function(event, context) {
    var status = {message: '', error: false};
    //console.log(JSON.stringify(event, null, 2));
    console.log('command:', event.Records[0].Sns.Subject);
    var message = {};
    try {
        message = JSON.parse(event.Records[0].Sns.Message);
    } catch (e) {
        console.log('SNS Message:', event.Records[0].Sns.Message);
        console.log(e);
        context.fail("Parse message failed.");
    }
    //console.log(JSON.stringify(message, null, 2));
    console.log('region:', message.region);
    console.log('instanceId:', message.instanceId);
 
    if ("ec2-start-instance" == event.Records[0].Sns.Subject) {
        startEC2Instance(message, startEC2InstanceHandler);
    } else if ("ec2-stop-instance" == event.Records[0].Sns.Subject) {
        stopEC2Instance(message, stopEC2InstanceHandler);
    } else {
        context.fail("Unsupported command.");
    }
 
    function startEC2Instance(instance, next) {
        var ec2 = new AWS.EC2({region: instance.region});
        var params = {
          InstanceIds: [instance.instanceId],
          DryRun: false
        };
 
        ec2.startInstances(params, function(err, data) {
            if (err) {
                console.log(err, err.stack);
            }
            next && next.call(this, err, instance);
        });
    }
 
    function startEC2InstanceHandler(err, data) {
        var message = {};
        var d = new Date();
 
        message.subject = "ec2-start-instance: " + data.instanceId;
        message.message = "[" + d.toUTCString() + "]";
 
        if (err) {
            message.subject += " = FAILED";
            message.message += err.toString();
            status.message = "Start instance " + data.instanceId + " failed.";
            status.error = true;
        } else {
            message.subject += " = SUCCEED";
            status.message = "Start instance " + data.instanceId + " succeed.";
            status.error = false;
        }
        pushSNSMessage(message, responseContext);
    }
 
    function stopEC2Instance(instance, next) {
        var ec2 = new AWS.EC2({region: instance.region});
        var params = {
          InstanceIds: [instance.instanceId],
          DryRun: false,
          Force: false
        };
 
        ec2.stopInstances(params, function(err, data) {
            if (err) {
                console.log(err, err.stack);
            }
            next && next.call(this, err, instance);
        });
    }
 
    function stopEC2InstanceHandler(err, data) {
        var message = {};
        var d = new Date();
 
        message.subject = "ec2-stop-instance: " + data.instanceId;
        message.message = "[" + d.toUTCString() + "]";
 
        if (err) {
            message.subject += " = FAILED";
            message.message += err.toString();
            status.message = "Stop instance " + data.instanceId + " failed.";
            status.error = true;
        } else {
            message.subject += " = SUCCEED";
            status.message = "Stop instance " + data.instanceId + " succeed.";
            status.error = false;
        }
        pushSNSMessage(message, responseContext);
    }
 
    function pushSNSMessage(message, next) {
        var sns = new AWS.SNS({region: snsRegion});
        var params = {
          Message: message.message,
          Subject: message.subject,
          TopicArn: snsTopic
        };
 
        sns.publish(params, function(err, data) {
            if (err) {
                console.log(err, err.stack);
            }
            next && next.call(this, err, {});
        });
    }
 
    function responseContext() {
        if (status.error) {
            context.fail(status.message);
        } else {
            context.succeed(status.message);
        }
    }
};
{% endhighlight %}

哇啦! 這裡有便宜的EC2 Scheduler不用嗎?
