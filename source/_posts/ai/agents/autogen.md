---
title: AutoGen 入门
date: 2024-09-04 14:22:31
tags:
- AI
- Agent
- LLM
---

## AutoGen


### Agents

> an agent in autogen, is an entity that can sed and receive messages to and from other agents in its environment.
> An agent can be powered by models, code executors (such as an IPyhon kernel), human, or a combination of these and other pluggable and customizable components.

An example of such agents is the built-in `ConversableAgent` which supports llm, code executor, function and tool executor, a component for keeping human-in-the-loop

```python

from autogen import ConversableAgent, config_list_from_json

config_list = config_list_from_json(env_or_file="OAI_CONFIG_LIST")

agent = ConversableAgent(
    "chatbot",
    llm_config={"config_list": config_list},
    code_execution_config=False, # turn off code executor default off
    function_map=None, # no registerd functions, default none
    human_input_mode="NEVER" # never ask for human input,

)

reply = agent.generate_reply(messages=[
    {
        "content": "tell me a joke",
        "role": "user"
    }
])

print(reply)

```


### Roles and Conversation

> assgin roles to agents and have them participate in conversations or chat with each other. 
> A conversation is a sequence of messages exchanged between agents.
> for example, we can assign different roles to two agents by setting their `system_message`.

```python

cathy = ConversableAgent(
    "cathy",
    llm_config={"config_list": config_list},
    system_message="Your name is Cathy and your are a part of a duo of comedians.",
    human_input_mode="NEVER"
)

joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",
    llm_config={"config_list": config_list},
    human_input_mode="NEVER"
)

result = joe.initiate_chat(cathy, message="Cathy, tell me a joke", max_turns=2)

```

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

```

### 总结
主要是初步了解是试用autogen，但这个工具和一年前我了解的metagpt很类似，就是角色环境工具使用，但autogen在这方面的设计应该更灵活一些，最近也是有一些想法想要通过mutiple-agents系统来实现看看效果，目前简单的使用上没有什么问题，接下来想看看里面的其他的模块，比如工具使用，代码执行部分的内容，能够和图数据进行结合的话更好了。