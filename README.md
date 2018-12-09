Save 'rerun2' on your PATH, and invoke it using:

    rerun COMMAND

It runs COMMAND every time there's a filesystem modify event within your current
directory (recursive.) For example, it can re-run tests every time you hit
'save' in your editor. Or it can re-generate rendered output (html, SVG, etc)
every time you save the source (eg. Markdown or GraphViz dot, etc.)

COMMAND needs to be a single parameter to 'rerun', hence if it contains
spaces, either quote the whole thing:

    rerun "ls ~"

or escape them:

    rerun ls\ ~

Things one might like about it:

* It uses inotify, so is more responsive than polling. Fabulous for running
  sub-millisecond unit tests, or rendering graphviz dot files, every time you
  hit 'save'.
* Because it's so fast, you don't have to bother telling it to ignore large
  subdirs (like node_modules) just for performance reasons.
* It's extra super responsive, because it only calls inotifywait once, on
  startup, instead of running it, and incurring the expensive hit of
  establishing watches, on every iteration.
* It's just a few lines of Bash
* Because it's Bash, it interprets commands you pass it exactly as if you had
  typed them at a Bash prompt. (Presumably this is less cool if you use another
  shell.)
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

* It uses inotify to recieve filesystem events, so won't work outside of Linux.
  For a cross-platform solution, consider this project's precursor,
  https://github.com/tartley/rerun (see below.)
* Because it uses inotify, it will barf on trying to watch directories
  containing more files than the max number of user inotify watches. By default,
  this seems to be set to around 5,000 to 8,000 on different machines I use, but
  is easy to increase. See
  https://unix.stackexchange.com/questions/13751/kernel-inotify-watch-limit-reached
* If you are editing files on your host, but executing 'rerun2' on a VM with
  those files mounted in the VM, then rerun2 won't see any events. You must
  either start editing files within the VM, or else use my other project,
  https://github.com/tartley/rerun (see below.)
* It fails to execute commands containing Bash aliases. I could swear that this
  used to work. In principle, because this is Bash, not executing COMMAND in a
  subshell, I'd expect this to work. I'd love to hear If anyone knows why it
  doesn't. Many of the other solutions on this page can't execute such commands
  either.
* Personally I wish I was able to hit a key in the terminal it's running in to
  manually cause an extra execution of COMMAND. Could I add this somehow,
  simply? A concurrently running 'while read -n1' loop which also calls execute?
* Right now I've coded it to clear the terminal and print the executed COMMAND
  on each iteration. Some folks might like to add command-line flags to turn
  things like this off, etc. But this would increase size and complexity
  many-fold.

## Alternatives

This project is the successor to a Python program I wrote:

https://github.com/tartley/rerun

which polls for filesystem changes rather than using events. So this 'rerun'
works on all operating systems, works on mounted filesystems in VMs. In
practice I've never seen it be more than a couple hundred milliseconds, even
on what I consider large projects (eg including the Django source.)

