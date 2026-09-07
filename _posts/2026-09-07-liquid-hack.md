---
layout: post
title:  "Neha's Writings"
post_title: "Quick thoughts on the Liquid hack"
date: 2026-09-07 12:47:00
comments: true
---

Yesterday Liquid, a federated sidechain created by Blockstream, was compromised and 4,000 BTC was taken from its bridge by a whitehat hacker:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">We are aware of a security incident on <a href="https://x.com/Liquid_BTC?ref_src=twsrc%5Etfw">@Liquid_BTC</a>. Purported white-hat hackers have withdrawn ~4,000 BTC (~$320 million) from the Liquid Federation wallet. The <a href="https://x.com/Blockstream?ref_src=twsrc%5Etfw">@Blockstream</a> team is working on contacting them on-chain with a signed message.<br><br>What we know so far is that the funds…</p>&mdash; Liquid Network 🌊 (@Liquid_BTC) <a href="https://x.com/Liquid_BTC/status/2096696272447218108?ref_src=twsrc%5Etfw">September 6, 2026</a></blockquote> <script async src="https://platform.x.com/widgets.js" charset="utf-8"></script>

This is a very quick post on what happend and a few thoughts. This is all unfolding, so it's quite likely I got something wrong. I'll try to post updates and corrections quickly as I understand them. Also, please keep in mind that Bitcoin itself was not compromised. As far as I know, this has nothing to do with a bug or issue in Bitcoin itself.

There are a few important things to understand about this hack:

## The bug

Liquid supports [confidential assets](https://blockstream.com/bitcoin17-final41.pdf). This means that asset amounts
are shielded cryptographically, and are not observable. However,
certain properties about the transactions containing these shielded
assets should still be publicly verifiable: importantly, that the
transactions _balance_, or that the sum of the spent inputs is greater
than or equal to the sum of the newly created outputs. This ensures
that money that didn't exist before isn't created out of
thin air. Because the transactions are shielded, this is the _only_ way
to ensure this property about the transaction -- the raw amounts are
not visible.

This is done using a Pedersen commitment and range proof. The Pedersen
commitment lets one check that the inputs and outputs balance, while the range
proof ensures that you aren't playing a sneaky trick where you use
negative numbers to inflate supply. For example, you could spend a
10 value input and create two outputs, one -4000 and one +4010, and
abandon the -4000 output. The Pedersen check will pass, but you've now
created 4000 new units of money out of nowhere. At a high level, the
range proof makes sure there are no negative numbers.

Range proofs are expensive to verify, so it makes sense to cache the output of verification 
if you can. This caching was [implemented
incorrectly](https://github.com/ElementsProject/elements/commit/c26d719c29),
and the whitehat hacker was able to cleverly circumvent the range
proof and, using something like the above, mint new L-BTC on Liquid.

See [here](https://x.com/mononautical/status/2096928595432374706) for
what seems like a good description and timeline of the bug, with links
to the transactions.

## Liquid

This all happened on Liquid, which is a sidechain on Bitcoin developed
by a company called Blockstream. Liquid uses a set of 15 signers (they
call them
[functionaries](https://help.blockstream.com/liquid-network/faqs/what-is-the-liquid-federation))
in a multisig to custody bitcoin on the Bitcoin blockchain; [11/15
signatures are needed to move the bitcoin on the Bitcoin
blockchain](https://help.blockstream.com/liquid-network/faqs/how-does-the-liquid-federations-multisig-work). So
taking advantage of the bug as described above _only affects
L-BTC on Liquid_. ~4,000 new L-BTC was minted on Liquid only.

The next issue is with how L-BTC is withdrawn from Liquid and
converted into real bitcoin. It seems that all of the
functionaries were operating as intended (aside from the bug) -- none of the functionary
signing keys were compromised.

Obviously, you don't just want to withdraw bitcoin from a bridge
willy-nilly; you want to be careful and make sure the real bitcoin
only moves the way it should. 

In the case of Liquid, this means checking at least two things are
true: 1) the request to withdraw real bitcoin burns an appropriate
amount of L-BTC on the Liquid sidechain, so the amount of L-BTC on
Liquid matches the amount of real BTC held in the Liquid bridge
multisig on the Bitcoin blockchain and 2) the withdrawal is going to
an authorized address.

Liquid uses something called Peg-out Authorization Keys, or PAKs, for
its allowlist functionality. The federation can only transfer Bitcoin
out of the sidechain to a [pre-specified set of
addresses](https://help.blockstream.com/liquid-network/faqs/what-is-a-liquid-peg-out).

Every functionary checks these two things, and both of these checks
passed, because in the peg-out transactions the whitehat hacker was
burning the fabricated L-BTC they created above.  The withdrawal was
to an authorized PAK owned by a service called [SideSwap](https://sideswap.io/), which
proxies Liquid withdrawals for users who don't have a PAK.


## SideSwap

Unfortunately, SideSwap didn't seem to implement any guardrails or
brakes beyond "this was valid and signed appropriately by the
functionaries" and didn't seem to notice or care that they received
almost all the BTC in Liquid. So they happily further transferred the
BTC to the whitehat hacker on the Bitcoin blockchain (none of their
keys were compromised either). Finit.

Note that the whitehat hackers have indicated they intend to return
"most" of the funds. This communication is all playing out in
OP_RETURN messages on the Bitcoin blockchain. You can't make this
stuff up.

## Takeaways

**Formal verification of cryptography wouldn't have helped in this case.** There
was a bug here connected to cryptographic machinery, but it was a
cache-key collision bug. Unfortunately, properties like integrity
against inflation are a property of the _system as a whole_. Formally
verifying the cryptographic code in isolation is not sufficient; you
would need to verify the whole system _around_ the code. This is much,
much more challenging. I don't think anyone in crypto is actually
verifying databases, file systems, networking, or caches.

That said, any formal verification is certainly a good idea! There
is a lot of value to formally verifying components, even if you can't
formally verify _everything_. But it's important to remember it's not
a panacea.

**Federation is security theater?** There could have been 10,000 or
100,000 functionaries in the federation and it wouldn't have helped
security in this case, at least not in a meaningful way. I'm being a
little loose with this; apparently different nodes were running
slightly different versions of Elements (some were fixed?) and so
[Liquid actually forked on the block containing the inflation
transaction](https://x.com/wiz/status/2096706364206887049). But
clearly overall security is not linear in the number in the
federation!

**The liability question**. Who gets sued? I think there are a few
different cases to be made here, if users aren't made whole but even
if they are. Not sure what kind of expectations holders of L-BTC had
on members of the Liquid network to keep their L-BTC properly backed
with Bitcoin. Or is it Blockstream's fault for false promises? I mean
their
[website](https://web.archive.org/web/20260616125738/https://help.blockstream.com/liquid-network/faqs/what-is-a-liquid-peg-out)
literally says "The number of LBTC on the Liquid Network always
verifiably matches the number of BTC locked on the mainchain
one-to-one". If a user was using a regulated exchange that supports
Liquid, maybe they can sue them? Even if users are made whole, Liquid
was halted for a while, and maybe people who wanted to trade on Liquid
lost money.

I find it difficult to believe that traditionally regulated financial
institutions are going to take on the risk of interacting with a
system like Liquid in the future. But some (not all) of what happened
here could also happen to various Ethereum L2s. Caveat emptor.

I am not a lawyer, none of this is legal or financial advice.

**Cryptographic privacy (without guardrails?) might be too scary.**
This makes me sad, because I care deeply about building
cryptographically privacy-preserving financial systems. But it might
just be too risky. At the very least, we need a lot of guardrails
around them, like speed bumps on withdrawals and many defense-in-depth
checks. It's unfortunate that didn't happen here.

## Updates

The whitehat hackers have returned 3,400 BTC out of the ~4,000 BTC ([mempool.space](https://mempool.space/tx/a6d697a25266ce3c78774fd1d75f896b7af522ada209b0f6228ea497bc49a46d)).
