<script>
    import { onMount } from "svelte";
    import { Chart } from "chart.js/auto";


    let graficoBurbujas;
    let chart;
    let datosBurbujas = [];

    onMount(async() => {

        const res = await fetch ("/data/curva_movilidad_nacional.csv");
        const rows = (await res.text()).trim().split("\n").map(r => r.split(","));
        datosBurbujas = rows.slice(1);


        chart = new Chart (graficoBurbujas, {
            type: 'bubble',
            data: {
                datasets: [
                    {
                        label: "Prueba",
                        data: [{ x: 10, y: 20, r: 5}],
                        backgroundColor: "red"
                    }

                ]},
            options: {
                responsive: false,
                plugins: {
                    legend: { position: 'top'},
                    title: { display: true, text: 'Curva de movilidad intergeneracional' }
                }
            }
        });
    });

</script>

<div style="text-align: center;">
    <canvas bind:this={graficoBurbujas} width="600" height="500"></canvas>
</div>