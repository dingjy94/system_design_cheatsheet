# system_design_cheatsheet
This is my system design interview cheatsheet.

## Basic Steps
1. Clarify requirements
   Some comman questions:
   - which part to focus on
   - who is the users
   - how many function need to implement (such as for wechat, do we need group chat...)
   - file types
   - need push notifications?
   - permenant storage? log, history...
2. (optioal) Design interface (API)
3. Consider restriction and target
   - what scale
   - how much storage
   - network bandwidth usage (latency)
4. Decide data flow
   - identify different entities (can also design sample schema for each entity)
   - How each parts interact with each other
   - data storage and manangement
5. High level design
6. Detailed design
   - dig deeper in two or three components (follow interviewer's feedback)
   - different approach, explain tradeoff
   - corner case
   - security
7. Find and resolve bottlenecks
   - avoid single point failure
   - replicas data and service
   - monitor performance and bugs

## Examples
### Typehead suggestions (auto complete)
1. Clarify requirements
   how many terms? (top 10 maybe)
   spell check? language? case sensetive?
   personal suggestions?

2. Restriction and target
   real time, 200ms latency
   we can assume that we need to store 100 million terms, each terms on average 30bytes, which means we need 3GB. This will increase 2% day by day, so 25 GB for a year.

   Since the latency requirement and the increase storage size, we need to make the system scalable.

3. Data flow
   Basiclly two flow: search and data update flow.
   - search flow: Store terms in tries. The basic flow: user - load balancer - server - redis (if hit return) - Trie database, for trie databse, we need do data sharding
   - data update flow: data source (logs...) - data process (Map reduce maybe) - result (phrase-weight pair) - trie. This process is done offline and for update, we can use master-slave database.

4. High level
   user - load balancer - server (distribute) - redis(distribute) - zookeeper - trie(distribute)

5. Detail
   - For the trie, for each node we store the top k (in this case 10) terms.
   - For the data sharding (partition), we can store tries based on the capacity of the server, or hash the trie and store it to corrsponding server.
   - To caculate the weight, consider the time and other factors.
   - Also when update trie, can push trending terms directely to high ranking position.
   - The server can also return some terms for popular prefix, such as for 'g', also return the top 10 of 'ga, gb, gc', and store it on client side.
   - CDN to store the result.
   - Client use local storage to cache. Set a delay (like 50ms) to wait for the typing.

6. Bottleneck
   All server, database are distributed, multiple replicas.
