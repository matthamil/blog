---
layout: post
title: Introduction to Bash Scripts
---

When I first started learning programming, I wondered how other developers automated certain tasks on their computers.

> Oh, I have a script that does X for me!

What did that mean? What language were they using to write a script that did _what?_

This is where Bash comes in.

Bash is a Unix shell and a command language. It's part of OSX and most Linux distrubutions. You can do so much crazy stuff with Bash.

> If you can do it in your terminal, you can write a Bash script for it.

This is my go-to vetting for Bash when I tell people about it. Some people don't see the benefit to using scripts to automate tasks, but I use them almost daily ([and I have a repo of my scripts too](https://github.com/matthamil/Bash-Scripts)).

Let's write our first Bash script! We'll build a simple `Hello World` script and extend it into a script to let us create Github repositories from the command line.

#### MyFirstScript.sh
```bash
#!/bin/bash
echo "What is your name? "
read name
echo "Hello $name!"
```

If you save that file and run `bash MyFirstScript.sh` in the terminal, you'll be prompted with `What is your name?` You can enter anything, hit enter, and be greeted! `Hello Matt!`

The command `echo` outputs text to the terminal by default. You can do other things with `echo` like echoing text into a file.

```bash
touch .gitignore
echo "node_modules" >> .gitignore
```

The `>>` characters act as an way to tell Bash to take whatever is before it, and put it into the file on the right. Now if you run `cat .gitignore`, you'll see the contents of the .gitignore.

```bash
node_modules
```

The first line of our script, `#!/bin/bash` is a convention so your shell knows what kind of interpreter to run.

Let's keep going on our quest to build a Github repo maker from the terminal!


#### The First Half
```bash
#!/bin/bash
printf "💻  Github Username: "
read user
printf "🤔  Repo Name: "
read name
myname=${name//[^[:alnum:]]/}
printf "$name changed in $myname for compatibilty with github\n"
printf "✏️  $name Description: "
read description
printf "✨  Creating repo: $myname\n"
```

I like to add some quirk to my scripts sometimes, so we're adding emojis to our script.

You'll notice a new command, `printf`. This has the same as the default behavior of `echo` except it adds a new line at the end of each output. If we used `echo`, we would need to add `\n` at the end of every string.

On the sixth line, you'll see a regex. This selects all of the characters in the string `name` except for special characters. This is to help us prevent any bugs in the later part of our script that we're about to write.

We can gather input from a user about his/her Github username, a repo name, and a description. We have all of the information we need to make a request to Github to make a new repository now!

#### The Second Half
```bash
curl -u $user https://api.github.com/user/repos -d '{ "name": "'"$myname"'", "description": "'"$description"'" }'
git init
touch README.md
echo $myname >> README.md
git add .
git commit -m "first commit"
git remote add origin "git@github.com:matthamil/${myname}.git"
git push -u origin master
printf "👍  Done!\n"
```

This is the meat and potatoes of our script. This uses the `curl` command to hit the Github API.

The `-u` flag lets `curl` know to prompt the user for a password for the account name saved in the `name` variable. The `-d` flag tells `curl` to post the following data to the API. We're sending a chunk of JSON to the API with information about our repository. You can [read up on the Github API](https://developer.github.com/v3/) and learn about other ways to send data to Github with a `POST` request.

Here's the script with both parts together.

#### githubRepoMaker.sh
```bash
#!/bin/bash
printf "💻  Github Username: "
read user
printf "🤔  Repo Name: "
read name
myname=${name//[^[:alnum:]]/}
printf "$name changed in $myname for compatibilty with github\n"
printf "✏️  $name Description: "
read description
printf "✨  Creating repo: $myname\n"
curl -u $user https://api.github.com/user/repos -d '{ "name": "'"$myname"'", "description": "'"$description"'" }'
git init
touch README.md
echo $myname >> README.md
git add .
git commit -m "first commit"
git remote add origin "git@github.com:matthamil/${myname}.git"
git push -u origin master
printf "👍  Done!\n"
```

I use this script very frequently. When I'm inside a directory that I want to push to Github, I run this command and _voila!_ A repository is created. If you'd rather clone the code from Github, you can find it [in my Bash scripts repository](https://github.com/matthamil/Bash-Scripts/blob/master/githubRepoMaker.sh) along with other scripts I use often.

You can also set up an `alias` to this script to be able to use it like a normal terminal command.

If you are using `oh-my-zsh` ([I highly recommend it](http://ohmyz.sh/)), you can set up an `alias` in your `~/.zshrc` file like so:

#### ~/.zshrc
```bash
alias mkrepo="bash path/to/githubRepoMaker.sh"
```

When you restart your terminal, you now have access to the `mkrepo` command to run the script. I went with the name `mkrepo` since the command `mkdir` exists in the terminal already, and it only seemed appropriate.

Now that you've written a super useful Bash script, try writing others to make your life a little easier every day.
