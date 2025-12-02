<script>
  import { onMount } from "svelte";

  let datos = [];
    //COMPROBACIÓN DE COMMIT
  // Nuevas variables para el sankey
  let nodes = [];
  let links = [];

  onMount(async () => {
    const res = await fetch("/data/distribucion_quintiles_nacional_padres_hijos.csv");
    const texto = await res.text();

    const filas = texto.trim().split("\n").map((row) => row.split(","));

    const cabeceras = filas[0];
    const cuerpo = filas.slice(1);

    datos = cuerpo.map((cols) => {
      const obj = {};
      cabeceras.forEach((h, i) => {
        obj[h] = i === 0 ? cols[i] : Number(cols[i]);
      });
      return obj;
    });
  });

  // Cuando cambie `datos`, calculamos nodos y enlaces
  $: if (datos.length > 0) {
    // nombres columnas de hijos (0-20, 20-40, ...)
    const columnasHijos = Object.keys(datos[0]).filter(
      (k) => k !== "quintil_padres"
    );

    // 5 nodos de padres
    const nodosPadres = datos.map((fila) => ({
      name: `Padres ${fila.quintil_padres}`
    }));

    // 5 nodos de hijos
    const nodosHijos = columnasHijos.map((q) => ({
      name: `Hijos ${q}`
    }));

    nodes = [...nodosPadres, ...nodosHijos];

    // Enlaces: de cada padre a cada quintil de hijo
    const tmpLinks = [];
    datos.forEach((fila, iPadre) => {
      columnasHijos.forEach((q, j) => {
        tmpLinks.push({
          source: iPadre, // índice nodo padre
          target: nodosPadres.length + j, // índice nodo hijo
          value: fila[q] // porcentaje
        });
      });
    });

    links = tmpLinks;
  }
</script>
<div style="text-align: center;">
  <h2>Movilidad por quintiles (Gráfico Sankey)</h2>
  <div>
    ESPACIO PARA EL GRÁFICO
  </div>

  {#if datos.length === 0}
    <p>Cargando datos...</p>
  {:else}
    <h3>Datos originales (CSV):</h3>
    <pre style="text-align: left; font-size: 0.8rem;">
{JSON.stringify(datos, null, 2)}
    </pre>

    <h3>Nodos:</h3>
    <pre style="text-align: left; font-size: 0.8rem;">
{JSON.stringify(nodes, null, 2)}
    </pre>

    <h3>Enlaces:</h3>
    <pre style="text-align: left; font-size: 0.8rem;">
{JSON.stringify(links, null, 2)}
    </pre>
  {/if}
</div>
