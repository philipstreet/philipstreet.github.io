---
layout: post
title: Issues when using Scoop to install Git on Windows
tags: windows scoop git gcm WSL
image: 2024-05-13-git-clone-prompt-password.png
---
Using [Scoop](https://scoop.sh){:target="_blank"} to install tools without elevated privileges on Windows 11, especially in an Enterprise setting, is super helpful but there can be some knock-on effects to other apps & configuration. One such issue I discovered was to do with Git and Git Credential Manager, so let's take a look at that.

## Installing Git for Windows

Install Git on Windows is easy, just run the following;

{% highlight powershell %}
scoop install main\git
{% endhighlight %}

I first realised something was wrong when I was trying to clone a repo from an Azure DevOps organisation I had not worked with for several weeks. I was getting the following;

{% highlight powershell %}
❯  git clone https://[organisation]@dev.azure.com/[organisation]/[project-name]/_git/[repository-name]
Cloning into '[repository-name]'...
Password for 'https://[organisation]@dev.azure.com':
{% endhighlight %}

That's weird! I'm sure I wasn't prompted to enter a password the last time I did this...In fact, I'm pretty sure I just logged in via the usual Microsoft Entra ID authentication flow (including MFA). The only thing that had changed was that I had re-installed Git using Scoop.

## So, what's missing?

After a bit of frantic Googling, I realised that [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager){:target="_blank"} had not been installed. Doh!

this is easily resolved by simply installing GCM using Scoop:

{% highlight powershell %}
scoop bucket add extras
scoop install extras/git-credential-manager
{% endhighlight %}

Additionally, I need to configure Git to use GCM:

{% highlight powershell %}
git config --global credential.store manager
git config --global credential.helper manager
{% endhighlight %}

## What about WSL?

The above will fix things for Windows, but what about WSL? Well, you _can_ install GCM in WSL as well but means having separate credential stores. The best experience is let GCM share credentials & settings between WSL and the Windows host. This can be achieved by doing the following.

_Inside your WSL installation_, run the following command to set GCM as the Git credential helper installed by Scoop, which will not be in the default location indicated on [GCM on Windows Subsystem for Linux (WSL)](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/wsl.md):

{% highlight bash %}
git config --global credential.helper "/mnt/c/Users/[username]/scoop/shims/git-credential-manager.exe"

# For Azure DevOps support only
git config --global credential.https://dev.azure.com.useHttpPath true
{% endhighlight %}

> **Note:** change [username] so that the path is valid.

## Wrap-up

I learned two things from this experience:

1. Always check what is - and what isn't - being installed by Scoop.
1. Additional configuration may be required when performing a user-only install, instead of a system-wide install.

## Conclusion

I hope you've found this useful.

As always, if you think there is a better way this could've been fixed then please leave a comment below.