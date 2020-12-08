# shell-session-logger

# overview

log interactive shell session

works in "real" systems

## what's installed
* session-logger.sh - a shell fragment installed to /etc/profile.d
* session-logger-keep-ssh-env-vars - a sudoers fragment installed to /etc/sudoers.d to preserve SSH env vars

## operation

This uses the shell 'trap' DEBUG mechanism  to capture shell execution one line at a time.

It uses the /usr/bin/logger command to send a formatted message to the local system logger with facility auth and severity info.

The line captured in auth.log looks like this:

Oct  1 10:25:33 ubu1604s64 cmdlog: joe-the-lion pts/3 10.1.15.125 echo blort

The timestamp and hostname are provided by the system logging subsystem (rsyslog in this example).

The subsequent fields are:

  * cmdlog - the signature of this log capture
  * username - name of user
  * tty - tty of session
  * ssh-whence - ip of system from which ssh connected (or nothing for console)
  * command - text of executed command

## build notes

The makefile is named "Mockfile" because the Debian packaging tools make assumptions about a legit Makefile I didn't want to deal with.

To install tools required for building:
* make -f Mockfile setup

To run shellcheck:
* make -f Mockfile check

To build the deb (in the parent directory):
* make -f Mockfile

To cleanup:
* make -f Mockfile clean

## known limitations

* Command capture can be defeated by disabling history collection using "set +o history". However, the 'set +o ...' command itself is captured

* Command capture can be evaded by prefixing line with a space character  (this can be blocked by forcing env var HISTCONTROL to "ignoredups")

* "sudo su -" blocks the propagation of SSH_CLIENT and related env vars. Use "sudo -i" instead. The 'sudo su -' is captured.

* The command is captured before execution so exit status are not avaialable to be logged

* When using the "screen" command, by default shells are "sub-shells" and are not fully initialized. This behavior can be changed by adding the following to either /etc/screenrc or ~/.screenrc to force the shells to be login shells:

   shell -/bin/bash

 tmux does not seems to have this issue

* command capture can be evaded for a sequence of operations by manually creating a subshell - i.e. by executing "bash". However, the creation itself is captured.

* The initialization is aimed at bash - attempts to use /bin/sh as a login shell will fail.

* It's really not compatible with /bin/sh on ubuntu (which is apparently /bin/dash)

* It is not tested with zsh
