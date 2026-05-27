# Freeing Aerospike CE from Its Limits

## Introduction

Most engineers working with high-load systems have likely heard of [Aerospike](https://aerospike.com).

For those who still don't know what it is, here’s a short excerpt from the documentation:

> The high-performance, distributed, 
> NoSQL database designed for real-time applications requiring low latency and high availability.

Basically, it’s a commercial product and it costs a ton of money. The price is so high that our small company simply can't afford it.

However, Aerospike is distributed both as a commercial build and a community edition, which, although limited, was quite usable for production in older versions.

Back then (about 10 years ago), our company adopted this DBMS and used it successfully.

But as the years passed, with each new version, the free edition’s limits have become stricter and stricter. 
Eventually, the restrictions became so severe that we couldn’t update anymore and got stuck somewhere around version 5 (the current release is version 8).

As a result, we’re still running very old versions of Aerospike CE because, since v5, Aerospike has introduced limits that make it impossible to use effectively in production environments.

Here are the restrictions I'm talking about:
- cluster size limit — 8 nodes;
- no more than 2 namespaces per cluster;
- storage limit per node for unique data (don’t remember the exact figure for older versions, but for newer ones it’s just 640 GB).

All the above prevents horizontal scaling and, consequently, creates problems for us and for development. The 640 GB storage limit per node, combined with the 8-node cluster cap, is precisely the problem that makes it impossible to use such a cluster in production.  
Also, since we’re stuck on older Aerospike versions, we face cluster support issues that only get worse every year.

In the past, we tried to purchase the commercial version, but it turns out their pricing for our use case is astronomical.  

Thus, we’re blocked on outdated software, and our service codebase is so tied to Aerospike that migrating to an alternative would basically mean "never".

This post proposes a solution to this problem.

## Solution

So, Aerospike is commercial software, but, guess what — the community edition source code is public.  
And it’s not just any open-source license — it's under AGPLv3.  
[Source here.](https://github.com/aerospike/aerospike-server/tree/master)

This definitely gives us hope that we can tweak things for the better.

Here’s what I discovered digging into the source:

1. The sources are under AGPLv3 (excluding a few files as mentioned in the LICENSE file).
2. There’s no complex logic preventing you from removing the limits.
3. All the limits are deliberately enforced in the code via C preprocessor macros,  
   and the files in question are under AGPLv3.

Naturally, I couldn’t ignore this and went ahead to increase several limits, after which I built my own Aerospike binary:

- cluster node limit is now 64;
- namespace limit per cluster is now 32;
- storage limit — infinity.

## The Result

Actually, modifying the code is quite straightforward — just a few sed-commands.  
So I wrote a Dockerfile and published it [here](https://github.com/vkhodor/aerospike-unchained).

I built and tested the version and confirmed the limits have indeed been lifted. 
I also ran performance tests to make sure everything still works fine, that the app doesn’t core dump or slow down.

> **Important!** I haven’t yet tested this build in a real production cluster.  
> This is still on my to-do list.
> So, you’ll have to try this path on your own.
