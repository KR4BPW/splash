---
title: Welcome to my blog
---

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>QRZ Propagation Dashboard</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            dark: '#0f172a',
            neon: '#00ffe7',
            hot: '#ff006e'
          }
        }
      }
    }
  </script>
</head>
<body class="bg-dark text-white font-mono">
  <div class="max-w-6xl mx-auto p-6 space-y-10">
    <header class="text-center">
      <h1 class="text-4xl md:text-6xl font-bold text-neon drop-shadow-lg">📡 QRZ Propagation Station</h1>
      <p class="text-lg text-gray-300">Live HF Conditions • Solar Weather • Sunspot Stats • Aurora Forecast</p>
    </header>

    <!-- HF Propagation Chart -->
    <section class="bg-gray-800 p-6 rounded-xl shadow-lg">
      <h2 class="text-2xl text-hot mb-4">HF Propagation Chart (SNR)</h2>
      <canvas id="hfChart"></canvas>
    </section>

    <!-- Sunspot Activity Chart -->
    <section class="bg-gray-800 p-6 rounded-xl shadow-lg">
      <h2 class="text-2xl text-hot mb-4">Sunspot Activity (Monthly)</h2>
      <canvas id="sunspotChart"></canvas>
    </section>

    <!-- Solar Weather Panel -->
    <section class="bg-gray-800 p-6 rounded-xl shadow-lg">
      <h2 class="text-2xl text-hot mb-4">Solar X-ray Flux (GOES)</h2>
      <canvas id="xrayChart"></canvas>
    </section>

    <!-- Aurora Forecast Overlay -->
    <section class="bg-gray-800 p-6 rounded-xl shadow-lg">
      <h2 class="text-2xl text-hot mb-4">Aurora Forecast (Ovations Model)</h2>
      <div class="overflow-x-auto">
        <img src="https://services.swpc.noaa.gov/images/animations/ovation/north/latest.jpg" alt="Aurora Forecast" class="w-full rounded-lg border border-neon">
      </div>
    </section>

    <footer class="text-center text-sm text-gray-500 mt-10">
      Made for badass hams. Updated live. ⚡<br>
      Data from NOAA, KC2G, and SWPC.
    </footer>
  </div>

  <script>
    async function renderCharts() {
      // HF Propagation
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
              label: 'HF Propagation (1=Poor, 3=Good)',
              data: snrData,
              backgroundColor: '#00ffe7'
            }]
          },
          options: { scales: { y: { beginAtZero: true, max: 3 } } }
        });
      } catch (err) {
        console.error('HF Chart Error', err);
      }

      // Sunspot Activity
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
              borderColor: '#ff006e',
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
              label: 'X-ray Flux',
              data: recent.map(d => parseFloat(d.flux)),
              borderColor: '#00ffe7',
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
