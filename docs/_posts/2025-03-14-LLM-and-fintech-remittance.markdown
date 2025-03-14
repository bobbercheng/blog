---
layout: post
title:  "LLM for remittance service in Fintech"
date:   2025-03-14 10:39:13 -0400
categories: Fintech
tags: Fintech MCP Anthropic LLM Agent
---

I talked to talents engineers, co-founders from Fintech, they said that Fintech is hard as it needs lots of domain knowledges to understand all regulatory compliance. High reliability and no open source sprite makes it every harder. You cannot use LLM for fintech. Today I challenges it with 3 hours quick coding. My goal is to have Cursor agent write a POC version remittance service of transferring Money from India to Canada in real time with UPI and  [Wise](https://www.linkedin.com/company/wiseaccount/).  


Here is what I got, 5 versions of POC:  
- Java,  [remit-java](https://github.com/bobbercheng/remit-demo/tree/main/remit-java)  
- Typescript,  [remit-typescript](https://github.com/bobbercheng/remit-demo/tree/main/remit-typescript)  
- Elixir,  [remit-elixir](https://github.com/bobbercheng/remit-demo/tree/main/remit-elixir)  
- Python,  [remit-python](https://github.com/bobbercheng/remit-demo/tree/main/remit-python)  
- Rust,  [remit-rust](https://github.com/bobbercheng/remit-demo/tree/main/remit-rust)  
  
You can check my prompts at  [remit-demo](https://github.com/bobbercheng/remit-demo). BTW, I use my hosted sequential-thinking as SSE at  [http://34.123.61.175:47963/sse](http://34.123.61.175:47963/sse)
  
Few takeaways:  
- LLM could be used for Fintech coding.  
- Java version is not the best for everyone.  
- I really enjoyed insights from different tech stack. E.g. Elixir version has a great style to log each remit method  [remittance_service.ex](https://github.com/bobbercheng/remit-demo/blob/8dde2979c21ffdde87642b660ce7048cd3a3701f/remit-elixir/lib/remit/core/remittance_service.ex)  
- LLM already also address advancing topic like finite state machine.  
- It takes more time to think over prompts than coding. Next time I will share how to compose prompts.


[my Resume]: https://raw.githubusercontent.com/bobbercheng/blog/main/docs/assets/Resume_Bobber-Cheng_2025_shared.pdf
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
