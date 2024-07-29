<script>
  import { onMount, afterUpdate } from "svelte";
  import Chart from "chart.js/auto";

  export let chartId = '';
  export let data = [];
  export let total = 0;
  export let scale = 100;

  export let backgroundColor;
  export let borderColor;

  let chartContainer;
  let chart;

  const createChart = () => {
    if (!chartContainer || !data.length) return;
    const ctx = chartContainer.getContext('2d');
    if (chart) chart.destroy();

    chart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: ['-60m', '-50m', '-40m', '-30m', '-20m', '-10m', 'Now'],
        datasets: [
          {
            label: 'Capacity',
            data: data,
            backgroundColor: backgroundColor,
            borderColor: borderColor,
            borderWidth: 1,
          },
        ],
      },
      options: {
        responsive: true,
        plugins: {
          legend: {
            position: 'top',
            labels: {
              font: {size: 14},
            },
          },
          tooltip: {
            callbacks: {
              footer: (tooltipItems) => {
                return 'Total: ' + total;
              }
            }
          }
        },
        scales: {
          x: {
            grid: {display: false},
            ticks: {font: {size: 12}},
          },
          y: {
            beginAtZero: true,
            grid: {color: 'rgba(200, 200, 200, 0.3)'},
            ticks: {font: {size: 12}},
          },
        },
      },
    });

    chart.update()
  };

  onMount(() => {
    createChart();
  });

  afterUpdate(() => {
    if (data.length) {
      createChart();
    }
  });

  // $: if (data.length) {
  //   createChart();
  // }
</script>

<style>
  .card {
    width: 33.333333vw;
    height: 50vh;
    background-color: var(--card-bg);
    padding: 8px;
    text-align: center;
    display: inline-block;
  }

  h3 {
    margin: 0;
  }

  .stat {
    font-size: 24px;
    margin: 10px 0;
  }

  .label {
    font-size: 16px;
    color: #777;
  }

  canvas {
    max-width: 100%;
    margin: 0 auto;
    pointer-events: none;
  }
</style>

<div class="card" style="--card-bg: { backgroundColor }">
    <div class="stats">
        <div class="stat">
            <h3>Total: { (total / scale).toFixed(2) }<small>L</small></h3>
            <small><span>{ data[data.length - 1] }<small>ticks</small></span></small>
            <span>{ (data[data.length - 1] / scale).toFixed(2) }<small>L</small></span>
            <div class="label">At last 10m</div>
        </div>
    </div>
    <canvas bind:this={chartContainer} id={`chart-${chartId}`}></canvas>
</div>
