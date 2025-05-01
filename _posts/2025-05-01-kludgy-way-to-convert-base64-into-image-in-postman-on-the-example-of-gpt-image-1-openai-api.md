---
layout: post
title: 'Kludgy Way to Convert Base64 into Image in Postman on the Example of gpt-image-1 OpenAI API'
date: 2025-05-01 09:40:03 +0200
tags: postman
---

If you need to save Base64 code as an image on the local drive through Postman, then there is no simple way.

The initial state of the problem:

- You use Postman UI;
- You have a [base64](https://en.wikipedia.org/wiki/Base64)-encoded image in the body response;
- How to save this image as a file (JPG or PNG) on a local drive? [Click here to scroll down to the solution immediately]().

There are a few questions in Postman’s community forum on the topic of how to save a base64-encoded code as a file to the local drive in Postman, but none of them have answers.

In my case, I faced this problem when I switched from `dall-e-3` to `gpt-image-1` model when [creating images through the OpenAI API](https://platform.openai.com/docs/api-reference/images/create).

The previous image generation model ([`dall-e-3`](https://platform.openai.com/docs/models/dall-e-3)) responded with a URL of the generated image, which was valid for 60 minutes, so that you could download the image file without any trouble.

```json
{
  "created": 1745934226,
  "data": [
    {
      "revised_prompt": "Generate an image of a… ",
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/… "
    }
  ]
}
```

New model ([`gpt-image-1`](https://platform.openai.com/docs/models/gpt-image-1)) responds with an image inside the response body. On the one hand, it is convenient to have the final result immediately in the response, but on the other hand, it requires a conversion step on the client side.

```json
{
  "created": 1745930246,
  "data": [
    {
      "b64_json": "oAIdaLleApIPR0DuctUSAAAAA… "
    }
  ],
  "usage": {
    "input_tokens": 28,
    "input_tokens_details": {
      "image_tokens": 0,
      "text_tokens": 28
    },
    "output_tokens": 1056,
    "total_tokens": 1084
  }
}
```

I prefer to perform as few actions as possible, especially when using UI tools (Postman) when handling an API. The ideal workaround is to get a generated image in one click, but there is even a list of restrictions that stop you from doing that:

- `gpt-image-1` API always returns only base64-encoded images (as mentioned above).
- I was hoping that it would be possible to convert Base64 code into an image file and save it on the local drive through a [post-response script](https://learning.postman.com/docs/tests-and-scripts/write-scripts/intro-to-scripts/). But Postman runs scripts in a sandboxed JavaScript environment and does not have direct file system access ⇒ it is not possible to import and use standard Node.js methods like [fs](https://nodejs.org/api/fs.html) in a script. Postman allows you to save only the response body through UI, by [Save response to file] button, but not by the post-script’s code.
- I tried [Postman Visualizer](https://learning.postman.com/docs/sending-requests/response-data/visualizer/), but it does not allow saving its visualization. Moreover, DevTools in the window, which opens by [Inspect visualization] button in the Visualization screen, has disabled the left-click context menu (maybe it is a bug).

But I discovered that the image from Postman’s Visualization screen can be dragged & dropped! Thereby, you can «visualize» Base64 into the image and move it into a folder on the local drive.

### Steps to save a base64-encoded image created by gpt-image-1 API through Postman:

1. Create Postman’s request with post-response script:

![Postman Post-response Scripts tab](/assets/2025-05-01/01-post-response-script.png)

_Fig. 1. Postman Post-response Scripts tab_

```javascript
function visualizerPayload() {
  const jsonData = pm.response.json();

  return { response: jsonData };
}

// {{response.<path>}} is your base64 image in your response body
const visualizerTemplate = `
<img src="data:image/png;base64,{{response.data.[0].b64_json}}" />
`;

pm.visualizer.set(visualizerTemplate, visualizerPayload());
```

2. Send the request. Example of `gpt-image-1` OpenAI API request:

```javascript
curl --location 'https://api.openai.com/v1/images/generations' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer •••••' \
--data '{
    "model": "gpt-image-1",
    "prompt": "Generate an image of a… ",
    "n": 1,
    "size": "1024x1024",
    "quality": "low"
}'
```

3. Choose [Visualisation] on the response’s Body tab:

![Postman Visualisation](/assets/2025-05-01/02-visualization.png)

_Fig. 2. Postman Visualisation_

4. Drag and drop the image from Postman to a local folder:

![Drag & drop from Postman to folder](/assets/2025-05-01/03-drag-and-drop.png)

_Fig. 3.1. Drag & drop from Postman to folder_

![Drag & drop from Postman to folder](/assets/2025-05-01/03-drag-and-drop.gif)

_Fig. 3.2. Drag & drop from Postman to folder_

**Now you can view your GPT’s generated images just inside Postman.**

The benefit of this way is that you can also view all your previously generated images in Postman’s history:

![Postman History Visualization](/assets/2025-05-01/02-history.png)

_Fig. 4. Postman History Visualization_

This trick can be performed with any kind of base64-encoded images (just update the data type and path to the base-64 code in the template for visualizer).

Copy @ [Medium](https://adequatica.medium.com/kludgy-way-to-convert-base64-into-image-in-postman-on-the-example-of-gpt-image-1-openai-api-1b291eb40325)
