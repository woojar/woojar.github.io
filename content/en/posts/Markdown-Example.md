---
title: "Markdown example"
date: 2022-09-03T00:00:00+08:00
aliases: ["/2022/09/03/Markdown-Example/"]
draft: false
tags: ['tech','markdown']
categories: ["Tech"]
---

## General text

``` markdown
#this is header

**hi, this is bold**

*this is italic font*

this is another line

## this is header two
```

You’re really, really going to want to see this.
Shortcut for markdown all in one[preview][toggle preview]

[Visit Markdown Guide](https://www.markdownguide.org)
![Alt text for image](/images/like.svg)

## Key	Command
Ctrl/Cmd + B	Toggle bold
Ctrl/Cmd + I	Toggle italic
Alt+S (on Windows)	Toggle strikethrough1
Ctrl + Shift + ]	Toggle heading (uplevel)
Ctrl + Shift + [	Toggle heading (downlevel)
Ctrl/Cmd + M	Toggle math environment
Alt + C	Check/Uncheck task list item
[toggle preview]: Ctrl/Cmd + Shift + V	Toggle preview
Ctrl/Cmd + K V	Toggle preview to side
The latest news from BBCimage link[orange cat]: 
Inline link[orange cat][orange cat]

## blockquotes
>The sin of doing nothing is the deadliest of all the seven sins. It has been said that for evil men to accomplish their purpose it is only necessary that good men should do nothing.


>this is a odd paragraph

## Lists
- one
- two

- one
- two

- hello
    - hello again
* one
* two
* three
  
## paragraphs

We pictured the meek mild creatures where
They dwelt in their strawy pen,
Nor did it occur to one of us there

- To doubt they were kneeling then.

## Code block
```cpp

#include <iostrem>
int main()
{
    std::cout << "Hello cpp world" << "/n";
    return 0;
}

```

## Tables

<!--more-->

| Feature | Syntax |
| :--- | :--- |
| Bold | `**text**` |
| Italic | `*text*` |
