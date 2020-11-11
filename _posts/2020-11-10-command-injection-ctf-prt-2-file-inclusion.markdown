---
layout: post
title:  "Command Injection CTF Part 2: File Inclusion"
date:   2020-11-10 14:16:20 -0800
categories: Laravel PHP
---

Going off the last [post]({% post_url 2020-11-10-command-injection-ctf %}), we had the ability to execute arbitary code. But let's go one step further and add [local file inclusion](https://en.wikipedia.org/wiki/File_inclusion_vulnerability) to the mix. Let's take another look at the validation rules in the form request again:
{% highlight php %}
public function rules()
{
  return [
    'command' => 'starts_with:ls,man,touch,echo,open',
  ];
}
{% endhighlight %}
Open seems like a good start, but it's rife with edge cases that might make it unpredictible, so let's try something easier. From the man page of touch: `[...]  If any file does not exist, it is created with default permissions`. Meaning we can use touch to create arbitary files. putting `touch file.txt` will create a text file in the public folder. Meaning we can browse to it.

Because the devs forgot about the `>` character, we can use that to give our file output. For example, using echo `whoami` > file.txt will output the results to that file we can browse to. 

What would happen if we tried to create a .php file? Sadly we can't use PHP files, because `;` is in the list of removed characters which, as we know, PHP uses for code termination. But what if we had another way?

What if we tried shell script code? It has similar syntax to PHP, works with any UNIX system and allows us to pipe it to any file to run it. By echoing it to the .txt file, it can be used as a script. For this example, Lets try having it delete our .txt file. In the request type: echo "rm file.txt" > file.txt . Next let's try to run it. Using the command echo `./file.txt` does... nothing? Remember that line from the manual about `touch`, the one about default permissions? By default, the PHP user cannot execute scripts created using touch. But this is an easy fix using echo \`chmod +x file.txt\`. Afterwards we can then execute the command, and browsing to the file reveals it was deleted!