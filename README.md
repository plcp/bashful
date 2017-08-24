# bashful
Fiddle with bash internals through `ptrace` to creates uncommon or dynamic
shell variables (answering
[this](https://stackoverflow.com/questions/29463778/how-to-create-a-bash-variable-like-random)
on the way).

Here are some user testimonials:
 - *« Bashendous »*
 - *« Bashusting ideas »*
 - *« The most bashulsive thing I've ever seen »*
 - *« A bashomination »*
 - *« The bash puns is the best part »*

# Quick setup
Here:
```sh
git clone https://github.com/plcp/bashful
cd bashful
echo "$0" | grep -q bash && ./bashful
```

# TL;DR

Uses `gdb` to perform a `ptrace` call and to mess with bash internals. You can
now set arbitrary flags to your bash variables and abuse
[loadable bash builtins](http://git.savannah.gnu.org/cgit/bash.git/plain/examples/loadables/README?id=bash-4.2)
to create dynamic shell variables, just like `$RANDOM`.

Works only if `ps -q $PPID -o comm=` is `bash` and if `ptrace` is accessible.

# Examples

Here are some examples:
```
$ declare foo
$ ./bashful foo uppercase
$ foo="hello"; echo $foo
HELLO
$ ./bashful foo lowercase
$ foo="Hello"; echo $foo
hello
$ ./bashful foo capcase
$ foo="hello"; echo $foo
Hello
```

Another one:
```
$ var=PWD
$ ./bashful var nameref
$ echo $var
/home/plcp/plabs/bash/bashful
```

Some more:
```
$ declare bar
$ ./bashful bar nounset integer
$ unset bar
bash: unset: bar: cannot unset
$ bar=4+3 ; echo $bar
7
$ ./bashful bar
integer
nounset
$ ./bashful bar clean
$ unset bar
```

Now with dynamic variables:
```
$ ./bashful RANDOM dynamic head -c 8 /dev/urandom
Injecting ["head" "-c", "8", "/dev/urandom", ]...

$ echo $RANDOM
L-{Sgf
```

Having fun:
```
$ ./bashful PATH dynamic bash -c "echo '$PATH'|sed 's/$/:/g;s/:/____\\n/g'|shuf|tail -n 3|xargs|sed 's/____ \\?/:/g;s/:$//g'" > /dev/null
$ date
bash: date: command not found
$ date
bash: date: command not found
$ date
bash: date: command not found
$ date
Thu Aug 24 15:15:44 CEST 2017
```

# Usage

You can retrieve the usage by running `bashful` without arguments:
```
$ ./bashful
Versatile redneck bash utility.

Usage:
    ./bashful
         Display this help.
    ./bashful <name>
         List tags of <name>.
    ./bashful <name> [tags...]
         Replace tags of <name> by the provided tag list.
    ./bashful <name> dynamic[-debug] <cmdline>
         Evaluate <cmdline> when $<name> is evaluated.

Available tags:
     clean               (no tag specified)
     exported            export to environment
     readonly            cannot change
     array               value is an array
     function            value is a function
     integer             internal representation is int
     local               variable is local to a function
     assoc               variable is an associative array
     trace               function is traced with DEBUG nrap
     uppercase           word converted to uppercase on assignment
     lowercase           word converted to lowercase on assignment
     capcase             word capitalized on assignment
     nameref             word is a name reference
     invisible           cannot see
     nounset             cannot unset
     noassign            assignment not allowed
     imported            came from environment
     special             requires special handling
     nofree              do not free value on unset
     tempvar             variable came from the temp environment
     propagate           propagate to previous scope
```

# Troubleshoothing

If `bashful` is asking for your password (i.e. `[sudo] password for user:`), it
may be because `ptrace` calls are restricted. You can run `gdb -p $$` then
check for a `ptrace: Operation not permitted.` to found out.

You may enter your password to run the script or disable
[security](https://www.kernel.org/doc/Documentation/security/Yama.txt)
with:
```
su -c "echo 0 > /proc/sys/kernel/yama/ptrace_scope"
```

Please don't forget to re-enable it after playing:
```
su -c "echo 1 > /proc/sys/kernel/yama/ptrace_scope"
```

For `dynamic` tag debugging, use `dynamic-debug` instead.

# Contribute

Feel free to contribute in any way.
