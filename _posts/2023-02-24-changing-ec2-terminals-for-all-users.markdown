---
title:  "How to change the terminal prompt on AWS EC2 instances for all users"
---

#### Recently I thought it would make more sense for EC2 terminal prompts to use the public hostname, rather than a prompt determined from the private IP. 

---

```console
# Which is better?
user@ip-10.1.1.1:~$
user@ip-10.1.1.2:~$
# Or
user@go.getdane.co.uk:~$
user@go.staging.getdane.co.uk:~$
```

It's too easy to mix up terminal windows, especially when a busy dev is flitting from server to server. Nobody wants to break a production server, so let's look at how we can make the prompt memorable. 

Surely, if the prompt tells you which server you're on then it's harder to make mistakes.

{% capture notice-2 %}
**Here's what you'll need to do this..**
* An EC2 instance up and running
* Sudo privileges, or root access
* Some method of editing files, I'll use `sed`, `echo` and `tee` but you could just use a text editor

**This is what we'll achieve**
* Replace `user@ip-x.x.x.x` with `user@go.getdane.co.uk` for every user
* No user will be able to change the value of that prompt without a determined effort

{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

## Step 1

Review the [AWS docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-hostname.html) (or just follow these steps). 


We need to edit the `cloud.cfg` file to tell AWS to preserve the hostname that we'll use for a prompt.


The `preserve_hostname` was present in my `cloud.cfg`, so I used `sed` to replace the text.
{% highlight bash %}
sed -i 's/preserve_hostname: false/preserve_hostname: true/g' /etc/cloud/cloud.cfg
{% endhighlight %}

## Step 2

Next, we need to set the hostname. It's mentioned in the docs above, but here's the command:

```bash
hostnamectl set-hostname go.getdane.co.uk

# or on your staging server
hostnamectl set-hostname go.staging.getdane.co.uk
```

## Step 3. Here's the hard bit (well, it took longer to figure out)

If you rebooted now, you'd see the prompt changes to `go`. Well thats no good if we have staging domain too, we'd always just see `go` when we really want `go.getdane.co.uk` or `go.staging.getdane.co.uk`.

We want this prompt to change for all users. It's only fair that anyone logging in gets this new benefit and reduces their risk of error.

We also want to make sure this isn't easily changed.

Check out `PROMPT_COMMAND` in the [bash docs](https://linux.die.net/man/1/bash). The value we give to this is executed prior to issuing a prompt (every time). You'll see in the docs that we're interested in `PS1`, that's what defines the relevant part of the prompt.

If you `echo` out `PS1` you'll see a crazy string that is interpreted to provide the prompt. Luckily there's only one change to make here.

```bash
# From
\h # This provides everything up to the first dot of the hostname
# To
\H # We'll swap to this, which gives us the full hostname
```

So, my `PS1` needs to look like this:

```bash
# I changed the second h to H, giving me the full hostname
\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\H\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$
```

If we swap `PS1` with that value using `PROMPT_COMMAND` in `/etc/bash.bashrc` then we can ensure this is applied to all users.

Here's the command I used, you could use `nano` or `vi`.

```bash
echo 'PROMPT_COMMAND="PS1=\"\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\H\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$\""' | tee -a /etc/bash.bashrc
```

That's it! Now, you can start a new terminal and you'll see a prompt like this:

```bash
# Production server
user@go.getdane.co.uk:~$

# Staging server
user@go.staging.getdane.:~$

```