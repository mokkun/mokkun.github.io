+++
title = "Coding with LLMs: Ollama"
date = "2023-10-27T15:04:05+09:00"
description = "Discover how to quickly setup LLMs with Ollama and configure a ChatGPT like UI."

tags = ["CodingWithLLMs", "LLM", "Ollama", "CodeLlama", "Docker", "ChatGPT"]
+++

A few months ago, I learned about [Ollama](https://ollama.ai) and how it enables you to easily run LLMs locally. I finally got the time to give it a go, and it is surprisingly easy to get it working.

Go to https://ollama.ai, download the installer for your system, and once completed, open the terminal and run `ollama run codellama:7b`. Congratulations, you now have access to Code Llama locally! A quick test.

```shell
>>> write a function in swift that adds two numbers
 
func add(a: Int, b: Int) -> Int {
    return a + b
}

This is a simple function that takes two integers as input and returns their sum.

You can call this function by passing in the values you want to add, like this:

let result = add(a: 2, b: 3)
print(result) // Output: 5
```

I've been using ChatGPT for a while now, and to have the possibility to run and try out different LLMs locally is incredible.

Although running it locally on a laptop could be enough, I want to take it one step further and repurpose my gaming PC to run the LLMs on it instead and access it from all the computers on my network.

## Installing Ollama with Docker

### Docker setup

[Ollama offers a Docker image](https://ollama.ai/blog/ollama-is-now-available-as-an-official-docker-image) and can run with GPU acceleration inside Docker containers. I don't have Docker set up on my machine, so I need to address that first.

For future reference, I've used their [Install Docker Engine on Debian - Install using the apt repository](https://docs.docker.com/engine/install/debian/#install-using-the-repository) guide to set up Docker on my [Pop_OS!](https://pop.system76.com). I initially tried [Docker Desktop](https://docs.docker.com/desktop/), but [I had issues with my NVIDIA card](https://github.com/NVIDIA/nvidia-docker/issues/1711) and needed to run Docker with `sudo`.

1. Set up Docker's `apt` repository.
    ```shell
    # Add Docker's official GPG key:
    sudo apt update
    sudo apt install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Add the repository to Apt sources:
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    ```
2. Install the Docker packages.
    ```shell
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```
3. Verify that the installation is successful by running the hello-world image:
    ```shell
    sudo docker run hello-world
    ```

### NVIDIA GPU setup

With Docker up and running, the next step was to install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation) so I can run Ollama with my GPU.

1. Configure the repository:
    ```shell
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
    && \
    sudo apt-get update
    ```
2. Install the NVIDIA Container Toolkit packages:
    ```shell
    sudo apt-get install -y nvidia-container-toolkit
    ```

### Ollama setup

With all the Docker setup done, installing Ollama is pretty simple.

```shell
sudo docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

To install a LLM in the container:

```shell
sudo docker exec -it ollama ollama run codellama:7b
```

This whole setup effectively has the same result as the initial setup that ran on my laptop but with the advantage of much better hardware and the possibility to use GPU, making it possible to use larger models.

## Web UI

Now that I got Ollama set and running on a dedicated machine, I can start to explore some integrations. The first thing I wanted was to have a UI where I could interact with the model, something similar to the ChatGPT UI. Lucky me, many open-source projects provide such UI: [LlamaGPT](https://github.com/getumbrel/llama-gpt), [Chatbot Ollama](https://github.com/ivanfioravanti/chatbot-ollama) and [Ollama Web UI](https://github.com/ollama-webui/ollama-webui) to mention a few.

I went with Ollama Web UI because it fits my use case the best and has good documentation. LlamaGPT is by far the most popular of them. That said, it is meant as an easy-to-deploy solution with a simple setup but limited.

I've set up Ollama Web UI on my NAS using Docker Compose. After cloning their Git project, I ran the following command to build the Docker image with the configuration for my Ollama server.

```shell
sudo docker build --build-arg OLLAMA_API_BASE_URL='http://my.ollama.server:11434/api' -t ollama-webui .
```

They don't provide a `docker-compose.yml` file, but creating one is very simple.

```yml
version: '3.3'

services:
  ollama-webui:
    image: ollama-webui:latest
    restart: always
    ports:
      - "3000:8080"
```

Running `sudo docker-compose -f ./docker-compose.yml up -d` starts the container and I can access the web UI at `http://my.nas.server:3000`.

![Screenshot of the Ollama Web UI interface showing a dropdown selector to choose one of the available LLMs. At the bottom, there are a few examples of things you can ask the model and an input field where you can write a message.](/coding_with_llms/ollama_ollama_web_ui.png)

I like that it allows me to select different models depending on what is available on my Ollama server, which is great for trying out different LLMs and comparing results.

The ultimate goal here is to use these LLMs for different applications that help me to write code more efficiently. Let's see how far it can go.
