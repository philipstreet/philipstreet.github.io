---
layout: post
title: Starship for Command Prompt on Windows 11
tags: windows shell prompt starship command-prompt
image: 2024-05-08-starship-command-prompt-extract.png
---
In a follow-up to my previous post [Starship prompt for PowerShell & WSL on Windows 11](https://blog.philipstreet.co.uk/Starship-prompt-for-PowerShell-and-WSL-on-Windows-11/){:target="_blank"}, I'll show how to configure Starship for the good old native Windows shell cmd.exe.

## Can I really customise the humble Windows shell cmd.exe?

Yes you can! Here's mine, which looks almost exactly like my PowerShell and WSL instance;

![Command Prompt](/images/2024-05-08-starship-command-prompt.png)

These instructions assume you have already configured Starship as per my previous blog post. If not, follow those instructions first before proceeding.

### Step 1: Install Clink using Scoop

Customising the Command Prompt using Starship is achieved using a tool called [Clink](https://chrisant996.github.io/clink/clink.html){:target="_blank"}.

The first step is to install Clink, which is as simple as opening Windows Terminal and running the following PowerShell;

{% highlight powershell %}
scoop bucket add main
scoop install main/clink
{% endhighlight %}

### Step 2: Configure Clink to load Starship

Add the following to a file called ```starship.lua``` and save in a Clink scripts directory:

{% highlight lua %}
-- starship.lua
os.setenv('STARSHIP_CONFIG', 'C:\\Users\\?\\.config\\starship\\starship.toml')
load(io.popen('starship init cmd'):read("*a"))()
{% endhighlight %}

Replace "?" with the relevant Windows "username" so that the path is valid, or modify the path to where you have your Starship TOML file saved. This means that the Command Prompt will use the same Starship configuration file as both your Windows Terminal AND WSL instance, ensuring consistency across all prompts!

## And there's more!

Clink as a powerful tool for customising the cmd.exe, including [features](https://chrisant996.github.io/clink/clink.html#features){:target="_blank"} such auto-suggestions, completions, persistent history and key bindings!

If you spend a lot of time using Command Prompt then take a look at the multitude of [configuration options](https://chrisant996.github.io/clink/clink.html#configuring-clink){:target="_blank"}, but be careful you don't conflict with customisations provided by Starship.

## Wrap Up

All of the above code & config snippets are available on my [dev-machine-configuration](https://github.com/philipstreet/dev-machine-config/tree/main){:target="_blank"} GitHub repo.

## Conclusion

I hope this will be of help to you. Let me know if you have any suggestions for improvements in the comments.
