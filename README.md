# MNS SDK for PHP
官方地址:['https://help.aliyun.com/document_detail/32381.html?spm=5176.doc27477.6.679.CKsLzZ'](https://help.aliyun.com/document_detail/32381.html?spm=5176.doc27477.6.679.CKsLzZ)

* SDK版本:Version 1.3.4
* 更新日期:2017-04-13


## 操作
```
<?php
require_once('./vendor/autoload.php');

use AliyunMNS\Client;
use AliyunMNS\Requests\SendMessageRequest;
use AliyunMNS\Requests\CreateQueueRequest;
use AliyunMNS\Exception\MnsException;

$accessId = '';
$accessKey = '';
$endPoint = '';

/* 发送数据 */
$queue = $client->getQueueRef($queueName);
$messageBody = '数据,字符串类型';
// 3个参数(数据,指定的秒数延后可被消费，单位为秒,消息的优先级权值，取值范围：1~16，其中1为最高优先级)
// 参考 https://help.aliyun.com/document_detail/35134.html?spm=5176.doc35135.6.719.L2LVs6#h2-request
$request = new SendMessageRequest($messageBody,0,4);
try {
    $res = $queue->sendMessage($request);
    echo "MessageSent! \n";
} catch (MnsException $e) {
    echo "SendMessage Failed: " . $e;
    return;
}

/* 获取数据 */
$receiptHandle = NULL;
try{
    // 1. 直接调用receiveMessage函数
    // 1.1 receiveMessage函数接受waitSeconds参数，无特殊情况这里都是建议设置为30
    // 1.2 waitSeconds非0表示这次receiveMessage是一次http long polling，如果queue内刚好没有message，那么这次request会在server端等到queue内有消息才返回。最长等待时间为waitSeconds的值，最大为30。
    $res = $queue->receiveMessage(30);
    echo "ReceiveMessage Succeed! \n";
    // 2. 获取ReceiptHandle，这是一个有时效性的Handle，可以用来设置Message的各种属性和删除Message。具体的解释请参考：help.aliyun.com/document_detail/27477.html 页面里的ReceiptHandle
    $receiptHandle = $res->getReceiptHandle();
    $messageBody = $res->getMessageBody();
    var_dump($messageBody);
} catch (MnsException $e) {
    // 3. 像前面的CreateQueue和SendMessage一样，我们认为ReceiveMessage也是有可能出错的，所以这里加上CatchException并做对应的处理。
    echo "ReceiveMessage Failed: " . $e . "\n";
    echo "MNSErrorCode: " . $e->getMnsErrorCode() . "\n";
    return;
}

/* 删除拿到的消息 */
try {
    // 5. 直接调用deleteMessage即可。
    $res = $queue->deleteMessage($receiptHandle);
    echo "DeleteMessage Succeed! \n";
} catch (MnsException $e) {
    // 6. 这里CatchException并做异常处理
    // 6.1 如果是receiptHandle已经过期，那么ErrorCode是MessageNotExist，表示通过这个receiptHandle已经找不到对应的消息。
    // 6.2 为了保证receiptHandle不过期，VisibilityTimeout的设置需要保证足够消息处理完成。并且在消息处理过程中，也可以调用changeMessageVisibility这个函数来延长消息的VisibilityTimeout时间。
    echo "DeleteMessage Failed: " . $e . "\n";
    echo "MNSErrorCode: " . $e->getMnsErrorCode() . "\n";
    return;
}
```