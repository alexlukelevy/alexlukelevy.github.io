---
layout: post
title:  "Ticker Tape: Raspberry Pi meets Bloomberg"
date:   2016-11-13 15:26:00 +0000
categories: experiments
comments: true
---

**tl;dr** - Introducing [ticker-tape](https://github.com/alexlukelevy/ticker-tape), a Python library for Raspberry Pi that prints scrolling news to an LED matrix.

![Cover](/img/ticker-tape-cover.jpg)

### Concept
Last Christmas my friends were kind enough to buy me a Raspberry Pi B+. Eager to get tinkering, I starting thinking of interesting projects right away. I was hoping to think of something that would give me some direct exposure to the hardware of the Pi but would also make my life marginally better in some way.

Staring at a stock price ticker scrolling by at work, I thought 'wouldn't it be cool if I could make my own one of these and print whatever I wanted on there?'. This seemed like it would incorporate a good mix of hardware and software, so I ran with the idea and started researching.

### Setup
First of all I had to get my hands on a display. Luckily I was not the only one in the world trying to control an LED matrix from a Raspberry Pi, so I headed over to [Adafruit](https://www.adafruit.com/) and purchased the following items:

* [16 x 32 LED matrix](https://www.adafruit.com/products/420)
* [Jumper wires](https://www.adafruit.com/products/266)
* [Power supply](https://www.adafruit.com/products/276)
* [Power adapter](https://www.adafruit.com/products/368)

Now that I had all the hardware I needed, the fiddly stage of wiring everything together was up next. Given my admittedly weak electronics knowledge I wasn't sure how much further I was going to get, so like all good software engineers I turned to Google for help. Fortunately I stumbled across an excellent [Raspberry Pi](https://github.com/hzeller/rpi-rgb-led-matrix) library from hzeller. hzeller has created a C++ library that allows you to print all sorts of things to an RGB LED matrix and comes with selection of bindings for different languages.

Following the excellent wiring guide [here](https://github.com/hzeller/rpi-rgb-led-matrix/blob/master/wiring.md), I managed to successfully connect my matrix and run the basic demo.

![Wired](/img/ticker-tape-wiring.jpg)

### Ticker Tape
With the perils of physical labour out of the way, I could concentrate on writing the software. I decided on Python as my language of choice as it seemed like a good fit for the problem and I was keen to improve my skills.

I drew up a high-level design for the software components which would allow for multiple news sources, aggregation of the news items and printing news to the tape. The basic components were:

* `FeedHandler`: consumes news events from a given source (e.g. BBC) and publishes `FeedEvents` to the `Reporter`
* `Reporter`: responsible for keeping track of all the latest `FeedEvents` from various sources and reporting the events to the `Tape` when requested
* `Tape`:  represents the LED matrix and contains the logic to convert a string to an LED output
* `Director`: oversees the whole operation and coordinates the threads of the `FeedHandlers` and the `Reporter`

Being a big TDD fan, I jumped straight in with some `nosetests` and started building a BBC news `FeedHandler`. The BBC publish an [RSS feed](http://feeds.bbci.co.uk/news/rss.xml) of their latest news stories and with the help of the Python [feedparser](https://pypi.python.org/pypi/feedparser) library, I was able to pull them out with relative ease.

Once I had the BBC `FeedEvents` flowing through nicely to the `Reporter`, I began to write the logic to print the text to the matrix. Leveraging off the Python examples provided by hzeller, I selected a suitable font and wrote the algorithm to gradually scroll the text across the tape.

Plumbing this altogether in `tickertape.py`, I added a basic CLI that would allow you to run the ticker tape for a given amount of time, refreshing periodically.

```sh
$ sudo python tickertape.py --runtime 20 --refresh 5
```

This command would allow all registered `FeedHandlers` to refresh and publish new events every 5 minutes, whilst the `Reporter` would continously report on the latest events for 20 minutes.

Finally, adding some `cron` to the mix and I'm now able to wake up to the blinking green headlines of Donald Trump's presidential campaign. So I'll let you decide whether this has improved my life...

![Demo](/img/ticker-tape-demo.gif){: .center-image }

### Further work
Like all software projects, the code is never really finished. Below are a few of the bits I'd like to add in the coming months:

* Add a [metacritic](http://www.metacritic.com/) `FeedHandler`
* Add an online banking `FeedHandler`
* Hook up another matrix to increase the tape length
* Create a `FeedConfiguration` class to allow you to finely tune the display options of a `FeedEvent`

### Contributing
[ticker-tape](https://github.com/alexlukelevy/ticker-tape) is open-source library, so if you wish to make use of or extend the library please do so on GitHub!
