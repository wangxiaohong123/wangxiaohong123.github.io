##### 安装

可以直接使用vibe coding让工具参考hermes的github文档帮我们安装，也可以执行下面命令安装：

```shell
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 安装完推出刷新环境变量
source ~/.bashrc
```

配置基模

```shell
hermes model 或者进入hermes执行/model
# 根据提示选择模型，输入模型对应的ak，我使用的是deepseek v4 pro：sk-25f60572491f44b9a93e6559ed5e4b7c
```

##### 接入飞书

登录开放者平台[https://open.feishu.cn/app?lang=zh-CN]创建应用，并添加机器人能力；在权限管理中添加如下权限：

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:write",
      "contact:contact.base:readonly",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "docs:document.content:read",
      "event:ip_list",
      "im:chat",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.group_msg",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource",
      "sheets:spreadsheet",
      "wiki:wiki:readonly"
    ],
    "user": [
      "aily:file:read",
      "aily:file:write",
      "contact:contact.base:readonly",
      "im:chat.access_event.bot_p2p_chat:read"
    ]
  }
}
```

添加完提交发布审批并审批，在凭证与基础信息中获取app ID和app Secret

```
cli_a97e259cfdb85bdf		ikhIQYwu1CKgdmmXhWj1rQZ8zOQFzPGB
```

在本地终端中输入如下命令进入hermes聊天平台配置：

```shell
hermes gateway setup
# 然后选择飞书，第一项是直接扫码就能配置好，第二项需要填写App ID ，App Secret ，User IDs
# userIds参考：https://open.feishu.cn/document/faq/trouble-shooting/how-to-obtain-user-id
```

然后在飞书开放平台刚刚创建的应用页面选择事件与回调，订阅方式选择长链接并添加im.message.receive_v1接收消息事件，重新发布。

创建一个群，添加群机器人选择刚刚创建的应用，@机器人开始对话。

##### 配置浏览器客户端

使用open web UI。

##### 使用

外挂知识库：使用google的notebookLm。