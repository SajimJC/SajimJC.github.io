---
title: "License"
permalink: /license/
author_profile: false
---

[View raw LICENSE on GitHub](https://raw.githubusercontent.com/{{ site.repository }}/master/LICENSE)

<style>
  #license-content {
    white-space: pre-wrap;
    overflow-wrap: anywhere;
    word-break: break-word;
  }
</style>

<pre id="license-content"></pre>

<script>
  (() => {
    const repo = "{{ site.repository }}";
    const candidates = [
      `https://raw.githubusercontent.com/${repo}/main/LICENSE`,
      `https://raw.githubusercontent.com/${repo}/master/LICENSE`,
    ];

    const target = document.getElementById("license-content");
    if (!target) return;

    const tryFetch = async () => {
      for (const url of candidates) {
        try {
          const r = await fetch(url, { cache: "no-store" });
          if (!r.ok) continue;
          target.textContent = await r.text();
          return;
        } catch (e) {}
      }
      target.textContent = "Failed to load LICENSE content.";
    };

    void tryFetch();
  })();
</script>
