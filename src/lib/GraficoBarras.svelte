<script>
  import { onMount } from "svelte";

  let graficoBarras;
  let datosCCAA = [];

  onMount(async () => {
    const resCCAA = await fetch("/data/ranking_ccaa_centil_padres_20.csv");
    const filas = (await resCCAA.text()).trim().split("\n").map(r => r.split(","));

    datosCCAA = filas.slice(1).map(r => ({
      ccaa: r[0],
      value: parseFloat(r[1])
    })).sort((a,b) => b.value - a.value);

    console.log("Datos cargados:", datosCCAA);
  });
</script>

<div style="text-align: center;">
  <h2>Selecciona Comunidad:</h2>
  <canvas bind:this={graficoBarras} width="400" height="400"></canvas>
</div>