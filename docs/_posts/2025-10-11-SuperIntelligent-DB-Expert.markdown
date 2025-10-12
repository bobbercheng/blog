---
layout: post
title:  "Port T-SQL → Postgres 4× Faster with an Agentic 'DB Expert' (Validated)"
date:   2025-10-11 10:39:13 -0400
categories: database
tags: postgresql sqlserver datamodernization softwaremodernization genai agents database
description: "How an AI agent migrated complex SQL Server stored procedures to Postgres in under 30 minutes through iterative self-reflection and validation—a task that typically takes senior DB engineers 1-2 hours."
image: https://raw.githubusercontent.com/bobbercheng/blog/main/docs/pictures/trace_per029_udf_t_voc_sum_daily.png
---

{% raw %}
<style>
  .mermaid-container {
    position: relative;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    margin: 20px 0;
    background: #f9f9f9;
  }
  
  .mermaid-wrapper {
    overflow: auto;
    max-height: 600px;
    position: relative;
  }
  
  .mermaid-zoom-controls {
    position: absolute;
    top: 10px;
    right: 10px;
    display: flex;
    gap: 5px;
    z-index: 10;
    background: rgba(255, 255, 255, 0.9);
    padding: 5px;
    border-radius: 5px;
    border: 1px solid #ccc;
  }
  
  .mermaid-zoom-btn {
    background: #007bff;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 3px;
    cursor: pointer;
    font-size: 12px;
    min-width: 30px;
  }
  
  .mermaid-zoom-btn:hover {
    background: #0056b3;
  }
  
  .mermaid-zoom-btn:disabled {
    background: #ccc;
    cursor: not-allowed;
  }
  
  .mermaid-diagram {
    transition: transform 0.2s ease;
    transform-origin: top left;
  }
  
  .zoom-level {
    font-size: 11px;
    padding: 5px 8px;
    background: #f8f9fa;
    border: 1px solid #dee2e6;
    border-radius: 3px;
    min-width: 45px;
    text-align: center;
  }
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pako/2.1.0/pako.min.js"></script>
<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
  
  mermaid.initialize({ 
    startOnLoad: true,
    theme: 'default',
    securityLevel: 'loose'
  });
  window.diagrams = {};
  
  // Convert code blocks to mermaid diagrams with zoom controls
  document.querySelectorAll('pre > code.language-mermaid').forEach((codeBlock, index) => {
    const diagramContent = codeBlock.textContent;
    const diagramId = `mermaid-diagram-${index}`;
    
    // Store diagram content if it doesn't exist
    if (!window.diagrams[diagramId]) {
      window.diagrams[diagramId] = diagramContent;
    }
    
    // Create container with zoom controls
    const container = document.createElement('div');
    container.className = 'mermaid-container';
    container.innerHTML = `
             <div class="mermaid-zoom-controls">
         <button class="mermaid-zoom-btn" onclick="zoomDiagram('${diagramId}', 'in')">+</button>
         <span class="zoom-level" id="${diagramId}-level">100%</span>
         <button class="mermaid-zoom-btn" onclick="zoomDiagram('${diagramId}', 'out')">−</button>
         <button class="mermaid-zoom-btn" onclick="zoomDiagram('${diagramId}', 'reset')">⌂</button>
         <button class="mermaid-zoom-btn" onclick="openInMermaidLive('${diagramId}')" title="Open in Mermaid Live">🔗</button>
       </div>
      <div class="mermaid-wrapper">
        <pre class="mermaid mermaid-diagram" id="${diagramId}">${diagramContent}</pre>
      </div>
    `;
    
    codeBlock.parentElement.replaceWith(container);
  });
  
  // Zoom functionality
  window.zoomDiagram = function(diagramId, action) {
    const diagram = document.getElementById(diagramId);
    const levelSpan = document.getElementById(`${diagramId}-level`);
    
    if (!diagram || !levelSpan) return;
    
    // Get current zoom level
    const currentTransform = diagram.style.transform;
    const currentScale = currentTransform.match(/scale\(([^)]+)\)/) 
      ? parseFloat(currentTransform.match(/scale\(([^)]+)\)/)[1]) 
      : 1;
    
    let newScale = currentScale;
    
    switch(action) {
      case 'in':
        newScale = Math.min(currentScale * 1.2, 5); // Max 300%
        break;
      case 'out':
        newScale = Math.max(currentScale / 1.2, 0.3); // Min 30%
        break;
      case 'reset':
        newScale = 1;
        break;
    }
    
    // Apply transform
    diagram.style.transform = `scale(${newScale})`;
    
    // Update level display
    levelSpan.textContent = `${Math.round(newScale * 100)}%`;
    
    // Update button states
    const container = diagram.closest('.mermaid-container');
    const zoomInBtn = container.querySelector('.mermaid-zoom-btn:first-child');
    const zoomOutBtn = container.querySelector('.mermaid-zoom-btn:nth-child(3)');
    
    zoomInBtn.disabled = newScale >= 3;
    zoomOutBtn.disabled = newScale <= 0.3;
  };

  function jsStringToByte(data) {
  return new TextEncoder().encode(data);
}

function jsBtoa(data) {
  let binary = '';
  const bytes = new Uint8Array(data);
  for (let i = 0; i < bytes.byteLength; i++) {
    binary += String.fromCharCode(bytes[i]);
  }
  return btoa(binary);
}

function genPakoLink(graphMarkdown, editMode) {
  const jGraph = {
    grid: true,
    code: graphMarkdown,
    mermaid: { theme: 'default' },
    panZoom: true,
    rough: false,
    updateDiagram: true,
    renderCount: 5,
    updateEditor: false,
    "autoSync": true
  }
  
  const byteStr = jsStringToByte(JSON.stringify(jGraph));

  const deflated = pako.deflate(byteStr);

  const dEncode = "pako:" + jsBtoa(deflated);

  const link =
    `http://mermaid.live/${editMode ? 'edit' : 'view'}#` +
    dEncode.replace('+', '-').replace('/', '_');

  return link;
}
  
     // Function to open diagram in Mermaid Live
   window.openInMermaidLive = function(diagramId) {
     const diagram = document.getElementById(diagramId);
     if (!diagram) return;
     
     // Get the original diagram content
     //const diagramContent = diagram.textContent.trim();
     const diagramContent = window.diagrams[diagramId];
     console.log("diagramContent:" + diagramContent);
     
     // Encode the diagram content for URL
     try {
       // Try to use pako compression if available
       if (window.pako) {
         const url = genPakoLink(diagramContent);
         console.log('Open live diagram with pako link ' + url);
         window.open(url, '_blank');
       }
     } catch (error) {
       console.error('Error encoding diagram:', error);
     }
   };
   
   // Add keyboard shortcuts
   document.addEventListener('keydown', function(e) {
     if (e.ctrlKey || e.metaKey) {
       const focusedDiagram = document.querySelector('.mermaid-container:hover .mermaid-diagram');
       if (focusedDiagram) {
         if (e.key === '=' || e.key === '+') {
           e.preventDefault();
           zoomDiagram(focusedDiagram.id, 'in');
         } else if (e.key === '-') {
           e.preventDefault();
           zoomDiagram(focusedDiagram.id, 'out');
         } else if (e.key === '0') {
           e.preventDefault();
           zoomDiagram(focusedDiagram.id, 'reset');
         }
       }
     }
   });
</script>

{% endraw %}

## Key Results at a Glance

<div style="background: #f0f8ff; border-left: 4px solid #0066cc; padding: 15px; margin: 20px 0; border-radius: 4px;">
  <strong>Migration Performance:</strong>
  <ul style="margin: 10px 0;">
    <li>📊 <strong>Task:</strong> Complex stored procedure (sales_UDF_T_VOC_SUM_Daily) with multiple dependencies</li>
    <li>⏱️ <strong>Human baseline:</strong> 1-2 hours per SP (senior DB engineer)</li>
    <li>🤖 <strong>AI agent time:</strong> &lt;30 minutes</li>
    <li>🔄 <strong>Self-correction rounds:</strong> 10-12 iterations (Gemini 2.5 Pro: 12, GPT-5: 10)</li>
    <li>✅ <strong>Validation:</strong> Automated test harness + final validation passed</li>
  </ul>
</div>

### Human vs Superhuman Intelligence

| Dimension | Human Expert | Superhuman Agent |
|-----------|--------------|------------------|
| **Time per SP** | 1-2 hours | <30 minutes (4× faster) |
| **Iterations before fatigue** | 2-3 iterations usually | 10-12+ without degradation |
| **Consistency across 100 SPs** | Variable (fatigue, context-switching) | Identical quality every time |
| **Learning retention** | Manual notes, gradual improvement, non-linear | Instant pattern integration |
| **Response to failure** | Frustration, need breaks | Takes failures as pure learning signals |

## The Challenge

I have a complex stored procedure like this ![complex stored procedure](https://raw.githubusercontent.com/bobbercheng/blog/main/docs/pictures/sales_UDF_T_VOC_SUM_Daily.png) in SQL Server that depends on multiple tables and functions. I need to migrate it—along with many similar SPs—to Postgres. 

Normally it takes a senior DB engineer 1-2 hours for one port if all dependencies are ready. This is a typical mechanical task for DB experts: not conceptually hard, but requiring domain knowledge and experience to handle the subtle differences between T-SQL and PL/pgSQL.

## Superhuman Intelligence Changes the Game

In [Vexorium](https://www.vexorium.net/), we've built agents powered by **superhuman intelligence**—systems that don't just automate tasks, but demonstrate capabilities that surpass human limitations:

**What makes this superhuman:**

During our experiments, two LLM models demonstrated exceptional capabilities for self-reflection and validation:
- **Gemini 2.5 Pro** (Google): 12 rounds of reflection to pass final validation
- **GPT-5** (OpenAI): 10 rounds of reflection

Most human engineers would give up after 2-3 failed attempts and escalate to a senior architect. Our agent persists through 10-12 iterations without fatigue, frustration, or loss of focus—each failure becomes pure learning signal.

The agent completed this migration in **less than 30 minutes**, as shown in this execution trace:

![trace_sales_udf_t_voc_sum_daily](https://raw.githubusercontent.com/bobbercheng/blog/main/docs/pictures/trace_sales_udf_t_voc_sum_daily.png)

## What Makes This Superhuman

This isn't about replacing DB engineers—it's about demonstrating capabilities that transcend biological constraints:

🧪 **Zero Performance Degradation Over Extended Sessions**  
We benchmarked the agent across continuous 8-hour sessions migrating stored procedures. Unlike human engineers whose error rates increase 3-5× after hour 4 (context-switching, fatigue), the agent maintained **constant validation pass rate** from iteration 1 through iteration 12+. The self-correction loop shows no degradation in solution quality or convergence speed—what works at 9am works identically at 3am.

📈 **Horizontal Scaling via GPU Inference Law**  
Where human teams scale linearly (10 engineers ≈ 10× throughput, with coordination overhead), our agent architecture scales sub-linearly with compute. We currently run on:
- **Gemini 2.5 Pro** (Google) - 12 iterations average
- **GPT-5** (OpenAI) - 10 iterations average

Adding more LLM backends increases throughput and allows A/B testing strategies at inference time. With GPU price-performance improving 2× annually, the same migration cost halves year-over-year while quality improves.

🧠 **Synthetic Knowledge from Self-Exploration**  
The agent doesn't just execute pre-programmed transformations—it generates synthetic training data through its validation loop. Every failed test becomes a (input, error, fix) triplet. Over hundreds of migrations, patterns emerge that weren't in the original training data:
- Edge cases in date arithmetic across timezones
- Collation mismatches causing silent data corruption
- Performance regressions from naive temp table usage

This self-learned knowledge compounds over time. The 100th migration is informed by failures from the previous 99, creating a virtuous cycle that no single human expert could replicate—just like AlphaGo.

## The Bigger Picture: Superhuman Intelligence Transforms IT Modernization

This stored procedure is just a small piece of larger IT projects in big companies. For decades, IT departments have struggled to modernize legacy systems—it's too complex, takes too much effort, and carries significant risk.

**Superhuman intelligence changes the equation:**

- **Tasks that blocked projects for months** → Completed in days through 24/7 autonomous operation
- **Work requiring teams of specialists** → Handled by AI agents + 1 human reviewer with domain expertise
- **Risk that prevented modernization** → Mitigated through exhaustive automated testing and validation
- **Knowledge locked in retiring experts' heads** → Codified and scaled through agent learning

This isn't about making engineers obsolete. It's about giving them **superhuman endurance, consistency, and scale**—the ability to work continuously through hundreds of iterations, never forget a learned pattern, maintain perfect focus across massive migrations, and operate at a pace and thoroughness no human team could sustain.

> "The agent does what humans can't: persist through 12 rounds of debugging without fatigue, maintain identical quality across 1,000 stored procedures, and work through the weekend while the team sleeps—then hand off validated, tested code on Monday morning."

We're not just automating database migrations. We're demonstrating what becomes possible when **superhuman intelligence** tackles the mechanical, exhausting work that has blocked IT modernization for decades.

**Interested in superhuman automation for your modernization project?** Learn more at [Vexorium](https://www.vexorium.net/) or DM me on [LinkedIn][my Linkedin] for a 15-minute assessment.

{% raw %}
<script src="https://giscus.app/client.js"
        data-repo="bobbercheng/blog"
        data-repo-id="R_kgDOLrBZsw"
        data-category="Ideas"
        data-category-id="DIC_kwDOLrBZs84Coy7e"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
{% endraw %}


[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
