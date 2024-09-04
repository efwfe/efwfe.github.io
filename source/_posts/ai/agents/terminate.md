---
title: AutoGen聊天终止
date: 2024-09-04 14:47:31
tags:
---

什么时候终止聊天对话，一般来说对很多复杂的问题或者自主工作流知道什么时候停止是很重要的，目前有两种主要的机制来控制agent之间的交流
1. `initiate_chat`设定中设定参数
2. 设置agent终止信号

### `initiate_chat`
设置`max_turns`参数为的数值能够控制交流对话的次数。设定为3轮，六次对话：

```shell

joe (to cathy):

Cathy, tell me a joke

--------------------------------------------------------------------------------
cathy (to joe):

Sure, Joe! Why did the scarecrow win an award?  

Because he was outstanding in his field! 🌾😄

--------------------------------------------------------------------------------
joe (to cathy):

That's a classic! Alright, how about this: Why don't skeletons fight each other? Because they don't have the guts! 🦴🤣

--------------------------------------------------------------------------------
cathy (to joe):

Good one, Joe! Alright, here’s another: What do you call a fish wearing a bowtie? 

Sofishticated! 🎩🐟

--------------------------------------------------------------------------------
joe (to cathy):

Haha, that’s great! Okay, here’s my turn: Why did the bicycle fall over? Because it was two-tired! 🚲😆

--------------------------------------------------------------------------------
cathy (to joe):

That’s a good one, Joe! How about this: What do you get when you cross a snowman and a vampire? 

Frostbite! ❄️🧛‍♂️😄

--------------------------------------------------------------------------------

```

### Agent trigger

可以使用agent trigger参数来停止交流，有两个参数可以设置
1. `max_consecurive_auto_reply`: this condition tiggers termination if the number of automatic responses to the same sender exceeds a threshold.如果向同一个发送者的响应次数超过阈值就停止，通过实例参数`max_consecutive_auto_reply`参数设置，但是这个计数可能会被重置。
2. `is_termination_msg`: this condition can tigger termination if the received message satisfies a particular condition, 比如它的回答里面包含了'TERMINATE'能够通过条件判断来终止,通过实例参数`is_terminate_msg`来设置。

#### max_consecurive_auto_reply
```python


joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",
    llm_config={"config_list": config_list},
    human_input_mode="NEVER",  # Never ask for human input.
    max_consecutive_auto_reply=1,  # Limit the number of consecutive auto-replies.
)

cathy = ConversableAgent(
    "cathy",
    llm_config={"config_list": config_list},
    system_message="Your name is Cathy and your are a part of a duo of comedians.",
    human_input_mode="NEVER"
)

result = joe.initiate_chat(cathy, message="Cathy, tell me a joke.")

```
这里joe发起会话，但是他只能对cathy回答一次就结束会话。



#### is_termination_msg


```python

joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",
    llm_config={"config_list": config_list},
    human_input_mode="NEVER",  # Never ask for human input.
    is_termination_msg=lambda msg: "good bye" in msg['content'].lower()
    )

cathy = ConversableAgent(
    "cathy",
    llm_config={"config_list": config_list},
    system_message="Your name is Cathy and your are a part of a duo of comedians.",
    human_input_mode="NEVER"
)

result = joe.initiate_chat(cathy, message="Cathy, tell me a joke.and then say the words GOOD BYE.")

```
output: 
```shell

joe (to cathy):

Cathy, tell me a joke.and then say the words GOOD BYE.

--------------------------------------------------------------------------------
cathy (to joe):

Sure, Joe! Here’s one for you: 

Why don’t skeletons fight each other? 

Because they don’t have the guts!

GOOD BYE!

--------------------------------------------------------------------------------

```

### 总结
我刚开始没有设置good bye结束词，我们他们俩能够一直聊，担心自己的token费，后来才注意到应该在提示词中设定结束触发词才行。
