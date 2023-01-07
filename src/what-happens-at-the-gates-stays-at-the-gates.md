# What happens at the gates, stays at the gates

Welcome to the gateway (⛩️) of the Nym network, your entry into the wider privacy infrastructure, the watchtowers at the edge of the mixnet, the digital post office, or our favourite one…the bouncers guarding the coolest party. 

Gateways are the interface which bridge the physical world of users and computers with the digital network. The technical and financial design of gateways in the Nym ecosystem incentivise different actors to enable the best possible privacy as a service for users.

Do you trust us yet? Verify this information @[Nym's gateway article](https://blog.nymtech.net/gateways-to-privacy-51196005bf5) or in detail @[Nym whitepaper](https://nymtech.net/nym-whitepaper.pdf).

All users connect through a ⛩️ to the mixnode of their choosing. The great thing here is that there’s no commitment required. Now, since a user is free to leave at any time, ⛩️’s are incentivised to optimise and improve the quality of their service, and access to privacy. These bouncers only have one goal in mind: to guard your vibes as you traverse the party taking place on the internet. Since we believe in caring for eachother and reciprocity, the ⛩️ gets rewarded from the pool of collected transaction fees. This amount depends on how many users the ⛩️ can keep happily at their party — ultimately, all together enjoying privacy in eachother’s company.

W**hat does a** ⛩️ **do?**

Gateways are responsible for checking if a user has the credentials to use the mixnet (yes, the bouncers are also cashiers).
⛩️ = the only actors whoever check and store your IP and **bandwidth credentials** (see below), they also shield your IP from everyone else
⛩️ = the guardians of shared privacy, trusted by all nodes to handle their money

**What are bandwidth credentials?**
Bandwidth credentials are a form of anonymous credentials using the decentralised [Coconut signature scheme](https://constructiveproof.com/posts/2020-03-24-nym-credentials-overview/). It proves payment for the mixnet fees in a private manner, essentially moving us away from “who you are” to a privacy-preserving paradigm based on “right to use.” 

**How do gateways check your bandwidth credentials?**
⛩️s are actually **smiling social oracles,** they query the [NYM blockchain](https://nym.explorers.guru/validators) to verify if the credentials have yet to be ‘spent’ before moving traffic through to the mixnet.

**How does this happen in a** **private manner?**

The Nym client on a user’s device encodes data into Sphinx packets, these are encrypted on multiple layers to make all the packets look the same. Then all the Sphinx packets are sent over a secure channel with end-to-end encryption to your ⛩️. They will forward these packets to the mixnet, made up of mix nodes who anonymously relay these SPhinx packets multiple times. At the other end, another gateway will receive the packets from the mixnet. It’ll remove the last layer of the Sphinx encryption and send the message to its final destination (which could be the service provider or end user). Just to be clear: the gateway doesn’t see any plaintext or data because the content is still end-to-end encrypted by the original sender. It’s only decrypted by the receiving client! 

Your very own private delivery person! 

![](what-happens-at-the-gates-stays-at-the-gates/73ew7b.jpg)

**What else can gateways do?**

If a receiving user is offline when a message arrives, the ⛩️ will store messages for later retrieval. Once the user connects they will receive all their messages. So it’s important for ⛩️ to be online consistently and not running on a mobile phone or a laptop (this is a detail you might want to keep in mind as you start to see how ⛩️ operators start to take shape). ⛩️’s also play an important role in preserving anonymity by sending “dummy packets” - empty packets which make it impossible to distinguish between packets, and thus impossible to track traffic 🤠

**Want to run a ⛩️ yourself?** 

To register and run your own ⛩️, you’ll need to go through a process to ensure that ⛩️ operators can’t be taken over for a Sybil attack. You’ll need to put up a registration deposit, think of it as a locked amount of NYM tokens. These will be unlocked when you unregister the ⛩️. Information about your ⛩️ will be published in the blockchain as a descriptor with keys and address, so that users can interact with you. This familiarity between users and their ⛩️ fosters a long-term relationship and incentivises better services too. 

Have any questions about choosing a ⛩️ (or running your own)? [Find the people](https://nymtech.net) and ask, we’re all learning and building together!
