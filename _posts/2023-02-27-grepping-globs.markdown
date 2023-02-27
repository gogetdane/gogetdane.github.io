---
title:  "Searching Multiple Log Files Quick Time"
---

#### If you're using something like PM2 on a busy server this might save some time.


Searching through log files can be laborious.  But, with some thought before just ploughing on it's possible to save some time.

---

```console
# ./pm2/logs
- process_1_out__2023-02-16_00-00-00.log
- process_1_out__2023-02-17_00-00-00.log
- process_1_out__2023-02-18_00-00-00.log
- process_1_out__2023-02-19_00-00-00.log
- process_1_out__2023-02-20_00-00-00.log
- process_1_out__2023-02-21_00-00-00.log
- process_1_out__2023-02-22_00-00-00.log
- process_1_out__2023-02-23_00-00-00.log
- process_1_out__2023-02-24_00-00-00.log
- process_1_out__2023-02-25_00-00-00.log
```

Now multiply that by _n_ servers and look for a regex pattern in each. That's a lot of `grep`, unless you know how to search them all in one command. Caveat: You'll have to run it on each server.

{% capture notice-2 %}
**Here's what you'll need to do this..**
* `grep`

**This is what we'll achieve**
* One command to search matching files for a pattern

{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

## The [`grep`](https://www.gnu.org/software/grep/manual/grep.html){:target="_blank"} command
```bash
grep -R # or --deference-recursive. We use this to looks in all files in the directory
grep -R "start.*end" # We're looking for a pattern that starts with 'start', followed by any number of anything, then ends in 'end'
grep -R "start.*end" ./some_dir # We want to look at files in the some_dir directory
grep -R "start.*end" --include "process_1*" ./some_dir # We'll use a glob pattern in the --include option to only look at files starting in 'process_1'
```


Using that command we can search multiple files for a regex pattern. So, rather than trying to `grep` each file now we can just run one command per server and quickly determine if there are matches.