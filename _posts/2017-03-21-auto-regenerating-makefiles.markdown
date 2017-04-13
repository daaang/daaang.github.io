---
layout: post
title:  "Auto-regenerating makefiles"
date:   2017-03-21 11:08:11 -0400
---
I've been rebuilding makefiles all wrong---turns out, you can just set
`Makefile` as a target, and it will automatically rebuild itself if the
files it depends on are newer than it is. So, for example, if you're
using a `./configure` script to generate the makefile, you could include
this default makefile:

{% highlight make %}
CONFIG_STATUS = config.status
CONFIG_SHELL = /bin/sh

Makefile: $(CONFIG_STATUS)
	$(CONFIG_SHELL) $(CONFIG_STATUS)

$(CONFIG_STATUS): configure
	if [ -s $(CONFIG_STATUS) ]; then \
	  $(CONFIG_SHELL) configure `$(CONFIG_SHELL) $(CONFIG_STATUS) --config`; \
	else \
	  $(CONFIG_SHELL) configure; \
	fi
	
{% endhighlight %}

In this case, let's say we have a standard unconfigured package. When
you type `make`, this file will:

1.  See that it depends on `config.status`.
2.  See that `config.status` is not there, and that it depends on
    `configure`.
3.  Run that five-line `if` statement as a single line of shell code,
    which will in turn:

    1.  See that `config.status` is not present.
    2.  Run `/bin/sh configure --no-create`, which will create a default
        `config.status` without running it.

4.  Run `/bin/sh config.status`, which will overwrite `Makefile` (while
    also possibly creating other files).
5.  Run `make` using the new makefile.

One of the things I like about this is that it doesn't matter what
additional arguments the user supplies to `make`; it will always rebuild
itself before attempting to actually run the command.

If, for example, the user typed `make test`, the exact same thing as
above would happen, except that it would pass `make test` to the new
makefile (instead of just `make`). If the makefile needs to be rebuilt,
it rebuilds itself before paying attention to the user-specified target.

I'm still trying to figure out precisely how `$(srcdir)` works and why
anyone would want to mess with it. But I do know that `$(CONFIG_STATUS)`
and `$(CONFIG_SHELL)` are environment variables that `configure` and
`config.status` are supposed to recognize.

Overall, I think I'm pretty safe to ignore `$(srcdir)` in a placeholder
makefile though. It's really only there in case a user just blindly
enters the directory and types `make` without even looking. I think the
`$(srcdir)` is something for advanced users who can probably be counted
on to actually run `configure`.

Oh! And in other news, I figured out a fun detail about heredocs: you
can write `<<'EOF'` to get it to parse the data literally (like with
single quotes). For example:

{% highlight shell_session %}
$ cat <<EOF
> like double quotes: \$
> EOF
like double quotes: $
$ cat <<'EOF'
> like single quotes: \$
> EOF
like single quotes: \$
$ 
{% endhighlight %}

Also, I can use heredocs to read multiple variables from a single line
without resorting to a subshell. O, the joys of programming under the
constraints of the original Bourne shell:

{% highlight shell %}
categorize_arg() {
  # A lot of sed expressions that convert the arg to three columns. For
  # example, --enable-some-feature=no would become something like:
  #
  #     "arg_with_value    enable-some-feature   no"
  #
  # and --no-create might become:
  #
  #     "long_arg          no-create             "
}

process_categorized_arg() {
  # Takes three arguments: arg_type, arg_name, and arg_value
  case "$1" in
    arg_with_value)
      # etc
      ;;

    long_arg)
      # etc
      ;;

    # etc

    *)
      # error out
      ;;
  esac
}

# With subshell
for arg in "$@"; do
  categorize_arg "$arg"
done | while IFS="^I" read arg_type name value; do
  # We are now in a subshell, and any variables we set here will not
  # persist beyond this loop.
  process_categorized_arg "$arg_type" "$name" "$value"
done

# Without subshell
for arg in "$@"; do
  IFS="^I" read arg_type name value <<EOF
`categorize_arg "$arg"`
EOF

  # This is the same as above except that we haven't entered a subshell,
  # so we can set variables without worrying about losing them.
  process_categorized_arg "$arg_type" "$name" "$value"
done
{% endhighlight %}

Thanks for reading!
