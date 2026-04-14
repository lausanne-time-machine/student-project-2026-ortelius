---
title: Visualisation
toc: false
---

<div class="content">

<h3>Visualisation des données</h3>

<p> TODO </p>

```js
const lausanneLayers = [
  {label: "Lausanne Pictorial Plan (1721)",          name: "lausanne-1721-melotte-pictorial"},
  {label: "Lausanne Land Owners (1723)",             name: "lausanne-1723-melotte-owners"},
  {label: "Lausanne Official Map (1806)",            name: "lausanne-1806-official-emery"},
  {label: "Lausanne Map (1824)",                     name: "lausanne-1824-spengler"},
  {label: "Cadastre Berney (1831)",                  name: "1831_Berney"},
  {label: "Cadastre Berney - LZW (1831)",            name: "1831_Berney_LZW"},
  {label: "Lausanne Pichard Belt Project (1836)",    name: "lausanne-1836-ceinture-pichard"},
  {label: "Lausanne Berney Ensemble Map (1838)",     name: "lausanne-1838-berney-ensemble"},
  {label: "Lausanne Geological Map (1852)",          name: "lausanne-1852-zollikofer"},
  {label: "Lausanne House Numbers Plan (1854)",      name: "lausanne-1854-weber"},
  {label: "Lausanne Plan (1856)",                    name: "lausanne-1856-bazar"},
  {label: "Lausanne Weber Map (1858)",               name: "lausanne-1858-weber"},
  {label: "Lausanne Siegfried Topographic Map (1865)", name: "lausanne-1865-siegfried"},
  {label: "Lausanne Map (1871)",                     name: "lausanne-1871-spengler"},
  {label: "Lausanne Funicular Map (1871)",           name: "lausanne-1871-spengler-funiculaire"},
  {label: "Lausanne Official Plan (1875)",           name: "lausanne-1875-decrousaz"},
  {label: "Lausanne City Directory (1875)",          name: "lausanne-1875-indicateur-reber"},
  {label: "Lausanne City Directory (1880)",          name: "lausanne-1880-indicateur-wurster"},
  {label: "Lausanne Map (1890)",                     name: "lausanne-1890-lebet"},
  {label: "Lausanne Parcel Plan (1900)",             name: "lausanne-1900-payot"},
  {label: "Lausanne Tramway Plan (1903)",            name: "lausanne-1903-tramway-reber"},
  {label: "Lausanne Parcel Plan (1910)",             name: "lausanne-1910-payot"},
  {label: "Lausanne Official Plan (1913)",           name: "lausanne-1913-plan-officiel"},
  {label: "Lausanne Transports Map (1925)",          name: "lausanne-1925-transports"},
  {label: "Lausanne Pictorial Map (1926)",           name: "lausanne-1926-ubs"},
  {label: "Lausanne Parcel Plan (1937)",             name: "lausanne-1937-cadastre"},
  {label: "Lausanne Parcel Plan (1959)",             name: "lausanne-1959-cadastre"},
  {label: "Lausanne Parcel Plan (1968)",             name: "lausanne-1968-cadastre"},
  {label: "Lausanne Atlas Map (1970)",               name: "lausanne-1970-imhoff"},
];
```

```js
const selectedLayer = view(Inputs.select(lausanneLayers, {
  label: "Historical map",
  format: d => d.label,
  value: lausanneLayers.find(d => d.name === "1831_Berney")
}))
```

```js
// Build the WMTS tile URL for a given layer name
function wmtsUrl(layerName) {
  return `https://geo-timemachine.epfl.ch/geoserver/gwc/service/wmts/rest/TimeMachine:${layerName}/{style}/{TileMatrixSet}/{TileMatrixSet}:{z}/{y}/{x}?format=image/png`;
}

// Create map with OSM base layer
const mapDiv = display(document.createElement("div"));
mapDiv.style = "height: 560px; margin: 1em 0;";

const historicalMap = L.map(mapDiv).setView([46.55, 6.632273], 12);

L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  opacity: 0.4
}).addTo(historicalMap);

invalidation.then(() => historicalMap.remove());
```

```js
// Reactively swap the historical tile layer whenever selectedLayer changes
{
  const tileLayer = L.tileLayer(wmtsUrl(selectedLayer.name), {
    style: "raster",
    TileMatrixSet: "EPSG:900913x2",
    tms: false,
    attribution: '&copy; <a href="https://www.epfl.ch/schools/cdh/time-machine-unit/">EPFL Time Machine Unit</a>'
  }).addTo(historicalMap);

  // Remove this layer when selectedLayer changes (cell re-runs)
  invalidation.then(() => historicalMap.removeLayer(tileLayer));
}
```

</div> 