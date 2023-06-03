<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  export let datapoints;
  export let dataType;
  export let labels;
  export let stockName

  let ctx;
  let myChart;
  
  $: if (myChart){
    myChart.data.datasets[0].data = datapoints[dataType];
    myChart.data.labels = labels
    myChart.update();
  }

  onMount(async () => {
      myChart = new Chart(ctx, {
          type: "line",
          data: {
              labels: labels,
              datasets: [{
                label: stockName, 
                data: [],
                borderWidth: 1
              }]
          },
          options: {
              scales: {
                  y: {
                      beginAtZero: false,
                      title: {
                        text: 'Price (INR)',
                        display: true
                      }
                  },
                  x: {
                      title:{
                        text: 'Date',
                        display: true
                      }
                  }
              },
              responsive: true,
              maintainAspectRatio: false
          },
      });
  });
</script>

<div class="card bg-gradient-info container mt-1" style="height: 500px">
  <canvas id="myChart" bind:this={ctx} />
</div>
