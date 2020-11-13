# :books: LSINF2345 Project: The Peer Sampling service

**Autumn, 2020** -- *Raziel Carvajal-Gómez and Peter Van Roy*

The main objective in this project is the implementation and evaluation of a gossip-based algorithm using the message passing model in Erlang. Concretely, in this project you will:

- Deploy a Peer-to-Peer (P2P) network of Erlang process;
- Implement and deploy the gossip-based Peer Sampling (PS) service;
- Analyze and report the convergence behavior of this service.

## Context

The PS service is a decentralized gossip-based algorithm. In a nutshell, this service delivers an unbiased-random subset of peers from the whole network, nodes might gossip with. This is done via a periodic exchanges of neighboring peers between two nodes.

:bulb: **Recall.** The term *peer* as well as *node* will be used indistinctly in the rest of this text to represent one participant in a P2P network.

Several distributed algorithms rely on the PS service, it is also relatively easy to conceive other services thanks to this gossip algorithm. For instance, a decentralized social network may rely on the PS service to discover users (peers) following similar interest to form social groups. To do so, users in this social network fetch periodically some other peers from its local instance of the PS service and they keep only those that follows the topics the social group discuss about. Figure 1 depicts three iterations of the process that finds social groups interested in similar topics, where circles represent peers and colors represent topics.

<!-- [](images/) -->

Initially, peer A has some neighboring nodes however any of these neighbors are colored in red. Hence, the *red* social group contains A as its only member. After some iterations, the PS service delivers a different set of A's neighbors with two of them colored in red. Finally as depicted in Figure X, A (peer) has discovered new nodes to grow the social group of red peers.

:bulb: **Notice** that an iteration of the PS service results in changing the underlying topology of the whole social network. The network topology is also referred as an *overlay*, which represents an undirected graph where peers are vertex and edges are defined with the neighboring relation; i. e., if peer X has peer Y in its list of neighbors, then there is an undirected link between X and Y in the overlay.

## Your mission

In this project we ask you to form a team two students in order to complete with the following tasks.

1. Deploy a network of connected peers;
1. Provide an implementation of the PS service;
1. Deploy an scenario that allows you to analyze the convergence behavior of the PS service;
1. Write down your conclusions in a brief report.

:bulb: **Relax.** You will find more details about the previous requirements in this section.

It is **mandatory to use Erlang** to complete the previous tasks of development, this will allow you to instantiate a lightweight peer as an Erlang process in order to have a network of several peers on your laptop.

It is also mandatory to **deliver your project as a GitHub repository.** It is strongly recommended to fork the current repository, **your forked copy should be a private repository**, add the sources of your implementation in your local copy and deliver the report as a Markdown file. This will allow you to work in a more effective manner with your team mate as well as pulling the solutions to the exercises for learning Erlang, you will see (or have seen already) in the practical sessions.

### Network deployment

The PS service requires a connected network to bootstrap. We say that a network is connected when there exist at least a path between any two pair of nodes. A simple way to form a network with such characteristics is relaying on two well known data structures: a double-linked list or a binary tree. Figure X depicts those data structures.

:bulb: **Notice** that in the case of the list, every node have at most two neighboring peers, and at most three neighbors in the case of a balanced binary tree.

To deploy the PS service it will be enough to **use one of both data structures** as connected network to bootstrap the service. **We strongly encourage you to follow these steps** to deploy such network.

1. Once you chose either the double-linked list or the binary tree, provide an implementation of this data structure in a server process.
1. This server process should expose an API with at least two requests:
  - *add(Node)* adds a node to the data structure;
  - *getNeighbors(Node)* gets the list of neighboring peers of a node.

1. Implement a sequential procedure (in bash or any other programming language) that instantiates an Erlang process (peer) and invokes the call *add(Node)*.
  - :bulb: **Notice** that this sequential procedure should wait until the last peer has been added to the data structure.

As simple as that, you have a first connected network to initiate the PS service in every peer.

- :bulb: **Recall** that every peer in the network should have a first set of neighboring peers, also known as *peers view*, to start the active thread (infinite loop) of the PS service. You will get such view invoking the call *getNeighbors(Node)*.

### PS service implementation

You will find the implementation of the PS service in Jelasity M. et al., recall that in a previous practical sessions you were asked to read that article (journal version). If you haven't done so, read until the second section (where you will find the implementation) and we also recommend you to read the sections 4 and subsection 4.1.

Your implementation should be parametrized to deploy the service in any combination of the following policies:

- **Peer selection.** The neighbor to gossip with will be chosen at random among all peers in the view (the so call *rand* selection) or the chosen neighbor will be the one with the highest age (*tail* selection).
- **View propagation.** Either every peer sends its local view (*push* propagation) or exchanges its view with the one of its neighbor (*push-pull* propagation).
- **View selection.** When an exchange of views between two peers takes place, the resulted view will contains the most recent entries (*healer* selection), redundancy of descriptors (*swapper* selection) or a mixed selection (keeps some recent descriptors with a certain redundancy).
  - :bulb: **Notice** that this selection is tweaked based in the self-healing parameter *H* or the swapper parameter *S*.

Having a parametrized PS service is essential to deploy and evaluate the convergence behavior of it. One of the main advantages of using Erlang is that your final implementation will look very similar to the pseudo-code shown in Jelasity M et al, allowing your to have a first validation by meerly comparing ehther the operations in your implementation take place in the same order as shown in the pseudo-code. The correctness of your implementation will be assessed within several scenarios, discussed in the next section.

### Experimental scenario

The recommended number of peers in the network should be at most 128. You will assess your implementation in an elastic scenario that lets peers join the network, then it makes certain nodes to crash. This scenario also contains a final phase where nodes recover and create an instance of the PS service for a second time. Your deployment should follow this time line.

1. **Bootstrapping phase.** Initially, only 40% of the peers start the service using the view from the first connected network (with a topology of a double-linked list or binary tree).
1. **Growing the PS overlay.** At a pace of every 30 cycles, 20% of the remaining peers will join the network until all nodes (100%) are part of this network.
1. **Time to crash.** When the lifetime of the PS service have reached 120 cycles, 60% of the peers will crash. To mimic a more realistic scenario, that amount of peers should be chosen in a random way.
1. **Back to business (recovery phase).** After 30 cycles more, you will chose one alive peer P from the network. To test how the PS service behaves during a phase of recovery, the 60% of faulty nodes will join again the network with one condition: every node will have a list that contains only the peer P.

This elastic scenario should be deployed twice using the push-pull policy in the propagation of the view. In the first deployment you should use the healer policy and the swapper policy in the second deployment, to select descriptors in the peers' view.

### Convergence behavior of the PS service

<!-- TODO -->

## :pencil: Grading
