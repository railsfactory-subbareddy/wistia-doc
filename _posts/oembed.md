---
title: Wistia &amp; oEmbed
layout: post
category: For Developers
post_intro: <p>oEmbed is a simple API to get information about embedded content on a URL.</p><p>For Wistia, it's a great way to programmatically get embed codes for videos if you have their URLs.</p>
description: oEmbed is a simple API to get information about embedded content on a URL. For Wistia, it's a great way to programmatically get embed codes for videos if you have their URLs.
footer: 'for_developers'
---

## The Endpoint

Our oEmbed endpoint is: `http://fast.wistia.com/oembed`

Currently, our oEmbed endpoint recognizes two URL formats:

Type | Example URL
-----|-------------
iframe embed code URLs | http://fast.wistia.com/embed/iframe/b0767e8ebb?version=v1&controlsVisibleOnLoad=true&playerColor=aae3d8
iframed playlist URLs | http://fast.wistia.com/embed/playlists/fbe3880a4e?theme=trime&version=v1&videoOptions%5BvideoHeight%5D=360&videoOptions%5BvideoWidth%5D=640
Public media URLs | http://home.wistia.com/medias/e4a27b971d

It's likely we'll add more URLs to this list in the future.

---

## The Regex

If you're looking to automatically detect Wistia URLs and run them against our endpoint, we recommend using this regular expression:

<pre><code class='language-vim'>
/https?:\/\/(.+)?(wistia\.com|wi\.st)\/(medias|embed)\/.*/
</code></pre>


Or if you don't speak regex, here's what we're matching:

<pre><code class="language-vim">
http(s)://*wistia.com/medias/*
http(s)://*wistia.com/embed/*
http(s)://*wi.st/medias/*
http(s)://*wi.st/embed/*
</code></pre>

Note, it's likely we'll add support for more URLs in the future so feel free to use a more general regular expression so you don't miss out on future enhancements! Perhaps this:

<pre><code class="language-vim">
/https?:\/\/(.+)?(wistia\.com|wi\.st)\/.*/
</code></pre>

---

## An Example

Get the embed code and some information for a video at ''http://home.wistia.com/medias/e4a27b971d'' in JSON format:

<pre><code class="language-vim">
curl "http://fast.wistia.com/oembed?url=http://home.wistia.com/medias/e4a27b971d"
</code></pre>

This returns:

<pre><code class="language-markup">
{
  "version":"1.0",
  "type":"video",
  "html":"&lt;iframe src=\"http://fast.wistia.com/embed/iframe/e4a27b971d?version=v1&videoHeight=360&videoWidth=640\" allowtransparency=\"true\" frameborder=\"0\" scrolling=\"no\" class=\"wistia_embed\" name=\"wistia_embed\" width=\"640\" height=\"360\"&gt;&lt;/iframe&gt;",
  "width":640,
  "height":360,
  "provider_name":"Wistia, Inc.",
  "provider_url":"http://wistia.com",
  "title":"Brendan - Make It Clap",
  "thumbnail_url":"http://embed.wistia.com/deliveries/2d2c14e15face1e0cc7aac98ebd5b6f040b950b5.jpg?image_crop_resized=100x60",
  "thumbnail_width":100,
  "thumbnail_height":60
}
</code></pre>

If you're looking for XML instead of JSON, use: `http://fast.wistia.com/oembed.xml`

For all the fine details about the options supported, see the official [oEmbed spec](http://oembed.com).

---

## Parameters

Our endpoint supports all the options detailed at oembed.com.

The required url parameter that's passed in supports all the options detailed in the [Player API]({{ '/player-api' | post_url }}).

We also accept some additional parameters that can change the output of the embed code:

Name | Type  | Description
-----|-------|------------
callback | string | Only applicable to JSON requests. When specified, JSON is wrapped in a javascript function given by the callback param. This is to facilitate JSONP requests.
embedType | string | Only applicable to videos and playlists. Accepts "iframe", "api", "seo", "popover", "playlist_iframe", and "playlist_api".
width | integer | The requested width of the video embed. Defaults to the native size of the video or 360, whichever is smaller.
height | integer | The requested height of the video embed. Defaults to the native size of the video or 640, whichever is smaller.
handle | string | Only applicable to "api", "seo", and "playlist_api" embed types. Sets the javascript handle. Default is "wistiaEmbed" for medias and "wistiaPlaylist" for playlists.
popoverHeight | integer | Only applicable to "popover" embed type. The requested height of the popover. Defaults to maintain the correct aspect ratio, with respect to the width.
popoverWidth | integer | Only applicable to "popover" embed type. The requested width of the popover. Defaults to 150.
ssl | boolean | Determines whether the embed code should use https. Defaults to false.

If given a `width`, `height`, `maxwidth`, or `maxheight` parameter (or any combination of those),
the other dimensions in the resulting embed code may change so that the video's 
aspect ratio is preserved.

{{ "These parameters are attached to the Wistia media URL, and not the oEmbed call. So they must be URL-encoded to travel with the Wistia URL." | note }}

### Example

In this example, we'll request an `API` embed code type, with the javascript handle `oEmbedVideo`:

First, the media URL we'll request:

    http://home.wistia.com/medias/e4a27b971d

Next, we'll add the parameters for our request:

    http://home.wistia.com/medias/e4a27b971d?embedType=api&handle=oEmbedVideo

We'll URL-encode this request:

    http%3A//home.wistia.com/medias/e4a27b971d%3FembedType%3Dapi%26handle%3DoEmbedVideo

And now use the oEmbed endpoint:

<pre><code class="language-vim">
curl "http://fast.wistia.com/oembed?url=http://fast.wistia.com/oembed.json?url=http%3A//home.wistia.com/medias/e4a27b971d%3FembedType%3Dapi%26handle%3DoEmbedVideo"
</code></pre>

This returns:

<pre><code class="language-markup">
{
  "version":"1.0",
  "type":"video",
  "html":"<div id=\"wistia_e4a27b971d\" class=\"wistia_embed\" style=\"width:640px;height:360px;\" data-video-width=\"640\" data-video-height=\"360\">&nbsp;</div>\n<script charset=\"ISO-8859-1\" src=\"http://fast.wistia.com/static/concat/E-v1.js\"></script>\n<script>\noEmbedVideo = Wistia.embed(\"e4a27b971d\", {\n  version: \"v1\",\n  videoWidth: 640,\n  videoHeight: 360\n});\n</script>",
  "width":640,
  "height":360,
  "provider_name":"Wistia, Inc.",
  "provider_url":"http://wistia.com",
  "title":"Brendan - Make It Clap",
  "thumbnail_url":"http://embed.wistia.com/deliveries/2d2c14e15face1e0cc7aac98ebd5b6f040b950b5.jpg?image_crop_resized=640x360",
  "thumbnail_width":640,
  "thumbnail_height":360,
  "duration":16.43
}
</code></pre>

---

## Working With The Thumbnail

Part of the JSON returned by the oEmbed is the `thumbnail_url`. This URL is a direct link to the thumbnail image asset. If your implementation involves using the thumbnail image (i.e. building your own 'popover' embeds, displaying your own play button, etc.) you should use this thumbnail image, which by default has no Wistia play button overlaid on it.

See our [working with Wistia images]({{ '/working-with-images' | post_url }}) guide for more info!

---

## Troubleshooting

  1. If an invalid URL (one that doesn't match our regular expression above) is given, the endpoint will return <span class="code">404 Not Found</span>.
  2. If an unparseable URL is given in the url param, the endpoint will return <span class="code">404 Not Found</span>.
  3. If a media is found but has no available embed code, the endpoint will return <span class="code">501 Not Implemented</span>. Video, Image, Audio, and Document files all currently implement oembeds.
  4. If a playlist is found but has no videos, the endpoint will return <span class="code">501 Not Implemented</span>.

---

## Make Your Life Easier

If you're contemplating doing an oEmbed implementation with Wistia (or any oEmbed provider for that matter), we strongly recommend checking out [Embedly](http://embed.ly). By integrating with them you'll have immediate access to over 100 oEmbed providers. They also have great documentation and ready-made libraries for every popular language, plus they're just nice guys!
