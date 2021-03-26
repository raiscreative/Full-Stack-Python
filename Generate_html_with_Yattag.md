> Yattag is a Python library for generating HTML or XML in a pythonic way.

> With Yattag,
- you don't have to worry about closing HTML tags
- your HTML templates are Python code. Not a weird template language. Just Python.
- you can easily render HTML forms, with defaults values and error messages.

> It's actually easier and more readable to generate dynamic HTML with Yattag than to write static HTML.
***

# 1. Installation

`pip install yattag`
***

# 2. `tag` and `text`

```
from yattag import Doc

doc, tag, text = Doc().tagtext()

with tag('h1'):
    text('Hello Yattag!')

html_file = doc.getvalue()
print(html_file)
``` 

The `yattag.Doc` class works kinda like `append`.

You create a `Doc` instance and then use the `text` method to append some text and the `tag` method to append the tags.

### A little trick to avoid typing too much

When you repeatedly call the `append` method, you can use this:

```
mylist = []
append = mylist.append
append('Everybody')
append('likes')
append('pandas.')
mystring = ' '.join(mylist)

```
Similarly, you can use a little trick to make the `html` templates more concise and beautiful.

```
doc, tag, text = Doc().tagtext()
```
is the equivalent of a longer code:

```
doc = Doc()
tag = doc.tag
text = doc.text
```

The `tagtext` method is a helper method that returns a triplet composed of:

- the `Doc` instance itself
- the `tag` method of the `Doc` instance
- the `text` method of the `Doc` instance

### The `tag` method

The `tag` method returns a context manager, so you can use it in a `with` statement. 

It has `__enter__` and `__exit__` methods. The `__enter__` method is called at the beginning of the `with` block and the `__exit__` method is called when leaving the block.

#### No more headache looking for unclosed tags.

`with tag('h1')` creates a `<h1>` tag.

It will be closed at the end of the with block.

This way you don't have to worry about closing your tags.

> Isn't it awesome? The python interpreter will close all your tags for you. 

#### No limits to your imagination

The `tag` method will accept any string as a tag name. So you're not limited to valid HTML tag names. You can write very strange XML documents if you want. You can specify tag attributes as keyword arguments.

```
with tag('icecream', id = '2', flavour = 'pistachio'):
    text("This is really delicious.")
```
Since `class` is a reserved keyword of the Python language, we had to replace it with `klass`. A `klass` attribute will be replaced with a `class` attribute in the end result.

With

```
with tag('h2', klass='breaking-news'):
    text('Sparta defeats Athens')
```
you'll get the result:
```
<h2 class="breaking-news">Sparta defeats Athens</h2>
```
For any other situation where an attribute's name can't be expressed as a Python identifier, you can use **(key, value) pairs**. For example, HTML5 allows attributes starting with `"data-"`. You can do:
```
with tag('td',
    ('data-search', 'lemon'),
    ('data-order', '1384'),
    id = '16'
):
    text('Citrus Limon')
```
and get:
```
<td data-search="lemon" data-order="1384" id="16">Citrus Limon</td>
```

### Escaping

Attributes values are escaped, that is, the `&`, `<` and `"` characters are replaced with `&amp;`, `&lt;`, and `&quot`.

For attributes without a value, just pass a string to the `tag` method. For example,
```
with tag('html', 'ng-app'):
    with tag('body'):
        text('Welcome to my Python application.')
```
will give you
```
<html ng-app><body>Welcome to my Python application</body></html>
```
### The `text` method

The `text` method takes a string, escapes it so that it is safe to use in a html document and appends the escaped string to the document.

You can pass any number of strings to the text method. 
```
text('Hello ', username, '!')
```
If you don't want your strings to be escaped, see the `asis` method.
***

# 3. Appending strings 'as is'

The `asis` method appends a string to the document without any form of escaping.

```
from yattag import Doc

doc, tag, text = Doc().tagtext()

doc.asis('<!DOCTYPE html>')
with tag('html'):
    with tag('body'):
        text('Hello world!')

print(doc.getvalue())
```

In this example, you don't want the `<` and `>` characters of the `'<!DOCTYPE html>'` string to be replaced with `&lt;` and `&gt;`. You really want the `<` and `>` characters to be added as is. So you use the `asis` method instead of the `text` method. You get:
```
<!DOCTYPE html><html><body>Hello world!</body></html>
```
As with the `text` method, the `asis` method can take any number of strings as arguments.
***
# 4. Self closing tags

The `stag` method produces a self closing tag.

As with the `tag` method, tag attributes are passed as keyword arguments to the stag method, and attribute values are escaped. With the code:

```
from yattag import Doc

doc, tag, text = Doc().tagtext()

with tag('div', id='photo-container'):
    doc.stag('img', src='/salmon-plays-piano.jpg', klass="photo")

print(doc.getvalue())
```
you get:
```
<div id="photo-container"><img src="/salmon-plays-piano.jpg" class="photo" /></div>
```
***
# 5. Setting tag attributes after opening a tag

The `attr` method sets the value(s) of one or more attributes of the current tag.

As with the `tag` and `stag` methods, tag attributes are passed as keyword arguments.

With the code:
```
from yattag import Doc

from datetime import date
today = date.today()

doc, tag, text = Doc().tagtext()

with tag('html'):
    with tag('body'):
        if today.month == 1 and today.day == 1:
            doc.attr(klass = "new-year-style")
        else:
            doc.attr(klass = "normal-style")
        text("Welcome to our site")

print(doc.getvalue())
```
January the first, you get the string:
```
"<html><body class="new-year-style">Welcome to our site</body></html>"
```
On any other day, you get the string:
```
"<html><body class="normal-style">Welcome to our site</body></html>"
```
***
# 6. Shortcut for nodes that contain only text

Most HTML and XML tag nodes contain only text. In order to write these in a terser way, use the `line` method.
With:
```
doc, tag, text, line = Doc().ttl()

with tag('ul', id='grocery-list'):
    line('li', 'Tomato sauce', klass="priority")
    line('li', 'Salt')
    line('li', 'Pepper')
```
you get:
```
<ul id="grocery-list">
  <li class="priority">Tomato sauce</li>
  <li>Salt</li>
  <li>Pepper</li>
</ul>
```
The `ttl` method works just like the `tagtext` method, only it returns a quadruplet: (doc, doc.tag, doc.text, doc.line).
 ***
# 7. Indentation
We've added indentation in the example above for readability purpose, but actually the `getvalue` method doesn't add spaces, indentation or new lines between tags.

Consequently, for big documents, the HTML or XML output is not as readable as hand-made HTML or XML.

In most cases, this is not a problem at all. For example, when debugging a web application, you'll look at the Python/Yattag code directly. Yattag code is in fact, more readable than hand made HTML. The Python interpreter guarantees that all your HTML tags are closed, and contrary to hand-made HTML, you'll get a syntax error if you don't close a quoted string, so it's really not necessary to look at the HTML output.

Moreover, you can use the `Inspect Element` function in Firefox (right click somewhere in the page and click on `Inspect Element`) to see the HTML document tree in a well indented and browsable fashion.

However, there might be a few situations where you want to produce well indented documents (for example when generating XML configuration files intended to be editable by hand). For that, you can use the `indent` function.
```
from yattag import indent

...

result = indent(doc.getvalue())
```
The `indent` function takes a string representing a xml or html document, and returns a well indented version of this document. The `indent` function is completely decoupled from the rest of Yattag. You can indent documents that were not generated by Yattag if you want.

You can tweak the end result by supplying these keyword arguments:

- `indentation`: the indentation unit. By default, it's two spaces.
- `newline`: the string to be used for new lines. By default, it's `'\n'`. Maybe Windows users would like to set this to `'\r\n'`.
- `indent_text`: by default, the content of nodes that directly contain some text (for example `<p>Hello <b>world!</b></p>`) is left untouched. If you want the content of these nodes to be indented as well, use `indent_text=True`.
```
result = indent(
    doc.getvalue(),
    indentation = '    ',
    newline = '\r\n',
    indent_text = True
)
```
Don't use the `indent` function to output HTML in a web application. Although this function is relatively fast, this would remain a waste of cpu time and bandwidth.
> Think of the planet! 
***
# 8. HTML forms rendering

It is possible to generate HTML forms just by using the tag and text methods. After all, any html document could be generated this way.

But you would be missing out on a very cool Yattag feature: completion of html forms with default values and errors. You can create a Doc instance and feed it with a dictionnary of default values and/or a dictionnary or errors. The defaults values are then inserted inside the form elements, and the error values displayed around the relevant fields.
```
from yattag import Doc

doc, tag, text, line = Doc(
    defaults = {
        'title': 'Untitled',
        'contact_message': 'You just won the lottery!'
    },
    errors = {
        'contact_message': 'Your message looks like spam.'
    }
).ttl()

line('h1', 'Contact form')
with tag('form', action = ""):
    doc.input(name = 'title', type = 'text')
    with doc.textarea(name = 'contact_message'):
        pass
    doc.stag('input', type = 'submit', value = 'Send my message')

print(doc.getvalue())
```

We passed a `defaults` and an `errors` dictionary to the Doc constructor.
Inside our template, we created the `<form>` node with a call to the tag method.

We called call the `input` and `textarea` methods. These special methods look into the defaults and/or errors dictionaries before generating their html content.

So instead of writing:
```
doc.stag('input', name = 'title', type = 'text')
```
which would create a simple self closing input tag, without taking defaults or error values into account, you write:
```
doc.input(name = 'title', type = 'text')
```
That creates a self closing `input` tag with the correct default and errors values. You can still pass all the html attributes you want as keyword arguments to this method.

Similarly, instead of writing:
```
with tag('textarea', name = 'contact_message'):
    pass
```
which would append the string:
```
<textarea name="contact_message"></textarea>
```
you write:
```
with doc.textarea(name = 'contact_message'):
    pass
```
which appends the following string to the document:
```
<span class="error">Your message looks like spam.</span><textarea name="contact_message" class="error">You just won the lottery!</textarea>
```
***
# 9. Select and options tags

Use the `select` and `option` methods to create `select` and `option` form elements.
```
from yattag import Doc

doc, tag, text, line = Doc(
    defaults = {'ingredient': ['chocolate', 'coffee']}
).ttl()

with tag('form', action = ""):
    line('label', 'Select one or more ingredients')
    with doc.select(name = 'ingredient', multiple = "multiple"):
        for value, description in (
            ("chocolate", "Dark Chocolate"),
            ("almonds", "Roasted almonds"),
            ("honey", "Acacia honey"),
            ("coffee", "Ethiopian coffee")
        ):
            with doc.option(value = value):
                text(description)
    doc.stag('input', type = "submit", value = "Validate")

print(doc.getvalue())
```
You get (indented for the readability of the example):
```
<form action="">
    <label>Select one or more ingredients</label>
    <select name="ingredient" multiple="multiple">
        <option value="chocolate" selected="selected">Dark chocolate</option>
        <option value="almonds">Roasted almonds</option>
        <option value="honey">Acacia honey</option>
        <option value="coffee" selected="selected">Ethiopian coffee</option>
    </select>
    <input value="Validate" type="submit" />
</form>
```
The two ingredients specified in the `defaults` dictionary were selected.
***
# 10. checkboxes and radio inputs

To create a html `checkbox`, call the `input` method with the keyword argument `type = 'checkbox'`. 

Similarly, for a `radio input`, call the `input` method with `type = 'radio'`. It reads just like HTML.
```
from yattag import Doc

doc, tag, text, line = Doc(
    defaults = {'color': 'red', 'fun': 'yes'},
    errors = {'shipping-method':\
      "Error! You're an idiot for not having chosen a shipping method."
    }
).ttl()

line('h1', 'Spaceship delivery details')
with tag('form', action = ""):

    line('p', 'Please pick the color of the spaceship')
    for color in ('blue', 'red', 'pink', 'yellow', 'ugly-yellow'):
        doc.input(name = 'color', type = 'radio', value = color)
        text(color)

    line('p', 'What shipping method should be used?')
    doc.input(name = 'shipping-method', type = 'radio', value = '1')
    text('Priority mail')
    doc.input(name = 'shipping-method', type = 'radio', value = '2')
    text('Delivery by a very old monk travelling on a horse')

    with tag('p'):
        text("Check this box if you want some additional fun for free")
        doc.input(name = 'fun', type = 'checkbox', value = 'yes')

    doc.stag('input', type = 'submit', value = 'Confirm my order')

print(doc.getvalue())
```
You get the following html output (indented for readability):
```
<h1>Spaceship delivery details</h1>

<form action="">

    <p>Please pick the color of the spaceship</p>
    <input type="radio" name="color" value="blue" />blue
    <input type="radio" checked="checked" name="color" value="red" />red
    <input type="radio" name="color" value="pink" />pink
    <input type="radio" name="color" value="yellow" />yellow
    <input type="radio" name="color" value="ugly-yellow" />ugly-yellow

    <p>What shipping method should be used?</p>
    <span class="error">Error! You're an idiot for not having chosen a shipping method.</span>
    <input type="radio" name="shipping-method" value="1" class="error" />Priority mail
    <input type="radio" name="shipping-method" value="2" />Delivery by a very old monk travelling on a horse

    <p>Check this box if you want some additional fun for free<input type="checkbox" checked="checked" name="fun" value="yes" /></p>
    <input value="Confirm my order" type="submit" />

</form>
```
The desired radio button (red color) was selected, and that the checkbox was checked.
***
# 11. Using Yattag in a web application

In a web application, it's generally wise to separate business logic from presentation. When a client loads a web page, you compute all kind of stuff, update the database etc, then you pass what needs to be displayed to the presentation layer, for example as a Python dictionary.

A good way to do this with Yattag is simply to use Python functions.
```
def display_article(data):

    doc, tag, text, line = Doc(
        defaults = data['form_defaults'],
        errors = data['form_errors']
    ).ttl()

    doc.asis('<!DOCTYPE html>')

    with tag('html'):

        with tag('body'):

            line('h1', data['article']['title'])

            with tag('div', klass = 'description'):
                text(data['article']['description'])

            with tag('form', action = '/add-to-cart'):
                doc.input(name = 'article_id', type = 'hidden')
                doc.input(name = 'quantity', type = 'text')

                doc.stag('input', type = 'submit', value = 'Add to cart')

    return doc.getvalue()
To reuse templates inside other template, you can use the asis method. Say you have a connection box, that appears on all the pages.

def connection_box(data):

    doc, tag, text, line = Doc().ttl()

    with tag('div', id = 'connection-box'):
        if data['user']:
            text('Hello ', data['user']['username'], '!')
        else:
            with tag('form', action = '/connect'):
                line('label', 'Username:')
                doc.input('username', type = 'text')

                line('label', 'Password:')
                doc.input('password', type = 'password')

                doc.stag('input', type = 'submit', value = 'Connexion')

    return doc.getvalue()
```
To reuse templates inside other template, you can use the `asis` method. 

Say you have a connection box, that appears on all the pages.
```
def connection_box(data):

    doc, tag, text, line = Doc().ttl()

    with tag('div', id = 'connection-box'):
        if data['user']:
            text('Hello ', data['user']['username'], '!')
        else:
            with tag('form', action = '/connect'):
                line('label', 'Username:')
                doc.input('username', type = 'text')

                line('label', 'Password:')
                doc.input('password', type = 'password')

                doc.stag('input', type = 'submit', value = 'Connexion')

    return doc.getvalue()
```
You can insert it at the top of the article page:
```
def display_article(data):

    doc, tag, text, line = Doc(
        defaults = data['form_defaults'],
        errors = data['form_errors']
    ).ttl()

    doc.asis('<!DOCTYPE html>')

    with tag('html'):

        with tag('body'):

            doc.asis(connection_box(data))

            line('h1', data['article']['title'])

            with tag('div', klass = 'description'):
                text(data['article']['description'])

            with tag('form', action = '/add-to-cart'):
                doc.input(name = 'article_id', type = 'hidden')
                doc.input(name = 'quantity', type = 'text')

                doc.stag('input', type = 'submit', value = 'Add to cart')

    return doc.getvalue()
```

You can use all the Python patterns you like. For example, if you think you need inheritance, you can put your templates into Python classes. However, simple functions, combined with the `asis` method are often enough to do what you want.
***
# 12. Adding style

You can keep your CSS in separate files and use
```
<link rel="stylesheet" href="style.css">
```
to connect it.

But if you need to keep the CSS  in the same file, use the `style` tag.

```
 with tag('head'):
        with tag('style'):
            doc.asis('p {color:blue}')
        with tag('html'):
            with tag('p'):
                text('Hello Yattag!')
```
  
*****
If you're like me, you want to be able to build everything with Python, even your html files. Fortunately for you, Yattag can help you with that so you can finally become a Full Stack Python Developer.

I find that Yattag is not only a fabulous library for Python, but also one with an amazing [documentation](https://www.yattag.org/#tutorial).
