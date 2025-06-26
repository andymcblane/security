# Drop Into Macca's
[3rdSense](https://3rdsense.com/work/drop-into-maccas) was contracted by McDonald's Australia to develop a mobile application, with rewards including free food and EFTPOS gift cards. I was starting to get into web and API development, and in combination with [Charles Proxy](https://www.charlesproxy.com/), decided to look closer at how the iPhone application communicates with the back-end in order to generate an "attempt" in the game, and hence an attempt at winning the free prizes.

Given this was way back in 2015, I'm not sure how common SSL pinning was back then, regardless it was not implemented and as such, I was able to sniff traffic successfully. This would have been a complete show stopper had it been implemented.

The iPhone app was a simple down-scroller game, where you would progress through obstacles and acheieve a distance-based score. Once you crashed or otherwise reached the end of an attempt, the iPhone app would generate an attmept ID, and hit the backend URL to determine if a prize can be issued for this play.

The payload for the play attempt included a unique ID (the attempt ID), and the message was signed and this hash included in the message.

I intiallly attempted a simple replay attack, however found that the backend threw and error and complained that the attempt ID had already been submitted. No luck there.

I spent a lot of time looking into how the message signature worked, and whether I could, somehow, expose the key they were using to do the signing, it would mean I could sign my own messages with whatever attempt ID I wanted, sadly this was also unsuccessful. (I recently decoded [an android APK](https://gist.github.com/andymcblane/3ce396e5af219c1d96208e5c917ce2af) in an attmept to connect my pet feeder to Home Assistant, sadly this was unsuccessful as well, but I suspect the same appraoch here would have worked.) It would have been interesting to explore what mitigations 3rdSense were relying on to handle fake-signed messages. (We'll find out later how they mitigated what I did.)

I next moved onto unhappy path cases, like poor network connectivity or dropped requests or responses.

When the iPhone would make a request, it had a sensible timeout of around 30 seconds, and if it did not receive a response, a popup was presented to the user on the mobile app to "resend" the request. This actually generated a new attempt ID, and since the message was correctly signed, the backend would run it's RNG and potentially issue a second prize for the same attempt!

The reader should already be aware that this communication protocol can be exploited... (mitm by dropping the response) - but it gets worse..

I wanted to see if another vulnerability could be found that would allow me to generate attempts more quickly than the default HTTP timeout configured in the client. I tried changing the response payload to nonsense to see how the iPhone app behaved. Sadly a malformed response was considered invalid by the iPhone app and would force the user to play again before re-generating a new attempt.

I tried a malformed request (via MITM), to see what the backend would do, and it responded with a _signed_ error message back.

Next, instead of replacing the server response with a malformed reponse, I replaced it with the legit error message signed by the server. **To my surprise**, the iPhone app recognised it as a valid error message and happily re-generated a new attempt.

In fact, it turns out the iPhone app only validates the error message hash as being valid, **not even checking if it's for the same message** and also not ensuring each message hash is only once and only once (it was time based).

The vulnerabilities I identified are as follows:

* The iPhone app would generate a new attempt ID as part of the error handling logic flow
* The iPhone app did not ensure error messages were only used once, allowing additional attempts to be generated nearly instantly.
    * Additionally it did not check the message hash was for the specific message, resulting in potentially processing of unsafe payloads (would have been fun to explore more)
* The backend had no rate limiting, and would issue real-money prizes without human intervention

Additionally no SSL pinning left it vulnerable to a MITM attack in the first place.

They ended up revoking all prizes I won, and sent me a lovely letter (on a McDonald's letter head, sadly I do not still have this) stating to _please stop_ or they would need to contact AFP regarding this.

I was slightly concerned about that situation occuring, though as I was only 17 at the time, I hoped if anything, I would go down the "gifted" route in cybersecurity.