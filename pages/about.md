---
layout: default
title: About
permalink: /about/
---

<style>
  .about-section { padding: 60px 0; }
  .about-content { max-width: 640px; }
  .about-content p { font-size: 1.05rem; margin-bottom: 1.4em; }
  .skills-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
    gap: 12px; margin-top: 40px;
  }
  .skill-item {
    background: var(--bg-alt); border: 1px solid var(--border);
    border-radius: var(--radius); padding: 16px 20px;
    font-family: 'JetBrains Mono', monospace; font-size: 0.82rem;
  }
  .skill-item .skill-name { color: var(--text); margin-bottom: 4px; font-weight: 700; }
  .skill-item .skill-level { color: var(--amber); font-size: 0.7rem; }
</style>

<div class="about-section">
<h1>// <span class="accent">about</span></h1>
<div class="about-content">

**bits & bytes** is a technical blog about C and C++ programming.
Not the "here's how to write a for loop" kind — the kind where we read assembly output,
measure cache miss rates, and dig into why the compiler makes surprising choices.

I'm a systems programmer who spends too much time on Compiler Explorer and not enough
time sleeping. I work on performance-critical software and write about what I learn along
the way: memory models, undefined behavior, modern C++ idioms, and the occasional foray
into kernel internals.

All code examples are compiled and tested. If something is wrong, open an issue on GitHub.

<div class="terminal-banner" style="margin-top:40px">
  <div class="term-dots">
    <div class="term-dot"></div><div class="term-dot"></div><div class="term-dot"></div>
  </div>
  <div class="term-line"><span class="prompt">$</span> <span class="cmd">whoami</span></div>
  <div class="term-line"><span class="output">Alex Mercer</span></div>
  <div class="term-line"><span class="prompt">$</span> <span class="cmd">cat interests.txt</span></div>
  <div class="term-line"><span class="output">systems programming, compilers, performance, embedded C, Linux internals</span></div>
  <div class="term-line"><span class="prompt">$</span> <span class="cmd">uname -a</span></div>
  <div class="term-line"><span class="output">Linux 6.8.0-generic x86_64 GNU/Linux</span> <span class="cursor"></span></div>
</div>

<div class="skills-grid">
  <div class="skill-item">
    <div class="skill-name">C (C11/C17)</div>
    <div class="skill-level">████████░░ Expert</div>
  </div>
  <div class="skill-item">
    <div class="skill-name">C++ (up to C++23)</div>
    <div class="skill-level">████████░░ Expert</div>
  </div>
  <div class="skill-item">
    <div class="skill-name">x86-64 Assembly</div>
    <div class="skill-level">██████░░░░ Proficient</div>
  </div>
  <div class="skill-item">
    <div class="skill-name">Linux Internals</div>
    <div class="skill-level">███████░░░ Advanced</div>
  </div>
  <div class="skill-item">
    <div class="skill-name">Embedded C</div>
    <div class="skill-level">██████░░░░ Proficient</div>
  </div>
  <div class="skill-item">
    <div class="skill-name">CMake / Build</div>
    <div class="skill-level">█████░░░░░ Intermediate</div>
  </div>
</div>

</div>
</div>
