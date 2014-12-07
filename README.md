Nod v.2.0
=========

Frontend input validation


Practical info
--------------

* **License:** MIT.
* **Dependencies:** None.
* **Browser support:** Chrome (newest) / FF (newest) / ie9+.
  * Please help me test in ie (9 and up). I have very limited access to windows machines.
* **Backwards compatibility:** ver. 2 is *not* compatible with previous versions.


Example
-------

Cloning the project, and checking out the examples is the best way to go. But here are some basic stuff to give you and idea and get you going.

```html
<form action=''>
    <label>
        <span>Name</span>
        <input type='text' class='name'>
    </label>
    <label>
        <span>Email</span>
        <input type='text' class='email'>
    </label>
    <label>
        <span>Email again</span>
        <input type='text' class='email-again'>
    </label>
    <label>
        <input type='checkbox' class='terms'>
        <span>I agree to terms &amp; conditions</span>
    </label>
    <button class='submit-btn' type='submit'>Sign up</button>
</form>
```

A basic sign up form. Let's add some validation.

```javascript
var n = nod();

// We disable the submit button if there are errors.
n.configure({
    submit: '.submit-btn',
    disableSubmit: true
});

n.add([{
    selector: '.name',
    validate: 'min-length:2',
    errorMessage: 'Your name must be at least two characters long.'
}, {
    selector: '.email',
    validate: 'email',
    errorMessage: 'That doesn\'t look like it\'s a valid email address.'
}, {
    selector: '.email-again',
    validate: 'same-as:.email',
    errorMessage: 'The email does not match.'
}, {
    selector: '.terms',
    validate: 'checked',
    errorMessage: 'You must agree to our terms and canditions.'
}]);
```


Documentation
-------------


### Adding input fields

`nod` is a factory, which spits out instances of nod when called (no parameters).

```javascript
var myNod = nod();
```

Elements can then be added to it via objects (or an array of objects) describing each.

```javascript
myNod.add({
    selector: '.foo',
    validate: 'presence',
    errorMessage: 'Can\'t be empty'
});
```

The `selector` property works much like a jquery selector (in fact, if jquery
is available, it will use jQuery to do the work). You can throw most anything
at it. A simple css type selector (like in the example above), a list of
selectors, raw dom elements, jQuery elements, even <a
href='https://developer.mozilla.org/en/docs/Web/API/NodeList'>NodeLists</a>.

The `validate` property can be a string with the name of the function you want
to validate the input of the element with (there should be a list somewhere on
this page), or your own function (see next example), a RegExp, or you can
directly extend `nod.checkFunctions` with your own function and reuse it
throughout your website.

The `errorMessage` is a string containing the error message (in case the
validation fails) that will be shown behind the field.

```javascript
myNod.add([{

    // Raw dom element
    selector: document.getElementById('foo'),

    // Custom function. Notice that a call back is used. This means it should
    // work just fine with ajax requests (is user name already in use?).
    validate: function (callback, value) {
        callback(value % 2 === 0);
    },

    errorMessage: 'Must be divisible by 2'

}, {

    // jQuery element
    selector: $('.bar'),

    // EegExp
    validate: /hello/g,

    errorMessage: 'Input must say hello'

}]);
```

There is one more setting you can add; `triggeredBy`. This is a selector for
another element (it will only match the first matched element), where changes
will also trigger a check on the current element. Example:

```javascript
// Feel free to use `nod.getElement` and `nod.getElements` to get the row dom
// element(s).
var someCheckbox = nod.getElement('.some-checkbox');

myNod.add({
    selector: '.some-input',
    // You could also just add the raw dom, just created above.
    triggeredBy: '.some-checkbox',
    validate: function (callback, value) {
        if (someCheckbox.checked) {
            callback(value.length > 0);
        } else {
            // If checkbox isn't checked, then it doesn't matter if .some-input
            // is filled out.
            callback(true);
        }
    },
    errorMessage: 'If the checkbox is checked, then we need your name'
});
```


### Removing input fields

Once added, you can also remove an input field from being checked. It's done
much like you'd expect (I hope).

```javascript
myNod.remove('.foo');
```

If `'.foo'` matches more than one element, they will all be removed.



### Some configuration

Let's go through each of the options in `nod.configure()`.

#### Classes

We can change the classes of both the "parent" and the "error span". I'll go
through each referring to this html:

```html
<div class='group'>
    <label>
        <span>What's the meaning of life?</span>
        <input type='text' class='foo'>
        <!-- 
            error span gets added here:
            <span class='nod-error'><%= errorMessage %></span>
        -->
    </label>
</div>
```


##### Parent

By default, the `label` (the parent of the input) will have the class
'nod-success' or 'nod-error' when the element is checked. These class names can
be changed via `nod.configure` like so:

```javascript
myNod.configure({
    successClass: 'myNewSuccessClass',
    errorClass: 'myNewErrorClass'
});
```

If you want, instead, that the "parent" isn't its immediate parent, but
something further out, you can tell nod to search for a parent with a class
like so:

```javascript
myNod.configure({
    parentClass: 'group'
});
```

Notice that the error span that the error span always gets appended to the
"parent", in the above example, it would now be added *after* the `</label>`
and not *inside* it.


##### Error Span

The class of the error message can also be changed:

```javascript
myNod.configure({
    successMessageClass: 'myNewMessageSuccessClass',
    errorMessageClass: 'myNewMessageErrorClass'
});
```

However, nod doesn't show a message behind the input field when it's valid. You
have to set that by the configure method as well:

```javascript
myNod.configure({
    successMessage: 'Well done!'
});
```

The message will be shown after every valid input field in `myNod`.

Changing the position of the error span is a bit more tricky (unless you do it
by changing the parentClass, see above). You can, specifically for each element
tell nod which dom element to use as the error span. See `setMessageOptions`
below.

    

#### Delay

By default nod waits 0.7 seconds before display an error message. This is so
the user isn't being bothered with error messages about something that he or
she is not done typing. This delay can be changed via `configure` method:

```javascript
myNod.configure('delay', 300);
```

Notice, however, that this delay only deals with the time before *showing* an
error message. Removing it will happen instantly, and there is currently no way
to change that.



#### Disable submit button

You can disable the submit button, until all added elements are considered
valid. This is done like so:

```javascript
myNod.configure({
    submit: '.submit-button',
    disableSubmit: true
});
```

However, be aware, that forms can often be submitted by hitting enter, and a
disabled submit button will not do anything to prevent that (see next section).



#### Prevent Submits

If you tell `nod` about the form, then it will prevent submits entirely, until
all added elements are considered valid.

```javascript
myNod.configure({
    form: '.myForm'
});
```

I should caution the use of this however, as it is hard to get it right in every case (from me, the designer's perspective). So test it well, and make sure it is working correctly in your use case. The last thing you want (I assume) is to prevent your users from submitting your form entirely.



#### Tap into the checks

You can "tap" into the internals, and get updates about the state of your
elements by adding a "tap" function via `configure`.

```javascript
myNod.configure({
    'tap': function (options) {
        console.log(options);
    }
});
```

This will give you *a lot* of calls to your console (basically one, per key
press). The `options` argument contains various properties used internally and
is *very* likely to change (that's why it's an object and not 5 different
arguments), but the most useful are probably `options.element`,
`options.result`, and `options.validate`.



### setMessageOptions

You can specifically for each element tell `nod` which is its parent and which
dom element to use as its error span:

```javascript
    myNod.setMessageOptions({
        selector: '.foo',
        parent: '.myCustomParent',
        errorSpan: '.myCustomErrorSpan'
    });
```

Notice, that the "parent" (despite its name) does not have to strictly be a
parent. There is also no need to set both the parent and the message.


### Manually checking validity

`nod` currently exposes two functions for for manually checking validity of the
elements. One for checking all elements (this is the same used, to
enable/disable the submit button internally:

```javascript
myNod.isAllValid(); // Returns a boolean
```

And you can also query one element specifically like so:

```javascript
myNod.getStatus('.foo'); // Returns: 'unchecked', 'valid', or 'invalid'
```

If you need to check them up against something, I suggest you use
`nod.constants` to do so. Like this:

```javascript
if (myNod.getStatus('.foo') === nod.constants.VALID) {
    // Do something
}
```


### List of check functions.

Most should be pretty self explaining.

`[String]` should be replaced with whatever string you feel is appropriate. See
examples below (or in the examples folder).

* `"presence"`
* `"exact:[String]"`
* `"contains:[String]"`
* `"not:[String]"`
* `"min-length:[Number]"`
* `"max-length:[Number]"`
* `"exact-length:[Number]"`
* `"between-length:[Number]:[Number]"`
* `"max-number:[Number]"`
* `"min-number:[Number]"`
* `"between-number:[Number]:[Number]"`
* `"integer"`
* `"float"`
* `"same-as:[Sring]"` (A css type selector)
* `"one-of"`
* `"only-one-of"`
* `"checked"` (For checkboxes only)
* `"some-radio"` (For a group of radio buttons)
* `"email"` (Uses the RFC822 spec to check validity)

#### A few examples of how to use the above list

```javascript
myNod.add({
    selector: '.foo',
    validate: 'exact:foo'
    errorMessage: 'You must write exalctly "foo" in the input field'
});
```

All are accessed through the `validate` property of adding an element.

Some more examples:

```javascript
// ...
{
    selector: '.foo',
    validate: 'between-length:2:4',
    errorMessage: 'Must be between 2 and 4 characters long'
}
// ...
{
    // This will check that at least one of the inputs matched by the selector
    // has a value.
    selector: '.phone-number-inputs',
    validate: 'one-of',
    errorMessage: 'You need to type in at least one phone number'
}
// ...
{
    // You can add validate functions in a list. Just remember to also have
    // errorMessages be a list with corresponding texts.
    selector: '.foo',
    validate: ['email', 'max-length:8'],
    errorMessage: ['Must be a valid email', 'Your email is too long. Sorry.']
}
// ...
{
    selector: '.foo',
    validate: 'same-as:.bar',
    errorMessage: 'Must be the same as in .bar'
}
```

`one-of`, `only-one-of`, and `some-radio` all match on their `selector`.
`same-as` is called as `same-as:[selector]`.


### Extending nod.checkFunctions

You can extend the available check functions in `nod` like this:

```javascript
    // Note that this is the general `nod` function. Not a particular instance.
    nod.checkFunctions['divBy2'] = function () {
        return function (callback, value) {
            callback(value % 2 === 0);
        };
    };
```

This function can then be reused:

```javascript
myNod.add({
    selector: '.foo',
    validate: 'divBy2',
    errorMessage: 'Must be divisable by 2'
});

We can also use arguments when setting up the function. Like so:

```javascript
    nod.checkFunctions['divByX'] = function (x) {
        x = parseInt(x, 10);

        return function (callback, value) {
            callback(value % x === 0);
        };
    };
```

And to define "x", we use it like so:

```javascript
myNod.add({
    selector: '.foo',
    validate: 'divByX:3',
    errorMessage: 'Must be divisable by 3'
});

If you need more arguments, you just separate them with ":".

```javascript
    validate: 'myFunc:a:b:c:d:e'
```

