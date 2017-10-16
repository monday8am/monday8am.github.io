# Greenfoot platform based games #

JULY 9, 2012

After the publication of the roadmap for the Adobe Flash platform everyone is aware that the platform and therefore developers will have two (and only two) addresses defined: games and streaming video. For this reason in recent months have come to light multiple frameworks for game development AS3: Starling, PushButton, FlashPunk, Flixel among others. But these frameworks are relatively new and few code examples.

What if we could use an existing Framework game with a community of developers and convert it to AS3 stable? This is the case of Greenfoot, a platform made in Java for creating simple games with thousands of examples and a community that exists since 2006.

The task would be to make a translation Greenfoot libraries to AS3, and then translating Java code from the examples, make games for it. It seems a big job, but who does not like to have the codebase hand the following games:

- Pacman
- Bubble Shooter
- Ping Pong

Given the similarity of the language, it will be much more profitable to do the translation code from scratch. As an example we can put the translation of the classic Bubble Shooter:



After translating my *own* and *unreliable* [version](https://github.com/monday8am/BubbleShooter_FLA/tree/initial_dev/src/com/monday8am/greenfoot) of the framework, we use this code as a base for a game portal for a major food firm in a commercial project. And the result was as follows: