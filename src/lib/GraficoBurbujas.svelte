<script>
    import { onMount } from "svelte";
    import { Chart } from "chart.js/auto";


    let graficoBurbujas;
    let chart;
    let datosBurbujas = [];
    let filtroBurbuja = "media";

    function actualizarGraficoBurbuja() {

        if (!chart) return;

        const puntos = datosBurbujas
        .filter(r => r[3] === filtroBurbuja)
        .map(r => ({
            x: parseInt(r[2]),
            y: parseFloat(r[5]).toFixed(1),
            r: 8
        }));

        chart.data.datasets = [{
            label: 'Movilidad Intergeneracional',
            data: puntos,
            fill: false,
            borderColor: "lightblue",
            backgroundColor: "black",
            tension: 0.3,
            pointRadius: 5
        }];

        chart.update();
    }

    onMount(async() => {

        const res = await fetch ("/data/curva_movilidad_nacional.csv");
        const rows = (await res.text()).trim().split("\n").map(r => r.split(","));
        datosBurbujas = rows.slice(1);


        chart = new Chart (graficoBurbujas, {
            type: 'bubble',
            data: { datasets: [] },
            options: {
                responsive: false,
                plugins: {
                    legend: { position: 'top', onClick: null},
                    title: { display: true, text: 'Curva de movilidad intergeneracional' },
                    tooltip: {
                        callbacks: {
                            label: function(context) {
                                const point = context.raw;
                                return `Padres: ${point['x']}, Hijos: ${point['y']}`;

                            }
                        }
                    }
                },
                scales: {
                    x: { type: 'linear', title: { display:true, text: 'Centil de padres' }, min: 0, max: 100},
                    y: { title: { display: true, text: 'Centil de hijos' }, min: 0, max: 100 }
                }
            }
        });

        actualizarGraficoBurbuja();
    });

</script>

<div style="text-align: center; margin-top: 6rem;">
    <h2>Curva de movilidad intergeneracional (Gráfico de Burbujas)</h2>
    <select bind:value={filtroBurbuja} on:change={actualizarGraficoBurbuja}>
        <option value="mediana">Mediana</option>
        <option value="media">Media</option>
    </select>
    <canvas bind:this={graficoBurbujas} width="600" height="500"></canvas>
      <p
    style="max-width: 600px; margin: 0.5rem auto 0;font-size: 0.9rem;"
  >
    Este gráfico de burbujas muestra, para cada centil de renta de los padres
    (eje horizontal), el centil que alcanzan sus hijos en la distribución de
    renta (eje vertical). El desplegable permite alternar entre usar la
    mediana o la media como medida de la
    posición de los hijos para cada centil de padres.
  </p>
</div>