# Share Button Type

## Summary

A value of “share” for the `type` attribute of the `button` element would allow authors to provide an interface element for sharing the current web page. This is already possible with the [Web Share API](https://github.com/w3c/web-share) but only by writing JavaScript. `button type="share"` provides a declarative alternative for simple use cases.

## Background

The [Web Share API](https://github.com/w3c/web-share) provides a means “for sharing text, URLs and images to an arbitrary destination of the user's choice”. The most basic sharing use case is sharing the current page (especially important in progressive web apps that launch with a `display` value of “standalone” or “fullscreen”). A declarative option that meets this basic use case would enable authors to provide this functionality without requiring knowledge of JavaScript.

## Proposed Plan

The `type` attribute of the `button` element currently accepts one of three possible values:

* `submit`
* `reset`
* `button`

The list of possible values can be extended in the same way that `type` values for the `input` element were expanded in HTML5.

Just as with the `input` element, the fallback behaviour for non-supporting browsers is consistent. Just as browsers will assume a default value of “text” for unknown `input` `type`s, browsers assume a default value of “submit” for unknown `button` `type`s. This makes it possible to use feature detection to provide polyfills for non-supporting browsers.

As it is, the Web Share API requires a user interaction in order to run so authors are already providing interface elements (like buttons) for this purpose. Currently authors also have to write JavaScript to listen for an event on that interface element and then invoke the Web Share API. A declarative `button type="share"` would remove that need (as long as the URL and title fields are from the current page).

A browser that supports the Web Share API and `button type="share"` could invoke the following actions when the button is activated:

1. Find the current page title and URL. This can be done using the same means that browsers currently use for bookmarking functionality using the value of `document.title` and the value of `window.location.href`.
2. Invoke an instance of the Web Share API, passing in those values for the `url` and `text` fields.

No special styling rules should apply to a `button` type of "share". The author is free to style the element in the same way as styling any other `button` element.

The content model for the `button` element remains unchanged regardless of the value of the `type` attribute.

## User Flow

This is based on [the user flow documented for the Web Share API](https://github.com/w3c/web-share/blob/master/docs/explainer.md#user-flow).

Here's how a user could share a link from a website to a native app of their choice on a site using `button type="share"`:

1. User is browsing a website with recipes. On a recipe page, the user clicks a share button.
2. A modal picker dialog is shown to the user, with a set of native applications and system actions (e.g., "Gmail", "Facebook", "Copy to clipboard"). On Android, this is the system intent picker, but the implementation may differ between browsers and operating systems. The user picks "Gmail".
3. The Gmail native app opens, and is pre-populated with the title and URL.

This functionality is currently provided by some browsers in the browser interface, but if this recipe site is a progressive web app that has been added to the user’s home screen, the browser interface may not be available.

## Examples

Authors can choose the content (and styling) of `button type="share"`:

```
<button type="share">Share this</button>
```

The content model of the `button` element also accepts images:

```
<button type="share"><img src="icon.png" alt="Share this"></button>
```

### Polyfill

Support for `button type="share"` can be detected in JavaScript:

```
const testButton = document.createElement('button');
testButton.setAttribut('type', 'share);
if (testButton.type != 'share') {
  // Polyfill needed!
}
```

[Here's a polyfill](https://gist.github.com/adactio/092b11a74eded2701335ba27f94d2484) that provides three levels of support:

1. Does this browser support `button type="share"`? If so, no extra code required.
2. Does this browser support the Web Share API? If so, pass in the current URL and page title.
3. If this browser supports neither, create a `mailto:` link with the current URL and page title.

If that polyfill is included via a `script` element, an author can use `button type="share"` today but their HTML will not be valid.

## Non-goals

A "share" type for the `button` element isn’t capable of providing all the functionality of the Web Share API. For anything more complex than sharing the URL and title of the current page, it will be necessary to write JavaScript.

The contents of the `url` and `text` fields passed to the Web Share API aren’t customisable with `button type="share"`. Rather than try to shoehorn in the ability for the author to over-ride these fields, it is simpler to defer to the JavaScript API.

Adding an additional value to the `type` attribute of the `button` element opens the door to other declarative options for functionality that currently requires JavaScript. For example, `button type="copy"` might be used to the invoke the Clipboard API. But use cases like this are beyond the scope of this proposal.

## Security considerations

The proposed functionality for `button type="share"` doesn’t extend what’s already possible with the Web Share API (in fact it provides *less* functionality). Any security concerns about the existing Web Share API may apply but no new security concerns are introduced.

Because the contents of the `button` element aren’t set by the browser, authors can trick users into beginning a share interaction:

```
<button type="share">Free beer!</button>
```

This is already possible with the existing Web Share API.

There is a potential privacy concern with the current Web Share API in that it uses a promise to report back to the author whether or not the sharing action was successful. But if the user invokes browser-native share functionality (in the browser interface), the author cannot detect that the share happened. From a privacy perspective, `button type="share"` behaves more like the browser UI than the JavaScript API.

## Considered alternatives

### Links with a URI scheme

[The explainer document for the Web Share API](https://github.com/w3c/web-share/blob/master/docs/explainer.md#why-not-make-a-share-uri-scheme-like-mailto-instead-of-a-javascript-api) discusses the possibility of using a URI scheme (similar to `mailto:`) to provide a declarative means of sharing.

```
<a href="share:?title=Example%20Page&amp;url=https://example.com/page">Share this</a>
```

But the concerns are:

> * Sharing the page's own URL (a very common case) would require some scripting to inject the page URL into the href attribute.
> * There is no API to determine whether share is supported on the user's system.
> * There is no promise or callback to signal the success or failure of the share action.

Using `button type="share"` address the first two concerns. The third item (about the lack of a callback) may actually be undesirable given the section on security considerations above.

## References & acknowledgements

Thanks to Ada Rose Cannon from Samsung Internet for [prompting ideas for a declarative way of sharing](https://twitter.com/Lady_Ada_King/status/1296844595414933508).