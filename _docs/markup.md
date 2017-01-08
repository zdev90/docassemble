---
layout: docs
title: Marking up text
short_title: Markup
---

**docassemble** allows you to format your text using [Markdown] and to
use [Mako] to make your documents "smart."  These [mark up] methods
are available for use in [`question`] text, field labels, [`interview
help`] text, the content of [documents], and other text elements.

# Markdown

The syntax of [Markdown] is explained well
[elsewhere](https://daringfireball.net/projects/markdown/).

When generating [documents], **docassemble** uses [Pandoc] to convert
your [Markdown] to PDF, RTF, and HTML.

Here are some examples of things you can do with Markdown:

{% highlight yaml %}
---
question: Markdown demonstration
subquestion: |
  This is *italic text*.
  This is **bold text**.
  This is __also bold text__.

  > This is some block-quoted
  > text

  ### This is a heading

  This is an image:

  ![Bass logo](https://upload.wikimedia.org/wikipedia/commons/thumb/9/9b/Bass_logo.svg/199px-Bass_logo.svg.png)

  Here is a bullet list:

  * Apple
  * Peach
  * Pear

  Here is a numbered list:

  1. Nutmeg
  2. Celery
  3. Oregano

  Here is a [link to a web site](http://google.com).
---
{% endhighlight %}

In the web application, this comes out looking like this:

![Markdown demo screenshot]({{ site.baseurl }}/img/markdown-demo.png)

All of these methods will format text in questions as well as PDF and
RTF documents, with the exception of the `!` image insertion command,
which does not work within PDF and RTF documents.

# <a name="mako"></a>Using Mako for logic and generated text

**docassemble** uses a templating system called [Mako] to allow
authors to insert variables and code into questions and documents.

You can insert the values of variables into question text using [Mako]'s
`${ ... }` syntax.

{% include side-by-side.html demo="mako-01" %}

You can use [Mako]'s `if/endif` syntax to insert text conditionally:

{% include side-by-side.html demo="mako-02" %}

You can also express more complicated logic:

{% include side-by-side.html demo="mako-03" %}

The [Mako] syntax for if/then/else statements is based on
[Python's `if` statement], but is a little bit different.

The `%` at the beginning of the line signifies that you are doing
something special with [Mako].

[Python] itself does not use `endif` -- it only uses indentation to
designate where the if/then/else statement ends.  [Mako] requires the
use of `endif` because it does not see indentation.

In [Python], `elif` is short for "else if."  In the example above, the
if/then/else statement means:

> If the day of the month is less than three, write "The month just
> started!", but otherwise if the day of the month is less than 15,
> write "It is the beginning part of the month."; otherwise, write "It
> is the latter part of the month."

As with [Python], it is critical that you include `:` at the end of
any line where you indicate a condition.

You can put `if/endif` statements inside of other `if/endif`
statements:

{% include side-by-side.html demo="mako-04" %}

In this example, the `% if`, `% else`, and `% endif` lines are
indented, but they do not have to be.  Since nested if/then/else
statements can be hard to read, the indentation helps make the
statement more readable.  Note that the the actual text itself is not
indented, even though the `%` lines are indented; this is because
indentation means something in [Markdown].  If you indent a line by
four spaces, [Markdown] will treat the line as a [code block], which
might not be what you want.

<a name="for"></a>[Mako] also allows you to work with lists of things
using `% for` and `% endfor`:

{% include side-by-side.html demo="mako-05" %}

This is based on [Python's `for` statement].

The `for` loop is useful for working with groups of [objects]:

{% include side-by-side.html demo="mako-06" %}

Within `for` loops, [Mako] provides a useful object called [`loop`],
which contains information about the current iteration of the loop.

{% include side-by-side.html demo="mako-09" %}

Note that `loop.index` is a number in a range that starts with zero.
The [`ordinal()`] function converts these numbers to words.

For more information about working with groups of things, see
[groups].

In addition to allowing you to insert [Python] expressions with the `${
... }` syntax, [Mako] allows you to embed [Python] statements using
the `<%`/`%>` syntax:

{% include side-by-side.html demo="mako-07" %}

[Mako] also allows you to insert special code that cuts short the text
being rendered:

{% include side-by-side.html demo="mako-08" %}

The same thing could also be accomplished with an `else` statement,
but using [`STOP_RENDERING`] may be more readable.

For more information about [Mako], see the [Mako documentation].
Note, however, that not all features of [Mako] are available in
**docassemble**.  For example, in normal [Mako], you can write:

{% highlight text %}
% if some_variable is UNDEFINED:
...
% endif
{% endhighlight %}

In **docassemble**, this will not work as intended.  Instead, you
would use the [`defined()` function]:

{% highlight text %}
% if defined('some_variable'):
...
% endif
{% endhighlight %}

If you want to use the [`<%def>`] construct of [Mako], see the
[`def` initial block].

# <a name="inserting images"></a>Inserting images

To insert an image that is located in the `static` folder of a custom
Python package, use the `FILE` command.  This works within PDF, RTF,
and DOCX documents as well as within questions.

For example:

{% highlight yaml %}
---
question: |
  Did your attacker look like this?
subquestion: |
  Please study the face below closely before answering.

  [FILE docassemble.crimesolver:mugshot.jpg]
yesno: suspect_identified
{% endhighlight %}

This example presumes that there is a Python package called
`docassemble.crimesolver` installed on the server, and there is a file
`mugshot.jpg` located within the `static` directory inside that
package.

If you omit the package name (e.g., `[FILE mugshot.jpg]`),
**docassemble** will assume you are referring to a file located in the
`static` directory of the package in which the question appears.

Optionally, you can set the width of the image:

    [FILE docassemble.crimesolver:mugshot.jpg, 100%]

or:

    [FILE docassemble.crimesolver:mugshot.jpg, 150px]

<a name="inserting uploaded images"></a>To insert an image that has
been uploaded, simply refer to the file variable using [Mako].  For
example:

{% highlight yaml %}
---
question: |
  Do you look cute in this picture?
subquestion: |
  ${ user_picture }
yesno: user_is_cute
---
question: |
  Please upload a picture of yourself.
fields:
  - Your Picture: user_picture
    datatype: file
---
{% endhighlight %}

Alternatively, you can call the [`show()`] method on the file object:

{% highlight yaml %}
---
question: |
  Do you look cute in this picture?
subquestion: |
  ${ user_picture.show() }
yesno: user_is_cute
---
{% endhighlight %}

The [`show()`] method takes an optional argument, `width`:

{% highlight yaml %}
---
question: |
  Do you look cute in this picture?
subquestion: |
  ${ user_picture.show(width='250px') }
yesno: user_is_cute
---
{% endhighlight %}

In the above example, the picture will be shrunk or expanded so that
its width is 250 pixels.

# <a name="emoji"></a>Inserting inline icons

If you have defined "decorations" in an [`image sets`] block (see
[initial blocks]), you can include these decorations as icons (having
the same size as the text) by referencing them "emoji-style," putting
colons around the decoration name.  This works not only in `question`
and [`subquestion`] areas, but also in question choices.

{% highlight yaml %}
---
generic object: Individual
question: |
  What is ${ x.possessive('gender') }?
field: x.gender
choices:
  - "Male :male:": male
  - "Female :female:": female
  - "Other": other
---
{% endhighlight %}

This works within PDF and RTF documents as well as within questions.
  
# <a name="audio and video"></a>Inserting audio and video

In addition to using the [`audio`] and [`video`] [modifiers], you can
insert audio and video into your [Mako] text in questions.

{% highlight yaml %}
---
question: Listen to this!
subquestion: |
  Best song ever:

  ${ my_file }

  Don't you think so?
---
question: Upload an audio file.
fields:
  - no label: my_file
    datatype: file
---
{% endhighlight %}

Or, if you have a file in `data/static`, you can write:

{% highlight yaml %}
---
question: Listen to this!
subquestion: |
  This excerpt of whalesong will give you goosebumps.

  [FILE whale_song.mp3]
---
{% endhighlight %}

It works the same with videos.

{% highlight yaml %}
---
question: Watch this!
subquestion: |
  This video of otters sunbathing is going to go viral.

  [FILE awesome_otters.mp4]
---
{% endhighlight %}

You can also embed [YouTube] and [Vimeo] videos.  For example, if you
want to embed a [YouTube] video for which the URL is
`https://www.youtube.com/watch?v=RpgYyuLt7Dx` or
`https://youtu.be/RpgYyuLt7Dx`, you would write this:

{% highlight yaml %}
---
question: Are you traveling to New York City?
yesno: going_to_nyc
video: |
  New York is such a happening place.  Check it out:

  [YOUTUBE RpgYyuLt7Dx]
---
{% endhighlight %}

See [modifiers] for more information about including audio and video.

# Inserting QR codes

You can also display or insert QR codes using `[QR ...]`, where `...`
is the text you want to encode.  This works like `[FILE ...]` in that
you can give the image a width.  The QR code images can be displayed
on the screen or inserted into a document.

This works within PDF and RTF documents as well as within questions.

For example, this interview provides a QR code that directs the user to
[Google News](http://news.google.com):

{% highlight yaml %}
---
mandatory: true
question: Here is a URL for you in a QR code
subquestion: |
  [QR http://news.google.com, 200px]
attachment:
  name: Your QR code
  filename: your_code
  content: |
    Use the QR reader on your smartphone to take a picture of this:
    
    [QR http://news.google.com]
---
{% endhighlight %}

([Try it out here]({{ site.demourl }}?i=docassemble.demo:data/questions/testqr.yml){:target="_blank"}.)

See also the [`qr_code()`] function, which allows you to insert the
`[QR ...]` markup using [Python].

[documents]: {{ site.baseurl }}/docs/documents.html
[modifiers]: {{ site.baseurl }}/docs/modifiers.html
[Mako]: http://www.makotemplates.org/
[Markdown]: https://daringfireball.net/projects/markdown/
[YAML]: https://en.wikipedia.org/wiki/YAML
[mark up]: https://en.wikipedia.org/wiki/Markup_language
[Pandoc]: http://johnmacfarlane.net/pandoc/
[YouTube]: https://www.youtube.com/
[Vimeo]: https://vimeo.com/
[initial blocks]: {{ site.baseurl }}/docs/initial.html
[function]: {{ site.baseurl }}/docs/functions.html
[Python]: https://en.wikipedia.org/wiki/Python_%28programming_language%29
[`question`]: {{ site.baseurl }}/docs/questions.html#question
[`subquestion`]: {{ site.baseurl }}/docs/questions.html#subquestion
[`interview help`]: {{ site.baseurl }}/docs/initial.html#interview help
[`show()`]: {{ site.baseurl }}/docs/objects.html#DAFile.show
[`image sets`]: {{ site.baseurl }}/docs/initial.html#image sets
[`audio`]: {{ site.baseurl }}/docs/modifiers.html#audio
[`video`]: {{ site.baseurl }}/docs/modifiers.html#video
[`qr_code()`]: {{ site.baseurl }}/docs/functions.html#qr_code
[code block]: https://daringfireball.net/projects/markdown/syntax#precode
[Python's `if` statement]: https://docs.python.org/2.7/tutorial/controlflow.html#if-statements
[Python's `for` statement]: https://docs.python.org/2.7/tutorial/controlflow.html#for-statements
[objects]: {{ site.baseurl }}/docs/objects.html
[groups]: {{ site.baseurl }}/docs/groups.html
[`loop`]: http://docs.makotemplates.org/en/latest/runtime.html#loop-context
[`STOP_RENDERING`]: http://docs.makotemplates.org/en/latest/syntax.html#exiting-early-from-a-template
[`ordinal()`]: {{ site.baseurl }}/docs/functions.html#ordinal
[Mako documentation]: http://docs.makotemplates.org/en/latest/index.html
[`defined()` function]: {{ site.baseurl }}/docs/functions.html#defined
[`<%def>`]: http://docs.makotemplates.org/en/latest/defs.html#using-defs
[`def` initial block]: {{ site.baseurl }}/docs/initial.html#def