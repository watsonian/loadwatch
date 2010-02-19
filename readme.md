# loadwatch: server monitoring and logging

Ever want to keep an eye on a particular server or a number of servers without having to have several terminals open with top running hoping to catch what's causing that load spike? If so, this is for you. It's a lightweight bash script that runs backgrounded on a server monitoring the load average. If it crosses a set threshold, it logs all active processes sorted by %CPU usage.

## Installation

Just checkout to your server somewhere:

    git clone git://github.com/watsonian/loadwatch.git

That's it!

## Usage

To get started, just execute the script:

    ./load.sh

That will fire it up and it will start monitoring in the background. When you do this, two logs will be created:

* load.std.log
* load.err.log

The first will be written to whenever the script detects the load of the server crossing the given threshold. When that happens, it keeps track of the last highest seen load level and will only trigger again if the load increases past that level. The threshold is reset after a specified number of checks. The second will contain any standard errors the script encounters.

When the script actually triggers, it will log the process list to this log file:

* load.cmd.log

If you want to run the script in the foreground you can do so by executing it like this:

    ./load.sh --

## Customization

There are a number of variables you can adjust to suit your needs. Here are the defaults:

    CMDOUT_LOG="load.cmd.log"
    STDOUT_LOG="load.std.log"
    ERROR_LOG="load.err.log"
    MAX_INITIAL=20.00
    MAX=$MAX_INITIAL
    RUNS=0
    MAX_RUNS=12 # to get minutes: $MAX_RUNS / (60 / $SLEEP_TIME)
    SLEEP_TIME=30
    COMMAND="ps aux --sort=-pcpu"

The log variables are fairly self-explanatory.

**MAX_INITIAL** is the load average threshold that will trigger logging initially.

**MAX** is the last highest seen load average.

**RUNS** is the current number of times the script has checked since the last reset.

**MAX_RUNS** is the number of runs that occur before a reset is performed. When a reset occurs, *MAX* is set to *MAX_INITIAL* again.

**SLEEP_TIME** is the number of seconds the script waits before checking again.

**COMMAND** is the command that's executed when the script is triggered.

## Examining the Logs

So, now you have a nice log file containing *ps aux* output from high load times on your server. That's great and all, but it would be great if you could quickly get an idea of which users might be causing the most trouble to get a quick idea of where to look. Try this on for size:

    grep -A 100 "USER" load.cmd.log | awk '{print $1}' | sort | uniq -c | sort -n | tail -20

Basically, that greps for the header column of each *ps aux* dump and grabs the first 100 lines of that dump (which is the highest CPU processes running at the time), it then pipes it to awk and pulls out the username, sorts it, gets a unique count of each username, sorts it numerically, and grabs the top 20 users. All this is really doing is showing you a sorted count of how many processes each user had running in the top 100 CPU processes during high load times, but that can prove quite helpful!