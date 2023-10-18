---
title: "Jenkins as a Job Dispatch Engine"
date: "2012-12-04T20:00:00.000Z"
description: "Automating the boring tasks"
warning: ancient
---

I get easily tired of doing the same thing over and over again, and will, wherever possible, script or automate it to make life easier for myself.  This could be in the form of a lightweight webapp/REST api for stuff, or in this case, I used Jenkins.

So on one server, we sometimes need to reload apache.  As we don't like developers randomly executing shells on live servers, it's better to just allow access to a few specific commands, in this case, a wrapper script on the target server's `/usr/local/bin` that just wraps "`/etc/init.d/httpd restart`" or "`/etc/init.d/httpd reload`". 

In "`/etc/sudoers`" there's a `Cmnd_Alias`

```
Cmnd_Alias RESTARTER = /usr/local/bin/restart-httpd.sh, /usr/local/bin/reload-httpd.sh
restarter ALL=(ALL) NOPASSWD: RESTARTER
```

And the restarter user can access this without specifying a password.

The restarter user has a `.ssh/authorized_keys` file containing the jenkins user's ssh public key.

On the jenkins job, there's a Parameterized Build flag, called "`ARE_YOU_SURE`" which prevents the accidental restart (as No is the default option).

The sole build step is:

 
```bash
if [ "$ARE_YOU_SURE" = "Yes" ]; then
echo "Restarting..."
ssh -tt restarter@server-to-restart.fqdn.tld sudo /usr/local/bin/restart-httpd.sh
else
echo "Aw, shucks"
fi
```
If you build and click "No" in the parameter, it will echo "Aw, shucks" and exit.  If you click yes, it will SSH to the remote server as the restarter user, and then execute the script.

If you don't specify `ssh -tt`, then you get pestered because the terminal it's trying to run sudo in isn't a TTY. 

Ta Da! Jenkins as a job dispatch engine.

