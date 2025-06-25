# Drop Into Macca's
[3rdSense](https://3rdsense.com/work/drop-into-maccas) was contracted by McDonald's Australia to develop a mobile application, with rewards including free food and EFTPOS gift cards. I was starting to get into web and API development, and in combination with [Charles Proxy](https://www.charlesproxy.com/), decided to look closer at how the iPhone application communicates with the back-end in order to generate an "attempt" in the game, and hence an attempt at winning the free prizes.

Given this was way back in 2015, I'm not sure how common SSL pinning was back then, regardless it was not implemented and as such, I was able to sniff traffic successfully. This would have been a complete show stopper had it been implemented.

The iPhone app was a simple down-scroller game, where you would progress through obstacles and acheieve a distance-based score. Once you crashed or otherwise reached the end of an attempt, the iPhone app would generate an attmept ID, and hit the backend URL to determine if a prize can be issued for this play.

The payload for the play attempt included a unique ID (the attempt ID), and the message was signed and this hash included in the message.

I intiallly attempted a simple replay attack, however found that the backend threw and error and complained that the attempt ID had already been submitted. No luck there.

I spent a lot of time looking into how the message signature worked, and whether I could, somehow, expose the key they were using to do the signing, it would mean I could sign my own messages with whatever attempt ID I wanted, sadly this was also unsuccessful. (I recently decoded [an android APK](https://gist.github.com/andymcblane/3ce396e5af219c1d96208e5c917ce2af) in an attmept to connect my pet feeder to Home Assistant, sadly this was unsuccessful as well, but I suspect the same appraoch here would have worked.) It would have been interested to explore what mitigations 3rdSense were relying on to mitigate fake-signed messages.

I next moved onto unhappy path cases, like poor network connectivity or dropped requests or responses.

When the iPhone would make a request, it had a sensible timeout of around 30 seconds, and if it did not receive a response, a popup was presented to the user on the mobile app to "resend" the request. This actually generated a new attempt ID, and since the message was correctly signed, the backend would run it's RNG and potentially issue a second prize for the same attempt!

The reader should already be aware that this communication protocol can be exploited... but it gets worse..

TBC