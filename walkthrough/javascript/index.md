---
title: JavaScript
permalink: /walkthrough/javascript/
redirect_from: /en/apv/walkthrough/javascript/
---

* TOC
{:toc}

Sometimes a quick popup alert or a confirmation dialog is the best solution to displaying a notification or question
to a website visitor. Imagine that you have to write a PHP script and a template to confirm deletion
of a record -- it would be a lot of code. You probably saw a dialog with native look of the operating system
on other websites. Such dialog can be displayed using a [JavaScript](/articles/javascript) code and it
is a built-int part of the browser ([frontend](todo)).

In this article I will not teach you how to build an application entirely with JavaScript. I will show you how to use
very basic functions of this language to overcome the most common problems with the user interface -- quickly confirm some action or
validate a form before it is send to the [backend](todo).

## Linking JavaScript to your HTML
[Similarly to CSS](/walkthrough/css/#linking-your-css-to-html-file), a JavaScript file is usually referenced in the `<head>`
element using a `<script>` tag (however it may be placed in other parts of the HTML too).
The `<script>` tag can contain either an URL to download a script or it can directly contain some
JavaScript code:

{% highlight html %}
{% include /walkthrough/javascript/attach.html %}
{% endhighlight %}

In the above example, the actual JavaScript code is `console.log("Some message.");` and `console.log("Another message.");`.
{: .note}

The basic JavaScript syntax is described in a separate [article](/articles/javascript/#javascript-basics).
The `console` is a global object that represents [browser's developer console](/course/not-a-student/#web-browser)
(usually activated by F12 key). The `log()` method puts some output into it. You should have the console activated every time
you develop some JavaScript functionality. Therefore the above code would print two messages in the console.

![Screenshot - Browser Console](console.png)

## Working with Elements
The source code of the linked scripts is executed immediately in the order of the `<script>` tag appearances. Usually you want
to *attach some [event handlers](/articles/javascript/#javascript-events)* ---   execute a piece of code after a visitor
performs some action (e.g. clicks a button). Event handlers can be attached to general events which take place globally
(like loading the whole page or scrolling the window) or to events that take place on a particular HTML element (clicking on
an element or focusing an element). To attach to an element event, you have to find the element first and then attach the handler in the JavaScript code.

There are multiple functions for locating HTML elements --- the easiest one is
`document.getElementById("id_of_element")` which can find and return one element using its `id`
attribute, `document` is a global object which contains the [tree structure](/articles/html/#hierarchical-structure) of your HTML elements.

Other useful functions to retrieve HTML elements:

- `document.getElementsByTagName("table")` --- returns an array of elements with a given tag name (i.e all `<table>` elements)
- `document.getElementsByClassName("some-class")` --- returns an array of elements with a given `class` attribute
- `document.querySelector(".some-css-selector")` --- returns the first element matched by a given [CSS selector](/articles/css/#selectors)
- `document.querySelectorAll(".some-css-selector")` --- returns an array of elements matched by a given [CSS selector](/articles/css/#selectors)

When you have elements, you can access their HTML attributes by their standard names (e.g. `console.log(link.href)` for
an `<a>` element). An exception is the `class` attribute which must be accessed using the `element.className` property.
To change the CSS styles use the `element.style` object followed by the [*camelCase*](https://en.wikipedia.org/wiki/Camel_case) CSS property name (e.g. the
CSS property `background-color` can be accessed with
`element.style.backgroundColor`). Another special attribute is `innerHTML` which can be used to change the content of an
element. You may remember about [user defined attributes](/articles/html/#data-attributes) which can be accessed
in the `element.dataset` object (e.g. `element.dataset.personid` for `data-personId` attribute).

{% highlight html %}
{% include /walkthrough/javascript/html-attributes-styles.html %}
{% endhighlight %}

{: .note}
Although it is possible, you should avoid changing particular CSS styles like in the above example. It is tedious and makes your code confusing. You should rather add or remove CSS classes (there is a [`classList`](https://developer.mozilla.org/cs/docs/Web/API/Element/classList)
field of the HTML element which allows efficient work with CSS classes).

## Working with Events
As described above, various **events** occur on the web page, either automatically, or through user interaction.
To be able to respond to events, you need to **register a handler** (or **subscribe for an event**). A *handler*
is a piece of code which *handles* the event (i.e. does something in response to that event).
The event handler is also commonly called a **callback** (as a piece of code which is "called back" from
an event situation).

Basic [event registration](/articles/javascript/#javascript-events) can be performed in the same manner
as [working with standard HTML properties](#working-with-elements) --- event
handlers are attributes of the HTML elements. The limitation of this basic approach is that you can attach only one
event handler to each type of event. You can attach [multiple events](/articles/javascript/#javascript-events)
using the `addEventListener('event', handler)` method of the element object.

{% highlight html %}
{% include /walkthrough/javascript/basic-events.html %}
{% endhighlight %}

If you are new, to Javascript (or any programming), you may be confused by the line `window.onload = function() ...`. This is indeed something new as
the `onload` property expects to be assigned a handler (which is a piece of code). An that is exactly what the line does, it assigns a function
which has no name and no parameters to the `onload` property of `window`. A function which has no name is called **anonymous function**. The same applies to
line `<button onclick="callMe()" ...` where I assign the code `callMe()` as the onclick handler. `callMe()` is a function which I have declared earlier.
(It might help you to write `<button onclick="callMe();"` where the semicolon emphasizes that the attribute content is an actual Javascript line.)
It is important that the function must be called in the handler, i.e. it is insufficient to write only `<button onclick="callMe"` as that will assign
function name to the handler and it will not execute the function. Though both approaches are valid and often interchangeable, they do not behave the
same (but that is rather complicated, so more on that later).

{: .note}
The above is still an example. A cleaner approach is to divide the HTML and the JavaScript code and attach the events in JavaScript. Also imagine
how ugly it looks if you write complex code inside the `onclick` attribute of an HTML element.

## Manipulating HTML
The same way you can manipulate HTML attributes, you can also manipulate with the elements themselves. This means that
the HTML code can be changed **dynamically** when something happens on the page.

Each element can have a set of child nodes, which can be manipulated using `appendChild` and `removeChild` methods. For example, you can
remove or add children (`elem2`) to an HTML element `elem` with `elem1.appendChild(elem2)` and `elem1.removeChild(elem2)`.
To create a new element you can use `document.createElement` method - e.g. `var newElem = document.createElement("<p>")` to create a new paragraph.

{% highlight html %}
{% include /walkthrough/javascript/creating-elements.html %}
{% endhighlight %}

In the above code, first I create a list `ul` element (`newList` variable). Then I pick an existing HTML element (with id `container`) and
add the list as a child of the container. Then I add two items to the list. Notice the difference between assigning `innerHTML` and `textContent`.
The latter is much safer approach, not susceptible to [XSS attacks](todo).

### Task -- Toggle Class Using a Button Click
Make a button and some HTML element with an `id` attribute. Attach a click event to the button using an `onclick` attribute.
Toggle CSS class named `highlight` on the element with the `id` using the `element.classList.toggle('className')` method.
You also need to define the CSS class `highlight` with some visual properties defined (e.g. green border).

{: .solution}
{% highlight html %}
{% include /walkthrough/javascript/events-toggle-css.html %}
{% endhighlight %}

{: .note}
If you came up with another solution, do not worry, there are always multiple working solutions when you write any code.

## Confirm User Actions
In the chapter about [records deletion](/walkthrough/backend-delete), you were referred to this article for
information about how to confirm a user action. Using JavaScript, we can prevent the visitor's browser from sending
a HTTP request (i.e. following a link or sending a form). This is called *preventing navigation* and might be useful for
example when the user is filling a form and you want to make sure that she does not accidentally leave the page (and loose the filled form).
Here is an example how to prevent navigation with a confirm popup for basic `<a>` element:

If you want to prevent navigation on a form, you have to use the `onsubmit` event:

{% highlight html %}
{% include /walkthrough/javascript/prevent-nav-form.html %}
{% endhighlight %}

You have to pass a true/false value returned by the `confirm` function from the called `confirmForm()` function
using the `return` keyword in the `onsubmit` attribute. This is because the attribute itself is the body of the event
handler function and has to return `true` or `false` to approve or cancel from submission.

Similar confirmation can be created on links:

{% highlight html %}
{% include /walkthrough/javascript/prevent-nav-a.html %}
{% endhighlight %}

Again, the handler must return a boolean value from the `confirmNav` function.

### Task -- Pass Parameters to the Handler
Let's create and alert for a form which takes some parameters. I have slightly modified the example above, there
are now two forms, but we would still like to use the same `confirmForm` function.

{% highlight html %}
{% include /walkthrough/javascript/prevent-nav-form-parameters-1.html %}
{% endhighlight %}

Modify the code so that you pass a parameter to the `confirmForm`, and display appropriate confirmation message.

{: .solution}
{% highlight html %}
{% include /walkthrough/javascript/prevent-nav-form-parameters-2.html %}
{% endhighlight %}

There are many different solutions. In the solution above, I used the `+` operator to concatenate strings in
the message and I directly passed the action being confirmed. The `+` operator in Javascript works both for
adding numbers and concatenating strings.

### Task -- Confirm Person Deletion
Now enhance the [script for deleting persons](/walkthrough/backend-delete). Insert a piece of
Javascript code which will prompt the user for confirmation before deleting the person.
Feel free to to insert the JavaScript code directly into the template, but be careful
where you insert the `<script>` tag -- remember that `{foreach ...}` duplicates all source inside
and you do not want multiple `<script>` tags with same function declaration in your markup.
Try to pass the person name into the confirmation message.

{: .solution}
{% highlight html %}
{% include /walkthrough/javascript/templates/delete.latte %}
{% endhighlight %}

{: .note}
Remember that the actions representing modifications of the database records should be transmitted to
the server using the [`POST` method](todo).

## Form validation
Nowadays you can use much more types of [input elements](/walkthrough/html-forms/#advanced-text-input)
than a few years ago. This means that many validations can be carried out by the browser.
You can adjust CSS styles using `:valid`, `:invalid` or `:required` [pseudo-classes](/walkthrough/css/#pseudoclasses)
to visually differentiate input states. You should use these capabilities as much as possible. Nevertheless
you may sometimes need to implement some custom logic. For example switch `required` state or
`enable`/`disable` some input in dependence of another input's value or dynamically
calculate some additional value such as price.

Here is an example with dynamic form which simulates a flight reservation. I used a `<fieldset>` element
which is capable of disabling or enabling multiple inputs within it. These inputs represent optional
baggage section of a flight reservation form. Based on the selected options, the price of flight is
adjusted immediatelly:

![Form validation](form-validation.png)

{: .solution}
{% highlight html %}
{% include /walkthrough/javascript/form-validation.html %}
{% endhighlight %}

I used `document.forms` which contains object with keys defined by the form `name` attribute.
Each form is again an object with keys defined by inputs' `name` attribute. Attributes of
JavaScript objects can be accessed using square brackets (where you can also use a variable)
or you can just use the dot notation `.key`. There is no functional difference between
`document.forms.formName` and `document["forms"]["formName"]`
or `document.forms["formName"]`. I prefer latter variant because attribute values can
contain characters like `-` which are reserved in JavaScript and cannot be used in the former variant.

{: .note}
This approach is ideal for manipulating form elements (such as enabling/disabling the fieldset as
in the above example). Computing the final price this way is not ideal, because it
means duplicated logic in the server and the client scripts (the price computed by the client
must be considered unreliable). A better solution in such case is to use
[AJAX](/articles/javascript#ajax).

### Task -- Add `required` attribute to Person & Address form inputs dynamically
Do you remember when I was talking about inserting [multiple records at once](/walkthrough/backend-insert/advanced/).
You should have already created a form where you can insert a person and a place where he lives.
Try to extend this form with JavaScript and add `required` attribute to `city` only when a user enters a `street_name`.

{: .solution}
{% highlight html %}
{% include /walkthrough/javascript/form-validation-address.html %}
{% endhighlight %}

## Summary
This brief chapter about JavaScript programming language should help you to improve your project with instant popup
dialogs and a bit of client-side form validation. You can find more in [articles](/articles/javascript)
section of this book.

### New Concepts and Terms
- JavaScript
- Event
- Event Handler
- Callback
- Anonymous function
- Dynamic HTML