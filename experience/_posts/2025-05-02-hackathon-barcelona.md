---
layout:     post
title:      "Building SkyGrouper at HackUPC: A Weekend of Innovation, Collaboration, and AI"
sitemap: false
excerpt_separator: <!--more-->
hide_last_modified: true
comments: true
---

# Building SkyGrouper at HackUPC: A Weekend of Innovation, Collaboration, and AI

Last weekend, I had the incredible opportunity to participate in HackUPC Barcelona, one of Europe's premier student hackathons. The energy was electric, with over 750 hackers from around the world converging on the city to build, learn, and connect. I joined forces with Taisiia Nekrasova, Aiman Himi, and Joel Calm Padrosa to take on the Skyscanner challenge—and what a challenge it was!

<!--more-->

## The Challenge: Multi-Origin Travel Planning

![Skyscanner Challenge](../images/hackathon-barcelona1.png)

Skyscanner posed a unique problem: how can we help people traveling from different locations agree on the ideal holiday destination? Coordinating group travel is often complex, especially when it involves different departure cities, budgets, and preferences.

## Our Solution: SkyGrouper

Enter **SkyGrouper**, our answer to this travel conundrum. SkyGrouper is a smart travel planning application powered by an AI Agent running on an MCP-based server. It integrates seamlessly with the Skyscanner API to analyze flight availability, pricing, and travel times to suggest optimal meeting points and itineraries for multi-origin groups.

Our AI Agent evaluates a range of destinations and weighs various constraints such as cost, travel duration, and convenience. Once the best destination is identified, SkyGrouper generates detailed flight itineraries for each traveler, making group coordination effortless.

You can check out our [demo video and source code here](https://github.com/jc2409/SkyGrouper-Agent).

## Technical Highlights

* **AI Agent on MCP Server**: We designed and deployed an intelligent agent capable of performing complex decision-making tasks.
* **Skyscanner API Integration**: Real-time data was pulled from Skyscanner to provide accurate travel options.
* **Multi-Origin Planning**: The app handles input from several starting points and recommends a centralized travel plan.

## Beyond the Hackathon: Sharing Knowledge

In the spirit of continued learning and community contribution, I recently published a new Learning Path on the [Arm Developer Platform](https://learn.arm.com/learning-paths/cross-platform/mcp-ai-agent/). It walks you through the process of deploying an MCP server on a Raspberry Pi 5—perfect for anyone looking to build intelligent, lightweight AI systems on edge devices.

## Reflections and Takeaways

![Team Picture](../images/hackathon-barcelona2.png)

HackUPC was a whirlwind of creativity, technical deep-dives, and late-night problem-solving. I'm incredibly proud of what our team accomplished in such a short time, and equally inspired by the ingenuity of fellow participants. Huge thanks to the HackUPC organizers and the Skyscanner team for making this event possible.

Looking forward to the next hackathon adventure—until then, I'll be iterating on SkyGrouper and exploring more ways to bring intelligent systems to life.
