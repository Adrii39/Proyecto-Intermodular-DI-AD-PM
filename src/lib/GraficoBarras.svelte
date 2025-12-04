<script>
    import { Chart } from "chart.js";
    import { onMount } from "svelte";

    let graficoBarras;
    let graficoBarrasInstancia;
    let filtroComunidades = "todas";
    let datosCCAA = [];

    function actualizarBarra() {
        if (!graficoBarras || datosCCAA.length === 0) return;

        let dataFiltrada = datosCCAA;
        if (filtroComunidades !== "todas") {
            dataFiltrada = datosCCAA.filter(
                (d) => d.ccaa === filtroComunidades,
            );
        }

        if (!graficoBarrasInstancia) {
            graficoBarrasInstancia = new Chart(graficoBarras, {
                type: "bar",
                data: {
                    labels: datosCCAA.map((d) => d.ccaa),
                    datasets: [
                        {
                            label: "Centil hijos (Loess)",
                            data: datosCCAA.map((d) => d.value),
                            backgroundColor: "lightblue",
                            borderColor: "black",
                            borderWidth: 2,
                        },
                    ],
                },
                options: {
                    indexAxis: "y",
                    responsive: false,
                    plugins: {
                        legend: { display: false },
                        title: {
                            display: true,
                            text: "Centil hijos por Comunidad Autónoma",
                        },
                    },
                    scales: {
                        x: {
                            title: { display: true, text: "Centil hijos" },
                            min: 0,
                            max: 60,
                        },
                        y: {
                            title: {
                                display: true,
                                text: "Comunidad Autónoma",
                            },
                        },
                    },
                },
            });
        }else{
            graficoBarrasInstancia.data.labels = dataFiltrada.map(d => d.ccaa);
            graficoBarrasInstancia.data.datasets[0].data = dataFiltrada.map(d => d.value);
            graficoBarrasInstancia.update();
        }
    }

    onMount(async () => {
        const resCCAA = await fetch("/data/ranking_ccaa_centil_padres_20.csv");
        const filas = (await resCCAA.text())
            .trim()
            .split("\n")
            .map((r) => r.split(","));

        datosCCAA = filas
            .slice(1)
            .map((r) => ({
                ccaa: r[0],
                value: parseFloat(r[1]),
            }))
            .sort((a, b) => b.value - a.value);

        actualizarBarra();
    });
</script>
<div style="text-align: center; margin-top: 6rem;">
  <h2>Centil medio hijos/CA (Grafico de Barras)</h2>
  <h3>Selecciona Comunidad:</h3>
  <select bind:value={filtroComunidades} on:change={actualizarBarra}>
    <option value="todas">Todas</option>
    {#each datosCCAA as d}
      <option value={d.ccaa}>{d.ccaa}</option>
    {/each}
  </select>
  <canvas bind:this={graficoBarras} width="400" height="400"></canvas>
</div>
<p
  style="max-width: 600px; margin: 0.5rem auto 0;font-size: 0.9rem;"
>
  El gráfico muestra, para cada comunidad autónoma, el centil medio de renta
  que alcanzan los hijos cuando sus padres se sitúan en el percentil 20 de la
  distribución nacional. Cuanto más a la derecha aparece una comunidad, mayor
  es la posición relativa que alcanzan sus hijos en la distribución de renta.
</p>
