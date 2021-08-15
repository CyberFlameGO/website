---
title: Developers can't seem to agree on what command should get rid of a package
description: Every package manager should accept both "uninstall" and "remove" commands
date: 2021-05-26
tags:
    - open-source
layout: layouts/post.njk
---

**Disclaimer**: This blog post complains about something inconsequential and silly. However, it's worth talking about.

Has this ever happened to you?

```bash
$ sudo snap uninstall x
error: unknown command "uninstall", see 'snap help'.
```
Me too! I'm used to apt, so I often try to "uninstall" with snap when the correct verb is "remove".

Then I go work on a Python project and I do this:
```shell
$ pip remove x
ERROR: unknown command "remove"
```
Whoops. pip uses "uninstall" just like apt. Then I connect to a machine running CentOS and try to uninstall a package using yum:
```shell
$ yum uninstall x
No such command: uninstall. Please use /usr/bin/yum --help
It could be a YUM plugin command, try: "yum install 'dnf-command(uninstall)'"
```
Oh, of course. yum uses "remove". 

The only program I know of that does it right is npm, accepting both "uninstall" and "remove". Why doesn't every package manager do this?

In case you're curious, here are some other package managers:
<table class="prettytable">
<tr>
    <th>Package manager</th>
    <th>Verb</th>
</tr>
<tr>
    <td>Brew</td>
    <td>uninstall</td>
</tr>
<tr>
    <td>yay</td>
    <td>-Rns 🙄</td>
</tr>
<tr>
    <td>vcpkg</td>
    <td>remove</td>
</tr>
<tr>
    <td>cargo</td>
    <td>uninstall</td>
</tr>
<tr>
    <td>NuGet</td>
    <td>delete</td>
</tr>
<tr>
    <td>conda</td>
    <td>remove (it makes me irrationally angry that this is different from pip)</td>
</tr>
<tr>
    <td>yarn</td>
    <td>remove (and only remove)</td>
</tr>
<tr>
    <td>flatpak</td>
    <td>uninstall</td>
</tr>
<tr>
    <td>choco</td>
    <td>uninstall</td>
</tr>
</table>

If you leave a comment with any more I'll add it to the table.

<h2 id="comments">Comments (62)</h2>

Discuss on [Reddit](https://www.reddit.com/r/programming/comments/nlb2c9/developers_cant_seem_to_agree_on_what_command/) / [Hacker News](https://news.ycombinator.com/item?id=27287455)