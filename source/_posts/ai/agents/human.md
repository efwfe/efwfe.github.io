---
title: Human FeedBack
date: 2024-09-04 15:20:11
tags:
- AI
- Agent
- LLM
---


### Human Input Modes

AutoGEN支持三种模式的用户输入。
1. `NEVER`, human input never request.
2. `TERMINATE`(default), human input only requested when a termination condition is met.
3. `ALWAYS`,huaman input is always reuquested and the human can choose to skip and tigger an auto-reply, intercept and provide feedback, or terminate the conversation. Note that in this mode termination based on `max_consecutive_auto_reply` is ignored.


#### MODE: Never

```python

from autogen import ConversableAgent

agent_with_number = ConversableAgent(
    "agent_with_number",
    system_message="You are playing a game of guess-my-number. You have the "
    "number 53 in your mind, and I will try to guess it. "
    "If I guess too high, say 'too high', if I guess too low, say 'too low'. ",
    llm_config={"config_list":config_list},
    is_termination_msg=lambda msg: "53" in msg["content"],  # terminate if the number is guessed by the other agent
    human_input_mode="NEVER",  # never ask for human input
)

agent_guess_number = ConversableAgent(
    "agent_guess_number",
    system_message="I have a number in my mind, and you will try to guess it. "
    "If I say 'too high', you should guess a lower number. If I say 'too low', "
    "you should guess a higher number. ",
    llm_config={"config_list":config_list},
    human_input_mode="NEVER",
)

result = agent_with_number.initiate_chat(
    agent_guess_number,
    message="I have a number between 1 and 100. Guess it!",
)


```


#### MODE: Always

这个模式中，总是需要用户输入，用户可以跳过，拦截，结束会话。

```python
human_proxy = ConversableAgent(
    "human_proxy",
    llm_config=None,
    human_input_mode="ALWAYS",
)

result = human_proxy.initiate_chat(
    agent_with_number,
    message='10'
)

```


#### MODE: TERMINATE

用户输入只在终止对话的条件出发的时候需要，如果用户选择拦截或者回复，那么计数将被重置；
如果用户选择跳过那么自动回复机制将启用，

在结束的时候需要用户输入，如果不是exit那么就会再次继续进入下一次循环。