---
title: "AI: MCP - Eeeeeek!"
date: "2025-03-14T15:00:01.123Z"
description: "Casting a critical eye over MCP for AI"
---

**The AI MCP Protocol: An Ambitious Idea - But this feels like VHS vs Betamax**

The AI Model Context Protocol (MCP) burst onto the scene in late 2024 as an Open Standard, introduced by Anthropic, suggesting a simple way for AI-powered apps—especially chatbots and assistants—to dynamically plug into external tools and data at runtime. Sounds great, right? But after some digging, it turns out MCP might not be as practical as it initially seems.

At first glance, MCP looks intriguing: it lets AI clients dynamically discover and integrate external tools and services, creating an ecosystem where your AI app can easily connect to databases, weather services, or even your smart home gadgets without pre-baked integrations. But once you start digging deeper, cracks begin to show.

Basically MCP defines a JSON-RPC based client-server framework, where AI Clients (currently only Claude Desktop, or Cursor 🙄) can discover and invoke functions (called tools), access resources (text files, documentation, etc...), or use preset prompts provided by an 'external' server component.   The supposed goal is to let users extend an AI application's capabilities at run-time, rather than design/build-time, by allowing live 'discovery' of new resources.

Whilst MCP has generated some excitement in AI developer circles, it has also faced well-founded criticism for various practical shortcomings.  This article examines MCP's key limitations; lack of stateless commands, missing authentication/authorization, scalability challenges, narrow applicability, and questions whether this new Open Standard really is a broadly useful standard, or a niche solution. 

### Too Much State, Too Little Simplicity
MCP uses a stateful session architecture built on JSON-RPC. Each MCP client, an AI application or agent, establishes a persistent connection to an MCP Server providing some functionality.   This long-lived, bidirectional channel (over stdio, or Server-Sent Events) allows the server to push notifications and maintain context with the client.  Through this session, the client and server exchange capabilities and discover features (what tools/resources/prompts) the server offers. 
After that, the client can send request to invoke server-provided tools, or access resources, and the server should respond with results. 

In practice however, this means MCP is more complex than a simple REST API.  It's not just calling a HTTP endpoint with inputs and getting a response, it involves running a dedicated server that speaks the right protocol, and maintaining an open session.  For example, to give an AI application access to a database, or home automation system via MCP, you'd have to run a MCP server that exposes relevant operations as tools, and the AI client would connect to it continuously rather than making one-off calls.  This architectural decision enables things like real-time notifications, and dynamic tool discovery but introduces some significant limitations and challenges...

So one of the biggest gripes with MCP is that because it requires a persistent, stateful connection between clients and servers; it becomes difficult to run a MCP server application serverlessly.  Unlike REST APIs, which you call statelessly (ask and receive a response), MCP demands maintaining continuous sessions. This means you can’t just quickly fetch some data—you must open and keep an active connection.

While persistent sessions allow real-time communication, they complicate scenarios where simplicity and scalability matter. Want to quickly scale or load-balance your MCP service across multiple servers? Good luck! Now you're stuck managing stateful sessions. What does that mean in practical terms, sticky sessions? How do you maintain the stickiness? It's cumbersome, heavyweight, and feels overkill for tasks as basic as fetching today's weather forecast. 

This alone feels like a nightmare waiting to happen for adoption in cloud-first environments, and that's before we look at Security.

### Security? What Security?
Another glaring issue with MCP is its lack of built-in authentication and authorization standards. The protocol spec (at time of writing), somewhat surprisingly, leaves security entirely to implementers. This means every MCP integration potentially handles authentication differently—or worse, not at all. 

In practice, developers have resorted to ad-hoc solutions like manually embedding API keys or OAuth tokens outside the MCP framework. This not only leads to inconsistent security practices but also opens up potential vulnerabilities, making MCP unsuitable for sensitive or enterprise-grade integrations out-of-the-box.

In an enterprise context, developers would have to wrap MCP connections in their own SSO/OAuth frameworks - extra work that undermines any benefits to MCP's supposed plug-and-play appeal.  
The core MCP development team are aware of this however, the [roadmap](https://modelcontextprotocol.io/development/roadmap) has better support for Remote MCP connections as a high priority.  

However, I can't help but wonder why MCP was released to the world in such an unfinished state - Why not spend a few more months on it, then release it when it's actually fit for purpose.

### Designed for a Niche, Struggling in the Real World
MCP’s original use case was narrow—primarily serving Anthropic's Claude Desktop application, a local AI assistant designed for privacy-conscious users. This narrow beginning means MCP isn't naturally built for web-scale cloud apps or enterprise scenarios. Its current complexity and overhead can be seen as remnants from an overly specific context — a protocol designed on someone's laptop, for a very particular set of problems.

The combination of statefullness and no intrinsic support for authentication leads to immediate scalability issues for MCP.  Each server must maintain session state with each client, which can strain resources as the number of clients grows - especially if you wanted to offer a public MCP Server as a Service.  Unlike stateless APIs that can be easily replicated and scaled behind a load balancer, a stateful MCP service requires careful coordination (or some kind of session stickiness), so clients don't lose their context, and their shit, when scaled up across multiple servers.  
This makes horizontal scaling more difficult, if not practically impossible.  There's an interesting discussion on the MCP github [here](https://github.com/modelcontextprotocol/specification/discussions/102) about what the possible solutions might be, but there's no clear winning suggestion currently.

As a result, general-purpose apps find little incentive to adopt MCP over simpler REST APIs or OpenAPI standards, which already address many of these integration needs more elegantly and efficiently. The MCP ecosystem, thus far, remains sparse and highly experimental.

I had a brief try earlier this week to set up a fairly generic MCP server as a proxy to a REST API, and my biggest finding is that whilst this protocol appears to have been built primarily for Claude Desktop, the desktop application doesn't actually support remote MCP servers, only local (on the same machine as the agent), connected via Stdio.  Cursor does support SSE transport, and so, remote MCPs, but the documentation for how to set it up is kinda... lacking. 

### In Conclusion
I don't like MCP.   

It feels like we're on the edge of a new format war, like VHS vs Betamax in the 1990s.  OpenAI have a [competing SDK set](https://openai.com/index/new-tools-for-building-agents/), which whilst currently only for OpenAI tools and models.  [Agents.json](https://github.com/wild-card-ai/agents-json) is an OpenAPI thing that I could definitely get behind, but making the python library for it AGPL3 feels like a proper footgun.
I also found another tool/library/protocol/thing positioning itself in the same field https://agent-network-protocol.com/ but honestly, I found the documentation to be too difficult to read. 
[SLOP](https://github.com/agnt-gg/slop) too, looks promising

Given that pornography may have been one of the deciding factor for the Betamax/VHS format war, perhaps we'll have to wait and see if someone builds a MCP or agent.json for pornhub.

I don't think I'm alone in my criticism of MCP, there's a [lot of skepticism on hackernews](https://news.ycombinator.com/item?id=43302297) about MCP’s practicality. 

Unless MCP evolves rapidly; adding essentials like built-in authentication, authorization, and perhaps a simpler stateless mode, it risks remaining a niche curiosity rather than becoming a practical industry standard.

### The Bottom Line
MCP is a fascinating experiment, an attempt at building a truly dynamic, plug-and-play AI environment. But today, it feels like a solution in search of a problem. Its stateful complexity, missing security mechanisms, and limited real-world utility make it an intriguing idea — but ultimately impractical for most real-world scenarios. Until these core shortcomings are resolved, MCP is more likely to inform future protocols rather than become the definitive standard itself.

It really feels like the embodiment of "Well, it worked on my machine!"

