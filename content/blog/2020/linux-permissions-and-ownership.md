+++
author = "Udara Bibile"
authors = ["s"]
authorImage = "/img/udarabibile.jpg"
title = "Linux Snippet: File Permissions & Ownership"
date = "2020-03-30"
description = "Using chmod and chown to work with permissions and ownership"
tags = ["chown", "chmod"]
categories = ["linux", "os"]
images  = ["img/2020/linux-permissions-cmd.jpeg"]
type = "post"
aliases = ["migrate-from-jekyl"]
draft = false
+++

Users familiar with Linux based operating systems should have at least once come across issues related to `Permission denied` when executing some commands in the terminal. This article aims to shed some light on user permissions and ownership related to files and directories.

<hr/>

In order to get an introduction to permissions and ownership, in the **terminal**, navigate to a location where files resides and execute `ls -al` to list files and directories with all associated details.

```
drwxr-xr-x  12 bibi         staff     384 16 Mar 11:19 public
-rw-r--r--   1 bibi         staff    1660 16 Mar 11:51 note.txt
```

Let’s break down what each segment of these entries stands for:

<img class="image featured" src="/img/2020/linux-permissions-cmd.png" alt="" />

<!--
<figure>
  <img class="image featured" src="/img/2020/linux-permissions-cmd.jpeg" alt="" style="width:100%" >
  <figcaption style="text-align:center">Permissions for start.sh script.</figcaption>
</figure>
-->

**Note**: File type could either be `-` for file and `d` for directory.

<hr/>

## File Permissions:

This section tries to capture more on **permissions** chunk introduced above, and how permissions are useful in Linux based operating systems.

### User types include:
* **User** → Owner of file. Creator of file will be owner unless changed later.
* **Group** → User group of file, which should also include owner of file.
* **Other** → All other users or groups excluding above two categories.

### Permission types include:
* **Read** → **For file**: view or copy file, **For directory**: list or copy files.
* **Write** → **For file**: modify file, **For directory**: add or delete files.
* **Execute** → **For file**: run if executable, **For directory**: enter directory.

There 3 types of users, and 3 types of permissions when considering files and directories. Therefore 9 combinations should be used to express how files or directories interact to a given user and given permission.

Hence 9 characters can **define permission types for each of the users, and this will define access levels for each user on a given file or directory**.


<figure>
  <img class="image featured" src="/img/2020/linux-permissions-table.jpeg" alt="" style="width:100%" >
  <figcaption style="text-align:center">Permissions for start.sh script.</figcaption>
</figure>

As per this example, if permission is granted it will be shown by alphabetic value such as `r`, `w`, `x` and if relevant permission is not granted it is denoted by `-`. So for this example:

* **User** have **read, write and execute** privileges. Thereby the owner can read this script, edit this script, and execute this script.
* **Group** have **read, write** privileges but not execute privileges. Hence users in the group will only be able to read, edit script but can’t execute script.
* **Others** have only **read** privileges, where any other user not the owner or in the owner group will only be able read content of the script.

<hr/>

### Using chmod command to change permissions:

File permissions can be altered according to need and `chmod` command is used. Here `chmod` simply meant for **change mode**.


#### 1. Using chmod with symbolic values

**User types** defined as follows: user → `u`, group → `g`, other → `o`, all → `a`.

**Permission types** defined as follows: read → `r`, write → `w`, execute → `x`.

Multiple operators can also be used as follows: add new permission → `+`, remove existing permission → `-`, set exact permissions → `=`.

Let's follow up with examples:

* **`chmod g+x abc.sh`** → This will **add executable** permission to **group**, and other permissions values for read and write remains the same for the group.
* **`chmod o=r abc.txt`** → This will **set exact** permissions to **others**, where only read permission is attached and will remove write and execute values.
* **`chmod a-w abc.sh`** → This will **remove** write permission from **all**, and doesn’t alter read or execute permissions related to the user.

For ease of use multiple users and permissions can be altered at once too:

```
chmod u=rwx,g=rx,o=r start.sh
```

**Note**: From stack overflow answers, you might have come across commands like **`chmod +x ./command.sh`**, which just means **`a+x`** command.

<hr/>

#### 2. Using chmod with octal values

Each user permission is declared using 3-digit binary value which contains `1` if that permission is granted, else given `0`. Since 3-digits are used it can be interpreted by octal value from 0 to 7. Then 3 octal values used to interpret permissions for user, group, and others.

<figure>
  <img class="image featured" src="/img/2020/linux-permission-single-octal.jpeg" alt="" style="width:100%" >
  <figcaption style="text-align:center">Binary calculation for permission.</figcaption>
</figure>

Here binary combination includes as follows. `read = 1`, `write = 0`, and `execute = 0`. This builds up `110` binary value, and if you’re familiar with binary it can be seen that this corresponds to value `6`. Even if unfamiliar with binary convert it can be easily done with multiply using `x 4`, `x 2` or `x 1` and sum it up for value of `6`.

This should be repeated for all permissions for each user to end up with value final value. This seems 3-digit value can interpret all which permission types for which user types:

<img class="image featured" src="/img/2020/linux-permissions-octal.jpeg" alt="" />

So let's break down command `chmod 764 command.sh`:

* **User** have 7 as the digit. It’s 111 in binary, hence `r, w, x` → `u=rwx`
* **Group** have 6 as the digit. It’s 110 in binary, hence `r, w` → `g=rw`
* **Other** have 4 as the digit. It’s 100 in binary, hence `r` → `o=r`

`ls -l command.sh` should return `— rwx rw- r--` as per previous learning.

1. **Note:** Fine grained access can be acquired by user-permission map as given, hence its not a good practice to grant all access like `chmod 777`, which leads to security vulnerabilities.
2. **Note:** In order to add permissions to the content of the entire directory, recursive can be used like this `chmod -R 764 public/`. However, this is different from changing permissions just to parent directory `chmod 764 public`, and not its content.
3. **Note:** Similar to `ls -l`, Linux status command can also be used to check permissions and ownership details like in `stat run.sh`

<hr/>

##### Let’s follow up about permission change with an exercise:

* Create shell script using `nano run.sh` and add simple command:
```
#!/bin/bash
echo "Executed...!"
```
* Checking existing permissions using `ls -l run.sh` or `stat run.sh`.
```
- rw- r-- r--
```
* For ease of exercise, we will only use current user which is owner of file.
* Remove all permissions for users: `chmod 000 run.sh` and check for permissions list using the previous command: `--- --- ---`.
* Try to read `run.sh` using `less run.sh` → `run.sh: Permission denied`.
* Grant read permission using `chmod 400 run.sh` and check if it can be read.
* Try to write `run.sh` using `nano run.sh` and make a change and try to save → `[Error writing run.sh: Permission denied]`.
* Grant write permission also by `chmod 600 run.sh` and check if you can edit.
* Try to execute `run.sh` using `./run.sh` →
```
-bash: ./run.sh: Permission denied
```
* Grant execute permission by `chmod 700 run.sh` and execute script for output of `Executed...`!.
* Check for file permission using `stat run.sh` → `rwx --- ---`.

<hr/>

## File Ownership:

From the above introduction to file permissions and `chmod` command, it can be permissions for **owner** and **group**. Thereby it’s important to take a look into these owners, groups, and how it can be changed if required.

* **Owner** → Refers to the user who owns a given file or directory. Typically the creator of the file is attached as its owner until it’s changed.
* **Group** → Refers to a group who owns a given file or directory. Generally, the user creator can be part of multiple groups, but will have primary group. This group will be set by default when the user creates a file or directory.

Users and groups in Linux based systems are used to properly manage security and is an advanced topic to be discussed here. But in the terminal, a simple command such as groups will list down all the groups the current user is a part of.

In regard to files are directories, from the output of `ls -l` it reveals **permissions and ownership**, and output example is:
```
-rw-r--r--  1 bibi  staff     272 12 Mar 12:00 assets.json
```

Here `bibi` is the owner of **assets.json** and `staff` is the group owner of this file. Moreover we can understand how permissions are attached: `— rw- r-- r--`. So user `bibi` being owner is attached `rw-` permission to read and write to the file, thereby this user will have permission to read, write to this file but doesn’t have execute permission. Users in the group `staff` is attached `r--` permission, where it could only read file.

**Note**: One of the use cases of changing permissions or ownership is when copying to server from using `scp` or accessing server using `ssh`. Here the user copying the script might not have access to execute the updated script. Thereby either the permissions of the script need to be changed so the current user can execute, or either ownership of script is changed to the existing user so it can be executed.

<hr/>

#### Using chown command to change ownership:

Similar to `chmod` command used before, there is `chown` command to alter ownership of file or directory. Here `chown` simply meant to **change owner**.

* **`chown bibi assets.json`** → This will change **only user** ownership to a given user of that file or directory.
* **`chown bibi:staff assets.json`** → This will change **user** ownership and **group** ownership to a given user and given group of that file or directory.
* **`chown :staff assets.json`** → This will change **only group** ownership to given staff. Alternatively `chgrp` command can also be used to only change group like in `chgrp staff assets.json`.

<br/>
##### Let’s follow up about ownership change with an exercise:

* Switch to `root` user by `su root`, and create a text file with some content using `nano note.txt`. Verify that its attached to the current user of root using `stat note.txt` → `rw-` permission for user `root`:
```
-rw-r--r--  1 root  staff
```
* Switch back to user `bibi` by exiting `root`’s terminal and try to edit this file from `bibi` using `nano note.txt` → **`Permission denied`**
* Try using `chown` to change current owner from `root` to `bibi` so that current user will inherit `rw-` permission to it: `chown bibi note.txt` →
```
chown: note.txt: Operation not permitted
```
* This was restricted due to security factors, as this can lead to users changing ownership to modifying files that are not granted.
* Hence this can be changed by `root`’s terminal by `chown bibi note.txt` or using `sudo chown bibi note.txt` from `bibi`’s terminal.
* Verify that user ownership changed to `bibi`:
```
-rw-r--r--  1 bibi  staff
```
* Edit from `bibi` user using `nano note.txt` to verify that changed saved without any issue.

<hr/>

I hope this article was able to provide insight into **permissions** and **ownership** in Linux, and to use **`chmod`** and **`chown`** commands to alter permissions and owners for **`files`** and **`directories`**.

<br/><br/>
