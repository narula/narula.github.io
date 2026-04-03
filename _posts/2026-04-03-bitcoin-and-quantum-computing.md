---
layout: post
title:  "Neha's Writings"
post_title: "Bitcoin and Quantum Computing"
date:   2026-04-03 12:42:14
categories:
comments: true
---


Bitcoin’s signatures are broken if a cryptographically-relevant quantum computer (CRQC) were to appear tomorrow.[^1] Bitcoin requires changes both to its code and to everyone’s wallets (at least a soft fork and many users moving coins to different types of addresses) to be secure in the presence of a CRQC.[^2]

[^1]: Though this statement is true, I’m not trying to scare you or say Bitcoin is *currently* broken. The likelihood of a CRQC appearing *tomorrow* is basically 0. *Bitcoin is not currently broken*.

[^2]: I don’t think the previous statements are controversial, but if you do disagree let me know! The rest of this post will probably not be interesting to you, but perhaps I need to write a different one explaining why they are true.

The remaining uncertainty is in two main areas: timeline and how to address this. I will frame these two issues in the following way:

1. What is the likelihood of a CRQC appearing, and on what timeframe?
2. What are the best paths for Bitcoin successfully upgrading so that it would not be broken in the presence of a CRQC, and at what cost to Bitcoin? What is the set of tradeoffs, and how should Bitcoin navigate this space of tradeoffs?

I think the following:

* The chance of (1) is non-zero for various timeframes
* We do not yet know the answer to (2), we don’t know if there will be
  agreement on how to navigate the tradeoffs once there is a defined
  set of possible paths forward, and it’s not clear there is agreement
  to even do anything. Therefore, it is not 100% clear that Bitcoin
  *will* successfully upgrade before a CRQC appears.

An important implication if you believe the, I think, pretty reasonable previous statements is:

A CRQC is a (you might believe low-likehood) existential threat to Bitcoin’s existence. Your measurement of this threat should literally be: 

(A) How likely you think it is a CRQC appears by a given time, multiplied by <br>
(B) How likely it is you think Bitcoin will not successfully upgrade by that time. 

Let’s put in some example numbers: Google [recently introduced a timeline for moving to PQC by 2029](https://blog.google/innovation-and-ai/technology/safety-security/cryptography-migration-timeline/) and a coauthor on [a recent Google Quantum paper](https://arxiv.org/abs/2603.28846) [suggested he wouldn't bet against a 10% chance of a CRQC existing by 2030](https://x.com/CraigGidney/status/2036850592375841249) (A=.1). The last soft fork in Bitcoin (taproot) took 3 years and 10 months from proposal to activation. Maybe there’s a 50% chance we could successfully roll out some as-yet-unknown PQ soft fork by 2029 (B=.5).[^3] If you believe these numbers there is at least a 5% chance (0.1*0.5=0.05) you should expect Bitcoin to no longer work due to a CRQC in 2030. 

[^3]:  Given what I’m seeing on X these days and the lack of developers working on it right now I think I’m being REALLY generous here.

## How to think about this an an investor

I personally care more about using Bitcoin than its price, but I know many of you do care about Bitcoin’s price and this has implications on it: This probability of Bitcoin no longer working is a floor for how much you should value Bitcoin at $0. There are many additional reasons to potentially value Bitcoin at $0, which is why I say floor. This should additively combine with all the other types of uncertainty you have about Bitcoin’s continued successful operation: keys might get stolen or hacked (yours or someone else’s, like Coinbase’s), Ethereum might overtake Bitcoin’s narrative as digital gold and render it irrelevant, someone might partition the network and conduct numerous destructive double spend attacks, cryptoeconomic security might break down by 2140 when the block reward drops to 0 and we have continuous block sniping attacks, etc etc etc…

IANAI (I am not an investor, also none of this is financial advice) but my understanding is most competent investors take this sort of thing into account when evaluating assets and creating their portfolios. You can fill in whatever values for A and B you think make sense and place your bets accordingly.

## How to think about this as a user

Even if you don’t really care about the Bitcoin price, a lot of the above thinking applies even if you just want to use or build on Bitcoin. [Some cryptographers are already saying they won’t bother working on non post-quantum cryptography anymore](https://www.linkedin.com/posts/sharon-goldberg-002803143_to-my-academic-cryptographer-friends-today-activity-7444843296838795264-LtS0). It’s hard to convince people to work on and engage with Bitcoin when it might be completely broken in a few years. 

## Moving forward

Assuming you think A is greater than 0, the clearest way to eliminate this existential threat is to make B zero and upgrade Bitcoin to post-quantum cryptography as soon as safely possible, specifically the supported signature schemes. The good news is PQ signature schemes exist and are usable! The bad news is there are many other valid things to worry about, so it’s not as simple as it sounds. I want to be clear that there are a few people doing important work in this area, but as far as I know it’s a very, very small number. As of today these questions remain unanswered: 

* **Which specific signature scheme(s) should we use in Bitcoin?** Unclear. Most of these schemes produce significantly larger signatures or require longer to verify. Choosing non-optimally might have downsides, like significantly increasing the cost to run a node, throttling Bitcoin’s supported number of transactions per second and available blockspace, or limiting future Bitcoin functionality like signature aggregation.
* **How should consensus consider transactions spending UTXOs that haven’t upgraded to PQ-schemes?** Even more unclear. Choosing badly on this front might violate Bitcoin’s core principles, thereby reducing trust in Bitcoin’s narrative of self-sovereignty and scarcity, or could render the economic security of Bitcoin mining unsound.
* **What are the exact steps and timeline(s) to add in optionality for PQ-signature schemes, and then to presumably require them?** Unclear. Do it too soon and you might accidentally pick non-optimal cryptographic primitives. Do it too late and Bitcoin is broken or it’s a mad rush. 
* **Before it’s very close to Q-day, how do you motivate wallets to actually make the effort to implement these upgrades and support the new schemes?** Not sure!

Which makes this all frustrating, and exciting! If you think A is basically 0, I get why you don’t want to bother with all this complexity. There is real overhead and risk to switching to post-quantum cryptography that needs to be weighed against the risk of Bitcoin breaking in the future. I do not have the background or interest to evaluate A from first principles myself, and I don’t trust most people on the internet to be able to do this effectively either. I rely on the [expertise of others who I know and trust](https://scottaaronson.blog/?p=9665). On a long enough timeframe, A gets high enough that it becomes a question of when, not if, Bitcoin should upgrade. Based on this, let me make my position clear: **I think A is high enough to warrant prioritizing designing, implementing, and evaluating post-quantum signature schemes and consensus upgrades in Bitcoin now.**

I think investors are also probably going to listen to the experts and price in this risk, so even if you  believe A is 0, if you care about the medium-term price of Bitcoin it’s probably in your best interest to either convince the experts they are wrong (good luck!), convince all the other investors not to listen to the experts, or get onboard with getting B to 0.

## Some common “counterarguments”

**Q: A quantum computer destroying Bitcoin is FUD.**

A: There certainly are many untrue claims out there, and it’s very frustrating to see [headlines](https://x.com/coinbureau/status/2038856197080785307) like “⚠️GOOGLE SAYS A QUANTUM ATTACK ON BITCOIN TAKES JUST 9 MINS WITH A 41% SUCCESS RATE” (this is not true. you cannot attack Bitcoin this way today).

But there’s also some truth to what’s going on. You have to filter out what’s just plain incorrect, on both sides. Calling everything FUD is lazy and unhelpful.

**Q: Isn’t there a chance a CRQC might never exist?**

A: Yes! Totally! But the word “might” is doing a lot of heavy lifting here. Unfortunately for Bitcoin there is also a non-trivial chance one will exist. Like I said above, [according to a quantum computing researcher at Google](https://x.com/CraigGidney/status/2036850592375841249), a 10% chance by 2030. To be clear, there will need to be a lot of work done and crazy advances made before one exists, and this is why A is not 1 for any short time frames. But I have found it’s not wise to bet against smart people figuring things out. For the reasons I described already, I think this chance is significant enough to warrant preparing for it if you still want Bitcoin to work.

**Q: Yeah ok but they can’t even factor 21 yet. Why should we do anything now?**

A: Sometimes your intuitions in one area don’t apply in another area. Turns out this is simply the wrong way to think about quantum computing progress. I think [Scott puts it well](https://scottaaronson.blog/?p=9665#comment-2029013):

> Once you understand quantum fault-tolerance, asking “so when are you going to factor 35 with Shor’s algorithm?” becomes sort of like asking the Manhattan Project physicists in 1943, “so when are you going to produce at least a *small* nuclear explosion?”

[More](https://bas.westerbaan.name/notes/2026/04/02/factoring.html) on why this is the wrong intuition and waiting till CRQC's can factor larger and larger numbers might be the wrong strategy. 

**Q: A CRQC also breaks banking, military communications, and most of the internet today! If one appears, isn’t Bitcoin the least of our problems?**

A: True! Banking software, military communications, and the internet also need to be upgraded. I have high confidence they will be, successfully (I’d put my B_{HTTPS} at close to 1). Unfortunately, I have less confidence that Bitcoin will upgrade successfully since upgrading a decentralized system of honey-badger-like participants is much more challenging and people like the questioner seem to think this is a valid argument that we shouldn’t even worry about it?

If you disagree and think there will be a CRQC and the rest of the internet won’t upgrade successfully, maybe you should consider shorting the stock market and buying gold. But not Bitcoin, because if we do nothing that won’t work anymore. Not investment advice.

**Q: This is probably just Google trying to destroy Bitcoin.**

A:  It’s possible for the following two things to both be true: “Google wants to destroy Bitcoin” AND “Google’s paper is technically sound”. The former does not negate the latter. 

I personally do not think Google cares much about Bitcoin outside of thinking it’s a great use case to demonstrate quantum prowess, but if you’d like to bet both that (1) this result is a psyops by Google to take Bitcoin down, and (2) all the authors are willing to compromise their reputations by putting out a paper with no actual technical merit, you do you.

**Q: Even if Google (or a nation-state) had a CRQC, why would they steal Bitcoin? Everyone would see it and it would tank the price!**

A: It’s pretty hard to speculate with any level of accuracy about how exactly a CRQC adversary would or would not engage with Bitcoin. There could be many motivations for various action paths, including an incentive to keep the CRQC secret and not touch Bitcoin for a while. You should factor your beliefs about this into your A. I think the real point is that in the presence of a CRQC, Bitcoin’s signatures are broken, and we cannot rule out the worst case (or even articulate the entire adversary strategy space). 

**Q: Stealing is illegal, so why would anyone use a CRQC to steal Bitcoin?**

A: If you truly believe this, you really should value Bitcoin at 0 -- it has many unnecessary components with a lot of overhead, like proof-of-work and digital signatures.

**Q: Bitcoin will figure this out. The "developers" have fixed things in the past.**

A: OK, great. How? Who is doing this? What’s the plan? Which primitives? Why those over others? Where is the discussion and resolution of the tradeoffs I mention above beyond a few threads on the mailing list? Do you agree with everything in [BIP 360](https://github.com/bitcoin/bips/blob/master/bip-0360.mediawiki)? If not, what are your concerns and what do you propose instead? What do we do with P2PK coins that are vulnerable and probably won’t move, like Satoshi’s? Where are the designs, BIPs, implementations, and evaluations of various options?

**Q: Well what are you doing about it?[^4]**

[^4]: I think it’s totally fair to call this sort of thing out, but will also note that it doesn’t engage with or invalidate any of the actual arguments.

A: So far not much! Trying to inform myself, encouraging other people who are working on it, and writing this post to hopefully convince more people to work on it. My work on Bitcoin is through my role at MIT running [DCI](dci.mit.edu). Maybe we should try to work on some of the technical challenges, or fundraise and hire people to work on this specifically. Should we? Are you interested?

What are you doing about it?

# Acknowledgements

Thanks to Ethan Heilman for feedback. No endorsement implied, all mistakes are my own.

*Edit*: A previous version of this post mistakenly cited the Google paper as saying "there is a 10% likelihood of a CRQC (can break a Bitcoin ECDSA or Schnorr key with reasonable likelihood in 9 minutes) by 2029 (A=.1)." That's been corrected above; the Google paper did not make this claim, an author on the paper did for 2030. Thanks to [@UnseenNight](https://x.com/unseennight) for pointing this out!


# Footnotes
