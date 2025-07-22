+++
date = '2025-07-22T16:01:07+08:00'
title = 'React Agent与MCP'
+++

2025年的今天，大语言模型的能力早已不再限于chatbot(只会聊天)，各家模型都在文本预测的基础上扩展LLM的能力，例如Structured Outputs(结构化输出)，Function calling(函数调用)，MCP(模型上下文协议)，同时，有人提出一种结合LLM思考与行动的协同机制-React。

## React Agent
通过让LLM循环执行 推理(Reasoning)->行动(Action)->观察(Observation) 来完成任务。
![alt text](image-1.png)
[参考-ReAct: Synergizing Reasoning and Acting in Language Models](https://react-lm.github.io/)

## Function calling
让LLM按照指定格式输出json格式的数据，来表明自己需要去使用什么函数，参数是什么，同时支持流式生成，
例如:
![alt text](image.png)
[参考-opneai-function-calling](https://platform.openai.com/docs/guides/function-calling?api-mode=chat)

## MCP
MCP (Model Context Protocol)是一种开放协议，用于标准化应用程序如何向大型语言模型（LLMs）提供上下文。可以将 MCP 想象为 AI 应用的 typec 接口。正如 typec 提供了一种标准化的方式将您的设备连接到各种外设和配件，MCP 也提供了一种标准化的方式，将 AI 模型连接到不同的数据源和工具。
![alt text](image-2.png)
此处不再过多赘述，详情参考:
[官方文档](https://modelcontextprotocol.io/introduction)
[OpenMcp-what-is-mcp](https://kirigaya.cn/openmcp/zh/plugin-tutorial/what-is-mcp.html)

## 整合
![alt text](image-3.png)
