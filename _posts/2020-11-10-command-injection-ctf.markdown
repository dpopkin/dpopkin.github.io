---
layout: post
title:  "Command Injection CTF"
date:   2020-11-10 12:16:20 -0800
categories: Laravel PHP
---

Consider the following:
{% highlight php %}
// CommandRequest.php
public function rules()
{
  return [
    'command' => 'starts_with:ls,man,touch,echo,open',
  ];
}

// CommandController.php
public function execute(CommandRequest $request)
{
  $output = [];
  $return = null;
  $command = $request->input('command');
  $command = Str::replaceArray(';', [' #'], $command);
  $command = Str::replaceArray('&', [' #'], $command);
  $command = Str::replaceArray('|', [' #'], $command);
  exec($command, $output, $return);
  if (!$return) {
    return view('command.output', ['output' => $output]);
  } else {
    return redirect()->back()->withErrors(['command' => "Error processing command"]);
  }
}
{% endhighlight %}
There's a lot to unpack here, so let's get started. First, this controller method will accept will accept a small set of commands using the starts_with rule in the form request file to only allow a whitelist of commands. It then takes that validated input and runs it through a set of Laravel helper commands to trim off a few special characters. Finally, it executes the command and displays the output if it works. I wonder if there's a way to execute arbitary code?

The first thing to notice is why they used `replaceArray` over a far more comprehensive method. Looking at the notes in the documentation for [exec](https://www.php.net/manual/en/function.exec), we can see that there are better and more battle-tested methods with [escapeshellarg](https://www.php.net/manual/en/function.escapeshellarg.php) and [escapeshellcmd](https://www.php.net/manual/en/function.escapeshellcmd.php). Looking at the cmd method also gives us a nice list of characters it escapes. We can also see it mentions something called the [backtick operator](https://www.php.net/manual/en/language.operators.execution.php). Put simply, the operator is used to execute commands in PHP and the UNIX command line (ex: \`ls\`).

This does give us an avenue to attack, but the validator requires our string starts with a valid command. So what if we try ls \`whoami\`? Well, that will execute code. But displays nothing on our end other than an error. I wonder if theres any way to ＥＣＨＯ out the output (sorry, not sorry). Typing echo `whoami` into the request gives us output. If you want to try this yourself, it's included in the commands tab in my [DVLA](http://dvla.test/command) project.

This is part one about command vulnerabilites. In [part two]({% post_url 2020-11-10-command-injection-ctf-prt-2-file-inclusion %}) I talk about how we can use this issue to achieve a file inclusion vulerability.