---
layout: post
title: "EM and Perfect Algorithms"
date: 2025-04-27 10:39:13 -0400
categories: Algorithm
tags: Algorithm
---

I went to the Bay Area for a tech session. Everyone was talking about “vibe coding” with cursor/windsurf, and many product managers and business people told me they could write amazing code to implement cool features—even though they had no formal coding background. This Friday, I visited Microsoft’s Toronto office, where a talent‐sales support manager shared that he’s also using AI to improve network‐protocol implementations. I’m thrilled to see that vibe coding empowers everyone to write code quickly. However, many seasoned engineers—especially in large companies—remain skeptical. One major concern is that LLMs only generate code that *looks* correct and still lack the reasoning ability to produce truly excellent or “perfect” solutions. I covered these LLM limitations in a previous blog post; the new point here is that you *can* guide LLMs toward perfection with vibe coding.

Let’s use a HackerRank problem—[Determining DNA Health](https://www.hackerrank.com/challenges/determining-dna-health/problem)—as an example. You can easily generate a solution with vibe coding and pass the basic test cases. But once you submit, you’ll find some test cases always fail. One key constraint is that each run must complete in under 10 seconds and use no more than 512 MB of RAM. In other words, you need high-performance, memory-efficient code. It took me three hours to arrive at a perfect solution with vibe coding. You can see my Rust/Trie implementation here:  
https://github.com/bobbercheng/algorithm-excellent/blob/main/hackerrank/determining-dna-health/main.rs

My perfect solution reflects my engineering-management experience:

- **Early perfection is rare.** Even with deep thinking, it’s almost impossible to craft a flawless solution on the first try.  
- **Build an evaluation pipeline.** Quickly measure each iteration’s correctness, runtime, and memory usage. I wrote a script to automate these checks.  
- **Start with a brute-force baseline.** Push it against constraints or just pass a subset of test cases—brute force often works for the baseline.  
- **Iterate toward improvement.** Use your pipeline to guide small, focused changes until that path yields no more gains. Document each step in a tree (DFS/BFS or even MCTS) to map your search.  
- **Refactor and synthesize.** Once you have several enhanced versions, consolidate your learnings and clean up the code.  
- **Know when to pivot.** Continuous tweaks eventually hit a wall because changes must stay consistent. Sometimes you need a foundational shift—like switching from Python to Rust—to break through.  

With this approach, vibe coding becomes not just quick but also precise enough to achieve “perfect” algorithms.  
