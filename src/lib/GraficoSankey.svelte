<script>
  import { onMount } from "svelte";
  import Plotly from "plotly.js-dist-min";

  let datos = [];

  // Datos para el sankey
  let nodes = [];
  let links = [];

  // Referencia al <div> donde dibujaremos el gráfico
  let chartEl;

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

  // Cuando ya tenemos datos, creamos nodos y enlaces
  $: if (datos.length > 0) {
    const columnasHijos = Object.keys(datos[0]).filter(
      (k) => k !== "quintil_padres"
    );

    const nodosPadres = datos.map((fila) => ({
      name: `Padres ${fila.quintil_padres}`
    }));

    const nodosHijos = columnasHijos.map((q) => ({
      name: `Hijos ${q}`
    }));

    nodes = [...nodosPadres, ...nodosHijos];

    const tmpLinks = [];
    datos.forEach((fila, iPadre) => {
      columnasHijos.forEach((q, j) => {
        tmpLinks.push({
          source: iPadre,
          target: nodosPadres.length + j,
          value: fila[q]
        });
      });
    });

    links = tmpLinks;
  }

  // Cuando tenemos el div y los datos, dibujamos el sankey
  $: if (chartEl && nodes.length && links.length) {
    const labels = nodes.map((n) => n.name);
    const source = links.map((l) => l.source);
    const target = links.map((l) => l.target);
    const value = links.map((l) => l.value);

    const data = [
      {
        type: "sankey",
        orientation: "h",
        node: {
          label: labels,
          pad: 15,
          thickness: 20
          
        },
        link: {
          source,
          target,
          value,
          color: "rgba(150,150,150,0.3)"
        }
      }
    ];

    const layout = {
      font: { size: 12 },
      paper_bgcolor: "rgba(0,0,0,0)",
      plot_bgcolor: "rgba(0,0,0,0)"
    };

    Plotly.newPlot(chartEl, data, layout, { responsive: true });
  }
</script>

<div style="text-align: center; margin-top: 2rem;">
  <h2>Movilidad por quintiles (Gráfico Sankey)</h2>

  <!--gráfico -->
  <div
    bind:this={chartEl}
    style="max-width: 900px; height: 500px; margin: 1rem auto;"
  ></div>
</div>
