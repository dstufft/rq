Times
=====

Times is a small, minimalistic, Python library for dealing with time
conversions to and from timezones, for once and for all.

It is designed to be simple and clear, but also opinionated about good and bad
practices.

[Armin Ronacher][1] wrote about timezone best practices in his blog post
[Dealing with Timezones in Python][2].  The **tl;dr** summary is that
everything sucks about our mechanisms to represent absolute moments in time,
but the least worst one of all is UTC.

[1]: http://twitter.com/mitsuhiko
[2]: http://lucumr.pocoo.org/2011/7/15/eppur-si-muove/


Rationale
---------

Python's `datetime` library and the `pytz` library are powerful, but because
they don't prescribe a standard practice of working with dates, everybody is
free to pick his or her own way.

`times` tries to make working with times and timezones a little less of
a clusterfuck and hopefully set a standard of some sort.

It still uses `datetime` and `pytz` under the covers, but as long as you never
use any timezone related stuff outside `times`, you should be safe.


Accepting time
--------------

Never work with _local_ times.  Whenever you must accept local time input (e.g.
from a user), convert it to universal time immediately:

    >>> times.to_universal(local_time, 'Europe/Amsterdam')
    datetime.datetime(2012, 2, 1, 10, 31, 45, 781262)

You can approach the conversion from the other side, too, as `to_universal` is
conveniently aliased to `from_local`:

    >>> times.from_local(local_time, 'Europe/Amsterdam')
    datetime.datetime(2012, 2, 1, 10, 31, 45, 781262)

The second argument can be a `pytz.timezone` instance, or a timezone string.

If the `local_time` variable already holds timezone info, you _must_ leave out
the source timezone from the call.

To enforce best practices, `times` will never implicitly convert times for you,
even if that would technically be possible.


POSIX timestamps
----------------
If you prefer working with UNIX (POSIX) timestamps, you can convert them to
safe datetime representations easily:

    >>> import time, times
    >>> print times.from_unix(time.time())
    2012-02-03 11:58:58.069461
    >>> print times.to_universal(time.time())
    2012-02-03 11:59:03.588419

Note that `to_universal` auto-detects that you give it a UNIX timestamp.

To get the correct UTC-based UNIX timestamp representation of a universal
datetime, use:

    >>> print times.to_unix(universal_time)


Current time
------------

When you want to record the current time, you can use this convenience method:

    >>> import times
    >>> print times.now()
    datetime.datetime(2012, 2, 1, 11, 51, 27, 621491)


Presenting times
----------------

To _present_ times to the end user of your software, you should explicitly
format your universal time to your user's local timezone.

    >>> import times
    >>> now = times.now()
    >>> print times.format(now, 'CET')
    2012-02-01 21:32:10+0100

As with the `to_universal` function, the second argument may be either
a timezone instance or a timezone string.

**Note**: It _is_ possible to convert universal times to local times, using
`to_local` (or the alias `from_universal`).  However, you probably shouldn't do
it, unless you want to `strftime()` the resulting local date multiple times.
In any other case, you are advised to use `times.format()` directly instead.
