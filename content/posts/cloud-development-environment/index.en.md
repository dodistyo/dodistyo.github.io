---
weight: 100
title: "Take Your Dev Setup Anywhere with CDE"
date: 2025-07-20T19:30:00+07:00
draft: true
author: "Dodi Prasetyo"
description: "A casual exploration into cloud development environments and browser-based IDE."
tags: ["Development Environment", "Cloud IDE", "VSCode", "Kubernetes"]
categories: ["Dev Experience", "Platform"]
---

Do you enjoy setting up a development environment each time you switch devices?

Well, I don't...

Setting up workspace is a tedious work, especially when you got a new device or just simply want to switch for another device.
<!--more-->
It could take hours or even days to install all of the necessary tooling, and set up the IDE and its extensions.

If you're a psycho, you'll probably be using
neovim as your IDE with 500+ lines of init.lua, custom LSP, treesitter, and a Tokyo Night colorscheme. Keyboard shortcuts so advanced even you forget them. Brags about how you wrote your own plugin manager or something...

But of course, you are a normal person, so you use VSCode instead.
Installing it in your desktops is pretty quick. and if you're on linux with zsh as your shell terminal, it will surely increase your productivity.
but in Windows, you need to properly set up your wsl first before you can use zsh.
Could you imagine if you switch devices often? keeping your workspace state the same across devices is not easy, right?
Now you're thinking... and then realize that VSCode is actually built with JavaScript.
Since it is built with JS, it should be possible to run it in the browser, right?

## Cloud Development Environment
Have you used GitHub Codespace for coding yet?
GitHub Codespace is so awesome. The user experience is near identical with VSCode Desktop. We can even directly expose and access our developed application by involving some proxying and port forwarding. GitHub Codespace is considered a CDE. And another CDE worth the mention is Gitpod.

A `Cloud Development Environment` is a remote-accessible coding workspace hosted on cloud or self-hosted infrastructure, allowing developers to write, build, and test software from anywhere using just a browser or lightweight client.

As we know it, Codespace is tightly integrated with GitHub services.
So is there any self-hosted/opensource alternative?

You might think hosting VSCode OSS in the browser is possible since it is made from javascript.

But unfortunately, we can't natively run VSCode in the browser. Since it was built using electron and specifically targeting desktop.

But don't worry. There are some projects being developed around VSCode. To keep the main VSCode experience and feature, they're developing them by forking the core component of the original VSCode OSS project. Some popular projects including openvscode, which is the data plane component used by GitHub Codespace and Gitpod.

There is another interesting project called code-server (also VSCode fork). As for now, it is more popular compared to openvscode.
But asically, they both over self-hosted VSCode that is accessable via browser.

Cool, right?

But there is no way we will host a single server for each developer.
It's not feasible. We need some kind of management tool for managing developer workpace.
In this blog, we'll take a look at the opensource project called `coder`. Coder is a CDE platform that can be self hosted. 
by the way, coder team is also the maintainer of the code-server project. and code-server is part of the coder component.

##  Kubernetes as CDE Orchestrator
By levereging the dynamic and the robustness of kubernetes, we'll get a strong platform to host our CDE manager and it's workspace data plane.
We will provision the coder controller first, then we will create a workspace template so users can use the template and spawn their workspace dynamically on demand.

Before we do the demo, here are some coder OSS key features:

## Core Features of Coder OSS

- **Self-Hosted Cloud Development Environments**  
  Run locally or in your cloud infrastructure (AWS, GCP, Azure, on-prem). Self-host all components for full control and no vendor lock-in.

- **Consistent Dev Environments via Templates**  
  Define standardized workspace templates (Docker, Kubernetes, VMs) using Terraform to ensure all developers work in identical environments.

- **IDE Flexibility**  
  Supports web-based IDEs like code-server and JetBrains Projector, plus desktop IDEs via SSH (VS Code Remote, JetBrains Gateway, Emacs, Jupyter, etc.).

- **Ephemeral + Persistent Workspace State**  
  Workspaces have ephemeral resources (e.g., runtime, secrets) and persistent ones (e.g., source code, dotfiles). When stopped, only persistent state remains, saving resources.

- **Cost Control via Auto-Shutdown**  
  Automatically stops inactive workspaces to reduce cloud costs while preserving the persistent workspace disk.

- **Web UI, CLI & API Access**  
  Provision and manage workspaces via web dashboard, command-line interface, or API—usable by both admins and developers.

- **Wide Platform Support**  
  Supports Linux, Windows, macOS, and ARM architectures. Workspaces can run as VMs, containers, or Kubernetes pods.

- **Open Source License & Community Edition**  
  Licensed under AGPL-3.0, offering unlimited templates, workspaces, members, IDE integrations, AI agents, and community support for free.

## Demo