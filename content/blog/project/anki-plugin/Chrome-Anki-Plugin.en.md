---
title: "Chrome Anki word add plugin" 
description: A side project 
draft: false
language: en
date: 2023-08-20
updated: 2023-08-20
taxonomies:
  categories:
    - In Progress 
  tags:
    - Project
---

My first side project , An Chrome plugin ,than can add words to Anki.
<!-- more -->

## Project Address
[ankichan](https://github.com/erzhiqianyi/anki_chrome_plugin)

## Background
I am learning Japanese and using Anki to remember words.
Since I mainly read Japanese online, I want to automatically add unfamiliar words to Anki to help with memorization.

Although there are applications that can achieve this functionality, like Yomichan.
But I'm not very fond of Yomichan's interface, so I've decided to create my own tool with the help of ChatGPT.

I don't have experience developing browser extensions.
So, I've decided to give it a try using ChatGPT for programming.

## Demo
First ,I would like to build simple demo plugin , that can select words from a webpage  

The simple Prompts is 
```
Creating a Chrome extension that selects words from a webpage 
```

The [chatgpt demo](https://github.com/erzhiqianyi/anki_chrome_plugin/tree/dev/chatgpt-demo) 
is work as expected.When select some words in the webpage ,then click the plugin and the select button , 
it will show an alert window and show the selected words.

![](../select-word-plugin.png)
