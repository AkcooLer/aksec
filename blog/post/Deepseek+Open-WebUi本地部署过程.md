---
title: Deepseek+Open-WebUi本地部署过程
date: 2025/02/18
desc: Deepseek+Open-WebUi本地部署过程
tags: ['#全部','#AI']
cover: /logo.png
---
一、软件需求
---------------

1.操作系统：Ubuntu  
2.软件安装：ollama、Open-webUI、Docker

一、Ollama搭建
---------------
Ollam官网：ollama.com

![图片1]( "/og/1.png")

进入首页选择Download，选择对应操作系统，这里我的为Linux

![图片2]( "/og/2.png")

复制curl传输命令下载install并运行SH文件

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

1、安装完成后  
2、输入ollama -Version  
3、验证是否安装成功  
4、运行ollama：ollama serve

```bash
查看运行状态：sudo systemctl status ollama
修改配置文件：/etc/systemd/system/ollama.
service
```
```bash
Environment="OLLAMA_HOST=0.0.0.0:8434"
```

安装完成后，你需要启动Ollama服务：

```bash
sudo systemctl start ollama  
sudo systemctl status ollama
```

#### DeepSeek-r1 相关版本及大小参考：

![图片3]( "/og/3.png")

![图片4]( "/og/4.png")

选择对应models与版本

执行命令下载和运行模型。  

```bash 
如：ollama run deepseek-r1:1.5b
```

访问Ollama API
Ollama还提供了一个REST API，允许你通过编程方式访问模型。默认情况下，API监听端口11434：

```bash
curl http://localhost:11434/api/generate -d '{"model": "qwen2.5:1.5b", "prompt": "Your prompt here", "stream": false}'
```
#### ollama 常用命令参考
![图片5]( "/og/5.png")


ollama 提供了丰富的命令行工具，方便用户对模型进行管理。

> ollama --help：查看帮助信息。  
> oollama serve：启动 ollama 服务。  
> oollama create <model-name> [-f Modelfile]：根据一个 Modelfile 文件导入模型。  
> oollama show <model-name:[size]>：显示某个模型的详细信息。  
> oollama run <model-name:[size]>：运行一个模型。若模型不存在会先拉取它。  
> oollama stop <model-name:[size]>：停止一个正在运行的模型。  
> oollama pull <model-name:[size]>：拉取指定的模型。  
> oollama push <model-name>：将一个模型推送到远程模型仓库。  
> oollama list：列出所有模型。  
> oollama ps：列出所有正在运行的模型。  
> oollama cp <source-model-name> <new-model-name>：复制一个模型。  
> oollama rm <model-name:[size]>：删除一个模型。

二、Open-WEBUI搭建
---------------

> Open WebUI 是一个开源的大语言模型项目，通过部署它可以得到一个纯本地运行的基于浏览器访问的 Web 服务。它提供了可扩展、功能丰富、用户友好的自托管 AI Web 界面，支持各种大型语言模型（LLM）运行器，可以通过配置形式便捷的集成 ollama、OpenAI 等提供的 API。

通过 Open WebUI 可以实现聊天机器人、本地知识库、图像生成等丰富的大模型应用功能。
在开始之前，请确保你的系统已经安装了 docker。
新建文件 docker-compose.yml，内容参考：
```docker
ervices:
  open-webui:
    image:ghcr.io/open-webui/open-webui:main
    environment:
      -OLLAMA_API_BASE_URL=http://ollama:11434/api
      -HF_ENDPOINT=https://hf-mirror.com
      -WEBUI_NAME="LZW的LLM服务"
      # 禁用 OPENAI API 的请求。若你的网络环境无法访问 openai，请务必设置该项为 false
      # 否则在登录成功时，会因为同时请求了 openai 接口而导致白屏时间过长
      -ENABLE_OPENAI_API=false
      # 设置允许跨域请求服务的域名。* 表示允许所有域名
      -CORS_ALLOW_ORIGIN=*
      # 开启图片生成
      -ENABLE_IMAGE_GENERATION=true
      # 默认模型
      -DEFAULT_MODELS=deepseek-r1:8b
      # RAG 构建本地知识库使用的默认嵌入域名
      -RAG_EMBEDDING_MODEL=bge-m3
    ports:
      -8080:8080
    volumes:
      -./open_webui_data:/app/backend/data
    extra_hosts:
      # - host.docker.internal:host-gateway
```

这里需注意 **environment** 环境变量部分的自定义设置。许多设置也可以通过登录后在 web 界面进行修改。

在该目录下执行该命令以启动服务：**docker-compose up -d**。成功后即可通过浏览器访问：http://localhost:8080。
服务镜像更新参考：
```bash
# 拉取新镜像
docker-compose pull
# 重启服务
docker-compose up -d --remove-orphans
# 清理镜像
docker image prune
```
> open-webui 详细文档参考：https://docs.openwebui.com/  getting-started/env-configuration  
> ollama 官方站：https://ollama.com  
>  ollama 中文站：https://ollama.org.cn
>  ollama 入门：https://ollama.readthedocs.io/quickstart/  
> ollama 常见问题：https://ollama.readthedocs.io/faq/  
> 魔塔社区：https://modelscope.cn  
> HF Mirror：https://hf-mirror.com  
> open-webui 文档：https://docs.openwebui.com