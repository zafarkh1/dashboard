<script>
  import BarChart from './Chart.svelte';
  import {onMount} from 'svelte';

  let ws;
  let dataBuffer = [];
  let labels = [];
  let currentData = [];
  let loading = true;

  // Function to parse the data string into values
  const parseDataValues = (dataStr) => {
    const dataPairs = dataStr.split(',').map(Number);
    const values = [];
    for (let i = 1; i < dataPairs.length; i += 2) {
      values.push(dataPairs[i]);
    }
    return values;
  };

  // Function to update data buffer
  const updateDataBuffer = (newData) => {
    dataBuffer.push(newData);
    if (dataBuffer.length > 10) {
      dataBuffer.shift();
    }
    currentData = transposeData(dataBuffer);
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

  onMount(() => {
    ws = new WebSocket('ws://10.0.1.31:81');

    ws.onopen = () => {
      console.log('WebSocket connection established');
    };

    ws.onmessage = (event) => {
      const dataStr = event.data;
      const newData = parseDataValues(dataStr);
      updateDataBuffer(newData);

      labels = generateLabels(currentData.length);
      console.log('Current data:', currentData);
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    ws.onclose = () => {
      console.log('WebSocket connection closed');
    };
  });

  const generateLabels = (numElements) => {
    return Array.from({length: numElements}, (_, i) => `${i + 1}`);
  };
</script>

<style>
  .chart-container {
    width: 32%;
    height: calc(100vh / 2);
    display: inline-block;
    vertical-align: top;
    margin-right: 10px;
  }

  .chart-container:last-child {
    margin-right: 0;
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

{#each currentData as data, index}
    <div class="chart-container">
        <BarChart {labels} {data} chartTitle={`Data Set ${index + 1}`}/>
    </div>
{/each}