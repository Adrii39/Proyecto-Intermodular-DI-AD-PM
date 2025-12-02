<script>
  import GraficoBarras from "./lib/GraficoBarras.svelte";
  import GraficoBurbujas from "./lib/GraficoBurbujas.svelte";
  import GraficoSankey from "./lib/GraficoSankey.svelte";
</script>

<main style="text-align: center;">
  <h1>Primer gráfico</h1>
  <br>
  <hr>
  
  <h1>Ranking por CCAA</h1>
  <GraficoBurbujas />
  <GraficoBarras />
  <GraficoSankey />
</main>