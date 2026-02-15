---
layout: post
title: "Supercharge Kubernetes with kubectl ai"
date: 2025-12-27 11:12:00 +0800
categories: kubernetes
tags: rke2
image:
  path: /assets/img/headers/kubectl-ai.jpg
---

*Google’s kubectl-ai is a revolutionary open-source project that seamlessly integrates with Large Language Models (LLMs) like gemini, bedrock and local LLM providers such as ollama with Kubernetes operations, enabling users to interact with Kuberntes clusters using natural language.*

{% include embed/youtube.html id='U4wzenY9Dfo' %}

🎞️ [Watch Video](https://youtu.be/U4wzenY9Dfo)

## Pre-requisits

- Kubernetes cluster
- Kubectl is installed and configured
- Helm installaed and configured


## Install ollama on your machine

```sh
curl -fsSL https://ollama.com/install.sh | sh
```

### Verify
```sh
ollama --version
```

### Pull a lightweight model — we’ll use Gemma 3 4‑B for the demo:
```sh
ollama pull gemma3:4b
```

### kubectl‑ai

*Ensure that you have kubectl is installed and configured. because kubectl-ai is a plugin that sits next to kubectl in your terminal.*

```sh
curl -sSL https://raw.githubusercontent.com/GoogleCloudPlatform/kubectl-ai/main/install.sh | bash
```

### Verify 
```sh
kubectl ai version
```

### Lets Ask our cluster a question :
```sh
kubectl ai --llm-provider ollama --model gemma3:4b --enable-tool-use-shim
```

### You can also configure kubectl-ai using a YAML configuration file at ~/.config/kubectl-ai/config.yaml to persist those flags so you don’t have to re‑type them in. 

```sh
{
mkdir -p ~/.config/kubectl-ai/
cat <<EOF > ~/.config/kubectl-ai/config.yaml
model: gemma3:4b
llmProvider: ollama
enableToolUseShim: true
toolConfigPaths: 
    - ~/.config/kubectl-ai/tools.yaml
EOF
}
```

###  verify your configuration
```sh
kubectl-ai model
```

### We can Extend the Agent with Custom Tools

*kubectl-ai can call any binary you describe in ~/.config/kubectl-ai/tools.yaml.*

```sh
{
cat <<EOF > ~/.config/kubectl-ai/tools.yaml
- name: helm
  description: "Helm is the Kubernetes package manager."
  command: "helm"
  is_custom: true
EOF
}
```

### To enable the custom tools, you must point `kubectl-ai` to the directory containing the tool configuration YAML files using the `--custom-tools-config` flag.
```sh
kubectl-ai --custom-tools-config=~/.config/kubectl-ai/tools.yaml "helm list"
```

## Install Python and Create a Virtual Environment

### First, make sure you have Python installed. You can verify this by running:
```sh
python3 --version
```
### In the directory where you want your project, open a terminal and run:
```sh
python3 -m venv ollama-env
```
### Activate the environment by running:

```sh
source ollama-env/bin/activate
```

### Install the Ollama Python library:
```sh
pip3 install ollama
```

### Run the python app
```sh
python3 python-app/kubectlai-app.py
```

### 🔗 Reference Links:

- [kubectl-ai](https://github.com/GoogleCloudPlatform/kubectl-ai.git)

- [Github repo](https://github.com/mkbntech/kubectl-ai-demo.git)
