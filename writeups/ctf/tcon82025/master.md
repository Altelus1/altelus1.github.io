---
layout: page
title: Master
category: Web
type: ctf_tcon82025
desc: Master
img-link: /writeups/ctf/tcon82025/img/master/1.png
---


# [Web] Master - 200

We are given with the link to to https://master.project-ag.org

![Master Home Page](img/master/1.png)

What stands out immediately is the admin panel. But we need to have valid creds, with admin privs, to access it.

![Master Admin Panel Error](img/master/2.png)

Checking out the source HTML, we can see that an Authorization Bearer may be needed to access the admin. Possilby a JWT string.

![Master Admin Panel Error](img/master/3.png)

So we tried to construct in Cyberchef but instead of using any Signing Algorithm, we just put `"alg":"none"` within the header. We also wanted to test if the `alg` header parameter is being taken into consideration.

![Master Admin Panel Error](img/master/4.png)

The string above actually worked but with "Access Denied" due to insufficient privileges. So we tried different privilege parameters or usernames within the body of JWT. What had workd was `"role":"admin"`.

![Master Admin Panel Error](img/master/5.5.png)

![Master Admin Panel Error](img/master/5.png)

With this, we got the flag.

![Master Admin Panel Error](img/master/6.png)
