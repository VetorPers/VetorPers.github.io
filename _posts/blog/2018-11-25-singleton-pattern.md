---
layout:         post
title:          单例模式踩坑
categories: blog
description:    单例模式问题
keywords: singleton pattern
---


>问题的起因:
我们有一个消息系统，处理各种异步消息。比如：管理人员给用户的公告，用户的动态发布需要通知管理人员审核，审核通过后需要通知用户和更新搜索引擎的数据。这就存在着在某个地方会发两个及以上的消息。

消息系统基于 nsq ，我们先来看看基本的代码, `EventSender` 类：
```
    private $sender;
	
	public static function getInstance()
    {
        if (!(self::$instance && (self::$instance instanceof self))) {
            self::$instance = new self();
        }

        return self::$instance;
    }

	public function sendEvent($eventName,  $topic, $data = [])
    {
        $this->init($topic);
        return $this->sender->sendEvent($eventName, $data);
    }
	
	private function init($topic)
    {
        $this->sender = new Sender($this->senderName, $this->topicHost, $this->topicName, $this->appEnv);
    }
```

上面只是抽取了部分代码。调用发消息的方法看起来像这样：

```
EventSender::getInstance()->sendEvent('UPDATE_INDEX', 'TOPIC_USER', ['user_id' => 1]);
EventSender::getInstance()->sendEvent('MESSAGE', 'TOPIC_MESSAGE', ['user_id' => 1]);
```

当用户成功发布动态以后，我们需要异步的去更新用户信息，并给用户发送一个通知，告知他的信息已被更新。因为更新操作和通知用户是在两个 TOPIC 上，所以上述代码只能收到更新消息，通知用户的信息不能成功送达。

开始一直以为是不能同时驱动 nsq 的两个 TOPIC ，在同步的情况下，TOPIC_USER 和 TOPIC_MESSAGE 只有一个能消化。在网上也没找到相应的回答，无论怎么调试都收不到两个消息。

自以为对单例模式有一定的认识，之前没遇到这样的问题也没注意。在仔细查看代码后发现，每次发送消息都会去执行 `init` 方法，都会重新实例化 `Sender` 类,而每次实例化都会传递 TOPIC 参数。如果使用单例模式的话，当再调用发消息方法的时候，因为实例已经存在，就不会再去初始化 TOPIC 。所以我们需要这样来调用：

```
(new EventSender)->sendEvent('UPDATE_INDEX', 'TOPIC_USER', ['user_id' => 1]);
(new EventSender)->sendEvent('MESSAGE', 'TOPIC_MESSAGE', ['user_id' => 1]);
```
