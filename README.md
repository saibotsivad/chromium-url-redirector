# chromium-url-redirector

Replicate the Firefox keyword bookmark functionality by going through an API. Sad.

This is a server-side script that you install in Cloudflare as a Worker.

## Replicated Browser Functionality

It sounds like many people not familiar with Firefox don't even know what this functionality is.

Inside most chromium-based browsers, suppose you want to go to a commonly visited website that starts with `f`.

You might click on a bookmark or use some external tool to get there.

If you opened a new tab, typed `f`, and hit enter... it wouldn't reliably take you to the same URL every time. If you wanted to go to `fastmail.com` and later visited `facebook.com` that's alphabetically earlier, so `f --> enter` would no longer go to Fastmail, it'd take you to Facebook.

But inside Firefox, if you set a keyword on a bookmark, typing that exact keyword will open that bookmarked URL.

The closest thing chromium browsers have are "search engines", which is a partial implementation of the superior Firefox bookmark: you _must_ include a `%s` in the URL (not required in Firefox) and when using it you _must_ type a space and at least one additional character.

So at best in chromium you could create a "search engine" to `fastmail.com#%s` with the keyword `f`, then type `f x` (or something unimportant) and it would take you to `fastmail.com#x` which is very much less than ideal.

## Solutions?

There was a browser extension, but it's not developed, and this is apparently niche functionality, so I don't expect chromium browsers to care much.

What I think is that I can install a "search engine" and give it a keyword like `b` for `bookmark`, then whatever you type after a space will translate into `my.api.com?redirect=%s`.

So to open my bookmark with an `f` keyword I would do: _new tab_, `b f`, `enter`, this would call my API, which would redirect to whatever I had mapped `f` to be.

Because it's cheap, I'm building this in Cloudflare Workers to store redirect maps of e.g. `f` to `https://fontawesome.com/` or whatever.

It's a manual setup, so this README is the documentation to set it up.

## Implementation

First go to the Cloudflare "Workers & Pages" page and click "Create".

There should be some sort of "Hello World" you can click to set up.

It'll suggest a Worker name, but...

- You might want to rename so you know what it's for, but since this domain will never be manually typed it can be very long.
- Make note of what it says the URL will be.

Go ahead and deploy the "Hello World" code, so you can make sure it's working correctly and serving responses.

After it's done, go visit the URL to make sure it's deployed and responding correctly.

There'll be a link on the page somewhere that takes you to a web-based code editor. Click through it until you can edit, and then paste in this:

```js
const MAP = {
	f: 'https://fontawesome.com/',
}
export default {
	async fetch(request) {
		const redirect = MAP[new URL(request.url)?.searchParams?.get?.('redirect')]
		return redirect ? new Response(null, { status: 307, headers: { Location: redirect } }) : new Response('Not Found')
	},
}
```
