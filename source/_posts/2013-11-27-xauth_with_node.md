---
layout: post
title: "Node's XAuth Support"
description: "Node based XAuth client."
category: 
tags: [node, xauth, fanfou]
---

[node-oauth](https://github.com/ciaranj/node-oauth)模块提供了OAuth和XAuth支持，根据[stackoverflow](http://stackoverflow.com/questions/7518795/instapaper-api-javascript-xauth/9645033#9645033)提供的信息，使用如下接口即可:

    getOAuthRequestToken(extra_params, callback_function)
其中extra_params:
    
    extra_params = {
        "x_auth_username":"__username__",
        "x_auth_password":"__password__",
        "x_auth_mode":"client_auth"
    }
    
饭否的XAuth实测可用。
