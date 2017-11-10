This, and other exemplars for self-adaptive systems, can be found at 
[http://self-adaptive.org](https://www.hpi.uni-potsdam.de/giese/public/selfadapt/exemplars/)

# The Znn Self-Adaptive Exemplar

## Description 

Based on real sites like *cnn.com* and *RockyMountainNews.com*, Znn.com is a news service 
that serves multimedia news content to its customers. Architecturally, Znn.com is a web-based 
client-server system that conforms to an N-tier style. As illustrated in the figure below, 
Znn.com uses a load balancer to balance requests across a pool of replicated servers, the size 
of which is dynamically adjusted to balance server utilization against service response time. 
A set of client processes (represented by the C component) makes stateless content requests to 
one of the servers. Let us assume we can monitor the system for information such as server 
load and the bandwidth of server-client connections. Assume further that we can modify the 
system, for instance, to add more servers to the pool or to change the quality of the content. 
We want to add self-adaptation capabilities that will take advantage of the monitored system 
and adapt the system to fulfill Znn.com objectives.

![ZNN example](documents/img/znn-arch.png)

The business objectives at Znn.com are to serve news content to its customers within a 
reasonable response time range while keeping the cost of the server pool within its operating 
budget. From time to time, due to highly popular events, Znn.com experiences spikes in news 
requests that it cannot serve adequately, even at maximum pool size. To prevent unacceptable 
latencies, Znn.com opts to serve minimalist textual content during such peak times in lieu of 
providing its customers zero service. The Znn.com system administrators (sys-admins) adapt the 
system using two actions: adjust the server pool size or switch content mode. When the system 
comes under high load, the sys-admins may increase the server pool size until a cost-determined 
maximum is reached, at which point the sys-admin would switch the servers to serve textual 
content. If the system load drops, the sys-admin may switch the servers back to multimedia mode 
to make customers happy in combination with reducing the pool size to reduce operating cost.

The adaptation decision is determined by observations of overall average response time versus 
server load. Specifically, four adaptations are possible, and the choice depends not only on the 
conditions of the system, but also on business objectives:

1. Switch the server content mode from multimedia to textual
2. Switch the server content mode from textual to multimedia
3. Increment the server pool size, and 
4. Decrement the server pool size

We want to help Znn.com automate system management to adjust the server pool size vs. switch 
content between multimedia and textual modes. In reality, a news site like cnn.com already
supports some level of automated adaptation. However, automating decisions that trade off
multiple objectives to adapt a system is still unsupported in most systems today. For instance, 
while automating adaptations on performance concerns is possible (e.g., load balancing), it is 
much harder to do so for potentially conflicting qualities such as performance and security. This 
work is an important step in that direction: to allow automation of adaptations that must strike a 
balance between multiple objectives.

## Relevance: Slashdot Effect

The motivation for the Znn.com instantiation comes from an Internet phenomenon known as the 
*Slashdot* effect, where an otherwise low-traffic website, shortly after being featured on 
slashdot.org, gets inundated with visitors for a period of time, anywhere from a few hours to a 
couple days [Ter04]. Now a common term, it describes a similar phenomenon where a website experiences 
a sudden, unanticipated rise in requests due to a popular event (or sometimes, anticipated, but not to 
the scale observed), for example, breaking news or superbowl craze [Wik08d]. To illustrate a few 
instances, on the morning of August 17, 2006, a break in the JonBenet Ramsey case resulted in a local 
news website, rockymountainnews.com, being swamped by news readers. Browsing the site at 10:18 AM took
more than 2 minutes to retrieve a blank page with only the local weather showing in the corner. On 
September 4, 2006, the shocking death of Steve “Crocodile Hunter” Irwin caused the Australian 
Broadcasting Corp website to be temporarily shut down; similarly, numerous other news sites groaned to a 
halt. Of note is that Australian Broadcasting Corp’s site resumed a few hours later with a low-bandwidth 
format to cope with the high traffic [Tai06]. A more problematic, revenue-losing case arises when an 
e-commerce site is shut down by the Slashdot-like effect, as exemplified by Wal-Mart’s shopping site on 
Black Friday 2006, which remained inaccessible into the afternoon after the initial morning 
rush [San06, Sch06]. The Australian Broadcasting Corp illustrates the present coping mechanism of most 
website administrators to the Slashdot effect. When they first notice a sudden rise in visit requests, 
they make a decision to temporarily shut down the site for manual reconfiguration, where usually entails 
resorting to lower fidelity content. In extreme cases, the site might temporarily shut down or simply post 
a notice to “visit later.” There are a number of disadvantages with this scheme:

- The problem may not be discovered in time;
- The manual process means slower response time;
- The chosen solution may be suboptimal; and
- It may result in a potential loss of confidence, revenue, or future business.

An ideal approach would be to achieve site adaptation automatically, and our aim is to demonstrate such 
self-adaptation with the Znn.com instantiation. It is worth noting that a few modern systems with 
sophisticated infrastructures, such as Google Gmail, have equipped themselves with similar but limited 
adaptation capabilities, for example, to instruct its users to “reload in a few seconds” when it detects a 
sudden peak or other underlying connectivity issues to its servers.

## Adaptation Challenges

A typical news content service such as Znn.com needs to serve content to its customers within a reasonable 
response time range while keeping the cost of the server pool within a certain operating budget. In normal 
operating circumstances, the appropriate trade-offs can be made at design-time. However, from time to time, 
due to highly popular events, Znn.com experiences spikes in news requests that are not within the originally 
designed parameters. This means that it cannot serve the clients adequately (i.e., clients will not receive 
content in a timely manner). To the client, the site will appear to be down, and customers will go elsewhere, 
resulting in lost revenue.

The challenge for self-adaptation is to be able to still provide content at these peak times. There are several 
ways to deal with this, such as serving minimalist content, increasing the number of servers serving content, 
redirecting to alternative sites, choosing to prioritize serving clients to paying customers.

In this exemplar, there are currently three runtime parameters than can be adjusted to adapt the system 
dynamically:

- Performance
- Cost
- Content Fidelity
