<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  export let labels = [];
  export let data = [];
  export let chartTitle = "Chart Title";

  let chartContainer;
  let chart;
  let total = 0;

  // Function to calculate the total of data
  const calculateTotal = () => {
    total = data.reduce((sum, value) => sum + value, 0);
  };

  // Function to create or update the chart
  const createChart = () => {
    if (!chartContainer) return; // Ensure chartContainer is defined
    const ctx = chartContainer.getContext('2d');
    if (chart) chart.destroy(); // Destroy the previous chart instance

    chart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: labels,
        datasets: [
          {
            label: 'Capacity',
            data: data,
            backgroundColor: 'rgba(255, 159, 64, 0.6)',
            borderColor: 'rgba(255, 159, 64, 1)',
            borderWidth: 1,
            barThickness: 10,
            hoverBackgroundColor: 'rgba(255, 159, 64, 0.8)',
            hoverBorderColor: 'rgba(255, 159, 64, 1)',
          },
        ],
      },
      options: {
        responsive: true,
        plugins: {
          legend: {
            position: 'top',
            labels: {
              font: { size: 14 },
            },
          },
          title: {
            display: true,
            text: chartTitle,
            font: { size: 18 },
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
            grid: { display: false },
            ticks: { font: { size: 12 } },
          },
          y: {
            beginAtZero: true,
            grid: { color: 'rgba(200, 200, 200, 0.3)' },
            ticks: { font: { size: 12 } },
          },
        },
      },
    });
  };

  onMount(() => {
    calculateTotal();
    createChart();
  });

  $: if (data.length) {
    calculateTotal();
    createChart();
  }
</script>

<style>
  canvas {
    max-width: 100%;
    /*height: 45vh !important;*/
  }

  .total {
    text-align: center;
    font-weight: bold;
    margin-top: 10px;
  }
</style>

<canvas bind:this={chartContainer}></canvas>
<div class="total">Total: {total}</div>
