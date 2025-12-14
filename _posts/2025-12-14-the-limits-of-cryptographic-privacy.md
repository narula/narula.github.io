---
layout: post
title: "Neha's Writings"
post_title: "The limits of cryptographic privacy"
date: 2025-12-14 11:56 -0500
comments: true
---

*The following are remarks I gave at the [2025 DC Privacy
Summit](https://www.dcprivacysummit.org/). TL;DR we don't talk enough
about potential dystopian outcomes for privacy tech. It's not an
answer by itself, we probably need thoughtful policy. Watch the
remarks [here](https://www.youtube.com/watch?v=NsRFuTn3WbU) and a
fireside chat with Commissioner Peirce
[here](https://www.youtube.com/watch?v=XjAg0bEstj0). Thanks to [Mike
Orcutt]() for inviting me and writing this [thoughtful
summary](https://www.projectglitch.xyz/p/how-to-prevent-online-privacy-from) of the event.*

Here's a scenario some of us have lived: You use Signal because it's
encrypted. You've turned on disappearing messages. Then someone
screenshots your chat and shares it (or you [accidentally add a
reporter](https://abcnews.go.com/Politics/messages-yemen-war-plans-inadvertently-shared-reporter-timeline/story?id=120128447)). All
that encryption didn't help at all.  This illustrates something
crucial we often miss in privacy conversations. When we talk about
privacy today, the conversation quickly turns to technology:
encryption, verifiable credentials, zero-knowledge proofs, multi-party
computation, fully homomorphic encryption. I work on some of these
technologies, and they are extraordinary. But we need to be precise
about what they can and cannot do.

# The Limits of Cryptographic Privacy

Take zero-knowledge proofs. They let you show that a computation was
carried out correctly without revealing the underlying data.  Yet they
don't tell you whether the data *itself* is accurate or complete. They
don't preclude the need for an authority to source that data, like a
government indicating citizenship. The mere use of verifiable
credentials cannot prevent an authoritarian government from denying
some of its citizens that credential, or stop a platform from using
the credential to kick certain types of people off.  In blockchains,
cryptographic techniques like ZKPs are uniquely powerful because
consensus and computation together define ground truth, at least as it
pertains to the blockchain. A proof of correct computation is
definitionally always correct, no single source of authority
required. But the rest of the world doesn't work that way, and,
honestly, that's a feature, not a bug. In the real world we still
depend on people and organizations, and reality to define what data is
"correct."

# Privacy Tech as a Tool for Control?

Here's what worries me most: if used unwisely, instead of reinforcing
autonomy, these cryptographic tools may help replace it with automated
control.

Imagine a world where producing cryptographic proofs is easy and
automatic, and so asking for them becomes routine. You need to prove
you're creditworthy to use a rideshare app. You have to prove your
health status to enter a building. Prove your spending patterns to get
insurance. Prove your income to access a dating platform. Services ask
for proofs of income, political affiliation, or health because they
can, and users might not even think about whether or not they should
comply, because it’s automatic and easy to do.

The fact that these requests don't leak specific data makes them feel
more acceptable. Over time, the ability to prove becomes an
expectation. A technology originally designed to help with privacy
turns into continuous permissioning. We could end up in a world where
constant proving replaces constant disclosure. The information is
hidden, but the control remains.

# Why Markets Won't Save Us

Even if we get the technology and rules around how to use it right,
technology does not operate in a vacuum. Unfortunately, markets don't
reward privacy. Individuals are often unwilling to pay for it, they
favor convenience and ease of use. Platforms favor access to data so
they can monetize it. And there's a circular benefit where the more
data, the better the results, and the more users prefer the less
private service.  So even if we have the right technology, the reality
of market forces means it might not get adopted.

# Where Policy Can Help

This is why the mere existence of technology is not enough. Privacy
requires not just good cryptography, but good law. We have to work
within the frameworks of law and policy as well, and I think this
should happen in three ways.

1. **Stop criminalizing the creation and use of privacy-preserving
   technologies.** There is still a belief in many circles that tools
   like CoinJoin and privacy coins are only for criminals, and that
   running software to help users unlink transactions is somehow
   wrong. There is a fundamental lack of understanding of how
   unprivate the default is.

   Here's an example: Imagine that a user gets paid by their employer
   on-chain in bitcoin. The employer can then directly observe how the
   user further spends that money on-chain, and might be able to see,
   for example, if a user pays to a well-known charity address or a
   medical professional. It makes perfect sense that users would not
   want this and would try to employ techniques to avoid it.

   This is especially important because we need to make the use of
   privacy-preserving systems the norm, not the exception. It cannot
   be treated as a red flag.

2. **Establish legal protections for user rights like self-custody and
   data portability.** It is important to protect the ability to
   control one's own assets and information without being forced
   through intermediaries who must report on them.

   Right now, the default assumption in regulation is that
   intermediaries are necessary surveillance points. But individuals
   can custody their own assets and manage their own data directly,
   and this is important to promote competition. We should enshrine
   the right to self-custody without it being treated as inherently
   suspicious, create real data portability requirements, and not
   require that every participant route through a monitoring
   gatekeeper.

3. **Stop mandating that private-sector intermediaries engage in
   specific data collection.** More broadly, stop using the private
   sector as an extension of law enforcement. Many existing laws
   assume that gathering and storing personal data--often in very
   specific formats--is the only way to ensure accountability. This
   is overly prescriptive and also not true. This means revisiting
   laws like the Bank Secrecy Act.

   This is not about rejecting oversight entirely, or ignoring
   risks. It is about starting from the question that should guide
   both technologists and regulators: what are we actually trying to
   achieve? If the goal is to prevent harm, detect fraud, and preserve
   fairness, we know there are ways to do that that do not rely on
   continuous data collection.

Technology can protect privacy, but only if we also carefully design
our systems, markets, and laws to respect it.
