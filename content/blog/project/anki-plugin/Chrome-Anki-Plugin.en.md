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

### Adding Note to Anki
See [How to manage Anki By Api](../how-to-manage-anki-by-api) for more information about add note to anki.
Ask chatgpt to send the word to a specified server
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

### Creating Word Assistant Prompt 
Can ask chatgpt to create a Word Assistant Prompt.
I already created one . 
```
Please provide definitions for the following Japanese words. 
Definitions should include the word, its kana, romaji, definition, Chinese translation, and part of speech. 
Additionally, provide 5 example sentences at the N1 level,including kana and Chinese translations. 
Output the information in JSON format.
The JSON format like '''{ word: 学生, kana: がくせい, romaji: gakusei, definition: student, chinese: 学生, speech: noun, sentences: [ { sentence: 私は大学生です, kana: わたしはだいがくせいです, chinese: 我是大学生 } ] }'''
```

Another version is in Japanese 

```
日本語の単語に対して日本語の定義を提供し、
その後単語、ひらがな、ローマ字、定義、中国語訳、品詞を出力してください。
定義はできるだけ詳細に、N1レベルの語彙を参考にしてください。
定義は日本語で出力してください。
また、N1レベルの例文5つを提供し、それぞれの例文にひらがなと中国語訳を含めてください。
出力はJSON形式です。The JSON format like 
'''{ word: 学生, kana: がくせい, romaji: gakusei, definition: student, chinese: 学生, speech: noun, sentences: [ { sentence: 私は大学生です, kana: わたしはだいがくせいです, chinese: 我是大学生 } ] }'''

```
See [How To Create Prompt](../../chatgpt/how-to-create-prompt) for more details

### Getting Definition From Openai
The main feature of this plugin is to explain words and provide example sentences using ChatGPT.
You can customize the learning material instead of relying on a dictionary.

We can use a cloudflare to proxy openai api ,Because some region can't access to openai api .
Here is a simple worker to proxy openai api

See [Openai api Proxy](https://github.com/erzhiqianyi/openai-api-proxy) for more details.

Here is function that get Definition from Openai

```javascript
function explainFromOpenAi(word) {
    const url = "xxx"
    const token = "xxx"
    const wordData = {
        "model": "gpt-3.5-turbo",
        "messages": [
            {
                "role": "system",
                "content": "日本語の単語に対して日本語の定義を提供し、 その後単語、ひらがな、ローマ字、定義、中国語訳、品詞を出力してください。 定義はできるだけ詳細に、N1レベルの語彙を参考にしてください。 定義は日本語で出力してください。 また、N1レベルの例文5つを提供し、それぞれの例文にひらがなと中国語訳を含めてください。 出力はJSON形式です。The JSON format like '''{ word: 学生, kana: がくせい, romaji: gakusei, definition: student, chinese: 学生, speech: noun, sentences: [ { sentence: 私は大学生です, kana: わたしはだいがくせいです, chinese: 我是大学生 } ] }'''"
            },
            {
                "role": "user",
                "content": word
            }
        ]
    }
    return fetch(url, {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            "Authorization": token
        },
        body: JSON.stringify(wordData),
    })
        .then(response => response.json())
        .then(data => {
            return data
        })
        .catch(error => {
            console.error("Error sending request:", error);
        });

}
```

### Converting Definition and Sentences to Anki Note
For the purpose of simplifying the demo, I will use a pre-defined card template.

Here is the simple createModel payload
```json
{
  "action": "createModel",
  "version": 6,
  "params": {
    "modelName": "MyWordNote",
    "inOrderFields": [
      "word",
      "speech",
      "chinese",
      "definition",
      "sentence",
      "sentence_translation"
    ],
    "css": "@font-face {\n font-family: IPAexMincho; src: url('_ipaexm.ttf');\n}\n\n.jp {\n  font-family: IPAexMincho;\n}\n\n.card {\n font-family: arial;\n font-size: 20px;\n text-align: center;\n color: black;\n background-color: white;\n}",
    "isCloze": false,
    "cardTemplates": [
      {
        "Name": "WordDefinition",
        "Front": "<span class=\"jp\">{{furigana:word}}</span> {{speech}}",
        "Back": "{{FrontSide}}\n\n<hr id=answer>\n\n<span class=\"jp\">{{furigana:word}}</span> <hr>{{definition}}<hr>{{chinese}}"
      },
      {
        "Name": "Sentence",
        "Front": "<span class=\"jp\">{{sentence}}</span>",
        "Back": "{{FrontSide}}\n\n<hr id=answer>\n\n<span class=\"jp\">{{furigana:sentence_translation}}</span> "
      }
    ]
  }
}

```

And add a new Note
```json
{
  "action": "addNote",
  "version": 6,
  "params": {
    "note": {
      "deckName": "test_decl",
      "modelName": "MyWordNote006",
      "fields": {
        "word": "学生[がくせい]",
        "speech": "[名词]",
        "definition": "A person who is enrolled in school and studying",
        "chinese": "学生",
        "sentence": "私は学生です",
        "sentence_translation": "I am a college student"
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
```
The word definition card front
![Front](../04-definition-card-front.png)

The word definition card back
![Back](../05-definition-card-back.png)

The sentence card front
![Front](../06-sentence-card-front.png)

The sentence card back
![Front](../07-sentence-card-back.png)


