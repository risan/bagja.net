---
title: Express Responds with Base64 Encoded Image
date: 2018-12-04 10:16:00
tags: [javascript, express]
excerpt: Using Express to send response with base64 encoded image.
image_thumb: ../img/express-logo-thumb.png
image:
  path: ../img/express-logo-og.png
  width: 1200
  height: 800
---
I want to create an endpoint that will respond with a 1x1 pixel PNG file. Instead of using the `sendFile` method, I choose to use the base64 encoded representation since the file is size super small. This way I can avoid accessing the file system.

Here's the gist:

```js
const express = require("express");

const app = express();

app.get("/test.png", (req, res) => {
  // A 1x1 pixel red colored PNG file.
  const img = Buffer.from("iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8z8DwHwAFBQIAX8jx0gAAAABJRU5ErkJggg==", "base64");

  res.send(img);
});
```

Note that when you pass a `Buffer`, the `content-type` header will be set to `application/octet-stream`.
