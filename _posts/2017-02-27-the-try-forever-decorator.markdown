---
layout: post
title:  "The try_forever decorator"
date:   2017-02-27 14:41:53 -0500
---
Here's a code structure I find myself using from time to time that I
quite dislike:

{% highlight python %}
def get_data_from_api():
    while True:
        try:
            return make_api_call()

        except ConnectionError:
            pause_between_attempts()
{% endhighlight %}

I feel there's too much of a march to the right. Also it gets even worse
if I want the option of limiting the number of attempts.

{% highlight python %}
def get_data_from_api(limit=0):
    n = 0
    while True:
        try:
            return make_api_call()

        except ConnectionError:
            pause_between_attempts()

        n += 1
        if n == limit: # will never happen if limit=0
            raise RuntimeError("Gave up after {:d} attempts".format(n))
{% endhighlight %}

The problem, as I see it, is the necessity of breaking out of the loop
(or returning, in this case). This means I can't extract methods in
order to clean up this mess.

My solution, at first, was to turn to the promise pattern, but it isn't
actually what I need---it does little more than add a third-party
library that I don't need.

At least in the case of python, I think a decorator solves my problem
best:

{% highlight python %}
def try_forever (func):
    def result (*args, **kwargs):
        while True:
            try:
                return func(*args, **kwargs)

            except ConnectionError:
                pause_between_attempts()

    return result

@try_forever
def get_data_from_api():
    return make_api_call()
{% endhighlight %}

This doesn't make the problem go away, but at least it puts the ugliness
into its own place, with its own purpose. However, this can't as written
be given a limit, and it's coupled to `pause_between_attempts()` and the
`ConnectionError` in particular.

{% highlight python %}
from time import sleep

def try_forever (*args, **kwargs):
    if len(args) == 1 and not kwargs and callable(args[0]):
        return try_forever()(args[0])

    elif args:
        raise RuntimeError("No positional args allowed")

    else:
        sleep_time = kwargs.get("pause_in_seconds", 1)
        limit = kwargs.get("limit", 0)

        if "error" in kwargs:
            assert "errors" not in kwargs
            errors = kwargs["error"]

        else:
            errors = kwargs.get("errors", Exception)

        def decorator (func):
            def result (*r_args, **r_kwargs):
                n = 0
                while True:
                    try:
                        return func(*r_args, **r_kwargs)

                    except errors:
                        sleep(sleep_time)

                    n += 1
                    if n == limit:
                        raise RuntimeError(
                                "Gave up after {:d} attempts".format(n))

            return result
        return decorator


@try_forever
def get_data_from_api():
    return make_api_call()
{% endhighlight %}

This is pretty ugly, but it's the beginning of a solution that can be
hidden away somewhere away from code that could otherwise be clean.

Time to start over with some unit tests.
