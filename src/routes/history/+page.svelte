<script lang="ts">
    import { onMount } from "svelte";
    import "chartjs-adapter-date-fns"
    import { Chart, TimeSeriesScale, Legend, LineController, LineElement, Tooltip, LinearScale, PointElement, Filler } from "chart.js";

    Chart.register(TimeSeriesScale, Legend, LineController, LineElement, Tooltip, LinearScale, PointElement, Filler)

    // @ts-nocheck
    import { clearHistory, getHistory } from "../../Shared.svelte";
    
    function populateTable() {
        console.log('dom content loaded')
        let records = getHistory() as Array<Array<any>>
        let historyTable = document.getElementById("historyTable") as HTMLTableElement

        records.reverse().forEach(element => {
            console.log(element)
            let row = historyTable.insertRow()
            let date = new Date(element[0])
        
            row.insertCell().innerHTML = date.toLocaleString()
            let denominator = element[1]
            row.insertCell().innerHTML = `${(20/denominator*20).toFixed(2)}/20`
            
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

        chart.resize()
    }

    function loadPage() {
        populateTable()
        populateChart()
    }

    function handleClearHistory() {
        if (confirm("Clear all records?")) {
            clearHistory()
            document.location.reload()
        }

    }

    onMount(loadPage)

    // historyTable.insertRow()
    // historyTable.cell    


</script>

<style>
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

    * {
        font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    }



</style>

<span>
    <a href=".."><button class="secondary">&lt;&lt; Home</button></a>
    <button class="danger" onclick={handleClearHistory}>Clear History</button>
</span>
<h1>History</h1>
<p>Sorted from newest to oldest. Up to 30000 last entries</p>
<div style="display: flex; max-width: 100%">
    <table style="width: fit-content" id="historyTable" cellpadding="10">
        <thead>
            <tr style="font-weight: bold;">
                <td>Time</td>
                <td>Acuity</td>
            </tr>
        </thead>
        
    </table>

    <canvas style="margin-left: 15px; max-width: 70%" id="recordChart"></canvas>
</div>

