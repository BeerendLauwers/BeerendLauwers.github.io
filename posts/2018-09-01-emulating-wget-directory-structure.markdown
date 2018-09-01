---
title: Emulating wget's directory structure creation with aria2
---

`wget` is an excellent tool for collecting information from open directories of universities, free data sets, and so on.
Unfortunately, it's also quite old. 
By itself, this is not a problem, but it does not, for instance, support multiple downloads at once.

Starting and stopping a long-running `wget` is also fraught with peril, and you must ensure you are using the proper flags to avoid duplicates or overwriting your previous work.

It's also a purely command-line program with no long-running server process behind it, which means you *definitely* cannot close the terminal it's running in, or power down the machine, etc.

This is why many people use `aria2` instead, which provides a long-running process that can be communicated with over RPC or command-line commands.

However, `aria2` just takes URLs, downloads them and dumps the resulting files in a folder.
On top of that, it doesn't do any recursive downloading like `wget` does.

## Standing on top of internet giants 

(A.K.A. I googled.)

There are plenty of tools for taking `wget`'s `--spider` output and turning it into a list usable by `aria2`, [theripper.sh](https://github.com/kajeagentspi/Datahoarder/blob/master/theripper.sh) being one of them.

[Here's another](https://asdfghjkl.me.uk/blog/apache-to-aria2/). But this blog post actually provided me with an interesting idea: I can just run an `aria2` server, with `aria2-ui` enabled and add stuff via the JSON RPC.

Additionally, it also includes code for pausing and resuming downloads at particular times of the day, which is a godsend in countries that have bandwidth caps during the day, but not during the night.

## Let's slap something together!

### Get a file with a bunch of URLs

I edited `theripper.sh` and commented out the `download()` and some cleanup parts:

```
echo "Creating list of urls..."
spider
echo "Index created!"
#download

# Cleanup
rm opendir-$$.log
rm opendir-$$.log.tmp
#rm list-$$.txt
#rm link-$$.down

```

(The `.log` file is useful for `tail -f`'ing it to see if it's performing as expected, by the way.)


### Preprocess the list and send it off to `aria2`

Next up was a whole bunch of googling for regexes for pulling apart a URL into its constituent parts so I could recreate the directory structure, and putting it in a shell script.

The basis was the shell script from [this post](https://sourceforge.net/p/aria2/discussion/540046/thread/b7e1eb37/?limit=25).

After putting in the regex code for extracting the host name and path from the URL, I tweaked the RPC code from the aforementioned blog post and [ended up with this script](https://github.com/beerendlauwers/wget-directories-in-aria2/blob/master/add_files_to_aria2_rpc.sh).
To use it, just run `./add_files_to_aria2_rpc.sh URL_LIST_FILE` and you're in business.

Files are added paused so I can download them overnight to prevent impacting my bandwidth cap.
You can change this as desired, the RPC interface supports all of [`aria2`'s command-line options](https://aria2.github.io/manual/en/html/aria2c.html#options).

Be sure to add a secret token and username/password combination for HTTP basic authentication.
(And run it over HTTPS if you're exposing the RPC interface on the internet.)

### Sidenote: mounting a CIFS (SMB) share on startup

Everything's dumped to a box that has a SMB share available.
To mount it, just add this line to `/etc/fstab`:

```
//IP-ADDR/REMOTE/PATH /media/LOCAL/PATH cifs guest,uid=1000,iocharset=utf8,user 0 0
```

### Pausing and resuming `aria2`

Again, borrowed from the second blog post mentioned above:

```
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command

# Pause aria2 downloads at 11:30.
30 11 * * * curl http://127.0.0.1:6800/jsonrpc -H "Content-Type: application/json" -H "Accept: application/json" --data '{"jsonrpc": "2.0","id":1, "method": "aria2.pauseAll", "params":["token:secret_token"]}'

# Resume aria2 downloads at 00:30.
30 00 * * * curl http://127.0.0.1:6800/jsonrpc -H "Content-Type: application/json" -H "Accept: application/json" --data '{"jsonrpc": "2.0","id":1, "method": "aria2.unpauseAll", "params":["token:secret_token"]}'
```

(Also again: don't forget your secret token!)

### Suggestions? Improvements?

Feel free to open up an issue or PR on [the Github repository](https://github.com/beerendlauwers/wget-directories-in-aria2/).

Enjoy!

