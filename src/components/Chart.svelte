<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  export let labels;
  export let datapoints;
  export let stockName;


  let ctx;
  let myChart
  
  $: if (myChart){
    myChart.data.datasets[0].data = datapoints
    myChart.data.datasets[0].label = stockName
    myChart.data.labels = labels;
    myChart.update()
  }
  

  onMount(async () => {
      myChart = new Chart(ctx, {
          type: "line",
          data: {
              labels: labels,
              datasets: [
                  {
                      label: stockName,
                      data: datapoints,
                      backgroundColor: [
                          "rgba(255, 99, 132, 0.2)",
                          "rgba(54, 162, 235, 0.2)",
                          "rgba(255, 206, 86, 0.2)",
                          "rgba(75, 192, 192, 0.2)",
                          "rgba(153, 102, 255, 0.2)",
                          "rgba(255, 159, 64, 0.2)",
                      ],
                      borderColor: [
                          "rgba(255, 99, 132, 1)",
                          "rgba(54, 162, 235, 1)",
                          "rgba(255, 206, 86, 1)",
                          "rgba(75, 192, 192, 1)",
                          "rgba(153, 102, 255, 1)",
                          "rgba(255, 159, 64, 1)",
                      ],
                      borderWidth: 1,
                  },
              ],
          },
          options: {
              scales: {
                  y: {
                      beginAtZero: true,
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
          },
      });
  });
</script>

<div class="card bg-gradient-info">
  <canvas id="myChart" width="400" height="100" bind:this={ctx} />
</div>
