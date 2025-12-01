<script>
    import { Chart } from "chart.js";
    import { onMount } from "svelte";

    let graficoBarras;
    let graficoBarrasInstancia;
    let filtroComunidades = "todas";
    let datosCCAA = [];

    function actualizarBarra() {
        if (!graficoBarras || datosCCAA.length === 0) return;
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


