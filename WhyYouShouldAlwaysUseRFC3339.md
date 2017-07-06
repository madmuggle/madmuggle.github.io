# Why You Should Always use RFC3339
2017/06/09 14:43:00


Sometimes we need to use string of characters to represent time. And we have many standard choices right now. But I think we should always use [RFC3339][rfc3339](or [ISO8601][iso8601], same thing).

Let see what RFC3339 looks like:

```
2017-06-09T14:55:35+08:00
```

Pretty clear. And how about other formats? This is what RFC2822 looks like:

```
Fri, 09 Jun 2017 15:00:00 +0800
```

And this is the default output format of unix command `date`. (which is not RFC2822)

```
Fri Jun  9 14:58:22 CST 2017
```

You don't want to remember the difference between them, do you?

Besides that, RFC3339 contains no English word, so it's friendly to people of any countries, as long as they use arabic numbers.

And RFC3339 is easy to parse.


[iso8601]: https://en.wikipedia.org/wiki/ISO_8601
[rfc3339]: https://tools.ietf.org/html/rfc3339

