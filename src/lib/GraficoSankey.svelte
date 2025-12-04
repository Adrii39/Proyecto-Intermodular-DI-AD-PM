<script>
  import { onMount } from "svelte";
  import Plotly from "plotly.js-dist-min";

  let datos = [];

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

  // Base y opacidad al hacer hover
  const baseOpacity = 0.2;   
  const hoverOpacity = 1;  

  const baseColor = `rgba(150,150,150,${baseOpacity})`;
  const hoverColor = `rgba(150,150,150,${hoverOpacity})`;

  const linkColors = links.map(() => baseColor);
  const hoverColors = links.map(() => hoverColor);

  const data = [
    {
      type: "sankey",
      orientation: "h",
      node: {
        label: labels,
        pad: 15,
        thickness: 20,
        hovertemplate: 'Quintil: %{label}<extra></extra>'
      },
      link: {
        source,
        target,
        value,
        // color normal de los enlaces
        color: linkColors,
        // color con hover
        hovercolor: hoverColors,
        hovertemplate: '%{value}%<extra></extra>'
      }
    }
  ];

  const layout = {
    font: { size: 12 },
    paper_bgcolor: "rgba(0,0,0,0)",
    plot_bgcolor: "rgba(0,0,0,0)",
    margin: { t: 10, l: 0, r: 0, b: 0 } 
  };

  Plotly.newPlot(chartEl, data, layout, { responsive: true });
}
</script>


<div style="text-align: center; margin-top: 6rem;">
  <h2 style="margin-bottom: 0.25rem;">Movilidad por quintiles (Gráfico Sankey)</h2>

  <!--gráfico -->
  <div
    bind:this={chartEl}
    style="max-width: 900px; height: 500px; margin: 0 auto 0.25rem;"
  ></div>
  <p
   style="max-width: 500px; margin: 0 auto 0.25rem ;font-size: 0.9rem;"
   >Este gráfico Sankey muestra el porcentaje de hijos que acaban en cada
    quintil de renta dado el quintil de renta de sus padres. Cada flujo
    representa “el % de padres de un quintil que tienen hijos en otro”.</p>
</div>
