---
layout: post
title:  "Anthropic MCP"
date:   2025-03-08 10:39:13 -0400
categories: MCP
tags: MCP Anthropic LLM Agent
---


[Anthropic](https://www.linkedin.com/company/anthropicresearch/)  MCP is the hottest topic this week. What challenge MCP resolve here? It decouples agents to tools/services with standard protocol and an open platform.  
  
If MCP is good, can it create a bomberman game for me and my wife to play together? I tried this idea with claude 3.5 or gpt-4o before but never made it. I challenged it again with MCP sequential-thinking and claude 3.5 in cursor. It actually works this time after few tries. Here is what I got,  [bomberman](https://bobbercheng.github.io/mcp-demo/bomberman.html). You can try to play it with your wife/kids. Feel free DM feedback or modify it by yourself. Refer to my repo  [mcp-demo](https://github.com/bobbercheng/mcp-demo)  for all details.  
  
Here is my prompt, "Use sequential-thinking with 5 steps first, then create a bomberman game all in one html file to play bomberman with JKLI for direction and H for bomb while competing against another player, who will use ASDW for direction and F for bomb. Define all game settings like moving speed as constant variants with explaination. There are random blocks that block two players in the begin. Show grid postion, blowed blocks number for each player with same font color as player color in real time. Two player are in opposite conerns in the begin. The game doesn't end until one player is killed by bombs. Show winer in the end. Each play alway can move 1 block to connected open blocks by pressing direction key and release it quickly. The player can move more blocks by holding direction key. Make sure each play always stay in one block instead between two blocks once releasing direction key. Players cannot go through blocks. Bomb always can blow away expected blocks. Follow best practice to have well documented code to reflect game logic. Write defensive code to prevent primary bugs e.g. go through blocks, move out of the grid. Do not use alert()."  
Yes, I use lots of prompt engineering. Here are few takeaways:  
- I could use more than 5 step thinking but the result is not stable due to long context length and limit of LLM model.  
- I force LLM to write code show players' positions in real time. It's a trick to process internal data properly to avoid bugs.  
- You can try it by yourself with my hosted sequential-thinking as SSE at  [http://34.123.61.175:47963/sse](http://34.123.61.175:47963/sse).
  
There are other userful MCP like fetch, memory and firecrawl etc.  
  
What MCP can do better? Setting up MCP in stdio mode is much hard than just install a pip/npm package. Using other MCP server sounds skeptical as data privacy concern. Anthropic plans to release MCP registry service to resolve this issue soon.  
  
However, you may not always get good result as I show here as sequential thinking cannot resolve all challenges. The missed one here is LLM self consistency. Unfortunately there is MCP server for LLM self consistency. I will release a consistency MCP. If you are interested in it, please DM me.



[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
