---
layout: post
title: Seeking for Better Prompot
---

Since last March, I've been heavily working with Zsh + GNU screen + git, and experiencing some small glitches now and then with this settings (in Japanse). Below are the new settings that I believe makes my everyday life awesome.

### Problems
RPROMPT is cool when you're working with it. When you copy the console output and paste it somewhere else, however, the it becomes just garbage. Often I needed to manually erase them after pasting the output to, email, review board, wiki, etc.
Sometimes I want to run the same command on different host. copy &amp; paste is the only way afaik. (cannot share command history with the host, can't I?) Selecting the command line accurately to copy it to clipboard needs a bit concentration and unison of your eyes, arm, and fingers. If you miss even a single character, or select extra characters, it won't work.
I found the answer to them in the terminal of someone in office whose prompt is "foo=; " where foo is his hostname. The key here is "foo=; " is, when typed in a shell, interpreted as a shell variable assignment. Specifically this assigns an empty string to shell variable "foo".

So why don't we put everything we want to see on prompt in (left) PROMPT, and format them in a way that our shell can interpret them, but ignore them?

Here we go.
![weird prompot](/public/assets/terminal1.png){: .main-image }

You can see weird prompt on left. For example, "mbp13=/tmp/foo/bar" means I'm on machine "mbp13" and in directory "/tmp/foo/bar".

Let's list up the directory contents.
![directories](/public/assets/terminal2.png){: .main-image }


To copy-paste-reexecute this command, you don't need to select "ls -lah" part. Just triple-click the line and press Command+C to copy the whole line into clipboard,
![copy line](/public/assets/terminal3.png){: .main-image }


and press Command+V to paste the whole line to a new command prompt. Then you can execute it again without modifying anything.

(If you use GNU screen, enter scrollback mode, move the caret to the last command line, press "Y", and you can copy the whole line to screen's buffer)

Of course you can see git branch name if you're in git workspace.
![branch name](/public/assets/terminal4.png){: .main-image }

You can tell whether your workspace is dirty.
![when workspace is dirty](/public/assets/terminal5.png){: .main-image }

On second thought, you can do more in shell prompt than assigning to shell variables.
![cd in prompt](/public/assets/terminal6.png){: .main-image }

Now the prompt ("mbp13=; cd /tmp/foo/bar; ") looks more interesting.
![getting interesting](/public/assets/terminal7.png){: .main-image }

If you open a new screen or terminal, and you can paste the whole line, which is "mbp=; cd /tmp/foo/bar; ls -lah", and you'll get the same result, without cd-ing to the directory. pretty neat!

As long as all machines you often work on have the same directory structure (shouldn't they?), you can very easily re-run your command on any of them.

All the configuration is found on GitHub. Enjoy prompt hacking!

(edit)
* special thanks to A zsh prompt for Git users. I borrowed most of configuration.
* Actually, this doesn't really solve the first question. The prompt "mbp13=; cd /tmp/foo/bar; " seems garbage to other people, definitely.
