---
layout: articles
title: Android 为TabHost添加标签页 报错
tags: others android
date: 2017-04-07
category: android
---
# Android 为TabHost添加标签页报错

``` java
private void initTabView(){
    TabHost tab = (TabHost) findViewById(R.id.th_main);
    tab.setup();// <---这行wtf非常重要！
    tab.addTab(tab.newTabSpec("tab_login").setIndicator("登录").setContent(R.id.tab_login));
    tab.addTab(tab.newTabSpec("tab_sdk").setIndicator("SDK").setContent(R.id.tab_sdk));
    tab.addTab(tab.newTabSpec("tab_notify").setIndicator("发个推送").setContent(R.id.tab_notify));
}
```

在为TabHost添加标签页之前必须调用一下tabhost.setup()  

私以为这api设计是有问题的，即使是必须不得不如此，那能不能在「没有调用setup就添加标签页」的时候提示一下「添加标签页之前必须调一下setup」?
