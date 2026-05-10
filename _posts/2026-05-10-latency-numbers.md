---
layout: post
title: Latency numbers
subtitle: A list of latency numbers for system design.
cover-img: /assets/img/posts/2026-05-10/latency-cover.png
thumbnail-img: /assets/img/posts/2026-05-10/latency.png
share-img: /assets/img/posts/2026-05-10/latency.png
tags: [system design]
author: Pengfei Gao
---

> You may find the one-page cheatsheet [here](/assets/pdf/highlight-latency-numbers.pdf)

This is a test post with js enabled.

<div id="latency-widget" class="latency-widget">
  <div class="latency-header">
    <div>
      <h2>Latency Numbers Bucketed by Integer log₁₀(ns)</h2>
      <p>
        Bucket rule:
        <span class="formula">b = ⌊log₁₀(Time in ns)⌋</span>
      </p>
    </div>

    <div class="latency-controls">
      <input id="latency-search" type="text" placeholder="Search: cache, SSD, network..." />
      <select id="latency-unit">
        <option value="ns">ns</option>
        <option value="us">μs</option>
        <option value="ms">ms</option>
        <option value="s">s</option>
      </select>
    </div>
  </div>

  <div class="latency-scale">
    <span>faster</span>
    <div class="scale-line"></div>
    <span>slower</span>
  </div>

  <div id="latency-buckets" class="latency-buckets"></div>

  <div class="latency-memory-hook">
    <strong>Memory hook:</strong>
    each bucket is roughly one order of magnitude slower than the previous one:
    <span>cache → memory → local storage/network → datacenter RTT → disk/WAN</span>
  </div>
</div>

<style>
  .latency-widget {
    --ink: #202020;
    --muted: #666;
    --card-bg: rgba(255, 255, 255, 0.82);
    --card-border: rgba(32, 32, 32, 0.16);
    --shadow: 0 14px 40px rgba(0, 0, 0, 0.10);
    --radius: 22px;

    max-width: 980px;
    margin: 2rem auto;
    padding: 1.5rem;
    color: var(--ink);
    border-radius: 28px;
    background:
      radial-gradient(circle at 10% 10%, rgba(110, 168, 254, 0.22), transparent 28%),
      radial-gradient(circle at 90% 0%, rgba(255, 185, 120, 0.24), transparent 30%),
      linear-gradient(135deg, #fafafa, #f2f4f8);
    box-shadow: var(--shadow);
    font-family:
      -apple-system,
      BlinkMacSystemFont,
      "Segoe UI",
      Roboto,
      Helvetica,
      Arial,
      sans-serif;
  }

  .latency-header {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
    justify-content: space-between;
    margin-bottom: 1rem;
  }

  .latency-header h2 {
    margin: 0 0 0.35rem;
    font-size: clamp(1.35rem, 2.8vw, 2rem);
    line-height: 1.15;
    letter-spacing: -0.03em;
  }

  .latency-header p {
    margin: 0;
    color: var(--muted);
  }

  .formula {
    display: inline-block;
    margin-left: 0.25rem;
    padding: 0.22rem 0.5rem;
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.72);
    border: 1px solid rgba(0, 0, 0, 0.08);
    font-weight: 700;
    color: #111;
  }

  .latency-controls {
    display: flex;
    gap: 0.5rem;
    min-width: min(100%, 340px);
  }

  .latency-controls input,
  .latency-controls select {
    border: 1px solid rgba(0, 0, 0, 0.13);
    background: rgba(255, 255, 255, 0.84);
    border-radius: 14px;
    padding: 0.65rem 0.8rem;
    font: inherit;
    outline: none;
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.05);
  }

  .latency-controls input {
    flex: 1;
  }

  .latency-scale {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 0.75rem;
    align-items: center;
    margin: 1rem 0 1.25rem;
    color: var(--muted);
    font-size: 0.9rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }

  .scale-line {
    height: 10px;
    border-radius: 999px;
    background: linear-gradient(90deg, #65d6ad, #ffd166, #ef476f);
    opacity: 0.8;
  }

  .latency-buckets {
    display: grid;
    gap: 0.85rem;
  }

  .latency-card {
    position: relative;
    display: grid;
    grid-template-columns: 76px 1fr minmax(150px, 220px);
    gap: 1rem;
    align-items: stretch;
    padding: 1rem;
    border: 1px solid var(--card-border);
    border-radius: var(--radius);
    background: var(--card-bg);
    backdrop-filter: blur(12px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.06);
    overflow: hidden;
    cursor: pointer;
    transform: translateY(0);
    transition:
      transform 180ms ease,
      box-shadow 180ms ease,
      border-color 180ms ease;
  }

  .latency-card::before {
    content: "";
    position: absolute;
    inset: 0 auto 0 0;
    width: 7px;
    background: var(--accent);
  }

  .latency-card:hover {
    transform: translateY(-3px);
    box-shadow: 0 16px 42px rgba(0, 0, 0, 0.13);
    border-color: rgba(0, 0, 0, 0.24);
  }

  .bucket-num {
    display: grid;
    place-items: center;
    border-radius: 18px;
    background: rgba(255, 255, 255, 0.76);
    font-size: 2.2rem;
    font-weight: 900;
    line-height: 1;
    color: var(--accent);
  }

  .bucket-main {
    min-width: 0;
  }

  .bucket-title {
    display: flex;
    flex-wrap: wrap;
    gap: 0.45rem;
    margin-bottom: 0.55rem;
  }

  .op-pill {
    display: inline-flex;
    align-items: center;
    padding: 0.35rem 0.6rem;
    border-radius: 999px;
    background: rgba(0, 0, 0, 0.055);
    font-weight: 800;
    line-height: 1.25;
  }

  .bucket-detail {
    display: none;
    color: var(--muted);
    font-size: 0.94rem;
    line-height: 1.45;
  }

  .latency-card.expanded .bucket-detail {
    display: block;
    animation: fadeIn 160ms ease;
  }

  .bucket-time {
    display: grid;
    gap: 0.55rem;
    align-content: center;
    padding-left: 1rem;
    border-left: 1px solid rgba(0, 0, 0, 0.12);
    text-align: center;
  }

  .time-range {
    font-size: 1.05rem;
    font-weight: 900;
  }

  .time-caption {
    color: var(--muted);
    font-size: 0.82rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.07em;
  }

  .log-bar {
    height: 8px;
    border-radius: 999px;
    background: rgba(0, 0, 0, 0.10);
    overflow: hidden;
  }

  .log-fill {
    height: 100%;
    width: var(--fill);
    border-radius: inherit;
    background: var(--accent);
  }

  .latency-memory-hook {
    margin-top: 1.25rem;
    padding: 1rem 1.1rem;
    border-radius: 18px;
    background: rgba(255, 255, 255, 0.72);
    border: 1px solid rgba(0, 0, 0, 0.10);
    line-height: 1.55;
  }

  .latency-memory-hook span {
    font-weight: 800;
  }

  .empty-state {
    padding: 1.5rem;
    text-align: center;
    color: var(--muted);
    border: 1px dashed rgba(0, 0, 0, 0.22);
    border-radius: 20px;
    background: rgba(255, 255, 255, 0.58);
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(-4px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @media (max-width: 760px) {
    .latency-header {
      flex-direction: column;
    }

    .latency-controls {
      width: 100%;
    }

    .latency-card {
      grid-template-columns: 56px 1fr;
    }

    .bucket-time {
      grid-column: 1 / -1;
      border-left: 0;
      border-top: 1px solid rgba(0, 0, 0, 0.12);
      padding-left: 0;
      padding-top: 0.85rem;
      text-align: left;
    }

    .bucket-num {
      font-size: 1.7rem;
    }
  }
</style>

<script>
  (() => {
    const buckets = [
      {
        b: 0,
        ops: ["L1 cache reference", "Branch misprediction", "L2 cache reference"],
        minNs: 1,
        maxNs: 10,
        detail:
          "This is the world of CPU-local events. These are extremely fast, but still matter inside tight loops and high-QPS serving paths."
      },
      {
        b: 1,
        ops: ["Mutex lock / unlock"],
        minNs: 10,
        maxNs: 100,
        detail:
          "Synchronization can be cheap when uncontended, but it can become a serious bottleneck in concurrent systems."
      },
      {
        b: 2,
        ops: ["Main memory reference"],
        minNs: 100,
        maxNs: 1000,
        detail:
          "Main memory is much slower than cache. Cache locality and layout-aware data structures matter here."
      },
      {
        b: 3,
        ops: ["Send 2 kB over 10 Gbps network", "Compress 1 kB with Zippy"],
        minNs: 1000,
        maxNs: 10000,
        detail:
          "This bucket enters microsecond-level work: small network transfers and lightweight compression."
      },
      {
        b: 4,
        ops: ["Read 1 MB sequentially from memory", "SSD 4kB random read"],
        minNs: 10000,
        maxNs: 100000,
        detail:
          "Sequential memory reads are fast, but random SSD reads already move into a much slower latency regime."
      },
      {
        b: 5,
        ops: ["Round trip within same datacenter"],
        minNs: 100000,
        maxNs: 1000000,
        detail:
          "A same-datacenter round trip is often fast enough for distributed systems, but too expensive for very tight inner loops."
      },
      {
        b: 6,
        ops: [
          "Read 1 MB sequentially from SSD",
          "Read 1 MB sequentially from disk",
          "Read 1 MB over 10 Gbps network"
        ],
        minNs: 1000000,
        maxNs: 10000000,
        detail:
          "This is millisecond-level latency. For online serving, this is already a large part of the request budget."
      },
      {
        b: 7,
        ops: ["Disk seek"],
        minNs: 10000000,
        maxNs: 100000000,
        detail:
          "Disk seek latency is slow compared with memory and SSD access. Random I/O can dominate system performance."
      },
      {
        b: 8,
        ops: ["TCP RTT between continents"],
        minNs: 100000000,
        maxNs: 1000000000,
        detail:
          "Wide-area network latency is huge compared with local computation. Avoid synchronous cross-continent calls in hot paths."
      }
    ];

    const colors = [
      "#2bb673",
      "#3dbb9f",
      "#4f8cff",
      "#7b61ff",
      "#b45cff",
      "#f06aa6",
      "#ff8a4c",
      "#e85d5d",
      "#c0392b"
    ];

    const root = document.getElementById("latency-widget");
    if (!root) return;

    const list = root.querySelector("#latency-buckets");
    const search = root.querySelector("#latency-search");
    const unit = root.querySelector("#latency-unit");

    function convert(ns, selectedUnit) {
      const value =
        selectedUnit === "ns"
          ? ns
          : selectedUnit === "us"
          ? ns / 1_000
          : selectedUnit === "ms"
          ? ns / 1_000_000
          : ns / 1_000_000_000;

      if (value >= 1000) return value.toLocaleString(undefined, { maximumFractionDigits: 0 });
      if (value >= 10) return value.toLocaleString(undefined, { maximumFractionDigits: 1 });
      if (value >= 1) return value.toLocaleString(undefined, { maximumFractionDigits: 2 });
      return value.toLocaleString(undefined, { maximumSignificantDigits: 3 });
    }

    function formatRange(bucket, selectedUnit) {
      return `${convert(bucket.minNs, selectedUnit)}–${convert(bucket.maxNs, selectedUnit)} ${selectedUnit}`;
    }

    function render() {
      const q = search.value.trim().toLowerCase();
      const selectedUnit = unit.value;

      const filtered = buckets.filter((bucket) => {
        const haystack = `${bucket.b} ${bucket.ops.join(" ")} ${bucket.detail}`.toLowerCase();
        return haystack.includes(q);
      });

      list.innerHTML = "";

      if (!filtered.length) {
        list.innerHTML = `<div class="empty-state">No matching latency bucket found.</div>`;
        return;
      }

      filtered.forEach((bucket) => {
        const card = document.createElement("article");
        const fill = `${((bucket.b + 1) / 9) * 100}%`;

        card.className = "latency-card";
        card.style.setProperty("--accent", colors[bucket.b]);
        card.style.setProperty("--fill", fill);

        card.innerHTML = `
          <div class="bucket-num">${bucket.b}</div>

          <div class="bucket-main">
            <div class="bucket-title">
              ${bucket.ops.map((op) => `<span class="op-pill">${op}</span>`).join("")}
            </div>
            <div class="bucket-detail">${bucket.detail}</div>
          </div>

          <div class="bucket-time">
            <div>
              <div class="time-caption">Time range</div>
              <div class="time-range">${formatRange(bucket, selectedUnit)}</div>
            </div>
            <div class="log-bar">
              <div class="log-fill"></div>
            </div>
          </div>
        `;

        card.addEventListener("click", () => {
          card.classList.toggle("expanded");
        });

        list.appendChild(card);
      });
    }

    search.addEventListener("input", render);
    unit.addEventListener("change", render);

    render();
  })();
</script>