---
layout: post
title:  "Revisiting try_forever"
date:   2017-03-13 14:28:41 -0400
---
I figured out a better version of that `try_forever` decorator from
[earlier][1]:

{% highlight python %}
from time import sleep

class TryForever:

    __time_specifiers = (
        ("seconds_between_attempts", 1),
        ("minutes_between_attempts", 60),
        ("hours_between_attempts", 60*60),
    )

    def __init__ (self, kwargs):
        self.__set_properties(kwargs)
        self.__assert_no_remaining_keywords(kwargs)

    def __call__ (self, func):
        def result (*args, **kwargs):
            n = self.limit

            while True:
                try:
                    return func(*args, **kwargs)

                except self.base_error:
                    n -= 1
                    if n == 0:
                        # We'll never get here if n starts at zero,
                        # since it will already be negative. Otherwise,
                        # we'll get here after n failures.
                        raise

                sleep(self.seconds_between_attempts)

        return result

    def __repr__ (self):
        return "<{}>".format(self.__class__.__name__)

    def __set_properties (self, kwargs):
        self.base_error = kwargs.pop("base_error", Exception)
        self.limit = kwargs.pop("limit", 0)

        self.__set_pause_time(kwargs, 60)

    def __set_pause_time (self, kwargs, default):
        all_times_given = self.__get_all_pause_times_from(kwargs)

        if all_times_given:
            self.__set_pause_time_from_list(all_times_given)

        else:
            self.seconds_between_attempts = default

    def __get_all_pause_times_from (self, kwargs):
        return [m * kwargs.pop(k)
                for k, m in self.__time_specifiers
                if k in kwargs]

    def __set_pause_time_from_list (self, all_times_given):
        self.seconds_between_attempts = all_times_given.pop(0)

        if all_times_given:
            raise TypeError("choose only one time specifier")

    def __assert_no_remaining_keywords (self, kwargs):
        if kwargs:
            key, value = kwargs.popitem()
            raise TypeError(repr(key) + " is an invalid keyword " +
                            "argument for this function")

def try_forever (*args, **kwargs):
    obj = TryForever(kwargs)

    if args:
        return obj(*args)

    else:
        return obj
{% endhighlight %}

This time I started over using TDD, and it's longer but clearer, I
think. I'm assuming you aren't interested in seeing the tests,
but---probabaly needless to say---they're longer.

[1]: {% post_url 2017-02-27-the-try-forever-decorator %}
