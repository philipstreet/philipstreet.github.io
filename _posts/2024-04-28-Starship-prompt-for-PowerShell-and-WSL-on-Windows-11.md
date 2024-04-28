---
layout: post
title: Starship prompt for PowerShell & WSL on Windows 11
---
"One Prompt to config them all,<br>
One Prompt to style them,<br>
One Prompt to display in all<br>
and in the Shell bind them."<br>

by J. R. R. Tolkien....maybe.

## Introduction

I recently watched [Starship! One prompt to rule them all](https://www.youtube.com/watch?v=wXK4RGrBLuM&t=7s) by [Matt Field](https://www.youtube.com/@matt-ffffff) and loved the idea of having a consistent look and feel across all the prompts that I use on my Windows 11 machines, whether that be PowerShell or WSL Ubuntu in either Windows Terminal or Visual Studio Code. Such as;

![Windows Terminal](/images/2024-04-28-windows-terminal.png)
![WSL Ubuntu](/images/2024-04-28-wsl-ubuntu.png)

Briefly, the above images are showing (from left to right);

- The platform (Windows or Linux)
- The shell (PowerShell or Bash)
- I'm in the "dev-machine-config" folder
- This is a GitHub repo
- I'm in the "main" branch
- The name of the Azure subscription that AZ CLI is currently logged into

__*WARNING!*__ This article is written for Windows 11 users. Those on macOS or Linux may not have these issues and will need to follow slightly different instructions.

After trying to follow Matt's instructions I discovered that not everything was going to work on my work laptop because I don't have the permissions to start / run a prompt with elevated privileges. Also, some of the tasks required to get Starship setup can be done in several different ways, some easier than others. So I thought it might be useful to document my findings here for the benefit of others.

## What's Starship?

"What's Starhip?", I hear you ask. [Starship](https://starship.rs) is a cross-platform cross-shell prompt written in Rust, so is blazingly fast. It allows you to customise and augment the prompt with additional information that you may find useful.

[top](#title)

## Sold! How do I get this setup?

Before you run headlong into trying to install Starship, you need to install a few dependencies first. As I said before, I couldn't install Starship using Winget or Chocolatey as I did not have the permissions to start Windows Terminal with elevated privileges. After a bit of sluething, I discovered an alternative package manager called [Scoop](https://scoop.sh), which allowed me to install Starship without elevated privileges, as well as other helpful tools.

[top](#title)

### Step 1: Install Scoop in Windows

The first step is to install Scoop, which is as simple as opening Windows Terminal and running the following PowerShell;

{% highlight powershell %}
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri [https://get.scoop.sh](https://get.scoop.sh) | Invoke-Expression
{% endhighlight %}

[top](#title)

### Step 2: Install a Nerd Font

I'm using Microsoft's Cascadia Code TrueType font, which can be installed on Windows using, you guessed it, Scoop!

{% highlight powershell %}
scoop bucket add nerd-fonts
scoop install nerd-fonts/CascadiaCode-NF
{% endhighlight %}

I also needed to install it in my WSL Ubuntu instance as well. This can be done using a tool called *NFDL* ([Nerd Fonts Downloader](https://github.com/rubiin/nfdl)), which is a small NodeJS app. So, first install NodeJS and NPM - if you don't have them already - and then NFDL;

{% highlight bash %}
sudo apt-get install -y nodejs npm
npm install -g nfdl
{% endhighlight %}

Then run the cli;

{% highlight bash %}
nfdl
{% endhighlight %}

Select your desired Nerd Font from the menu and let the cli handle the rest. By default, fonts will be downloaded and installed in the .fonts directory in your home directory.

[top](#title)

### Step 3: Configure Windows Terminal to use the downloaded Nerd Font

By default Windows Terminal uses a standard font, so you need to tell it to use the new Nerd Font. Open Windows Terminal > Settings, and select Profile Defaults > Appearance, and select your Nerd Font from the the Font face dropdown list. For me that meant selecting "CaskaydiaCode NF". If you don't want to apply the font to all profiles then select each profile to set the Font face.

[top](#title)

### Step 4: Configure Visual Studio Code to use the downloaded Nerd Font

By default Visual Studio Code uses a standard font, so you need to tell it to use the new Nerd Font. Open Visual Studio Code > Preferences > Settings, then Text Editor > Font and type the name of your Nerd Font and click Save. For me that meant entering "CaskaydiaCove NF".

[top](#title)

### Step 5: Install Starship on Windows

To install Starship on Windows, open Windows Terminal and run the following PowerShell;

{% highlight powershell %}
scoop bucket add main
scoop install main/starship
{% endhighlight %}

You then need to edit your PowerShell profile to run Starship at startup. From the same Windows Terminal run;

{% highlight powershell %}
notepad $PROFILE
{% endhighlight %}

and then add the following code at the bottom;

{% highlight powershell %}
$ENV:STARSHIP_CONFIG = "$HOME\.config\starship\starship.toml"
Invoke-Expression (&starship init powershell)
{% endhighlight %}

You need to create a configuration file for Starship, which is the "starship.toml" path in the above code. Follow the [instructions](https://starship.rs/config/), or you can download and use [my configuration from my GitHub repo](https://github.com/philipstreet/dev-machine-config/blob/main/starship/starship.toml). Ensure the path to your starship.toml file matches the one you configure in your STARTSHIP_CONFIG environment variable.

[top](#title)

### Step 6: Install Starship in your WSL instance

To install Starship on your WSL instance, run the following Bash script;

{% highlight bash %}
curl -sS [https://starship.rs/install.sh](https://starship.rs/install.sh) | sh
{% endhighlight %}

Then add the following to your .bashrc file;

{% highlight bash %}
export STARSHIP_CONFIG=/mnt/c/Users/?/.config/starship/starship.toml
eval "$(starship init bash)"
{% endhighlight %}

Replace "?" with the relevant Windows "username" so that the path is relevant from WSL. This means that both your Windows Terminal AND WSL instance will use the same Starship configuration file, ensuring consistency across both platforms!

[top](#title)

## Wrap Up

All of the above code & config snippets are available on my [dev-machine-configuration](https://github.com/philipstreet/dev-machine-config/tree/main) GitHub repo.

When you open a PowerShell or WSL prompt, either from Windows Terminal or Visual Studio Code, you should now have a consistent user experience! Enjoy!

[top](#title)

## Conclusion

As always, if you think there is a better way this could've been fixed then please leave a comment below.

[top](#title)
