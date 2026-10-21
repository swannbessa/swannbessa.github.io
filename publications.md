---
layout: default
title: Publications
permalink: /publications/
---

# Publications

<div class="pub pub-wide">
  <p class="pub-authors"><span class="me">S. Bessa</span>, P.-E. Mazare, G. Szilvasy, H. Touvron, D. Picard, M. Douze, H. Jégou</p>
  <p class="entry-title"><i>e</i>-means: Outperforming k-means (in One-Tenth the Epochs)</p>
  <p class="entry-venue">ICLR 2027 (submission)</p>
  <p class="entry-meta"><a href="https://openreview.net/pdf?id=pt14gnV3dl">PDF (OpenReview)</a></p>
  <p><i>e</i>-means is a modern optimization rule for k-means, built around mini-batches with an exponential moving average rule.</p>

  <div class="widget-wrap" id="anim2d">
    <div class="anim-row">
      <div>
        <div class="panel-title" style="color:#d62728">Lloyd (full batch)</div>
        <canvas class="left" width="380" height="380"></canvas>
        <div class="readout" data-r="lloyd"></div>
      </div>
      <div>
        <div class="panel-title ours">e-means (batch and step from the scaling laws)</div>
        <canvas class="right" width="380" height="380"></canvas>
        <div class="readout" data-r="ema"></div>
      </div>
      <div>
        <div class="panel-title" style="color:#55555a">Training MSE</div>
        <canvas class="chart" width="360" height="380"></canvas>
      </div>
    </div>
    <div class="controls">
      <span style="font-size:0.85em;color:#55555a">epochs</span>
      <input type="range">
      <span class="val" style="font-size:0.85em;color:#55555a">0.0</span>
    </div>
  </div>
  <p class="entry-meta">2-D toy distribution adapted from Huy Vo's work on hierarchical k-means for data curation. Gray dots are the data points; colored dots are the centroids. Lloyd moves once per epoch (the staircase); e-means moves roughly 176 times per epoch. Plays automatically and loops; drag the slider to scrub.</p>

  <script>
  (function () {
    "use strict";
    var root = document.getElementById("anim2d");
    if (!root || root.dataset.wired) return;
    root.dataset.wired = "1";
    fetch("{{ '/assets/toy-anim-data.json' | relative_url }}").then(function (r) { return r.json(); }).then(function (D) {
      var cvL = root.querySelector("canvas.left"), cvE = root.querySelector("canvas.right");
      var cvC = root.querySelector("canvas.chart");
      var slider = root.querySelector("input[type=range]");
      var tval = root.querySelector(".val");
      var rL = root.querySelector("[data-r=lloyd]"), rE = root.querySelector("[data-r=ema]");
      var TMAX = D.lloyd.mse.length - 1, EMA_T = D.ema.passes[D.ema.passes.length - 1];
      var GRID = TMAX > 10 ? 5 : 1;
      var spe = Math.round(1 / D.ema.passes[1]);
      var BLUE = "#1f77b4", RED = "#d62728", DOT = "rgba(120,120,128,0.30)";

      function setup(cv) {
        var w = cv.width, h = cv.height;
        cv.style.width = w + "px"; cv.style.height = h + "px";
        cv.width = w * 3; cv.height = h * 3;
        var ctx = cv.getContext("2d");
        ctx.scale(3, 3);
        return { ctx: ctx, w: w, h: h };
      }
      var L = setup(cvL), E = setup(cvE), C = setup(cvC);

      function toPx(p, P) {
        var s = Math.min(P.w, P.h) / 6.2;
        return [P.w / 2 + p[0] * s, P.h / 2 - p[1] * s];
      }

      function drawPanel(P, cents, color) {
        var ctx = P.ctx;
        ctx.clearRect(0, 0, P.w, P.h);
        ctx.fillStyle = "#fbfbfa"; ctx.fillRect(0, 0, P.w, P.h);
        ctx.strokeStyle = "#d4d4d8"; ctx.lineWidth = 1; ctx.strokeRect(0.5, 0.5, P.w - 1, P.h - 1);
        ctx.fillStyle = DOT;
        D.points.forEach(function (p) { var q = toPx(p, P); ctx.fillRect(q[0] - 1.1, q[1] - 1.1, 2.2, 2.2); });
        cents.forEach(function (c) {
          var q = toPx(c, P);
          ctx.beginPath(); ctx.arc(q[0], q[1], 4.6, 0, 2 * Math.PI);
          ctx.fillStyle = color; ctx.fill();
          ctx.lineWidth = 1.1; ctx.strokeStyle = "#fbfbfa"; ctx.stroke();
        });
      }

      var lloydIn = D.lloyd.mse.slice(0, TMAX + 1);
      var yMin = Math.min.apply(null, lloydIn.concat(D.ema.mse));
      var yMax = Math.max.apply(null, lloydIn.concat(D.ema.mse));
      var X0 = 62, X1 = C.w - 12, Y0 = C.h - 56, Y1 = 4;
      function chartXY(t, m) {
        var lo = yMin * 0.96, hi = yMax * 1.02;
        return [X0 + (X1 - X0) * t / TMAX, Y0 - (Y0 - Y1) * (m - lo) / (hi - lo)];
      }
      function drawChart(t) {
        var ctx = C.ctx;
        ctx.clearRect(0, 0, C.w, C.h);
        ctx.font = "13px -apple-system,Arial,sans-serif";
        ctx.strokeStyle = "#ededed"; ctx.lineWidth = 1;
        var g, p;
        for (g = 0; g <= TMAX; g += GRID) { p = chartXY(g, yMin); ctx.beginPath(); ctx.moveTo(p[0], Y0); ctx.lineTo(p[0], Y1); ctx.stroke(); }
        var span = yMax - yMin, ticks = [];
        for (var i = 0; i <= 5; i++) ticks.push(yMin + span * i / 5);
        ctx.strokeStyle = "#333336"; ctx.lineWidth = 1.1; ctx.strokeRect(X0, Y1, X1 - X0, Y0 - Y1);
        ctx.fillStyle = "#333336"; ctx.textAlign = "center";
        for (g = 0; g <= TMAX; g += GRID) { p = chartXY(g, yMin); ctx.fillText(String(g), p[0], Y0 + 18); }
        ctx.fillText("epochs", (X0 + X1) / 2, Y0 + 38);
        ctx.textAlign = "right";
        ticks.forEach(function (v) { var q = chartXY(0, v); if (q[1] > Y1 + 6 && q[1] < Y0 - 2) ctx.fillText(v.toFixed(3), X0 - 6, q[1] + 4); });
        ctx.strokeStyle = RED; ctx.lineWidth = 2.2; ctx.beginPath();
        var q0 = chartXY(0, D.lloyd.mse[0]); ctx.moveTo(q0[0], q0[1]);
        for (var i2 = 1; i2 <= Math.min(Math.floor(t + 1e-9), TMAX); i2++) {
          var a = chartXY(i2, D.lloyd.mse[i2 - 1]), b2 = chartXY(i2, D.lloyd.mse[i2]);
          ctx.lineTo(a[0], a[1]); ctx.lineTo(b2[0], b2[1]);
        }
        var cur = chartXY(t, D.lloyd.mse[Math.min(Math.floor(t + 1e-9), TMAX)]); ctx.lineTo(cur[0], cur[1]); ctx.stroke();
        ctx.strokeStyle = BLUE; ctx.lineWidth = 2.6; ctx.beginPath();
        var last = null;
        D.ema.mse.forEach(function (m, k) {
          var tt = D.ema.passes[k];
          if (tt > t + 1e-9) return;
          var q = chartXY(tt, m);
          if (k === 0) ctx.moveTo(q[0], q[1]); else ctx.lineTo(q[0], q[1]);
          last = m;
        });
        ctx.stroke();
        if (t >= EMA_T - 1e-9) {
          var e1 = chartXY(EMA_T, last), e2 = chartXY(TMAX, last);
          ctx.save(); ctx.setLineDash([6, 5]); ctx.strokeStyle = BLUE; ctx.lineWidth = 1.6;
          ctx.beginPath(); ctx.moveTo(e1[0], e1[1]); ctx.lineTo(e2[0], e2[1]); ctx.stroke(); ctx.restore();
          ctx.fillStyle = BLUE; ctx.textAlign = "left";
          ctx.font = "italic 13px -apple-system,Arial,sans-serif";
          ctx.fillText("e-means after " + EMA_T + " epochs", chartXY(Math.max(1.4 * EMA_T, TMAX / 2.6), last)[0], e1[1] - 7);
        }
        var c0 = chartXY(t, yMin);
        ctx.strokeStyle = "rgba(29,29,31,0.55)"; ctx.lineWidth = 1.5;
        ctx.beginPath(); ctx.moveTo(c0[0], Y0); ctx.lineTo(c0[0], Y1); ctx.stroke();
      }

      function show(t) {
        var li = Math.min(Math.floor(t + 1e-9), TMAX);
        var ei = Math.min(Math.round(t * spe), D.ema.centroids.length - 1);
        drawPanel(L, D.lloyd.centroids[li], RED);
        drawPanel(E, D.ema.centroids[ei], BLUE);
        drawChart(t);
        tval.textContent = t.toFixed(1);
        rL.innerHTML = "Training MSE <b>" + D.lloyd.mse[li].toFixed(4) + "</b> after " + li + (li === 1 ? " epoch" : " epochs");
        rE.innerHTML = "Training MSE <b>" + D.ema.mse[ei].toFixed(4) + "</b>" + (t >= EMA_T ? " (finished after " + EMA_T + " epochs)" : "");
      }
      slider.min = 0; slider.max = TMAX; slider.step = 0.1; slider.value = 0;
      slider.addEventListener("input", function () { show(parseFloat(slider.value)); });

      var timer = null;
      function loop() {
        var t = 0, last = null, holdUntil = null;
        function frame(ts) {
          if (last === null) last = ts;
          var dt = (ts - last) / 1000; last = ts;
          if (holdUntil !== null) {
            if (ts >= holdUntil) { timer = requestAnimationFrame(function (ts2) { last = null; holdUntil = null; t = 0; frame(ts2); }); return; }
            timer = requestAnimationFrame(frame);
            return;
          }
          t += dt * (t < EMA_T ? 0.6 : 3.0);
          if (t >= TMAX) { t = TMAX; show(t); slider.value = t; holdUntil = ts + 1800; timer = requestAnimationFrame(frame); return; }
          slider.value = t; show(t);
          timer = requestAnimationFrame(frame);
        }
        timer = requestAnimationFrame(frame);
      }
      show(0);
      loop();
    });
  })();
  </script>
</div>

<div class="pub">
  <p class="pub-authors"><span class="me">S. Bessa</span>, P. Fernandez, G. Szilvasy, M. Douze, H. Jégou</p>
  <p class="entry-title">Cluster with Auctions for Vector Search<img class="entry-logo" src="{{ '/assets/cwa-logo.png' | relative_url }}" alt="CwA logo: an auction gavel fanned by colored cluster graphs"></p>
  <p class="entry-venue">NeurIPS 2026</p>
  <p class="entry-meta"><a href="https://arxiv.org/abs/2607.13728">arXiv:2607.13728</a></p>
  <img class="pub-figure" src="{{ '/assets/cwa-figure.png' | relative_url }}" alt="CwA alternates training the neural probing function and solving the auction assignment">
</div>

<div class="pub">
  <p class="pub-authors"><span class="me">S. Bessa</span>, D. Dabert, M. Bourgeat, Q. Cappart, L. Rousseau</p>
  <p class="entry-title">Learning Valid Dual Bounds in Constraint Programming: Boosted Lagrangian Decomposition with Self-Supervised Learning</p>
  <p class="entry-venue">AAAI 2025</p>
  <p class="entry-meta"><a href="https://arxiv.org/abs/2408.12695">arXiv:2408.12695</a></p>
  <img class="pub-figure" src="{{ '/assets/lagrangian-figure.png' | relative_url }}" alt="Learning Lagrangian multipliers from a graph representation of the constraint program via self-supervised learning">
</div>

<p><a href="{{ site.author.scholar }}">Google Scholar</a></p>
