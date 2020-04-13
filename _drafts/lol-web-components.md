---
layout: post
class: post
title: lol I made my own component framework
subtitle: why did I do this (oh yeah because of the pandemic)
syndications: {}
---

This website is a playground for me, so I decided to rewrite my [Blog Admin code][] to [use Web Components][Blog Admin web components]. It worked, and I liked it! 🎉 But then I didn't: writing HTML in JavaScript strings isn't great. Why can't we have both HTML and JavaScript defined in the same file _and_ encapsulated separate from the rendered page?

<details markdown="1">
<summary>Table of contents</summary>
In this article, we will:

1. [Design our own component HTML file](#component-html-file-design)
2. [Create a Web Component that imports the component HTML file](#how-do-we-import-a-html-file)
3. [Add the template, style, and script from the component HTML file into the `ShadowRoot` of the Web Component](#add-the-css-and-javascript-from-the-component-file)
4. [Ensure the script executes](#ensure-the-script-executes)
5. [Export a `View` class from the script](#export-a-view-class-from-the-script)
6. [Turn the script into an importable module](#turn-the-script-into-an-importable-module)
7. [Import and initialise the `View` class with the `ShadowRoot`](#import-and-initialise-the-view-class-with-the-shadowroot)
8. [Change the template dynamically](#change-the-template-dynamically-in-the-view-class)
9. [Notes](#notes)
</details>

### That sounds like the HTML Imports spec

([HTML Imports explained beautifully on html5rocks.com](https://www.html5rocks.com/en/tutorials/webcomponents/imports/))

Unfortunately, you may have heard, HTML Imports has been abandoned – the only browsers that supported it were Blink-based (Chrome, Opera, Edge) and now they've deprecated it. The reasons why are explained succinctly by the proposal to replace it: [HTML Modules W3C proposal to replace HTML Imports](https://github.com/w3c/webcomponents/blob/7e4476e32a69a5de2374e4b8e1ef4f9c4ad47fdc/proposals/html-modules-proposal.md).

<a id="html-imports-problems"></a>Namely:
1. Global object pollution
2. Inline scripts block the document parsing
3. Inability for script modules to access declarative content

So what can we do? It would be so nice (super nice) to have everything about a component all in the same file: HTML, CSS (`<style>` and `<link rel=stylesheet>`), and JavaScript.

<small>cough [Vue][]</small> – hm? What's that? <small>cough [Svelte][] cough</small> – huh? Oooooohhhhhhhhh, _Frameworks._

Yeah... nah 🙃

### I'm not hating on frontend frameworks

They're great! Easy! Convenient! Well supported and fantastic browser compatibility! Fun to write in! And I would _absolutely_ choose a framework for a production system because writing something custom will be painful, no doubt 😅.

However, I wanted to keep my website free of a frontend build step, so I could write and commit HTML, CSS, and JavaScript anywhere at any time. This is a Jekyll site, but that compilation is handled for me by GitHub Pages, so I can commit via git or the GitHub file editor.

If you don't mind a build step, that's great! You can probably get a lot more functionality out of something from ["The Simplest Ways to Handle HTML Includes" on css-tricks.com][css-tricks.com ways to include HTML] than this weird little mismash of code that you're about to see here 🙃

### "_...because writing something custom will be painful, no doubt_ 😅"

We're here for fun, so let's do it anyway!

What do we want? _HTML and JavaScript defined in the same file!_
~~When~~ Where do we want it? _!n the same file!_

### Component HTML file design
So something like this then?

```html
<template>
  <p>This is a lovely component</p>
</template>

<style>
  p {
    color: red;
  }
</style>

<script>
  alert('It works!');
</script>
```

👉<i class="fab fa-vuejs" aria-label="Vue JS"></i> 👈👀

lol yes ok, this is pretty much a [Vue single-file component][]. We could use VueJS (or Svelte) – but again, that would require a build step, so ❌ (_buzzer sound_)

Anyway, this can very easily go into a Web Component: we have the `<template>`, and everything else is regular HTML ✅

### How do we import a HTML file?

The [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) is our friend here: we can `fetch()` the HTML file from the webserver[*](#ft-jekyll-fetch-html) as plain text, perfect for inserting into a holding element to get the browser to build the HTML for us:
```js
fetch('/test.component.html')
  .then(response => response.text())
  .then(html => {
    const holder = document.createElement('div');
    holder.innerHTML = html;
    const template = holder.querySelector('template'); // our <template> from above
  });
```

Let's put this into a Web Component:
```js
// test.component.js
export class TestComponent extends HTMLElement {
  constructor() {
    super();
    const shadowRoot = this.attachShadow({ mode: 'open' });

    fetch('/test.component.html')
      .then(response => response.text())
      .then(html => {
        const holder = document.createElement('div');
        holder.innerHTML = html;

        const template = holder.querySelector('template');
        shadowRoot.appendChild(template.content.cloneNode(true));
      });
  }
}
```
```html
<!-- index.html -->
<body>
  <test-component></test-component>
  <script type="module">
    import { TestComponent } from './test.component.js';
    customElements.define('test-component', TestComponent);
  </script>
</body>
```

![Browser dev tools showing the paragraph content from the test web component correctly loaded into the DOM](/images/lol-web-components-test.png)

Great! It works!

### Add the CSS and JavaScript from the component file
```js
const style = holder.querySelector('style');
const script = holder.querySelector('script');
shadowRoot.appendChild(style);
shadowRoot.appendChild(script);
```

![Browser dev tools showing the red paragraph style correctly applied to the test component](/images/lol-web-components-test-with-styles.png)

The `<style>` is working, but there was no alert: the `<script>` didn't run 🤔 Why is that?

### Ensure the script executes

Oh: the [HTML5 spec on innerHTML][] says
>Note: script elements inserted using innerHTML do not execute when they are inserted.

<small>Thanks to Daniel Crabtree's article: [Gotchas with dynamically adding script tags to HTML][]</small>

But we _can_ execute inline JavaScript as long as we use `document.createElement('script')`, then we can insert the contents with `innerHTML`:
```js
const script = holder.querySelector('script');
const newScript = document.createElement('script');
script.getAttributeNames().forEach(name => {
  // Clone all attributes.
  newScript.setAttribute(name, script.getAttribute(name));
});
// Clone the content.
newScript.innerHTML = script.innerHTML;
// Adding will execute the script.
shadowRoot.appendChild(newScript);
```

![Browser alert saying "It works!"](/images/lol-web-components-test-with-script.png)

<p style="text-align:center; font-size: 3em;">🎉 🎉 🎉</p>

### That's it!

All done now. Nothing else to do. Bye-bye, see you later 👋

<br>

<br>

<br>

<br>

<br>

<br>

 ………<br>

<br>

<br>

<br>

<br>

<br>

Oh, you wanted interactivity in your component? 😅 Sure ok, let's continue!

### Export a `View` class from the script

```js
class View {
  constructor(el) {
    this.el = el;

    this.el.addEventListener('click', () => {
      alert('I was clicked');
    });
  }
}
```

Ok. That's good and all, very generic. Seems like a view that tells you when anything inside it is clicked. Great!

But… how do we give it the template?

### Turn the script into an importable module

Maybe we can export it?
```html
<!-- test.component.html -->
<script type="module">
  export class View {
    // ...
  }
</script>
```

Ok, now let's import it and give it the element! Let's just say that we always need to export a `View` class, so we don't have to do any kind of special detection: if a module exports `View`, we use it.

So, back in our Web Component, after appending the script to the `shadowRoot`, let's import this one:
```js
// test.component.js
export class TestComponent extends HTMLElement {
  constructor() {
    super();
    const shadowRoot = this.attachShadow({ mode: 'open' });

    fetch('test.component.html')
      .then(response => response.text())
      .then(html => {
        const holder = document.createElement('div');
        holder.innerHTML = html;

        const template = holder.querySelector('template');
        const style = holder.querySelector('style');
        shadowRoot.appendChild(template.content.cloneNode(true));
        shadowRoot.appendChild(style);

        const script = holder.querySelector('script');
        const newScript = document.createElement('script');
        script.getAttributeNames().forEach(name => {
          newScript.setAttribute(name, script.getAttribute(name));
        });
        newScript.innerHTML = script.innerHTML;
        shadowRoot.appendChild(newScript);

        import { View } from newScript; // this is an error
      });
  }
}
```
Hold on: how can we import the script tag?

Firstly, we cannot use static imports, so we must use dynamic imports:
```js
import(newScript).then(module => {
  module.View; // our view class
});
```

Secondly, that doesn't work because the `script` variable is an `HTMLScriptElement`, not an importable string.

Hhmmm. This sounds like [HTML Imports Problem No.3](#html-imports-problems):
>Inability for script modules to access declarative content

### Import and initialise the `View` class with the ShadowRoot

_/me searches "inline script module export" … … … a-ha!_

I found an [example of "Inlining ECMAScript Modules in HTML"][stackoverflow inline module] on StackOverflow<sup>[&dagger;](#ft-stackoverflow-merits)</sup>. In it, the contents of the script can be turned into an "Object URL", which we can use to import!

So let's do that before adding the script to the `shadowRoot`:
```js
// test.component.js
const script = holder.querySelector('script');
const newScript = document.createElement('script');
newScript.innerHTML = script.innerHTML;

const scriptBlob = new Blob([newScript.innerHTML], {
  type: 'application/javascript'
});
newScript.src = URL.createObjectURL(scriptBlob);
shadowRoot.appendChild(newScript);
```

Now we should be able to import and initialise it:
```js
// test.component.js
import(newScript.src).then(module => {
  new module.View(shadowRoot);
});
```

### Change the template dynamically in the `View` class

Let's change our view to act on the element:
```html
<!-- test.component.html -->
<script type="module">
  export class View {
    constructor(el) {
      el.querySelector('p').innerText = 'The view has initialised!';
    }
  }
</script>
```

![Browser showing "The view has initialised!" text appended to our component](/images/lol-web-components-test-with-view.png)

All done!
_dusts off hands_

<p style="text-align:center; font-size: 3em;">🎉 🎉 🎉</p>

### Notes

This example was not intended for good (or even average!) performance or browser compatibility. I'm the only person using code like this, on two very specific devices. Other users of my site don't even receive this code: it's an Admin interface that is only loaded if I'm logged-in (see my other post: ["I can write this from my phone"]({% post_url 2020-04-08-i-can-write-this-from-my-phone %})).

The Web Component in this example can be totally generic by taking an import url as an input, so you don't have to create a new Web Component – which, to be honest, is a bit of a pain – for every HTML component you have. In fact, that's exactly what I've done with my Admin interface: I can put `<html-import data-href="./amazing.component.html"></html-import>` anywhere and it will load that component 🎉

You can read more about it here: [my blog admin interface HTML component on GitHub](https://github.com/henrahmagix/henrahmagix.github.io/commit/3f4f1cf0010fe01a215a4267f4adee3d571e2561#diff-21232f297a57a5a743894a0e4a801fc3) – it's a lot more complicated than this example at the time of writing, and probably doesn't need to be (I don't know if it will ever be finished 😅)

---

<small id="ft-jekyll-fetch-html">* Thankfully Jekyll doesn't transform it with a layout since it doesn't have any front matter (yaml at the top of the file), so the response will only be the contents of the file: perfect for inserting into a holding element to get the browser to build the HTML for us!</small>

<small id="ft-stackoverflow-merits"><sup>&dagger;</sup> StackOverflow is not kindest of places, brought to my attention by April Wensel ([here is an example][April Wensel on StackOverflow]). It is also where I have found answers for the majority of my problems. So reading it is usually fine, but interacting with it – posting, commenting, answering – can be a dire experience. I suggest reading April Wensel to find out more.</small>


[Vue]: https://vuejs.org/
[Svelte]: https://svelte.dev/
[Blog Admin code]: https://github.com/henrahmagix/henrahmagix.github.io/blob/fc8bc1d2596ba77e3c82cab28e2b8afa8d35ac8d/admin/post-list-admin.js
[Blog Admin web components]: https://github.com/henrahmagix/henrahmagix.github.io/commit/3296da3bd3ce8364fd12061f9094a63ca81d13df
[css-tricks.com ways to include HTML]: https://css-tricks.com/the-simplest-ways-to-handle-html-includes/
[Vue single-file component]: https://vuejs.org/v2/guide/single-file-components.html
[question: On what specific grounds were HTML Imports rejected, deprecated and removed?]: https://webmasters.stackexchange.com/questions/127482/on-what-specific-grounds-were-html-imports-rejected-deprecated-and-removed
[Gotchas with dynamically adding script tags to HTML]: https://www.danielcrabtree.com/blog/25/gotchas-with-dynamically-adding-script-tags-to-html
[HTML5 spec on innerHTML]: https://www.w3.org/TR/2008/WD-html5-20080610/dom.html#innerhtml0
[stackoverflow inline module]: https://stackoverflow.com/a/43834063/3150057
[April Wensel on StackOverflow]: https://medium.com/@Aprilw/suffering-on-stack-overflow-c46414a34a52 "April Wensel on StackOverflow"

<style>
  .entry-content h3,
  .entry-content h4,
  .entry-content h5,
  .entry-content h6 {
    margin-top: 2em;
  }
</style>