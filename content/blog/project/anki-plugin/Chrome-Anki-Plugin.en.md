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
    - Chatgpt
    - Anki
    - Chrome Extensions
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
### Showing selected words
First ,I would like to build simple demo plugin , that can select words from a webpage  

The simple Prompts is 
```
Creating a Chrome extension that selects words from a webpage 
```

The [chatgpt demo](https://github.com/erzhiqianyi/anki_chrome_plugin/tree/dev/chatgpt-demo) 
is work as expected.When select some words in the webpage ,then click the plugin and the select button , 
it will show an alert window and show the selected words.

![](../01-select-word-plugin.png)

There is one issue in this demo . 
```
Manifest version 2 is deprecated, and support will be removed in 2023.
```
Need change the manifest version to 3 .
[mv2-transition](https://developer.chrome.com/blog/mv2-transition/)

It is easy to fix the issue for this demo . 
change version from 2 to 3 and rename browser action to action 
Here is the v3 manifest file 
```json
{
  "manifest_version": 3,
  "name": "Word Selector Extension",
  "version": "1.0",
  "description": "Select words from a webpage.",
  "permissions": [
    "activeTab"
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "icons": {
    "16": "images/icon16.png",
    "48": "images/icon48.png",
    "128": "images/icon128.png"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["contentScript.js"]
    }
  ],
  "permissions": [
    "activeTab",
    "storage"
  ]
}
```
### Adding a context menu

Then I would like to add context menu that
provide a convenient way for users to select words directly from 
the right-click menu.

The simple Prompts is

```
Adding a context menu
```
[Here](https://chat.openai.com/share/2e129dab-91bc-4b64-9f13-40f2af2fd1ed) is the detail message 

![](../02-add_context_menu.png)

In this step , need add permissions for contextMenus in the permissions node 
```json
{
  "permissions": [
    "activeTab",
    "storage",
    "contextMenus"
  ]
}
```

Then add a background node to reference the service_worker js
```json
{
  "background": {
    "service_worker": "background.js"
  }
}
```

And create the background.js to create contextmenu.
```js
chrome.contextMenus.create({
  id: "wordSelection",
  title: "Select Words",
  contexts: ["selection"]
});

chrome.contextMenus.onClicked.addListener(function (info, tab) {
  if (info.menuItemId === "wordSelection") {
    const selectedText = info.selectionText;
    chrome.tabs.sendMessage(tab.id, { action: "selectWords", selectedText });
  }
});

```
Update the plugin ,then can see the contextmenu

![](../03-contextmenu.png)

### Adding card to Anki
See [How to manage Anki By Api](../how-to-manage-anki-by-api) for more information about add note to anki.
Ask chatgpt to send the word to a specified server
```
send a http request 
```
Here is the function to send a new card to Anki .
Need to create a deck and a model first.
```javascript
function sendSelectedWords(words) {
    //default url for anki server
  const url = "http://127.0.0.1:8765"; 
  

  add_words = {
    "action": "addNote",
    "version": 6,
    "params": {
      "note": {
        "deckName": "test_decl",
        "modelName": "TestModel",
        "fields": {
          "Field1": words,
          "Field2": words
        },
        "options": {
          "allowDuplicate": false
        },
        "tags": [
          "tester"
        ]
      }
    }
  }
  fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(add_words),
  })
          .then(response => response.json())
          .then(data => {
            console.log("Server response:", data);
          })
          .catch(error => {
            console.error("Error sending request:", error);
          });
}
```

This function creates a basic deck with identical values for the front and back sides.