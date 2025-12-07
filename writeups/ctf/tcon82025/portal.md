---
layout: page
title: Portal
category: Web
type: ctf_tcon82025
desc: Portal
img-link: /writeups/ctf/tcon82025/img/dashboard/1.png
---


# [Web] Portal - 150

We are given with the link to to https://portal.project-ag.org

![Portal Home Page](img/portal/1.png)

We can try clicking "Login" and a login page will be presented with working creds provided.

![Portal Login](img/portal/2.png)

Upon logging in, we can see that its main functionality is searching for employees.

![Portal Logged in](img/portal/3.png)

Searching for an employee with id of 1, it returns "employee@portal.com". Thus the functionality actually works and returns user.

![Portal Employee search](img/portal/4.png)

In these types of scenario, we can do a quick check for weaknesses in SQL injection. A payload like `2 OR 1=1-- -` can be a test case.

![Portal Employee search](img/portal/5.png)

As can be seen above, it returns employee 1 even though our main search is 2. This means that `OR 1=1-- -` actually executed and returns all employees (Turns out there is only 1 employee).

What we can do next is to test for the number of columns we need to successfully do a UNION based attack.

![Portal Employee search](img/portal/6.png)

Screenshot above shows that it is 5 columns. Screenshot below also shows that doing a UNION with 1,2,3,4,5 values returns 2 rows already.

![Portal Employee search](img/portal/7.png)

Next we need to search which column accepts a string. Screenshot below shows that the second column is the employee name thus accepts string.

![Portal Employee search](img/portal/8.png)

We can quickly get what database server we're dealing with. Turns out it is an SQLite version 3.46.1.

![Portal Employee search](img/portal/9.png)

Then we proceeded dumping the tables with payload below:

![Portal Employee search](img/portal/10.png)

Then we dumped the columns for "users" table.

![Portal Employee search](img/portal/11.png)

Then we dumped the "notes" columns of all users within the "users" table. Then we also got the flag.

![Portal Employee search](img/portal/12.png)
 
