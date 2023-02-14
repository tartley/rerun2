# rerun2

Runs a given COMMAND every time it detects filesystem update events in the current
directory, or any subdirectory.

    rerun2 COMMAND

For example, it can show you the latest test results in an adjacent terminal
every time you save your code from the comfort of your favorite editor:

    rerun2 pytest

Or it can re-generate rendered HTML when you save Markdown:

    rerun2 pandoc README.md -o README.html

Or call GraphViz to regenerate SVG when you save the .dot source file:

    rerun2 dot mydiagram.dot -T svg -o mydiagram.svg

COMMAND needs to be a single command (ie. don't use pipes (`|`), semi-colons (`;`) or
boolean operators (`&&`, `||`, etc). To give a compound command, use rerun2 to invoke
a shell, and pass the compound command as a single string:

    rerun2 bash -c "ls -l | grep ^d"

Things one might like about it:

* It uses *inotify*, so is more responsive than polling. Fabulous for running
  sub-millisecond unit tests, or rendering Graphviz dot files, every time you
  hit 'save'.
* Because it's so fast, you don't have to bother telling it to ignore large
  subdirs (like node_modules) just for performance reasons.
* It's extra super responsive, because it only calls `inotifywait` once, on
  startup, instead of running it, and incurring the expensive hit of
  establishing watches, on every iteration.
* It's just a few lines of Bash
* Because it's Bash, it interprets commands you pass it exactly as if you had
  typed them at a Bash prompt. (Presumably this is less cool if you use another
  shell).
* It doesn't lose events that happen while COMMAND is executing, unlike most of
  the other inotify solutions on this page.
* On the first event, it enters a 'dead period' for 0.15 seconds, during which
  other events are ignored, before COMMAND is run exactly once. This is so that
  the flurry of events caused by the create-write-move dance which Vi or Emacs
  does when saving a buffer don't cause multiple laborious executions of a
  possibly slow-running test suite. Any events which then occur while COMMAND is
  executing are not ignored - they will cause a second dead period and
  subsequent execution.

Things one might dislike about it:

* It currently only has a hard-coded list of files that it ignores changes to.
  So if you use it to run Python tests, and that generates or modifies the files
  in your __pycache__ directory, then it knows to ignore those, so you won't
  trigger a cascade of re-executions of COMMAND. But most filesystem side-effects
  of your COMMAND will trigger an endless cycle of updates. I was about to add
  something like an '--ignore' flag, when I discovered 'entr' (see below), so
  perhaps I don't have to.
* It uses *inotify* to recieve filesystem events, so won't work outside of Linux.
  For a cross-platform solution, consider this project's precursor,
  https://github.com/tartley/rerun (see below).
* Because it uses *inotify*, it will barf on trying to watch directories
  containing more files than the max number of user inotify watches. By default,
  this seems to be set to around 5,000 to 8,000 on different machines I use, but
  is easy to increase. See
  https://unix.stackexchange.com/questions/13751/kernel-inotify-watch-limit-reached
* If you are editing files on your host, but executing `rerun2` on a VM with
  those files mounted in the VM, then rerun2 won't see any events. You must
  either start editing files within the VM, or else use my other project,
  https://github.com/tartley/rerun (see below).
* It fails to execute commands containing Bash aliases. I could swear that this
  used to work. In principle, because this is Bash, not executing COMMAND in a
  subshell, I'd expect this to work. I'd love to hear If anyone knows why it
  doesn't. Many of the other solutions on this page can't execute such commands
  either.
* Personally I wish I was able to hit a key in the terminal it's running in to
  manually cause an extra execution of COMMAND. Could I add this somehow,
  simply? A concurrently running `while read -n1` loop which also calls execute?
* Right now I've coded it to clear the terminal and print the executed COMMAND
  on each iteration. Some folks might like to add command-line flags to turn
  things like this off, etc. But this would increase size and complexity
  many-fold.

## Alternatives

### rerun

`rerun2` is the successor to `rerun`, a similar utility written in Python:

[https://github.com/tartley/rerun](https://github.com/tartley/rerun)

which polls for filesystem changes rather than using events. Hence *rerun*
works on all operating systems, works on mounted filesystems in VMs. In
practice I've never seen it take more than a couple hundred milliseconds to
poll, even on what I consider large projects (e.g. including the Django
source).

### entr

Aha! I didn't discover this until looooong after writing rerun & rerun2:

[https://www.systutorials.com/docs/linux/man/1-entr/](https://www.systutorials.com/docs/linux/man/1-entr/)

This looks like a serious effort, and perhaps I'll migrate to using `entr`
myself in future.
