# Simple Setup for WriteFreely

**GOAL:  Run WriteFreely as a separate instance using Nginx between
the WriteFreely service and your Nginx virtual server.**

The documentation for the package is fairly tight and complete, so
I don't have a lot to add.  Here are some the gaps that might be
necessary to fill.

Let's start with *their documentation*

[WriteFreely Installation](https://writefreely.org/start)

## Database

The MySQL example is fine.  It creates the database, but you'll probably
want to create a user for that database.

They suggested (as of 11/2022):

```
CREATE DATABASE writefreely CHARACTER SET latin1 COLLATE latin1_swedish_ci;
```

Fine, but we need a user too, right?

```
CREATE USER 'jeff'@'localhost' IDENTIFIED BY 'elonsucks';
GRANT ALL ON writefreely.* to 'jeff'@'localhost';
FLUSH PRIVILEGES;
```

## NGINX

There's probably a slick way to do this in Apache, but I have decided
to eschew Apache in favor of Nginx.

It seems a lot of *Fediverse* server projects are doing this too?

Anyway.  Names are important.

So if you're running an `example.com` instance of something else
and then want to overlay WriteFreely on top of it you'll run into 
some problems unless you're really good at Nginx config to separate out
WF from the other service.  

I'm not interested in complexity so let's make a new `CNAME` for your
new WriteFreely instance.

Suppose that `example.com` is where you host something ELSE, so make
`wf.example.com`  a thing.   (replace `wf` with whatever your
CNAME is for this new WriteFreely instance).

Create the virtual-server configuration in Nginx (WriteFreely actually
offers a template in the page above.  Refer to it.  Use it.)


## Start the WriteFreely Config


They indicate it's as simple as:

```
$ writefreely config start
```

It is.  Just do that.

Here are the questions they ask (and what you should answer)

1. First question will be, *What kind of environment?*  Choose **Production, behind reverse proxy**

2. Next, *Local Port?*  This is the port number associated with the WF proxy.  Make up a port, but check it is not a WKS (`grep PORT /etc/services`) nor used by any other service you're aware of.  Use the default if you wish if this is the only service using it on your host.

3. Next, *Database driver?*  I use `MySQL`.

4. Next, *MySQL Username?*  Well, you already made that, so enter it `jeff`

5. Next, *MySQL Password?*  Easy, `elonsucks`. Just kidding. Use the password you created above.

6. Next, *Database name?*.  If you followed WF's standard example the database is called `writefreely`.  You could have called it anything you wanted to.

7. Next, *Host?*  That will be `localhost` unless you have some other 
situation (and I'm not into complex situations at the moment..) so Pick `localhost`

8. Next, *Port?*  This is the port that `MySQL` is listening.  Again, pick the default unless you have something unique in mind.

9. Next, *Site type?*  You get to decide now.  Is your blog or a service for anyone to make blogs?   I chose *Single user blog* for simple reasons.  If you want to cater to users, then pick *Multi-user instance*.  I didn't pick that option, so I cannot detail the next questions that would follow.  Maybe you can Pull-Request this file with those answers?

10. Next, *Admin username?*  This is the public facing username on your
new Federated Blog.  If your hostname of the WF instance is `wf.example.com`, and if you picked the name `joshua`,  Then the Admin will be effectively `@josuha@wf.example.com`   You might want to pick the Admin name to be the same
as your name in more popular Fediverse instances... Just a thought. Or not.

11. Next, *Admin password?*   This is the password for that Admin user
on your instance.  Pick strongly.

12. Next, *Blog name?*  This is a name for the blog and you change it later.

13. Next, *Public URL?*  Ok, so all of this about `wf.example.com` now is important.  We DO NOT want to set the Public URL to `wf.example.com` since
we're running through the Nginx as a proxy.  LEAVE IT as `http://localhost:PORT` (you remember that Port number you picked earlier?  Use it there.)

14. Next, *Federation?*   Yes, Enabled. Unless you don't want to.

15. Next, *Usage Stats public?*  Yes, Public.  Unless you don't want to.

16. Next, *Instance metadata privacy?*  Yes, Public.  Unless you don't want to. (I am not sure of the impact there.  Maybe some deeper reading on the WF web page is in order.)

Then it creates the config and sets up the Database.

The next step in the WF generic instructions was to generate keys.

Do that.

```
$ writefreely keys generate
```

Now you have a WF instance that is **ready, but not running**.

The rest of the instructions in their setup-guide (Link at the top)
is pretty clear.  They offer two suggestions that I echo.


1.  Use the *Behind a Reverse Proxy* steps/template they provide.  Wherever you need to change something, they actually put it in **bold** for you.  Namely, the **domain** and the **port** of the proxy.
2.  Setup the `systemd` config per your flavor of OS.  Because, honestly, you just want to be able to start it it as `systemctl start writefreely`

## certbot

One more thing.  After this you'll need to establish the `https` flavor
of the server hosted by `nginx`.  

```
# certbot --nginx -d wf.example.com
```

(not `-d example.com`)

Let `certbot` do configure it for redirect.  Now your WF instance
is prim and proper.

# Test

Once you've started it (`systemctl start writefreely`) you can now
hit the site via browser and should see the WF beauty before your eyes.

Problems, try their web page for installation.  It's fairly complete.

I'm no expert in WF but it seems straight-forward.  Questions, ping 
me in case I can help.  But, I have no knowledge of how the software
was developed.  I don't do `Go` or any scripting languages (except
`bash` and `perl`).  I'm just an embedded `C` and `C++` programmer 
for the most part making real hardware do real things in the vacuum 
of space.  I don't (anymore) write
software that runs within an operating system either.  Bare metal, baby.


