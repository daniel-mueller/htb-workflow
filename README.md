# Hack The Box Workflow

The following describes my go-to approach when starting on a new box on hackthebox.eu. At the moment it's targeted at doing linux boxes since my experience with Windows servers is still somewhat limited.

# Setup

I usually create a new dir where I put my notes and results. I start with a blank thouhts.txt file where I write down everything of interest I notice. I found this to be crucial to keep thoughts somewhat ordered and come back to it if I'm out of ideas.

# Enumeration from the outside

First thing to do is always a comprehensive nmap scan:

```bash
nmap -p 1-65535 -v -sS -A -Pn -T5 -p- -oN nmap.txt $IP
```  
I now take the time to study the result thorougly and put the found ports in a file called enum.txt. The usual ports are 22/ssh and 80,43/http(s). If we get other ports (e.g. 3306 for mysql) I look up which service they represent. The nmap results bear a lot of additional information like e.g. the version of the web server at port 80. This also gets written down in enum.txt as any other info which could be relevant, even if it's a far shot. I only move forward when I'm sure the harvest of information is really complete.

In most cases we get some kind of website at port 80 or 443. In this cases I run gobuster and dirb simultaneously to scan for directories:

```bash
gobuster dir -w /path/to/wordlist -t 100 -u $IP >> gobuster.txt
dirb $IP >> dirb.txt
```
While the directory busters are at work I will use BurpSuite and my browser to poke around manually while keeping an eye on the network tab of the browser dev tools. I'll try to slowly work through every piece of content and interesting stuff will be written down in the thoughts.txt (or enum.txt if it's something that describes the environment). Is there custom JS at work? Are names mentioned which might also be user names? Are there any login forms or other types of input processing? Are directories public which shouldn't? A lot of of questions should be asked. After navigating manually use the results of the dir busters and go through everything you missed. Again I think it's crucial to take the time to gather everyting and ensure you don't overlook stuff. The more complete your notes and initial enumeration, the less jumping forth and back later on.

# Poking the discoveries

After the initial enumeration we should have a somewhat clear picture of the environment. I now try work off on everything one by one. I look up the specific versions of services and used technology on ExploitDB and with searchsploit and note possible exploits in the enum.txt. If an exploit is supported in metasploit I use the 'check' command to try to figure out if the target is vulnerable.

Giving a comprehensive overview of all possible methods is hardly doable in the scope of the write-up, so I will give some examples:

* SQL-Injections (I try by hand and use sqlmap)
* Finding out which kind of authentication is used (e.g. JWT)
* Remote Code Execution with some exploit from ExploitDB or Metasploit
* Upload of some script (e.g. PHP Reverse Shell) and execute it
* Using default credentials on admin pages
* Finding exposed application code and analyzing it
* Anonymous login allowed on ftp
* placing your public ssh-key in a authorized_hosts file

I again try to take the time and analyze the different areas of the environment one by one. Since hackthebox always follows the same layout (first getting user.txt from the home directory of a user, later root.txt from /root) you should sooner or later be able to login on the server via ssh or get a reverse shell of some kind.  

# Enumeration from the inside

After a brief moment of celebrating the success so far it's time explore the machine. The goal is to somehow find way to get access to the content of the precious root.txt, usually through some kind of privilege escalation. First thing I do is checking out which tools I have at hand. curl/wget/nc and vi/vim are the most useful at this point. 

If it's possible to connect to the outside world I'm able to place some handy tools on the server. I start a local HTTP-Server which serves a directory I created exactly for this:

```bash
python -m SimpleHTTPServer
```

Now I download scripts like linenum.sh, lse or the fantastic pspy which monitors every process even as non-root user. If downloading neat stuff is not possible I try to at least copy/paste the scripts into a file via vi/vim.

The tools are a shortcut to answering questions you could also get yourself. Before writing down a probably non-exhaustive list of stuff I will just link to the excellent guide https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/ by g0tmi1k and encourage everyone to just read the code of the two bash scripts from above.

I create a file enum-box.txt and try to write down everything of interest I find. It's a bit harder to filter out the noise because there's a lot to check. Next I read up on software that's running which I don't know. Reading the docs properly gives the awesome benefit of getting to know something new and that's the reason of the whole exercise after all, at least for me :)

With the intel from the linux enumeration and the knowledge about the running software I try to see the connection. Sometimes it's necessary to escalate priviledge horizontally and become another user first before escalating up to root. Anyways, this is the moment where I just try to see it. It's gotta be somewhere. If looking around for hours doesn't bring the result I just stop. Getting rid of tunnel vision after some sleep and a day full of distractions often works wonders.
