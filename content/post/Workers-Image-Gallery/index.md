---
title: Image gallery on Cloudflare Workers
date: 2025-05-11T00:00:00-05:00
draft: false
type: post
tags:
  - software
  - cloudflare-workers
description: |
  May 2025:
  Basic image viewer and uploader from Google Photos hosted on Cloudflare Workers
---

Earlier this month I adopted an adorable puppy, so I wanted an easy way to share
photos of him (tl;dr skip [here][Luca]) without relying on social media. From
working at Cloudflare I've been exposed to all the features available on our
developer platform, and this felt like a perfect opportunity to play around and
set something up (other than this blog). Note to any of my co-workers or anyone
familiar with Typescript, I don't know what I'm doing and I'm sorry 😀

## Image viewer

This was shockingly easy. These images will be stored in an [R2][R2] bucket on
the free plan, so we just need to have a site that will:

* Fetch all the images in the bucket
* Display them in a decent looking gallery

The first step can be done with a very basic Worker (it's almost identical to
the example code in the developer docs). We just need to provide a `/list`
endpoint which provide a json array of all the keys that we should fetch, and
then an endpoint `/image/<image-id>` to return the image content itself.

```
router.get("/list", async ({env, req, ctx}) => {
  var objList: string[] = []
  var listOpts: R2ListOptions = {}
  // pagination
  while (true) {
    const objects = await env.PHOTO_BUCKET.list(listOpts);
    for (const obj of objects.objects) {
      objList.push(obj.key)
    }
    if (objects.truncated) {
      listOpts.cursor = objects.cursor
    } else {
      break
    }
  }
  return new Response(JSON.stringify(objList))
})

router.get('/image/:image_id', async ({req, env}) => {
  const object = await env.PHOTO_BUCKET.get(req.params.image_id);
  if (object === null) {
    return new Response("ImageNot Found", { status: 404 });
  }
  return new Response(object.body, {
    headers,
  });
})
```

Now, the front-end site needs to query that `/list` endpoint to find out which
images to display, and the point the `src` of those images to the `/image`
endpoint. Using [Fancybox] to display the image gallery itself was dead simple,
and [Tailwind] can easily make this into a responsive grid.

```
<!-- init tailwind responsive classes -->
    <div class="flex flex-wrap justify-center">
      <div
        id="photo-viewer"
        class="grid grid-cols-3 lg:grid-cols-6 justify-items-center gap-2"
      ></div>
    </div>
<!-- Load images from backend -->
    <script>
    function getImages() {
      fetch("/list").then( (resp) => {
        return resp.json()
      }).then( (body) => {
        console.log(body)
        var outHTML = ""
        const photoViewer = document.getElementById("photo-viewer")
        for (const obj of body) {
          newImg.alt = obj
          newImg.src = `/cdn-cgi/image/fit=cover,width=150/image/${obj}`

          const newHref = document.createElement("href")
          newHref.setAttribute(
            "href",
            `/cdn-cgi/image/fit=cover/image/${obj}`
          )
          newHref.dataset.fancybox = "gallery"
          newHref.appendChild(newImg)

          photoViewer.appendChild(newHref)
        }
      })
    }
    getImages()
    </script>
<!-- make it a photo gallery classes -->
    <script src="fancybox-cdn-src"></script>
    <script>
      Fancybox.bind('[data-fancybox="gallery"]', {
        Thumbs: {type: "modern"},
        contentClick: "toggleCover",
        wheel : "slide",
        Toolbar: {display: {right: ["close"]}},
        Images: {initialSize: "fit"},
      });    
    </script>
```

Keen viewers might note the `/cdn-cgi/image` tag for the thumbnail src. With
[Cloudflare Images][Images] we can do a live transform of the image backed by R2
to a scaled down size. I've also added a transformation for the full-page image
to ensure EXIF data is [stripped][images-strip] (it should be in the bucket
anyway, but can't hurt to do it twice). That's free for the first 5,000 *unique*
transforms, meaning since we're doing two here we can host 2,500 images per
account without spending a dime. That coupled with the free 10 GB we can host on
R2 should be more than enough for a simple gallery viewer.

Both the backend and frontend can thankfully be served by a single worker. The
Worker routes to a [static asset][Assets] if one exists, and otherwise forwards
all requests to the Worker script running in the background.

I'm not exaggerating when I say this portion took an afternoon at most. You can
view the source code [here][gallery-code]. Don't want to read raw HTML? You can
view this site live [here][Luca].

## Okay great, but how do we upload images to R2?

If you were sane you would just do it in the Cloudflare dash directly, or using
any S3 compatible API. I unfortunately did this on hard mode, and wanted to
directly upload images from [Google Photos][GPhotos]. They thankfully do provide
an [API][GPhotos api docs], but I needed to:

* Create a new OAuth app client
* Instantiate a "Picker session" to allow users to select media to share
* Poll the "Picker session" until the user is finished
* Upload the selected media to R2

Easy right? I thought so, and wrote a single-threaded Worker script to do the
above. It worked okay at first, but I noticed that it would fail to upload
images if I took more than a few seconds selecting images.

Turns out that Workers will [stop executing][Duration] if the client disconnects
or a timeout of 30 seconds is reached. The latter case isn't supposed to usually
happen (and the tasks are supposed to be extended with [waitUntil][waitUntil])
but I was still having issues. Maybe my colleagues were bouncing connections on
metals in Dallas more frequently that normal, maybe my phone was disconnected
when I changed tabs, or (more likely) my code was failing in odd ways.
Regardless, I needed a more robust solution.

The recommended solutions are using different paid products ([Queues][Queues] or
[Tail Workers][Tail Workers]), but I'm cheap so I figured I could use our [key
value store][KV]. Now the flow is:

* User hits the main static page, creates a session
* backend worker with the OAuth client secrets:
    * returns the Google Photos picker URL
    * saves session information in KV
* front end page polls a `/check_status` endpoint periodically, which:
    * updates the status text visible to the user
    * reads the users' session from KV, figures out what it needs to do
    * Checks "Picker session" status, set media to fetch if done
    * Check media to fetch, take one off top and upload to R2
    * *Importantly* - only writes back to KV once the operation it's doing is
      complete

This still does have quite a few issues (we can only write to the same key once
per second, the free KV plan allows only [1,000 writes a day][kv-limits], etc),
but it's able to consistently fetch and retrieve media from Google Photos, so
I'm happy.

One other thing to note - user auth to this picker page. Apologies but it's
locked down to just me. [Cloudflare Access][Access] made that really easy. With
using Google as the identity source I'm able to gate access to the worker to
only my account. Access also signs a JWT which I'm able to verify to ensure it's
really mean doing these write operations.

If you're curious to read more the source code for the photo-uploader is
available [here][picker-code]. It's much more involved than the image gallery,
and took me several days instead of an afternoon. I'll still poke around at it
trying to improve some things trying to improve it, but it's in a stable-ish
state if anyone wants to do something similar. My recommendation though: 
use an S3-api to manage the photos.

[R2]: https://developers.cloudflare.com/r2/
[Fancybox]: https://fancyapps.com/fancybox/
[Tailwind]: https://tailwindcss.com/
[Images]: https://developers.cloudflare.com/images/
[Luca]: https://luca.josephvoss.com
[GPhotos]: https://photos.google.com
[GPhotos api docs]: https://developers.google.com/photos/picker/reference/rest
[Duration]: https://developers.cloudflare.com/workers/platform/limits/#duration
[Queues]: https://developers.cloudflare.com/queues/
[Tail Workers]: https://developers.cloudflare.com/workers/observability/logs/tail-workers/
[KV]: https://developers.cloudflare.com/kv/
[gallery-code]: https://github.com/josephvoss/photo-gallery-worker
[Assets]: https://developers.cloudflare.com/workers/static-assets/
[waitUntil]: https://developers.cloudflare.com/workers/runtime-apis/context/#waituntil
[kv-limits]: https://developers.cloudflare.com/kv/platform/limits/
[Access]: https://developers.cloudflare.com/cloudflare-one/identity/
[images-strip]: https://developers.cloudflare.com/images/transform-images/transform-via-workers/#metadata
[picker-code]: https://github.com/josephvoss/cf-images-google-photos
