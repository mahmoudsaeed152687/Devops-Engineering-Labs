1. Problem

Create a Linux user named yousuf on App Server 3 with an account expiry date of April 15, 2027.

The important requirements are:

Username: yousuf — lowercase
Server: App Server 3
Expiry: 2027-04-15

2. How to Think

The task has two separate requirements:

Create a user → use useradd.
Set an account expiration date → useradd supports this with the -e option.

So we can handle both requirements in a single command.

3. Solution

On App Server 3:

sudo useradd -e 2027-04-15 yousuf

Here:

useradd → creates the user.
-e 2027-04-15 → sets the account expiration date.
yousuf → creates the user with the required lowercase username.

4. Verification

Check the user's account information:

sudo chage -l yousuf

