---
author: Andrei
comments: false
date: 2017-05-22 15:03:29+00:00
layout: post
link: https://dlang.org/blog/2017/05/22/introspection-introspection-everywhere/
slug: introspection-introspection-everywhere
title: Introspection, Introspection Everywhere
wordpress_id: 779
categories:
- Core Team
- D Foundation
- DConf
permalink: /introspection-introspection-everywhere/
redirect_from: /2017/05/22/introspection-introspection-everywhere/
---

#### Prelude: Orem, UT, May 29 2015


Just finished delivering my keynote. Exiting the character, I'm half dead. People say it needs to look easy. Yeah, just get up there and start saying things. Like it's natural. Spontaneous. Not for me it's not. Weeks before any public talk, I can only think of how to handle it. What to say. What angles come up. The questions. The little jokes made in real time. Being consistently spontaneous requires so much rehearsal. The irony.

So all I want now is sneak into my hotel room. Replenish the inner introvert after the ultimate act of extroversion. Lay on the bed thinking "What the heck was that?" for the rest of the day. Bit of slalom to get to the door. Almost there. In the hallway, an animated gentleman talks to a few others. He sports a shaved head and a pair of eyebrows that just won't quit. Stands just by the door, notices me in the corner of his eye, and it's instantly clear to both of us he's waiting for me. Still, he delicately gives me the chance to pretend I didn't notice and walk around him. "Hi, I'm Andrei." "Liran, co-founder and CTO of [Weka.IO](http://weka.io). I'm leaving a bit early and before that I wanted to tell you—you should come visit us in Tel Aviv. We've been using D for a year now in a large system. I think you'll like what you see. I might also arrange you a Google tech talk and visits at a couple of universities. Think it over."


#### Tel Aviv, May 8 2017


Coming out of the hotel, heat hits like a punch. We're talking 41 Celsius (before you pull that calculator: 106 Fahrenheit), if you're lucky to be in the shade. Zohar, software engineer and factotum extraordinaire at Weka, is driving us on the busy streets of Tel Aviv to his employer's headquarters. A fascinating exotic place, so far away from my neck of the woods.

First, Liran gives me an overview of their system—a large-scale distributed storage based on flash memory. Not my specialty, so I'm desperately searching my mind for trick questions—Information Theory—wait! peeps a lonely neuron who hasn't fired since 1993—Reed-Solomon and friends, used to know about it. (Loved that class. The professor was quite a character. Wrote anticommunist samizdat poetry before it was cool. Or even legal. True story.) "How do you deal with data corruption when you have such a low write amplification?" "Glad you asked!" (It's actually me who's glad. Yay, I didn't ask a stupid question.) "We use a redundant encoding with error correction properties; you get to choose the trade-off between redundancy and failure tolerance. At any rate, blind data duplication is mathematically a gross thing to do." I ask a bunch more questions, and clearly these guys did their homework. The numbers look impressive, too—they beat specialized hardware at virtually all metrics. "What's the trick?" I finally ask. Liran smiles. "I get that all the time. There's not one trick. There's a thousand carefully motivated, principled things we do, from the high level math down to the machine code in the drivers. The trick is to do everything right."

We now need to hop to my first stop of the tour—Tel Aviv University. Liran accompanies me, partly to see his alma mater after twenty years. Small, intimate audience; it's the regular meeting of their programming languages research group. A few graduate students, a postdoc, and a couple of professors. The talk goes over smoothly. We get to spend five whole minutes on an oddball—what's the role of the two semicolons in here?

    mixin(op ~ "payload;");
It's tricky. mixin is a statement so it needs a terminator, hence the semicolon at the very end. In turn, mixin takes a string (the concatenation of variable op, which in this case happens to be either "++" or "--", and "payload;") and passes it to the compiler as a statement. Mix this string in the code, so to say. It follows that the string ultimately compiled is "++payload;" or "--payload;". The trick is the generated statement needs _its own_ semicolon. However, within an expression context, mixin is an expression so no more need for the additional semicolon:

```d
auto x = mixin(op ~ "payload"); // mixin is an expression here, no semicolon!
```
This seems to leave one researcher a bit unhappy, but I point out that all macro systems have their oddities. He agrees and the meeting ends on a cordial note.

The evening ends with dinner and beers with engineers at Weka. The place is called "Truck Deluxe" and it features an actual food truck parked inside the restaurant. We discuss a million things. I am so galvanized I can only sleep for four hours.


#### May 9 2017


Omg omg OMG. The alarm clock rings at 6:50 AM, and then again at 7 and 7:10, in ever more annoying tones. Just as I'd set it up anticipating the cunning ways of my consciousness to become alert just enough to carefully press the stop—careful, not the snooze—button, before slumbering again. Somewhat to my surprise I make it in time for meeting Liran to depart to Haifa. Technion University is next.

Small meeting again; we start with a handful of folks, but word goes on the grapevine and a few more join during the act. Nice! I pity Liran—he knows the talk by heart by now, including my jokes. I try to make a couple new ones to entertain him a bit, too. It all goes over nicely, and I get great questions. One that keeps on coming is: how do you debug all that compile-time code? To which I quip: "Ever heard of printf-based debugging? Yeah, I thought so. That's pretty much what you get to do now, in the following form:"

    pragma(msg, string_expression);
The expression is evaluated and printed if and only if the compiler actually "goes through" that line, i.e. you can use it to tell which branch in a static if was taken. It is becoming clear, however, that to scale up Design by Introspection more tooling support would be needed.

Back at Weka, as soon as I get parked by a visitor desk, folks simply start showing up to talk with me, one by one or in small groups. Their simple phonetic names—Eyal, Orem, Tomer, Shachar, Maory, Or, ...—are a welcome cognitive offload for yours socially inept truly. They suggest improvements to the language, ask me about better ways to do this or that, and show me code that doesn't work the way it should.

In this faraway place it's only now, as soon as I see their code, that I feel at home. _They get it!_ These people don't use the D language like "whatevs". They don't even use it as "let's make this more interesting." They're using it as strategic advantage to beat hardware storage companies at their own game, whilst moving unfairly faster than their software storage competitors. The code is as I'd envisioned the D language would be used at its best for high leverage: introspection, introspection everywhere. Compile time everything that can be done at compile time. It's difficult to find ten lines of code without a static if in there—the magic fork in design space. I run wc -l and comment on the relatively compact code base. They nod approvingly. "We've added a bunch of features since last year, yet the code size has stayed within 5%." I, too, had noticed that highly introspective code has an odd way of feeding upon itself. Ever more behaviors flow through the same lines.

Their questions and comments are intent, focused, so much unlike the stereotypical sterile forum debate. They have no interest to show off, prove me wrong, or win a theoretical argument; all they need is to do good work. It is obvious to all involved that a better language would help them in the way better materials can help architects and builders. Some template-based idioms are awfully slower to compile than others, how can we develop some tooling to inform us about that? Ideas get bounced. Plans emerge. I put a smudgy finger on the screen: "Hmm, I wonder how that's done in other languages." The folks around me chuckle. "We couldn't have done that in any other language." I decide to take it as a compliment.

A few of us dine at a fancy restaurant (got to sample the local beer—delicious!), where technical discussions go on unabated. These folks are sharp, full of ideas, and intensely enthusiastic. They fully realize my visit there offers the opportunity to shed collective months of toil from their lives and to propel their work faster. I literally haven't had ten minutes with myself through the day. The only time I get to check email on my phone is literally when I lock myself into (pardon) a restroom stall. Tomorrow my plan is to sleep in, but there's so much going on, and so much more yet to come, that again I can only sleep a couple of hours.


#### May 10 2017


Today's the big day: the [Google Campus Tel Aviv talk](https://www.youtube.com/watch?v=es6U7WAlKpQ). We're looking at over 160 attendees this evening. But before that there's more talking to people at Weka. Attribute calculus comes on the table. So for example we have @safe, @trusted, and @system with a little algebra: @safe functions can call only @safe and @trusted functions. Any other function may call any function, and inference works on top of everything. That's how you encapsulate unsafe code in a large system—all nice and dandy. But how about letting users define their own attributes and calculi? For example, a "may switch fibers" attribute such that functions that yield cannot be called naively. Or a "has acquired a lock" attribute. From here to symbolic computation and execution cost estimation the road is short! To be sure, that would complicate the language. But looking at their code it's clear how it would be mightily helped by such stuff.

I got a lunchtime talk scheduled with all Weka employees in attendance, but I'm relaxed—by this point it's just a family reunion. Better yet, one in which I get to be the offbeat uncle. My only concern is novelty—everything I preach, these folks have lived for two years already. Shortly before the talk Liran comes to me and asks "Do you think you have a bit more advanced material for us?" Sorry mate, I'm at capacity over here—can't produce revolutionary material in the next five minutes. Fortunately the talk goes well in the sense that Design by Introspection formalizes and internalizes the many things they've already been doing. They appreciate that, and I get a warm reception and a great Q&A session.

As I get to the Google campus in Tel Aviv, late afternoon, the crowds start to gather. This is big! And I feel like I'd kill myself right now! I mentioned I take weeks to prepare a single public appearance, by the end of which I will definitely have burned a few neurons. And I'm fine with that—it's the way it's supposed to be. Problem is, this one week packs seven appearances. The hosts offer me coffee before, during, and after the talk. Coffee in Israel is delicious, but I'm getting past the point where caffeine may help.

Before the talk I get to chit chat incognito with a few attendees. One asks whether a celebrity will be speaking. "No, it's just me." He's nice enough to not display disappointment.

There's something in the air that tells you immediately whether an audience will be welcoming or not so much. Like a smell. And the scent in this room is fabulously festive. These folks are here to have a good time, and all I need to do to keep the party going is, in the words of John Lakos, "show up, babble, and drool." The talk goes over so well, they aren't bored even two hours later. Person in the front row seems continuously confused, asks a bunch of questions. Fortunately we get to an understanding before resorting to the dreaded "let's take this offline." Best questions come of course from soft-spoken dudes in the last row. As I poke fun at Mozilla's CheckedInt, I realize Rust is also Mozilla's and I fear malice by proxy will be alleged. Too late. (Late at night, I double checked. Mozilla's [CheckedInt](https://dxr.mozilla.org/mozilla-central/source/mfbt/CheckedInt.h) is just as bad as I remembered. They do a division to test for multiplication overflow. Come on, put a line of assembler in there! Portability is worth a price, just not any price.) My talk ends just in time for me to not die. I'm happy. If you're exhausted it means it went well.


#### May 11 2017


Again with the early wake-up, this time to catch a train to Beersheba. Zohar rides with me. The train is busy with folks from all walks of life. I trip over the barrel of a Galil. Its owner—a girl in uniform—exchanges smiles with me.

Two talks are on the roster today, college followed by Ben Gurion University. The first goes well except one detail. It's so hot and so many people in the room that the AC decides—hey, y'know, I don't care much for all that Design by Introspection. It's so hot, folks don't even bother to protest my increasingly controversial jokes about various languages. I take the risk to give them a coffee break in the middle thus giving them the opportunity to leave discreetly. To my pleasant surprise, they all return to the sauna for part deux.

The AC works great at Ben Gurion University, but here I'm facing a different problem: it's the dreaded right-after-lunch spot and some people have difficulty staying awake. Somebody gives up and leaves after 20 minutes. Fortunately a handful of enthused (and probably hungry) students and one professor get into it. The professor really likes the possibilities opened by Design by Introspection and loves the whole macro expansion idea. Asks me a bazillion questions after the talk that make it clear he's been hacking code generation engines for years. Hope to hear back from him.

Just when I think I'm done, there's one more small event back at Weka. A few entrepreneurs and executives, friends of Weka's founders, got wind of their successful use of the D language and were curious to have a sit down with me. One researcher, too. I'm well-prepared for a technical discussion: yes, we know how to do safety. We have a solution in the works for applications that don't want the garbage collector. "So far so good," told himself the lamb walking into the slaughterhouse.

To my surprise, the concerns these leaders have are entirely nontechnical. It's all about community building, leadership, availability of libraries and expertise, website, the "first five minutes" experience, and such. These people have clearly have done their homework; they know the main contributors by name along with their roles, and are familiar with the trendy topics and challenges within the D language community.

"Your package distribution system needs ranking," mentions a CTO to approving nods from the others. "Downloads, stars, activity—all criteria should be available for sorting and filtering. People say github is better than sourceforge because the latter has a bunch of crap. In fact I'm sure github has even more crap than sourceforge, it's just that you don't see it because of ranking."

"Leadership could be better," mentions a successful serial entrepreneur. "There's no shortage of great ideas in the community, and engagement should be toward getting work done on those great ideas. Problem is, great ideas are easy to debate against because almost by definition what makes them great is also what takes them off the beaten path. They're controversial. There's risk to them. They may even seem impossible. So those get pecked to death. What gets implemented is good ideas, the less controversial ones. But you don't want to go with the good ideas. You want the great ideas."

And so it goes for over three hours. My head is buzzing with excitement as Zohar drives us back to the hotel. The ideas. The opportunities. The responsibility. These people at Weka have a lot riding on the D language. It's not only money—"not that there's anything wrong with that!"—but also their hopes, dreams, pride of workmanship. The prime of their careers. We've all got our work cut out for us.

I tell Zohar no need to wake up early again tomorrow, I'll just get a cab to the airport. I reckon a pile of backlogged work is waiting for him. He's relieved. "Man, I'm so glad you said that. I'm totally pooped after this week. I don't know how you do it."

Funny thing is, I don't know either.
