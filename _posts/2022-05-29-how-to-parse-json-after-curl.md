---
layout: post
title: "How to Parse JSON After cURL"
date: 2022-05-29 04:45:16 +0400
tags: api curl json
---

If you use cURL for requests to an API with JSON response in the body, you know that feeling of frustration at the sight of solid text in the console.

![Terminal](/assets/2022-05-29/01-terminal.png)

_Fig. 1. Terminal_

I am using [cURL](https://curl.se/) as a tool for testing REST API — **making HTTP/HTTPS requests**. In most cases, the response body contains [JSON](https://www.json.org/). I assume that the reader knows what cURL ([wiki](https://en.wikipedia.org/wiki/CURL)) is for and is familiar with JSON ([wiki](https://en.wikipedia.org/wiki/JSON)) format.

All the methods described below will work if the cURL output contains **only valid** JSON ⇒ do not use `-i` option ([show the HTTP response headers](https://curl.se/docs/manpage.html#-i)), which will add additional data to the output except the response’s body like this:

![Response headers are included in the cURL output](/assets/2022-05-29/02-curl-output.png)

_Fig. 2. Response headers are included in the cURL output_

Otherwise, you will not be able to parse the response by any tool:

![Invalid JSON is not parseable](/assets/2022-05-29/03-invaid-json.png)

_Fig. 3. Invalid JSON is not parseable_

For all examples, I used macOS Big Sur and its default Terminal; all additional tools can be installed through [homebrew](https://brew.sh/).

## Python

If you do not want to install any additional tool — Python is your choice, of course, [if your OS comes with it](https://docs.python.org/3/using/unix.html). It has a built-in [JSON encoder and decoder](https://docs.python.org/3/library/json.html), which can be called from the command line [through the pipe](https://www.geeksforgeeks.org/piping-in-unix-or-linux/):

```
curl 'https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}' | python -m json.tool
```

The response will be in human-readable form:

![python -m json.tool](/assets/2022-05-29/04-python-m-json-tool.png)

_Fig. 4. \| python -m json.tool_

**Additional reading:**

- [How can I pretty-print JSON in a shell script](https://stackoverflow.com/a/1920585)
- [Parsing JSON files using Python](https://medium.com/analytics-vidhya/parsing-json-files-using-python-f8fb04172ce7)

## jq

[Jq](https://stedolan.github.io/jq/) is a flexible command-line JSON processor and the most popular solution:

```
curl 'https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}' | jq
```

The response will look nice and with syntax highlighting:

![jq](/assets/2022-05-29/05-jq.png)

_Fig. 5. \| jq_

The advantage of the tool is [an extended filtration system](https://stedolan.github.io/jq/manual/#Basicfilters). For example, you can immediately get the value of the required key:

```
curl 'https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}' | jq .timezone
```

![jq .json_key](/assets/2022-05-29/06-jq-json-key.png)

_Fig. 6. \| jq .json_key_

**Additional reading:**

- [Parse JSON data using jq and curl from command line](https://medium.com/how-tos-for-coders/https-medium-com-how-tos-for-coders-parse-json-data-using-jq-and-curl-from-command-line-5aa8a05cd79b)
- [Working with JSON using jq](https://sher-chowdhury.medium.com/working-with-json-using-jq-ce06bae5545a)
- [Parsing JSON with jq](http://www.compciv.org/recipes/cli/jq-for-parsing-json/)
- [jq Manual](https://stedolan.github.io/jq/manual/)

## fx

[Fx](https://github.com/antonmedv/fx) is a command-line JSON viewer and manipulation tool — after getting the JSON, you can navigate through it:

![fx navigation through JSON](/assets/2022-05-29/07-fx-navigation-through-json.gif)

_Fig. 7. fx navigation through JSON_

The highlighted response can be obtained without interactive mode by `.` option:

```
curl 'https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}' | fx .
```

![fx .](/assets/2022-05-29/08-fx.png)

_Fig. 8. \| fx ._

Despite that fx is less popular than jq, it also supports filters:

```
curl 'https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}' | fx .current.humidity
```

![fx .json_key](/assets/2022-05-29/09-fx-json-key.png)

_Fig. 9. \| fx .json_key_

**Additional reading:**

- [Discover how to use fx effectively, a JSON manipulation command line tool](https://medium.com/@antonmedv/discover-how-to-use-fx-effectively-668845d2a4ea)
- [Getting started with FX: Powerful and handy JSON manipulation from the command line](https://dev.to/sanexperts/getting-started-with-fx-powerful-and-handy-json-manipulation-from-the-command-line-362f)

## jless

[Jless](https://jless.io/) is the newest command-line JSON viewer with [a bunch of vim-inspired commands](https://jless.io/user-guide.html) (like `:q` to exit.) for navigation:

```
curl 'https://api.openweathermap.org/data/2.5/onecall?lat=40.1811&lon=44.5136&appid={api_key}' | jless
```

After request, you will immediately get into the beautiful interactive mode, where you can move and search through JSON:

![jless](/assets/2022-05-29/10-jless.png)

_Fig. 10. \| jless_

Jless is also a worthy tool to work with local JSON files.

**Additional reading:**

- [Jless, a command-line JSON viewer](https://pauljuliusmartinez.github.io/)

---

Earlier this year the maintainer of cURL [announced the beginning of implementation of JSON support](https://curl.se/mail/archive-2022-01/0043.html). I hope in the near future we could work with JSON output out of the cURL’s box.

**Additional reading:**

- [How to start using Curl and why](https://medium.com/free-code-camp/how-to-start-using-curl-and-why-a-hands-on-introduction-ea1c913caaaa)
- [An Introduction to JSON](https://www.digitalocean.com/community/tutorials/an-introduction-to-json)
- [Parsing JSON with Unix tools](https://stackoverflow.com/a/1955555)

Copy @ [Medium](https://adequatica.medium.com/how-to-parse-json-after-curl-71e9413daa0c)
