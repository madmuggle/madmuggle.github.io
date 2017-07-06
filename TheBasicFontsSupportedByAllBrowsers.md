# The Basic Fonts Supported By All Browsers
2017/05/23 10:13:00


## The Five Basic Fonts

Choosing fonts for a website can be very frustrating. [Google Fonts][googlefonts] are pretty, but they slow down the loading speed. And google is not accessible in some places like China. Using the local fonts is always a better choice, but you never know what kind of fonts your clients have.

But today I found that CSS has already solved the problem to some extent.

CSS has defined 5 general purpose fonts: *serif*, *sans-serif*, *monospace*, *cursive* and *fantasy*. These fonts are some kind of collection, which means they are not specific font names. For example, *monospace* can be any kind of monospaced fonts like *menlo*, *monaco*, *consolas*, *courier*, etc. It will choose the one that your operating system has.


## Example

The system may not support all basic fonts. For example, my cellphone can not render *fantasy* font while my laptop can. But I beleive that most operating systems do or will support all the 5 fonts.

Show time for the 5 fonts:

serif:
<p style="font-family:serif;border:1px solid #ccc;padding:2px;">
A quick brown fox jumps over the lazy dog.
</p>

sans-serif:
<p style="font-family:sans-serif;border:1px solid #ccc;padding:2px;">
A quick brown fox jumps over the lazy dog.
</p>

cursive:
<p style="font-family:cursive;border:1px solid #ccc;padding:2px;">
A quick brown fox jumps over the lazy dog.
</p>

fantasy:
<p style="font-family:fantasy;border:1px solid #ccc;padding:2px;">
A quick brown fox jumps over the lazy dog.
</p>

monospace:
<p style="font-family:monospace;border:1px solid #ccc;padding:2px;">
A quick brown fox jumps over the lazy dog.
</p>

I like the *fantasy* font most.


[googlefonts]: https://fonts.google.com/

