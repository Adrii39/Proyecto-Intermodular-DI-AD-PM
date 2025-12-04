<script>
  import GraficoBarras from "./lib/GraficoBarras.svelte";
  import GraficoBurbujas from "./lib/GraficoBurbujas.svelte";
  import GraficoSankey from "./lib/GraficoSankey.svelte";
</script>

<main style="text-align: center;">
  <h1>Ascensor Social</h1>
  
  <hr>
  <br>
  <GraficoBurbujas />
  <br>
  <GraficoBarras />
  <br>
  <GraficoSankey />
</main>