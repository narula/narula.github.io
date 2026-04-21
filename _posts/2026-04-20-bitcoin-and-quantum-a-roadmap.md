---
layout: post
title:  "Neha's Writings"
post_title: "Bitcoin and Quantum Computing: A Roadmap"
date:   2026-04-20 12:40:14
categories:
comments: true
---

I felt kind of guilty about [my last post](https://nehanarula.org/2026/04/03/bitcoin-and-quantum-computing.html), so I have spent a bit of time thinking about post-quantum Bitcoin. This includes:

1. Thinking through specifically what I think a good “roadmap” would be to get Bitcoin to a place where it is secure in the presence of a CRQC
2. Surveying everything that has been going on in Bitcoin-post-quantum-land
3. Identifying the remaining open questions

I have seen a few people construct lists or summaries of (2), and then point to them like “See! Look at all that’s happening! How could you say that there’s not enough going on?!” This is true, there is something going on. Maybe even, dare I say, LOTS going on. But how much is going on is not really the relevant question. (1) and (3) are the most important parts! What is necessary, but not yet completed, to make Bitcoin PQ-safe? What are the tradeoffs of various options and how do we choose between them? We need to know the gap from *here* to *there*: What needs to be done before we can declare Bitcoin secure in the presence of a CRQC, and are we making sufficient progress towards that?[^1]

[^1]: Side note: I really don’t want to say negative things about people who are trying to contribute, but I have a pet peeve about this. Just “doing things” is not always helpful. First, sometimes doing things that aren’t priorities distracts valuable review and evaluation from things that are priorities. Second, there is a lot of work required to actually get something launched. Not just invention or proof-of-concept implementation, but also design, evaluation, production implementation, testing, and activation. There is this concept of [cookie-licking](https://www.redhat.com/en/blog/dont-lick-cookie) in research, where someone kind of starts something or vaguely describes an idea, but doesn’t actually do the the rest of the hard work of getting that idea to fruition. Then no one else wants to work on it cause the idea has been claimed. This doesn’t happen all the time, obviously, there are many examples of someone coming up with an idea and someone else picking up the mantle, but it does happen sometimes. Don’t be a cookie-licker, and don’t be afraid to pick up cookies. Or whatever. I’m not the boss of you, do whatever you want, it’s a free country.

**In this post, I’m going to argue that we should provide a one-shot way for users to make their coins safe in the presence of a CRQC with minimal tradeoffs.** We should do this *even if we have not yet resolved* all the questions of what to do *when* a CRQC appears, like how to handle coins that are still not PQ-safe.

I do not think we have to have 100% of the answers immediately, nor can we. There are different mitigations that have different tradeoffs. We should make the low-harm, low-risk, high-benefit, safety-critical mitigations NOW, and save the high-harm, high-risk mitigations for LATER, when we know with more certainty a CRQC is close.

So what should we do now, in my opinion? 

1.  Design, implement, evaluate, test, and activate a soft fork for a PQ-secure output type and signature scheme(s) for Bitcoin outputs (in conversation with wallets, L2s, and Bitcoin applications to make sure it works for them) 
2.  Coordinate developers of wallets, L2s, and Bitcoin applications (who want to) to correctly support this output type
3.  Communicate to users why they should choose wallets, etc that support this output type and should move to this output type. Make it clear how to use this output type correctly and adapt flows to it, or when necessary, develop new flows.

If this is done, it gives Bitcoin users the ability to move their coins to a safe output type immediately, having confidence their coins are safe even if a powerful CRQC appears, without worrying about future softforks.

The best candidate for this I have seen so far is P2MR (BIP 360) in conjunction with a new PQ signature opcode and cryptographic agility (multiple branches with different signature schemes). This combination gives us the ability to say: Move your coins to this output type, and as long as you do not reveal your non-PQ-safe public key (do not reuse addresses):

* Your coins will be safe in the presence of a CRQC, even if there is no future soft fork to Bitcoin (for both short and long exposure attacks!)
* You can continue to use small, fast Schnorr signatures to move your coins for as long as a CRQC is not on the immediate horizon (and your coins are still PQ-safe!)

This is great. The downsides are the following:

* If you reveal the Schnorr public keys inside your P2MR, this does not help keep your coins safe in the presence of a CRQC *unless there is a future soft fork shutting off the insecure signature schemes*. Revealing public keys might be something that certain wallets/custodial flows do in order to reuse addresses. But you can’t do that. If you think *all* users will do that anyway, then might as well not bother and rely on a future soft fork shutting off ECC anyway. If you think everyone does address reuse, this is equivalent in security to an alternative proposal, P2TRv2. 
* This eliminates one of the so-called efficient privacy benefits of P2TR. Namely, this eliminates the key spend path, which previously you could use when spending the output to pretend like there weren’t other spending conditions, even if there were. Eliminating this leaks exactly 1 additional bit of information about the output (other contract paths exist or not). If you really care, you can avoid this leak in P2MR with an additional 32 bytes, but you are losing the efficiency argument, so the incentives are no longer aligned for this to be the default.

These downsides, for the ability to say that you can go away and not come back for a long time and your coins are safe in the presence of a CRQC, seem reasonable to me. The efficient privacy loss part is annoying, but maybe we can get it back later once we are all using PQ signatures.

# Bitcoin security as a common good

This can help you make *your* coins secure. What this does not help with is making *everyone else’s coins secure* if they don’t use it, or use it incorrectly. Since Bitcoin is a monetary system you should probably care about this. Most of the argument on the bitcoin-dev mailing list to date seems to be about this distinction; namely, there is an argument that if *too many other people’s coins are insecure*, your coins are insecure.

I am not yet sure how to think about this. At the very least I’d say it depends on exact numbers -- if only 0.0001% of coins are insecure, I think Bitcoin will be fine. If 20% of coins are insecure, I think things would probably get pretty chaotic if a CRQC would appear, but I’m not totally convinced there isn’t a *possible* path where Bitcoin might make it through. That said, I think it would be horrible, it wouldn’t really be the Bitcoin we know today, people might lose faith in it entirely, and it’s just way too high risk to bank on.[^2] Either way, I don’t know how many coins will remain insecure. Let’s call this percentage X. I think it’s safe to say X > 0 (not all coins will move to PQ-safe outputs), and X < 100 (at least *some* coins will move to PQ-safe outputs).

[^2]: Pun intended

The nice thing about this proposal is that we can give people an option to move without having an answer to what to do if X is large. AND, we will be able to get *real data*. We can see who moves on chain! We obviously want to design this such that we minimize X: But even if we disagree on exactly what X might be, or what to do with who doesn’t move, we can agree to do that. We can get started on giving people a way to make X small without knowing what to do yet.

Also important to note: In order to make progress on this, we do not need to answer the question of how to provide [escape-hatch-like-schemes](https://www.bitmex.com/blog/Mitigating-The-Impact-Of-The-Quantum-Freeze) to people who *didn’t* move in time but show up later and want to securely move their coins, post Q-Day. STARK proofs of HD-seed, commit/reveal, whatever. Those are all far away from being needed and orthogonal, unless you believe those solutions are so great that we should make one of them the way we handle *all* coins, now. I completely disagree with that. Those solutions suck, a lot. So while it’s interesting that people are looking into those, they are not the imminent fixes in front of us (and I would argue spending too much time on them is confusing and distracts everyone from what *are* pragmatic ways to address this problem). And, what are you going to recover your escape hatch coins *to*? We need a PQ-safe output option no matter what.

Most importantly, we do not have to decide what to do with people who are unlikely to show up to do anything at all (Satoshi’s coins) right now in order to make progress.[^3] Eventually, if a CRQC seems close, we will have to make a decision one way or the other (leave the coins to potentially be taken, freeze them, repurpose them for mining rewards, give them to me, etc. What? Just saying, it’s an option). That is going to be a difficult conversation and I’m glad we are starting it now. But resolving that conversation is not needed to make useful, meaningful progress. 

[^3]: If you believe Satoshi is not going to move their coins, [this puts a floor on X > 2.9](https://www.bitmex.com/blog/satoshis-1-million-bitcoin). If you believe all old P2PK is probably not going to move, [X > 8.1](https://arxiv.org/pdf/2603.28846).

![Roadmap for making coins PQ-safe](https://nehanarula.org/assets/pq-safe-bitcoin.png)

*Proposed roadmap for mitigations. The actions in the blue dashed boxes help with the blue triangle; the actions in the purple dashed boxes are only needed for the purple trapezoid. We cannot have PQ-safe outputs until the first activation happens; we do not need to consider whether or not to freeze coins until closer to Q-Day. Even if the second soft fork doesn’t activate, or we can’t reach agreement, the blue triangle Bitcoin is PQ-safe. Inspired by a slide from Ethan Heilman’s Cryptographic Agility talk, which you can see once recordings come online from the 2026 MIT Bitcoin Expo or 2026 OPNEXT.*


# Counterarguments

There are valid reasons why you might disagree with the above as a strategy, even if you want to move forward on making Bitcoin secure in a post-quantum world (If you don’t think a CRQC is a risk worth addressing see my [last post](https://nehanarula.org/2026/04/03/bitcoin-and-quantum-computing.html)). I might discuss the most reasonable ones in more detail in a future post; for now, see the recent discussion on the mailing list. I will very briefly summarize the arguments as I understand them here: 

First, you might think that P2MR is going to be too difficult for wallets to implement correctly (namely because they really like reusing addresses), so even if deployed and implemented, it will be implemented incorrectly and a significant number of wallets will not truly be PQ-safe. You mostly agree with the above strategy, you just don’t think P2MR is the specific right way to go about it.

Second, you might think the blue triangle will not be large enough (wallets won’t implement it, people won’t move), so we should spend most of our time and energy on the purple trapezoid instead. In fact, you might think the purple trapezoid is going to be so large that we might as well just wait until closer to Q-Day and cram this into one soft fork. 

Third, you might think that deploying a PQ-safe output type now is in some ways "giving up" because X will still be too high, and it will make it harder to motivate people to come up with better solutions. They might not appreciate the common good point raised above.

I am not sure I am best representing the counterarguments here, because quite frankly they don't seem like good reasons to me not to proceed. Please let me know if I missed yours. Also note that these counterarguments do not have anything to do with the question of whether to freeze or not.

See the FAQ for a shorter explanation of proposals that I do not think are as interesting, but are still worth explaining since they come up a lot (please stop bringing them up).

# FAQ

**Why P2MR and a new PQ-safe signature opcode? What about OP_CAT?**

Yes, soft-forking in OP_CAT would also enable us to implement PQ-safe signatures manually in script, or even verify STARKs (but if this is in tapscript, you’d still need a future soft fork to shut off the taproot keypath spend, or P2MR). We need to make the distinction between *technically possible* and *pragmatically a good idea*. Encouraging this technique for all users is not a good idea: scripts would be huge and gross and unwieldy. This is not a good way forward from an efficiency perspective.

**What about the [QSB paper](https://github.com/avihu28/Quantum-Safe-Bitcoin-Transactions)? Doesn’t that make Bitcoin quantum-safe right now?**

This paper does technically provide a way for people today to move their coins to an output and script that would be safe in the presence of a CRQC without any soft forks (assuming it's correct. I haven't evaluated it yet, so cannot vouch for that). But see above on technical possibility vs. pragmatically a good idea. Among other concerns, this technique is expensive, like hundreds of dollars per transaction, and the transactions would be non-standard so you’d have to ship them straight to a miner, bad for decentralization. As it currently exists, this is a neat research proof of existence. It is not a good solution to make Bitcoin PQ-safe for as many people as possible. 

**What about specific post-quantum signature schemes? You didn’t say anything about those.**

Why thank you for asking. I’m trying to keep this post manageable in size. Go watch Jonas Nick’s [talk from OPNEXT about OP_CHECKSHRINCS](https://blockspace.media/podcast/op_checkshrincs-a-hash-based-signature-opcode-for-post-quantum-bitcoin-opnext-26/). This seems like the most likely path forward. Basically, they’ve come up with a signature scheme that is not HORRIBLY large but requires keeping track of the number of times you signed, and is still like 5X as large as Schnorr today. If you lose track you can fall back to the *really* horribly large one. We probably also want a block size increase unless we want to accept half as many transactions per second (best case). I know plenty of you get riled up about that sort of thing but sorry, that’s life. Things are enormous in a PQ world and we really need to consider that as PQ signatures start having to go on chain. I’m willing to go on record saying a 2-8X block size increase is probably fine from a decentralization perspective, and we can do it as a soft fork. It is no longer 2016. I think I’d rather see that than a 2X (most optimistic) reduction in throughput.

I look forward to the BIP and people banging on the new signature scheme. People who work on wallets/L2s/etc should check this out now and see how much harder this makes your life. Also, if you can come up with a post-quantum hash-based signature scheme that's smaller, that would be really helpful for Bitcoin!

**I am very angry about BIP 361**

Yes. I acknowledge your anger. The one ray of hope about what I’m saying here is that there is a possible world where we never have to make the hard decision that BIP 361 attempts to address: if either 1) a CRQC never appears or 2) almost everyone moves their coins to PQ-safe outputs (including Satoshi). Unfortunately, in the long-term, I think the likelihood of those is low. So yes, I predict we will eventually have to struggle with what to do with Satoshi’s coins, but we probably have time. And the sooner we implement PQ-safe outputs, the more time people have to move! So maybe take a look at P2MR (BIP 360) and do some deep breathing.

**Aren’t you just arguing we should punt on the really hard problem?**

Yes. Yes indeed. This is my strategy in life as a serial procrastinator, and honestly so far it’s worked pretty well; sometimes if you just wait your problem goes away. OK but seriously, let me make a better argument: First, we need more information before we can answer the really hard problem well. I honestly just don’t know what to do with Satoshi’s coins.[^4] In order to feel more confident one way or the other, I would want to better understand the game theory around an adversary with a CRQC, especially as it applies to miner incentives. Hint hint, someone should fund this research.

[^4]: Before you yell at me for even *considering* the idea of turning off ECC (which would freeze Satoshi’s coins if they’re not moved in time), please understand that it is not something I want. Among all the horrible things described in this post, it is one of the most horrible. I am not advocating for it. I would only advocate for it if I was *convinced* Bitcoin as a whole cannot remain secure in a world where a powerful CRQC-adversary exists and Satoshi’s coins are vulnerable on chain (my current thinking is that this might be the case due to miner incentives for reorgs to battle for the coins. I don’t give a crap about dumping a bunch of coins on the market, this is not a good reason and I am pretty sure that’s recoverable. For example, Michael Saylor might go bankrupt and sell tomorrow, and no one is going to make an argument to freeze his coins). So if the option is “no Bitcoin” or “disable ECC (which is about to be badly broken) on Bitcoin, with years of warning” I would choose the latter. But I don’t know that’s the choice in front of us to make, yet. Others might have more certainty one way or the other.

Second, we should make progress as best we can on this question now, but also acknowledge that it might not be a question for us to decide. If a CRQC doesn’t appear for 100 years, then we should not prematurely make this decision for the Bitcoin users of 100 years from now. It’ll be their Bitcoin, and their choice.

# Acknowledgements

Thanks to Ethan Heilman for feedback on this post. No endorsement implied, all mistakes my own.

**Edit 4/21**: Thanks to Antoine Poinsot for pointing out according to BIP 360, Schnorr is the only signature option since outputs are confined to tapscript. Removed references to ECDSA above.

# Footnotes
