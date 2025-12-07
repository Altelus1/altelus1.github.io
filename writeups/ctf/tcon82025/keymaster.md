---
layout: page
title: Keymaster
category: Web
type: ctf_tcon82025
desc: Keymaster
img-link: /writeups/ctf/tcon82025/img/keymaseter/1.png
---


# [Web] Keymaster part 1 - 400

We are given with the link to to https://keymaster.project-ag.org

![Keymaster Home Page](img/keymaster/1.png)

There's some registration functionality within the website we successfully registered an account.

![Dashboard HTML](img/keymaster/2.png)

When we tried logging in, it returns also a JWT string for succeeding authentications.

![Dashboard flag](img/keymaster/3.png)

In the main UI (no screenshots, sorry :( ), you can create your own secret notes and then edit or delete them. As you can see below, we have no secrets yet.

![Dashboard flag](img/keymaster/4.png)

However, turns out there's an IDOR vulnerability allowing us to access other people's secret notes by adding a numeric value after `/api/secrets/` like `/api/secrets/1`.

![Dashboard flag](img/keymaster/5.png)

So we tried extracting other secrets through scripting.

![Dashboard flag](img/keymaster/6.png)

Once done, here's the result we managed to obtain:

![Dashboard flag](img/keymaster/7.png)

Out of all of this, what caught interest us is this specific secret note that seems to be a signing key for JWT.

![Dashboard flag](img/keymaster/8.png)

So we went to Cyberchef and sign our own JWT with admin privs...

![Dashboard flag](img/keymaster/9.png)

And when we set it on our Authorization header, it says that we are already admin!

![Dashboard flag](img/keymaster/10.png)

At this point of the challenge, we were stuck. Fortunately, we thought of trying to do insert an SQL Injection payload within the user\_id parameter from JWT body we have.

![Dashboard flag](img/keymaster/11.png)

As can be seen above, an SQL error returned. So we proceeded with checking out the columns:

![Dashboard flag](img/keymaster/12.png)

![Dashboard flag](img/keymaster/13.png)

We successfully reflected UNION injection with 1,2,3 columns values. Next we tried to see what database we're targetting. What worked is `version()` in the second column.


![Dashboard flag](img/keymaster/14.png)

![Dashboard flag](img/keymaster/15.png)

And we confirmed it's a PostgresQL database. Then we tried grabbing all tables from PostgreSQL server.

![Dashboard flag](img/keymaster/16.png)

![Dashboard flag](img/keymaster/17.png)

And then, as can be seen above, there's "flags" table that stands out a bit. So we tried querying its columns.

![Dashboard flag](img/keymaster/18.png)

![Dashboard flag](img/keymaster/19.png)

It has "id", "flag", and "description" columns. We can grab those directly.

![Dashboard flag](img/keymaster/20.png)

![Dashboard flag](img/keymaster/21.png)

And we got the flag!

# [Web] Keymaster Part 2 - 100

The part requires another flag and when we dumped all the secret notes, we have also found a string that has a tcon flag format. We submitted it and it worked.

![Dashboard flag](img/keymaster/22.png)

