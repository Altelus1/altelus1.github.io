---
layout: page
title: Dashboard
category: Web
type: ctf_tcon82025
desc: Dashboard
img-link: /writeups/ctf/tcon82025/img/dashboard/1.png
---


# [Web] Dashboard - 100

We are given with the link to to https://dashboard.project-ag.org

![Dashboard Home Page](img/dashboard/1.png)

Looking at the HTML source code, we can see the following base64 and some chinese characters as securityToken.

![Dashboard HTML](img/dashboard/2.png)

Turns out, the chinese characters are base65535 encoded and we decoded and got the flag:

![Dashboard flag](img/dashboard/3.png)

