<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.3/font/bootstrap-icons.min.css">

<script lang="ts">
    import { onMount } from "svelte";
    import "chartjs-adapter-date-fns"
    import { Chart, TimeSeriesScale, Legend, LineController, LineElement, Tooltip, LinearScale, PointElement, Filler } from "chart.js";

    Chart.register(TimeSeriesScale, Legend, LineController, LineElement, Tooltip, LinearScale, PointElement, Filler)

    // @ts-nocheck
    import { clearHistory, getHistory } from "../../Shared.svelte";
    
    let tableData: Array<Array<any>> = []

    function populateTable() {
        tableData = []
        let records = getHistory() as Array<Array<any>>

        records.reverse().forEach(element => {
            console.log(element)
            let date = new Date(element[0])
            
            let denominator = element[1]
            
            tableData.push([date.toLocaleString(), `${(20/denominator*20).toFixed(2)}/20`])

        });
    }
    // @ts-nocheck
    // let chart: Chart = undefined
    function populateChart() {

        let records = getHistory() as Array<Array<any>>
        let dataSet: Array<any> = []
        records.forEach(element => {
            let denominator = element[1]
            dataSet.push({x: new Date(element[0]), y: (20/denominator*20).toFixed(2)})
        })

        const ctx = document.getElementById('recordChart') as HTMLCanvasElement
        let chart = new Chart(ctx, {
            type: "line",
            data: {
                datasets: [{
                    label: "Acuity",
                    data: dataSet,
                    borderColor: "#6DB424",
                    fill: true,
                    backgroundColor: "#6DB42430"

                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                scales: {
                    x: {
                        type: 'timeseries',
                    },
                    y: {
                        min: 0,
                        max: 20,
                    }
            }
    }
        })   
        chart.reset()
        chart.resize()
    }

    function loadPage() {
        populateTable()
        populateChart()
    }

    function handleClearHistory() {
        if (confirm("Clear all records?")) {
            clearHistory()
            populateTable()
            populateChart()
        }

    }

    onMount(loadPage)

    // historyTable.insertRow()
    // historyTable.cell    


</script>

<style>

table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td, th {
  border: 1px solid #cacaca;
  text-align: left;
  padding: 8px;
}


    button {
        padding: 10px;
        text-transform: capitalize;
        background-color: rgb(33, 130, 227);
        color: white;
        font-size: 1rem;
        border: solid transparent;
        border-radius: 5px;
    }



    button.secondary {
        background-color: gray;
    }


    button.danger {
        background-color:  #cd0000;
    }

    button:hover {
        filter: brightness(0.7);

    }

    td.highlight {
        color: #447017;
        font-weight: bold;
    }


    * {
        font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    }



</style>

<span>
    <a href=".."><button class="secondary">&lt;&lt; Home</button></a>
    <button class="danger" onclick={handleClearHistory}>Clear History</button>
</span>
<h1><i style="color: grey;" class="bi bi-clock-history"></i> History</h1>
<p>Sorted from newest to oldest. Up to 30000 last entries</p>
<div style="display: flex; max-width: 100%;">
    <table style="width: fit-content;" id="historyTable" cellpadding="10">
        <thead>
            <tr style="font-weight: bold;">
                <td>Time</td>
                <td>Acuity</td>
            </tr>
        </thead>
        <tbody>

            {#each tableData as record}
                <tr>
                    <td>{record[0]}</td>
                    <td class="highlight">{record[1]}</td>
                </tr>
            {/each}

        </tbody>

        
    </table>

    <canvas style="margin-left: 15px; max-width: 70%; max-height: 300px" id="recordChart"></canvas>
</div>

