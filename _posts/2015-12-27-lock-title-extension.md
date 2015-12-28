---
layout: post
title: Chrome extension "Lock Title"
---

I often compare multiple charts or dashbaords on different Chrome tabs, each of which plots the same metrics of different instances (e.g. request latency of instance 1 and 2).
When I open ten such tabs, the title displayed on the tab is the easiest way to tell which tab shows which. The application that renders those charts, however, may not correctly set the title.
It would be handy if I override the title on the tab with a string that helps me differenciate the contents.

[Lock Title](https://chrome.google.com/webstore/detail/lock-title/cjkldkjflehlglaenengohmjddgfhlec?hl=en-US) is a Chrome extension that allows you to lock tab title with whatever you specify. (I couldn't find such an extension on Chrome web store.) Here's how to use it.

Right click on the page of which you want to override the title, and select "Lock tab title"
![choose "Lock tab title" on context menu](/public/assets/201512/contextmenu.png){: .main-image }

Then, type in the string you want to set as the title.
![type in tab title](/public/assets/201512/prompt.png){: .main-image }

The string appears on the tab.
![Title has been updated](/public/assets/201512/locked.png){: .main-image }

The title is locked until the browser shuts down, or explicitely unlocked by setting an empty string in the prompt dialog.


Developing such a simple chrome extension was actually straightfoward. It needed a bit more hassle to upload it to Chrome web store, though.
