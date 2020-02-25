- Start Date: 02/19/2020
- HIP PR:
- Tracking Issue:

# Summary
[summary]: #summary

Ensuring trust in the network when non-helium hotspots begin to
participate in the network.

# Motivation
[motivation]: #motivation

Helium's primary objective has always been to establish a decentralized
permissionless wireless network allowing anyone to be able to join and
provide RF Coverage with off-the-shelf hardware and open source
software.

This goal poses some interesting challenges specifically regarding
network growth. Once we allow non-helium hotspots to be able to join the
network, we lose the ability to verifiably prove that any sub-network
created by such new hotspots truly exist physically since it's easily
possible for dishonest actors to fake geographic locations and radio
transmit/receive and claim that they are providing network coverage.

# Stakeholders
[stakeholders]: #stakeholders

- 3rd party hotspot manufacturers
- DIY hotspot builders

# Detailed Explanation
[detailed-explanation]: #detailed-explanation

In order to ensure organic network growth while still maintaining
verifiable trust regarding the claim of location of newly joining 3rd
party or DIY hotspots we propose that every hotspot joins the network in
a probationary mode with limited PoC capability.

In the current network onboarding a helium manufactured hotspot allows
full proof-of-coverage priviledges as soon as the hotspots syncs fully
with the blockchain, however this behavior is insufficient to ensure
that non-helium manufactured hotspots or DIY hotspots by individual
developers adhere to the consensus rules and also honestly provide RF
coverage.

Currently we know that every single hotspot being added to the network
is manufactured by helium and has the required hardware to participate
in proof-of-coverage challenges, as soon as we allow other hotspots to
join the network we have no verifiable way of being able to separate a
virtual network from a real one.

To mitigate that, we propose introducing a probationary mode. Every
single hotspot which will join the network (including helium
manufactured) will do so in "probation". In probation mode, the PoC
activity of a hotspot would be limited to only participate in challenges
which involve the external device.

To gain full PoC priviledges, a hotspot must complete a pre-determined
number of successful challenges involving the external device. This
number can be set via chain variable on the blockchain.

Consider the below network where hotspot A-F are non-helium manufactured
hotspots, with the lines representing RF visibility between hotspots.

                     +------+
                     |      |
         +-----------+  A   |
         |           |      |
         |           +------+
         |
    +------+
    |      |
    |  F   |
    |      |
    +------+
        |            +------+
        |            |      |
        |            |  B   |
        |            |      |
        |            +------+            +------+
    +------+             |               |      |
    |      |             |               |  C   |
    |  E   +-----------------------------+      |
    |      |             |               +------+
    +------+             |
        |                |
        |             +------+
        |             |      |
        +-------------+  D   |
                      |      |
                      +------+

There exists no possible way to ensure that these hotspots do not form
part of a virtual network, since they can easily fake locations and RF
coverage and verify each other via the existing PoC mechanism. Therefore
these remain in probation mode with limited PoC priviledges.

Until and unless there is a helium (or an authorized retailer)
manufactured hotspot introduced in the above network, we cannot make any
conclusions regarding the claim of location of any hotspot A - F.

Reimagining the above network with the introduction of a helium hotspot,
consider the graph below:


                     +------+
                     |      |
                     |  A   +---------------+                      
                     |      |               |                      
                     +------+               |                      
                                         +--+---+                  
    +------+                             |      |                  
    |      |                             |  X   +<--+Helium Hotspot
    |  F   |             +---------------+      |                 
    |      |             |               +------+                 
    +------+             |                                        
                     +---+--+                                     
                     |      |                                     
                     |  B   |                                     
                     |      |                                     
                     +---+--+            +------+                 
    +------+             |               |      |                 
    |      |             |               |  C   |                 
    |  E   |             |               |      |                 
    |      |             |               +------+                 
    +---+--+             |                                        
        |                |                                        
        |             +--+---+                                    
        |             |      |                                    
        +-------------+  D   |                                    
                      |      |                                    
                      +------+                                    

We can verifiably prove that A, B, D and E can successfully complete
PoC challenges which involve hotspot X. Since we know that hotspot X is
bound to be trustworthy by design and any hotspot which can participate
in a PoC challenge involving X can only do so if it's operating within
the rules set by the consensus mechanism and also has an operating radio
chip.

However, we still cannot make logical conclusions about C and F since
they have not had any PoC activity with X. Therefore both C and F remain
in probation mode but A, B, D and E can transition to full PoC activity
once they all successfully complete `N` PoC challenges successfully
involving hotspot X.

## High-Level Workflow

All the below actions would happen behind chain var `diy` (or
something). Setting `diy` to true, triggers all of the below:

Another chain var, `probation_height_limit`, set to 2000 or whatever will
be set to allow witness collection for hotspots in probation mode.

Yet another chain var, `probabtion_required_pocs`, set to 100 or
whatever will be set to count down whether a probationary hotspot has
completed enough successful pocs with non-probationary hotspots.

- Add `probation` flag to gateway entries in the ledger. This would help
  us identify which hotspots have full/limited PoC priviledges.

- Add `added_at_block_height` to gateway entries in the ledger. This
  would help identify how long to keep collecting witnesses for hotspots
  in probation mode. Also helpful in general. Not quite sure what to set
  this for existing hotspots when upgrading the ledger gateway entries.

- Add `count_golden_pocs` to gateway entries in the ledger. This is the
  count of pocs which a probationary hotspot maintains every time it
  successfully and verifiably transmits a receipt with an active hotspot.

- Modify existing ledger gateway entries to have `probation` flag set to
  false.

- When a new hotspot is added to the network, `probation` is set to
  true.

- Hotspots with `probation` set to true are prohibited from creating
  `poc_request` transactions.

- Hotspots with `probation` set to true are allowed to be witnesses,
  however, they do not earn any tokens for being witnesses. This allows
  such hotspots to potentially witness helium manufactured hotspots
  opening chances of eventually getting pathed through those.

- At each `poc_receipt`, after all the existing validity checks, if a
  probationary hotspot successfully transmitted and received packets
  from a non-probationary hotspot, the probationary hotspot gains +1
  `golden_poc` count in its ledger entry. Furthermore, as soon as the
  total `golden_poc` count for the probationary hotspot reaches
  `probabtion_required_pocs` count, it transitions to `active` mode by
  setting the `probation` mode to false.


# Drawbacks
[drawbacks]: #drawbacks

TBD

# Rationale and Alternatives
[alternatives]: #rationale-and-alternatives

- Why the limitation for PoC-ing with helium hotspots?

The helium hotspot solves the problem of seeding trust in a new network
by serving as a "trust authority". Furthermore, this authority can be
delegated to other 3rd party manufacturers. 

# Unresolved Questions
[unresolved]: #unresolved-questions

TBD

# Deployment Impact
[deployment-impact]: #deployment-impact

TBD

# Success Metrics [success-metrics]: #success-metrics

TBD