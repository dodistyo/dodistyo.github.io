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
It could take hours or even days to set everything up, install all of the necessary tooling, and set up the IDE and its extensions.

If you're a psycho, you'll probably be using
neovim with 500+ lines of init.lua, custom LSP, treesitter, and a Tokyo Night colorscheme. Keyboard shortcuts so advanced even you forget them. Brags about how you wrote your own plugin manager or something...

But of course, you are a normal person, so you use VSCode instead.
Installing it in your desktops is pretty quick. and if you're on linux with zsh as your shell terminal, it will surely increase your productivity.
but in Windows, you need to properly set up your wsl first before you can use zsh.
Could you imagine if you switch devices often? keeping your workspace state the same across devices is not easy, right?
Now you're thinking... and then realize that VSCode is actually built with JavaScript.
Since it is built with JS, it should be possible to run it in the browser, right?

## Cloud Development Environment
Have you used GitHub Codespace for coding yet?
GitHub Codespace is a CDE.
A `Cloud Development Environment` is a remote-accessible coding workspace hosted on cloud or self-hosted infrastructure, allowing developers to write, build, and test software from anywhere using just a browser or lightweight client.

As we know it, Codespace is tightly integrated with GitHub services.
Is there any alternative?

Unfortunately, we can't natively run VSCode in the browser. It was built using electron and specifically targeting desktops.

But don't worry. There are some projects being developed around it. To keep the main VSCode feature, they're developing it by forking the core component of the original VSCode. Some popular projects including openvscode, which is the data plane component used by GitHub Codespace and Gitpod.

There is another interesting project called code-server (also VSCode fork). As for now, it has more github stars compared to openvscode.
Basically, they both over self-hosted VSCode that is accessable via browser.