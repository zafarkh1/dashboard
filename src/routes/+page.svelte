<script>
  import { onMount } from 'svelte';
  import ChartComponent from './Chart.svelte';

  const dataStrings = [
    "0,10, 1,13, 2,23, 3,43, 4,53, 5,92, 6,22",
    "0,15, 1,10, 2,20, 3,40, 4,50, 5,85, 6,25",
    "0,12, 1,14, 2,22, 3,41, 4,52, 5,90, 6,20",
    "0,11, 1,16, 2,24, 3,42, 4,51, 5,88, 6,23",
    "0,13, 1,12, 2,21, 3,44, 4,54, 5,91, 6,21",
    "0,14, 1,11, 2,25, 3,45, 4,55, 5,89, 6,24"
  ];

  let ws;
  let dataBuffer = [];
  let allData = [];
  let currentData = [];
  let totals = [];
  let loading = true;

  const parseDataValues = (dataStr) => {
    const dataPairs = dataStr.split(',').map(Number);
    const values = [];
    for (let i = 1; i < dataPairs.length; i += 2) {
      values.push(dataPairs[i]);
    }
    return values;
  };

  const updateDataBuffer = (newData) => {
    dataBuffer.push(newData);
    allData.push(...newData);
    if (dataBuffer.length > 7) {
      dataBuffer.shift();
    }
    currentData = transposeData(dataBuffer);
    calculateTotals();
    loading = currentData.length === 0;
  };

  const transposeData = (data) => {
    const transposed = [];
    const numValues = data[0].length;
    for (let i = 0; i < numValues; i++) {
      const values = [];
      for (let j = 0; j < data.length; j++) {
        values.push(data[j][i]);
      }
      transposed.push(values);
    }
    return transposed;
  };

  const calculateTotals = () => {
    totals = allData.reduce((acc, value, index) => {
      const dataIndex = index % 7;
      acc[dataIndex] = (acc[dataIndex] || 0) + value;
      return acc;
    }, []);
  };

  onMount(() => {
    let index = 0;
    const interval = setInterval(() => {
      const newData = parseDataValues(dataStrings[index]);
      updateDataBuffer(newData);
      index = (index + 1) % dataStrings.length;
    }, 1000);

    return () => clearInterval(interval);
  });

  // onMount(() => {
  // ws = new WebSocket('ws://10.0.1.99:81');
  // ws.onopen = () => {
  //   console.log('WebSocket connection established');
  // };
  //
  // ws.onmessage = (event) => {
  //   const dataStr = event.data;
  //   console.log(event.data)
  //   const newData = parseDataValues(dataStr);
  //   updateDataBuffer(newData);
  //
  // };
  //
  // ws.onerror = (error) => {
  //   console.error('WebSocket error:', error);
  // };
  //
  // ws.onclose = () => {
  //   console.log('WebSocket connection closed');
  // };
  //
  // const chartUpdateInterval = setInterval(() => {
  //   currentData = transposeData(dataBuffer);
  //   calculateTotals();
  // }, 20000);
  //
  // return () => {
  //   clearInterval(chartUpdateInterval);
  //   ws.close();
  // };
  // });

  const colorScheme = [
    {bg: '#FFF3CD', bar: 'rgba(255, 159, 64, 0.2)', border: 'rgba(255, 159, 64, 0.2)'},
    {bg: '#E3F2FD', bar: 'rgba(66, 165, 245, 0.2)', border: 'rgba(66, 165, 245, 1)'},
    {bg: '#E8F5E9', bar: 'rgba(102, 187, 106, 0.2)', border: 'rgba(102, 187, 106, 1)'},
    {bg: '#FFEBEE', bar: 'rgba(239, 83, 80, 0.2)', border: 'rgba(239, 83, 80, 1)'},
    {bg: '#F3E5F5', bar: 'rgba(171, 71, 188, 0.2)', border: 'rgba(171, 71, 188, 1)'},
    {bg: '#FFE0B2', bar: 'rgba(251, 140, 0, 0.2)', border: 'rgba(251, 140, 0, 1)'}
  ];

  function getColor(index) {
    return colorScheme[index % colorScheme.length].bg;
  }

  function getBarColor(index, isBackground) {
    return isBackground ? colorScheme[index % colorScheme.length].bar : colorScheme[index % colorScheme.length].border;
  }

</script>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    display: block;
    font-family: 'Roboto', sans-serif;
  }

  :global(*) {
    box-sizing: border-box;
  }

  .loader {
    text-align: center;
    display: flex;
    justify-content: space-around;
    width: 70px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }

  .loader div {
    width: 15px;
    height: 15px;
    background-color: #3498db;
    border-radius: 50%;
    animation: pulse 0.6s infinite alternate;
  }

  .loader div:nth-child(2) {
    animation-delay: 0.2s;
  }

  .loader div:nth-child(3) {
    animation-delay: 0.4s;
  }

  @keyframes pulse {
    to {
      opacity: 0.3;
      transform: scale(0.7);
    }
  }
</style>

{#if loading}
    <div class="loader">
        <div></div>
        <div></div>
        <div></div>
    </div>
{/if}

<div class="chart-container">
    {#each currentData as data, index}
        <ChartComponent
                chartId={index}
                {data}
                total={totals[index] || 0}
                backgroundColor={getBarColor(index, true)}
                borderColor={getBarColor(index, false)}
                style="background-color: {getColor(index)};"
        />
    {/each}
</div>