# Providing node incentive

The purpose of this document is to provide a non technical overview of implementing an incentive system for Nano node representatives. The core premise being that node operators would set a startup flag to provide verification and store alternative genisis forks of the Nano blockchain. These forks would be provided with: 

* A Guid identifier for their fork
* The ability to upload the base fork database including the initial wallet.
* Some level of wallet integration to allow users to easily receive and transact the newly minted fork. 
* A non unique fork name

The metadata for these forks should be stored on a new side chain as an optional flag for node operators. If they chose to participate in this program they will keep this data in sync and automatically become part of the network pending a full online status to receive rewards. The only input needed from the node operator is a Nano wallet address to deposit coins collected for hosting costs. 

The forks will need to be funded by the creator of the fork sending Nano directly to the live node representatives. 

The clients fork will undergo the same upgrades as the core Nano currency and remain instant and free for transactions.

This document is missing key portions of implementation, revolving around voting systems and distribution percentages. Any specific references to times/percentages or api specifications are only given as an example. 

## Why not just use Nano? 

Sometimes for an institution it is beneficial for there to be no price fluctuation for their issued currency, and for them to have absolute control over the supply and issuance IE: 

* gift card providers
* internally issued corporate tokens
* Financial education tools for parents and schools
* Ephemeral currency for events
* Pre-sales of products or services

The Nano network needs the boost also, without a sharding solution low power nodes could potentially go offline or fall behind on the blockchain. By promoting uptime and baking in constant stress testing through the Proof of storage checks from the network. You ensure that the backbone of the network is properly configured on hosted nodes that meet network demand. The raspberry pi nodes and other low cost VPS spun up and unmaintained would not achieve the uptime statistics nor the network load needed to be included in the payout pools. 

They would likely not participate in the secondary chain and be eliminated from it's consensus models. 

- [Server side](#server-side)
    + [The control system](#the-control-system)
    + [Optimal infrastructure](#optimal-infrastructure)
    + [Rewards for Hosts](#rewards-for-hosts)
    + [Chain restoration](#chain-restoration)
- [Client side](#client-side)
    + [Chain issuance](#chain-issuance)
      - [Payment services](#payment-services)
    + [Wallets](#wallets)

## Server side

The generation of these forked accounts should use the exact same methods used for Nano outside of wrapping them in a unique identifier. The storage of the chain consists of three clear filetypes. 

* Forked chain storage - The forked chains should store their database files in the exact same format as Nano core. These files live inside of a concatted secondary chain master named after their chainid. This ensures that all the routines currently used by the Nano node would stay nearly identical to the core. 

* Reference database storage- This file is a simple key value pair with the chainid and the metadata with a JSON string containing: 

```
{
    "name":"<Long Form Name>",
    "ticker_name":"<Short form name>",
    "funding":"[<array of wallets funds will come from>]"
}
```
The chains name and ticker_name should not be unique fields, What is distributed to users is a guid id. If people want to pay to attempt to duplicate a coins name, let them to curb name squatting. When a coin issuer sends data to the client to load the wallet it will be in the form of this guid preferably using a QR code. 

* The sidechain node network data- This is a classical time based blockchain, with a farmed block containing the current approve host list as a result of the logic used by the participating nodes to determine uptime containing: 

```
{

    "hostid":"<node rep guid>",
    "wallet":"<destination address for funds>",
    "uptime":"<Unix timestamp>"
}
```
This sidechain will be pruned in a sane manner as older than X historical data is not needed for operations. 

To access a side chain the requests for operations should use the existing rai protocol with an additional flag embedded in the messages containing the side chains guid. 

If the server is capable it will redirect the operations to the appropriate chain while nodes not participating will simply return an error. 

#### The control system

The api endpoints for fork generation should be as developer friendly as possible and accept a basic http post containing: 

```
{

    "name":"<Long Form Name>",
    "ticker_name":"<Short form name>",
    "funding":"[<array of wallets funds will come from>]",
    "blockchain":"BASE64"

}
```

The blockchain being the file blob that makes up the genesis chain the client used to define the supply in the wallet and the private key to access it. 

You can use any representative participating in the network to make this request and they will broadcast it to the network. But they will only do so when you pay their fee minimum which can be no lower than a pre-determined rate. This is to mitigate reflection attacks on the network. 

The return from the server contains a blockchainid and a a list of hosts to pay with amounts for 1 hour of service for a two hour floating window. Multiple hours can be purchased but then you risk paying potentially future off line or out of network hosts. The single hour at the tail of the contract funding is there to provide a warning threshold to be able to ping in and detect from any live node representative.

Proof of storage is split into 2 distinct checks: 

* Frequent uptime checks - This will consistently check the last 1000 transaction blocks across all chains by requesting a random block. 
* Less frequent historical checks - Check for a random block on the full blockchain spectrum. 

You need separate checks to avoid users from spinning up pruned nodes or nodes that only have a historical majority to roll the dice on node uptime checks.

Once a server is in the 72 hour fold it gains an important ability to now request the network delist and pull another node in the network out of their uptime window. This ability can only be used once for a set period of time with the list of potential targets coming from the current live host list on a rotating time based target list for that particular host. 

The reason for the rotating window of hosts for ping checks is to alleviate hosts from being singled out for attack. A node requesting a check of another node outside of its approved targets for the time period will be ignored by the network.
A system where you promote node operators to perform as many checks as possible on node peers ensures that operators will institute sane rate limiting on incoming requests and basic security measures to stay online. Someone spamming requests would be blocked and unable to reliably determine if the other nodes will actually fail the test.
Failed checks from the network should not be discouraged as you want people to request their proof of storage checks even if only based on a random host and block to keep the system stress tested. 

From the node requesting the delist they will send: 

* The node rep ID
* The failed block and chainid (blank for Nano Chain) 

 The delist request is broadcast to the whole network (in a sane sharded model) of hosts, if any of the hosts fail the proof of storage, they propagate the request with an also time based ability. If less than a % of the network comes back from these secondary requests the node will be delisted and shifted into the pool waiting to prove themselves. Part of this process is the server referencing their own local storage to check for the block on that particular chain and compare the output of the remote server. 

Under this model you reward a node that can block malicious attempts while providing legitimate users an endpoint that will return a valid proof of storage. 

#### Optimal infrastructure

This document has referred to nodes as individual computers, in reality to operate a functioning always online rep will require using clusters of hosts and storage with some kind of Highly available front end load balancing combined with DDOS protection. 

None of these security measures will be provided by the base software and will be up to the node operator to implement to avoid being delisted. 

At a minimum to survive in an active payment network you would need: 

* Multiple hot systems in distributed data center regions
* A shared storage pool between the systems
* A frontend CDN provider that could handle null routing DDOS attacks
* Basic monitoring system with an automated or human run NOC
* A growing storage model to handle the side chains and meta data

The rewards for running a node should reflect these types of costs, as the network grows.

#### Rewards for Hosts

Rewards for hosting a trusted representative should follow a free market model and essentially only cover the cost of running a node plus a small profit. If there are too many online nodes to make the system worthwhile to operators, then nodes should drop out. 

The system should avoid staking models to curb speculation on Nano.

This system should operate based on a pre-set distribution formula with a variable plugged in regarding hosting costs from a centralized server. This centralized server should be a community effort that acts like the federal reserve setting interest rates. The community will vote on modifying this number to support the network. Any initial implementation of this endpoint should be a static string as voting and community consensus is a complex issue that can set back development for some time.

The community will be based on a % split of voting power between the live nodes and live coin creator accounts. Votes will come in the form of small transactions and voting will occur on a time based classical blockchain. The tiny amounts of Nano will be placed in the communities wallet to support the effort for every voting period. 

The hosts will receive no rewards for hosting the main Nano chain, but it will be a core requirement for the representatives to participate, as they will need to verify transactions during proof of storage checks.

When a payment account goes unfunded for a set period of time it will be permanently removed from the secondary chain by the secondary chain representatives that have not received funding. In a partially paid deployment the user risks their clients encountering errors when trying to add a block to the chain. The nodes will also blacklist this chain from POS checks if any host on the network is not funded. 

The responsibility of funding that account and backing up their fork rests solely on the client or community efforts. 

In order to be part of the rewards pool your server must be up and running for a period of 72 hours and have 0% Proof of Storage Failures. If at any time during your operational window you fail another proof of storage the server will be removed until it achieves this status once again starting from 0 hours. 

Rewards distribution is calculated by uptime rank, with no more than a 100% bonus to funds received for the #1 spot.

Individual payments from a coin creator to a node representative will be a tiny amount of Nano and hosts leaving the live list before their hour of service should not be a large financial burden on the user issuing the coin. 

How much the coins chain owes to keep their chain funded will be a direct calculation related to the size of their fork within a bound set of growth rules that prevent rapid price increases/decreases. If the user pays for multiple hours their hours will be cut from service during the next recalculation if the data grows beyond the current pricing tier. This will still be constrained by the 2 hour window to allow coin issuers with a reasonable time to pay money owed for hosting. 

#### Chain restoration

If you have a backup of your chain and want to repost it under the same id, the process should be similar to the chain creation service with a pre-payment requirement based on data size if it breaks the new chain byte limit. Once funds have been payed to the network of hosts they will accept a chain of that size and automatically propagate the file in a distributed manner to hosts that have also been paid.

This can be used as an avenue of attack, but only against people that let their chain die by not paying hosting bills. The network should have no mercy during these situations. 


## Client side 

On the client side you have two end user experiences to consider. 

#### Chain issuance 

All efforts should be made to provide example scripts/libraries in multiple programming languages to help facilitate self hosting a wallet containing Nano to make these payments. 
These tools provided will also need to include a method for generating a new genesis wallet and defining the amount of currency issued. 

##### Payment services

Because of the need to keep sending Nano to a fluctuating list of hosts for fluctuating amounts, 3rd party payment services should be a relative norm. These services taking a small percentage of a funded wallet to send the transactions out in a timely manner. 

Savvy users would use multiple funding sources to cover for an operator outage. You can always pay more, but never less. 

Users will not depend on these payment services though, if they want to host their own system that can manage it using an API response from current reps that can be requested at any time for the chainid. 

Optimally these payment services will pay as little as possible to keep the chain online. 

#### Wallets 

The wallets should use existing representative rules for interacting with the Nano network. 
The sidechain will follow a specific set of rules. 

* The client will initially pull the list of live nodes
* The client will use a random node array from this list

This is impossible to police as a developer could just hard code a rep for the secondary chain data, but ignoring these rules could bring the integrated wallet application offline on host rotations, as hosts marked in the off-line pool are not allowed to broadcast new blocks to the network of side chains. 

The client experience in the wallets should be as seemless as possible with special QR codes used to send a user the new chain and let them generate a new wallet for it. 


















