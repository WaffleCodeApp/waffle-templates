FROM mcr.microsoft.com/vscode/devcontainers/python:0-3.11-bullseye
RUN pip install --upgrade pip

# curl
RUN type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
# GitHub CLI
RUN curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
    && sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && sudo apt update
RUN apt install gh -y

RUN mkdir /workspaces
RUN mkdir /var/task

WORKDIR /workspaces
