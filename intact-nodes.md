# From well-behaved to intact and correct nodes. Road to understanding the Stellar Consensus Protocol (SCP)  

Nodes in the Stellar Network follow the Stellar consensus protocol to reach agreement on the contents of their ledgers. 
Roughly speaking, they want safety: every node has the same copy of the ledger. And they want liveness: their ledgers should be updated in a timely manner to reflect the latest transactions.

However, the Stellar consensus protocol (SCP) can only guarantee safety and liveness to nodes if they are intact. 

In this blogpost we will dive deeper into the concept of intact nodes and how a well-behaved node can achieve safety and liveness. 

| <img src="https://github.com/user-attachments/assets/039f7bf7-5e26-441e-8972-bf3eeaa9b620" alt="well-behaved-to-correct" width="50%"/>| 
|:--:| 
| *From well-behaved to correct nodes* |

An understanding of FBAS, quorums, slices,... will help. Short descriptions will be given, but you can find a deep dive here: https://medium.com/stellarbeatio/LINK
If you want more math and precise definitions, you can check out the SCP whitepaper, section 3 and 4.

## Local behaviour of a node

When looking at the direct/local behaviour of a node we can observe the following: 
* Does it follow the consensus protocol?
* Is it configured with sensible quorum slices? 
* Does it (eventually) respond to all requests sent to it? 

If yes, then we call the node well-behaved. 

An ill-behaved node on the other hand can display random, byzantine behaviour and doesn't follow the protocol. 
This could be with malicious intent to try and take advantage of the system, or it could just simply have crashed. 

The local node behaviour is mostly under the control of the node operator. 
Keep your node up-to-date, provide a sensible quorum slice configuration, ensure good network access, setup monitoring,...

But being well-behaved is not enough. Nodes do not operate in isolation and the nodes they trust upon could cause them to fail.

## Safety and liveness in an FBAS

The nodes in the Stellar Network form a federated byzantine agreement system (FBAS). 
They choose themselves the nodes they trust through quorum slices to reach consensus on the ledger.

Safety and liveness are defined as follows in the SCP whitepaper: 

* safety: A set of nodes in an FBAS enjoy safety if no two of them ever externalize different values for the same slot.
Nodes that lack safety could externalize different values and 'diverge'.

* liveness: A node in an FBAS enjoys liveness if it can externalize new values without the participation of any failed nodes. 
Nodes that are well behaved, but lack liveness, are called 'blocked'.

Translated to the Stellar network: A slot is a block in the ledger, and externalize means adding a new set of transactions to the ledger. 

When a node has both safety and liveness, we call the node 'correct'. If not, it has failed.

| <img src="https://github.com/user-attachments/assets/6e8c550d-7446-418d-a8bf-d5f32dd58afe" alt="node failures" width="50%"> | 
|:--:| 
| *Fig. 5 Node failures SCP whitepaper* |

Now we come to the main question: How do go from a well-behaved node, to a correct node?

## Requirements for a Correct node they
 
Unlike being well- or ill-behaved, correctness is not a local property of a node. It depends on the system as a whole and the behaviour of other nodes. 

What do we need for a node to be correct?

1) The node should be well-behaved. A malicious or crashed node will never be classified as correct. 
2) We need a sound consensus protocol, like the trusty SCP used by Stellar.
3) We assume a (physical) network that eventually delivers all messages between well-behaved nodes, even though it can reorder or arbitrarely delay them. 
4) Robust quorum slices that are not tainted by too many ill-behaved nodes.

In this blogpost we will focus on the robustness of quorum slices. Starting with a short recap of quorum slices and quorums.

## From quorum slices to Quorums 

In an FBAS, every node chooses its own quorum slices. It chooses sets of nodes it trusts. 
A node is part of every one of its quorum slices and, rhoughly speaking, can be convinced of a statement if everyone in the quorum slice agrees. 

From the combinations of the quorum slices of the nodes in the network, quorums emerge. A quorum contains for every node it encompasses, one of its quorum slices. 
If a quorum slice can convince a single node, a quorum can convince every node in the quorum. 
Another way to look at it is that a statement needs a 100% approval rate from a quorum to be agreed upon. 
Because nodes choose multiple slices to trust, multiple quorums can emerge in an FBAS.

Note: If you lack intuition of quorums and slices, here are some worked out examples (TODO link).

## Quorum slice robustness: safety

The first issue with a node's slice selection could be lack of quorum intersection. Meaning that the quorums that are formed do not overlap.   
No quorum intersection means it is impossible to guarantuee safety. Remember we want a correct node that has both safety and liveness.

Lack of quorum intersection implies that two sets of nodes could decide upon different values and fork the FBAS.
If all the quorums overlap in at least one node, and armed with the knowledge that quorums decide unanimously (100% approval rate), forks would be impossible.
All the nodes in all the quorums, and by extension the entire FBAS, would decide on the same value.

But what if the overlapping nodes are malicious? They could lie to the overlapping quorums and cause them to diverge. 
Here we have the first dimension of quorum slice robustness. Choose your quorum slices in a way that the quorums you are a part of overlap in multiple nodes.

## Quorum slice robustness: liveness

Another issue with slice selection could crop up. There could be a node that is present in every single quorum. Meaning that if this node crashes,
no quorum could reach a 100% approval rate and your node would be stuck. When this happens we say that the 'Quorum availability' property is violated, 
which is crucial to ensure liveness for the node.

This is the second dimension of quorum slice robustness. Choose your quorum slices in a way that multiple node need to fail before there are no more quorums left for your node.  

Note: I want to emphasize that quorum intersection and quorum availability do not guarantee liveness and safety. It is the actual consensus protocol that guaruantees this.
But withouth these two properties no consensus protocol in an FBAS could. 

Now that we have an intuition about what robust quorum slices are, 'how' do we actually make an informed decission to select these slices?
To answer this question we will now introduce dispensable sets and the concept of intact nodes. 

## Dispensable sets and intact nodes

A dispensable set (DSET) is a set of nodes in an FBAS, that is, well... dispensable. 
The other nodes in an FBAS can function correctly without (or despite) the behaviour or presence of these dispensable nodes. 
Dispensable sets tell us which nodes _can_ fail. This is important information, we can for example ask the following questions:
- Does every individual node form a DSET onto themselves? Meaning no single node failure could harm the FBAS.
- Does every combination of two nodes form a DSET? Meaning are there combinations of two nodes that can fail without harming the rest?
- What are the weak spots in our FBAS? What nodes are more vulnerable then the others?

Dispensable sets can be found by examining the quorum slices and quorums that emerge from them. 
For safety, the FBAS should keep the quorum intersection property after deleting the nodes in a DSET from the FBAS and every one of its quorum slices. 
For liveness A DSET cannot hamper quorum availability. Meaning there should be quorums that do not contain these nodes. 
And finally there is the special case DSET that equals all the nodes in the FBAS. 

Now imagine we know what the ill-behaved nodes are in an FBAS with quorum intersection. Imagine, because it is impossible beforehand to know node behaviour.
We can now determine all the DSETS that contain all the ill-behaved nodes. Because we want to know if we can 'slice' them from our FBAS and still have functional nodes left.
It's not a given there exists a DSET that solely exists out of ill-behaved nodes, meaning that some well-behaved nodes could be affected. Even more so in the special case 
where the only DSET that contains all the ill-behaved nodes is the entire FBAS, meaning no nodes would survive.

To define if a well-behaved node will be affected we introduce the notion of intact nodes:
A node v is called 'intact', if there exists a DSET containing all the ill-behaved nodes, but does not contain v. 
Translated, we have found a set of dispensable nodes, that contain all the ill-behaved nodes but not v, that we can 'slice' from the FBAS without harming quorum intersection
and quorum availability. Node v will remain intact.   
If we can't find such a DSET, we call node v befouled. Note that the ill-behaved nodes are also called befouled.

This is the final step for a well-behaved node to be classified as 'correct' because a sound consensus protocol like SCP can garuantee safety and livenes for intact nodes.

Note: Stellarbeat uses a different approach to give insights into FBAS robustness. Separate safety and liveness buffers are determined through blocking and splitting set analysis.
(See FBAS blogpost). But to understand SCP, a good understanding of intact sets is helpfull.

To drive home the concept of intact sets, we will end this blogpost by deeply examining the intact nodes example from the SCP whitepaper.

## Dispensable set analysis example

Let's start by listing the quorum slices of the top tier. 
Q(1) = {{1,2,3},{1,2,4}, {1,3,4}} 
Q(2) = {{1,2,3},{1,2,4}, {2,3,4}}
Q(3) = {{1,2,3},{1,3,4}, {2,3,4}} 
Q(4) = {{1,3,4},{1,2,4}, {2,3,4}}

The quorums that can convince the top tier are:
{1,2,3}, {1,2,4}, {1,3,4}, {2,3,4}
All quorums overlap. 

Every single node is a DSET. 
For example the DSET {1}.
After deleting the node from the FBAS and every quorum slice we get: 
Q(2) = {{2,3}, {2,4}, {2,3,4}}
Q(3) = {{2,3}, {3,4}, {2,3,4}}
Q(4) = {{3,4}, {2,4}, {2,3,4}}
The quorums are:
{2,3}, {2,4}, {3,4} and {2,3,4}
There is quorum availability in the FBAS and the quorums overlap, which confirms that {1} is a DSET if the FBAS consisted out of top tier nodes only.

If look at the second tier, they only require two of the top tier nodes (unlike the top tier itself that requires three). Therefore we can conclude that for 
the second tier, {1} is dispensable.

For example:
Q(5) = {{5,1,2}, {5,1,3}, {5,1,4}, {5,2,3}, {5,2,4}, {5,3,4}}
After deletion:
Q(5) = {{5,2}, {5,3}, {5,4}, {5,2,3}, {5,2,4}, {5,3,4}}
The quorums that can convince node 5 are:
{5,2,3}, {5,3,4}, {5,2,4} and {5,2,3,4}
These quorums overlap, and overlap with the quorums of the top tier, thus the FBAS will remain in sync. Again {1} is dispensable.

The third tier only trusts node 1 transitively, thus because they depend on the second tier, and this tier is unafected by node 1, they will also be unaffected.
Feel free to write it out.

If we follow the same reasoning and write it out, we fill find that the nodes in the second tier are also DSETs onto themselves. 
Nodes in the in the bottom tier are also DSETS, because nobody depends upon them. They can be safely deleted. 

This FBAS can can tolerate failures of every individual node!

The top tier cannot tolerate failures of two top tier nodes, the quorums would no longer overlap. And because all other tiers are dependent on the top tier (transitively),
the entire FBAS would fail. Another way of saying this, is that the only DSET containing two of the top tier nodes is the entire FBAS. No intact nodes remain.

Is {6,7,8,9,10} a DSET?
Yes! the nodes 1 through 5 do not depend upon them and can keep functioning. 

Is {7,8,9,10} a DSET?
Yes! by the same logic as above.

Is {5,6} a DSET?
The answer is actually no. Deleting these nodes will cause problems for the bottom tier nodes. 
For example node 9. We take a look at its slices:

Q(9) = {{9,5,6},{9,5,7},{9,5,8},{9,6,7},{9,6,8}}
And after deletion of {5,6}:
Q(9) = {{9},{9,7},{9,8},{9,7},{9,8}}

The slice {9} will cause problems, because the node can be convinced in isolation. 
{9} is a quorum and does not overlap with the others.

But if we know that 5 and 6 are malicious, what are the DSETS that contain them? 
{5,6,9,10}, {5,6,7,9,10}, ..., {5,6,7,8,9,10},...,{1,2,3,4,5,6,7,8,9,10}
Node 9 and 10 are befouled, because there is no DSET possible in the FBAS that contains all the ill behaved nodes, but node 9 and 10.
Node 7 for example is not befouled, because there exists a DSET containing all ill-behaved nodes (for example {5,6,9,10}) that does not contain node 7.

If {5,6,9,10} is a DSET, and only node 5 is malicious. Does this mean that node 6 is also befouled?
No, Because there exists a DSET, namely {5}, that can be safely deleted, and does not contain node 6. 

Theorem 2 of the SCP whitepaper states that the intersection of two DSETS is also a DSET. Let's confirm this in our example.
{5,6,9,10} and {7,8,9,10} are DSET's. Their intersection is {9,10}. Is this also a DSET? 
Yes, they can be sliced from the FBAS, reflecting the fact that no other nodes depend upon the bottom tier, and thus it can be entirely malicious without impacting the well behaved nodes. 

## Conclusion
I hope you enjoyed and understood this explainer of intact nodes, dispensable sets,... Armed with these concepts you can go further in understanding the actual SCP,
Stellar consensus protocol. 

If you have feedback or spot any mistakes. Let me know on the Stellar dev discord or through pieterjan@stellarbeat.io
