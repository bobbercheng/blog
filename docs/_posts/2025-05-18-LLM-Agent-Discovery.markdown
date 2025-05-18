---
layout: post
title:  "Have LLM agent to discovery new science"
date:   2025-05-18 10:39:13 -0400
categories: discovery
tags: discovery
---

{% raw %}

  <style>
    body { color: #fff; }
    * { margin:0; padding:0; box-sizing:border-box; }
    html, body { width:100%; height:100%; overflow:hidden; background:#000; }
    canvas { display:block; width:100%; height:100%; }
  </style>

{% endraw %}

The hottest AI news this week is Google [AlphaEvolve](https://deepmind.google/discover/blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/). This news looks far away from AI engineer. After recovering from a bad fever, I started a one week challenge to ​have an agent read hundreds of arxiv papers until it finds a AI research idea that it verifies has never been thought before. I think it's a good way to bring AI discovery to engineers.

{% raw %}
  <script>
    // Find post content div and create canvas
      const postContent = document.querySelector('.post-content.e-content');
      if (postContent) {
        const canvas = document.createElement('canvas');
        canvas.id = 'gl-canvas';
        postContent.appendChild(canvas);
        console.log('created gl-canvas');
      }

    // Vertex shader code
    const vertexShaderSource = `
    attribute vec2 a_position;
    varying vec2 v_uv;
    void main() {
      v_uv = a_position * 0.5 + 0.5;
      gl_Position = vec4(a_position, 0.0, 1.0);
    }
    `;

    // Fragment shader code
    const fragmentShaderSource = `
    precision mediump float;
    uniform float u_time;
    uniform float u_speed;
    varying vec2 v_uv;

    void main() {
      vec2 uv = v_uv;
      float t = u_time * 0.001 * u_speed;
      
      // Calculate spotlight position moving from left to right
      float spotlightX = mod(t * 0.2, 1.5) - 0.25; // Controls horizontal movement
      float spotlightY = 0.5; // Fixed at middle height
      
      // Calculate distance from current pixel to spotlight center
      float dist = distance(vec2(spotlightX, spotlightY), uv);
      
      // Create spotlight effect with soft edges
      float spotlight = smoothstep(0.3, 0.1, dist);
      
      // Base colors (revealed by spotlight)
      float angle = (uv.x + uv.y) * 6.0 + t * 0.2;
      float r = 0.5 + 0.5 * sin(angle);
      float g = 0.5 + 0.5 * sin(angle + 2.0);
      float b = 0.5 + 0.5 * sin(angle + 4.0);
      
      // Mix with darkness based on spotlight
      vec3 color = mix(vec3(0.0, 0.0, 0.0), vec3(r, g, b), spotlight);
      
      gl_FragColor = vec4(color, 1.0);
    }
    `;

    // Main application code
    (function() {
      const canvas = document.getElementById('gl-canvas');
      const gl = canvas.getContext('webgl');
      if (!gl) return alert("WebGL not supported");

      // compile helper
      function compile(src, type) {
        const s = gl.createShader(type);
        gl.shaderSource(s, src);
        gl.compileShader(s);
        if (!gl.getShaderParameter(s, gl.COMPILE_STATUS))
          throw new Error(gl.getShaderInfoLog(s));
        return s;
      }

      // build program
      const vs = compile(vertexShaderSource, gl.VERTEX_SHADER);
      const fs = compile(fragmentShaderSource, gl.FRAGMENT_SHADER);
      const prog = gl.createProgram();
      gl.attachShader(prog, vs);
      gl.attachShader(prog, fs);
      gl.linkProgram(prog);
      if (!gl.getProgramParameter(prog, gl.LINK_STATUS))
        throw new Error(gl.getProgramInfoLog(prog));
      gl.useProgram(prog);

      // full-screen quad
      const posLoc = gl.getAttribLocation(prog, 'a_position');
      const buf = gl.createBuffer();
      gl.bindBuffer(gl.ARRAY_BUFFER, buf);
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
        -1,-1,  1,-1,  -1,1,
        -1,1,   1,-1,   1,1
      ]), gl.STATIC_DRAW);
      gl.enableVertexAttribArray(posLoc);
      gl.vertexAttribPointer(posLoc, 2, gl.FLOAT, false, 0, 0);

      // uniform locations
      const timeLoc  = gl.getUniformLocation(prog, 'u_time');
      const speedLoc = gl.getUniformLocation(prog, 'u_speed');

      // resize handler
      function resize() {
        canvas.width  = window.innerWidth;
        canvas.height = window.innerHeight;
        gl.viewport(0, 0, gl.drawingBufferWidth, gl.drawingBufferHeight);
      }
      window.addEventListener('resize', resize);
      resize();

      // render loop
      function render(time) {
        const speed = 0.8;
        gl.uniform1f(timeLoc, time);
        gl.uniform1f(speedLoc, speed);
        gl.drawArrays(gl.TRIANGLES, 0, 6);
        requestAnimationFrame(render);
      }
      requestAnimationFrame(render);
    })();
  </script>
{% endraw %}


[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
