Title: Lists in Web Components
Date: 2026-02-28
Tags: clojure

Suppose you have some UI in a website that should render a list of items. Let's call it `FancyList`. Usage locations of `FancyList` provide the list of items, each with a `label`, and, crucially, any other content that is rendered however the usage location requires.

In UI libraries like React and Svelte, components allow customized rendering using callback functions or slots. In React this might look like:


<p class="codepen" data-height="300" data-pen-title="Untitled" data-default-tab="html,result" data-slug-hash="OPRVBrG" data-user="tomconnors" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/tomconnors/pen/OPRVBrG">
  Untitled</a> by tom connors (<a href="https://codepen.io/tomconnors">@tomconnors</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>


In Svelte we use snippets but the concept is pretty much the same; the caller defines how items are rendered and the `FancyList` calls that logic.


We don't have a similarly convenient option for rendering child element lists when using web components. You can't pass a function to a web component with just HTML:

```html
<!-- This doesn't solve our problem -->
<fancy-list items="[...]"></fancy-list>
```

The best option I've found is conceptually pretty simple: instead of passing the items as an attribute of the web component, pass them as children. Structure the children so that the web component can easily pull out the information it needs, and include any customized rendering directly in the DOM.

<p class="codepen" data-height="300" data-pen-title="Untitled" data-default-tab="html,result" data-slug-hash="ZYpGmYe" data-user="tomconnors" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/tomconnors/pen/ZYpGmYe">
  Untitled</a> by tom connors (<a href="https://codepen.io/tomconnors">@tomconnors</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

There are some important things to note:
- We read the DOM the caller provided in `parseLightDom`. You can imagine more complex versions of this function for more complex situations.
- We use a mutation observer to keep track of the children provided by the caller so we can rerender if the items change.
- Instead of directly putting the elements provided by the caller into the WC's shadow DOM, we clone them. This works in more situations than just using innerHTML.
- We don't use `<slot>` elements because we have a *list* of items where each can render differently.
- This approach requires more HTML. Your HTML templating should make that manageable and compression should negate any performance concerns.
