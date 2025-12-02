<script>
  import { onMount } from "svelte";

  
  let datos = [];

  onMount(async () => {
    // Pedimos el CSV que está en public/data
    const res = await fetch("/data/distribucion_quintiles_nacional_padres_hijos.csv");
    const texto = await res.text();

    //CSV a array
    const filas = texto.trim().split("\n").map((row) => row.split(","));

    // Primera fila: nombres de las columnas
    const cabeceras = filas[0];
    const cuerpo = filas.slice(1); // el resto son datos

    //array { columna: valor }
    datos = cuerpo.map((cols) => {
      const obj = {};
      cabeceras.forEach((h, i) => {
        // Primera columna: quintil_padres
        // El resto son porcentajes numéricos
        obj[h] = i === 0 ? cols[i] : Number(cols[i]);
      });
      return obj;
    });
  });
</script>

<div style="text-align: center;">
  <h2>Movilidad por quintiles (Gráfico Sankey)</h2>
  <div>
    ESPACIO PARA EL GRÁFICO
  </div>

 
  {#if datos.length === 0}
    <p>Cargando datos...</p>
  {:else}
    <pre style="text-align: left; font-size: 0.8rem;">
{JSON.stringify(datos, null, 2)}
    </pre>
  {/if}
</div>
