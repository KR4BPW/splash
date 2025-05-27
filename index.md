---
layout: default
title: QRZ Tactical Dashboard
description: Live comms data for hardcore operators.
---

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ page.title }}</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            combat: '#1a1a1a',
            ops: '#2e2e2e',
            highlight: '#00ff87',
            warning: '#ff4444',
          },
          fontFamily: {
            mono: ['"Share Tech Mono"', 'monospace']
          }
        }
      }
    }
  </script>
  <link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&display=swap" rel="stylesheet">
</head>
<body class="bg-combat text-highlight font-mono">
  <div class="max-w-7xl mx-auto p-8 space-y-12">
    <header class="text-center border-b border-highlight pb-6">
      <h1 class="text-5xl font-bold uppercase tracking-widest">🛰 QRZ Tactical Ops</h1>
      <p class="text-md text-gray-400 mt-2">Signal Intel • Solar Threats • Tactical Awareness</p>
    </header>

    <!-- HF Propagation -->
    <section class="bg-ops p-6 rounded-lg border border-highlight shadow-md">
      <h2 class="text-2xl mb-4 font-bold text-warning">HF Propagation Signal Levels</h2>
      <canvas id="hfChart"></canvas>
    </section>

    <!-- Sunspot Activity -->
    <section class="bg-ops p-6 rounded-lg border border-highlight shadow-md">
      <h2 class="text-2xl mb-4 font-bold text-warning">Sunspot Activity (Last 24 Months)</h2>
      <canvas id="sunspotChart"></canvas>
    </section>

    <!-- Solar Weather -->
    <section class="bg-ops p-6 rounded-lg border border-highlight shadow-md">
      <h2 class="text-2xl mb-4 font-bold text-warning">Solar Threat Level (X-ray Flux)</h2>
      <canvas id="xrayChart"></canvas>
    </section>

    <!-- Aurora Overlay -->
    <section class="bg-ops p-6 rounded-lg border border-highlight shadow-md">
      <h2 class="text-2xl mb-4 font-bold text-warning">Northern Hemisphere Aurora Risk</h2>
      <img src="https://services.swpc.noaa.gov/images/animations/ovation/north/latest.jpg" alt="Aurora Forecast" class="rounded shadow-md border border-highlight">
    </section>

    <footer class="text-center text-gray-500 pt-10 border-t border-highlight">
      <p>Data stream: NOAA / SWPC / KC2G | Assembled with Jekyll and 💥</p>
    </footer>
  </div>

  <script>
    async function renderCharts() {
      // HF Chart
      try {
        const res = await fetch('https://prop.kc2g.com/propagation.json');
        const data = await res.json();
        const bands = Object.keys(data).filter(b => b.includes('m') && data[b].now);
        const snrData = bands.map(b => data[b].now === 'Good' ? 3 : data[b].now === 'Fair' ? 2 : 1);

        new Chart(document.getElementById('hfChart'), {
          type: 'bar',
          data: {
            labels: bands,
            datasets: [{
              label: 'Signal Quality (3=Good, 1=Poor)',
              data: snrData,
              backgroundColor: '#00ff87'
            }]
          },
          options: { scales: { y: { beginAtZero: true, max: 3 } } }
        });
      } catch (err) {
        console.error('HF Chart Error', err);
      }

      // Sunspot Chart
      try {
        const res = await fetch('https://services.swpc.noaa.gov/json/solar-cycle/observed-solar-cycle-indices.json');
        const data = await res.json();
        const recent = data.slice(-24);

        new Chart(document.getElementById('sunspotChart'), {
          type: 'line',
          data: {
            labels: recent.map(d => d.time_tag.substring(0, 7)),
            datasets: [{
              label: 'Sunspot Number',
              data: recent.map(d => d.ssng),
              borderColor: '#ff4444',
              fill: false
            }]
          },
          options: { responsive: true }
        });
      } catch (err) {
        console.error('Sunspot Chart Error', err);
      }

      // X-ray Flux
      try {
        const res = await fetch('https://services.swpc.noaa.gov/json/goes/primary/xrays-1-day.json');
        const data = await res.json();
        const recent = data.slice(-100);

        new Chart(document.getElementById('xrayChart'), {
          type: 'line',
          data: {
            labels: recent.map(d => new Date(d.time_tag).toLocaleTimeString()),
            datasets: [{
              label: 'Flux (W/m²)',
              data: recent.map(d => parseFloat(d.flux)),
              borderColor: '#00ff87',
              fill: true
            }]
          },
          options: {
            responsive: true,
            scales: { y: { type: 'logarithmic' } }
          }
        });
      } catch (err) {
        console.error('X-ray Chart Error', err);
      }
    }

    renderCharts();
  </script>
</body>
</html>
