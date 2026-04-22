---
title: Visualisation
toc: false
---

<div class="content">

<h3>Visualisation des données</h3>

<p> TODO </p>

```js
// overall layout

const container = display(document.createElement("div"));
container.style = "display: flex; gap: 1px;";

const leftCol = document.createElement("div");
leftCol.style = "display: flex; flex-direction: column; gap: 5px;";

const rightCol = document.createElement("div");
rightCol.style = "display: flex; flex-direction: column; gap: 5px;";

container.appendChild(leftCol);
container.appendChild(rightCol);
```

```js
// historical map layers

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
// historical map selection 

const selectLeft = Inputs.select(lausanneLayers, {
  label: "Historical map",
  format: d => d.label,
  value: lausanneLayers.find(d => d.name === "1831_Berney")
})
selectLeft.classList.add("observable-container");

const selectRight = Inputs.select(lausanneLayers, {
  label: "Historical map",
  format: d => d.label,
  value: lausanneLayers.find(d => d.name === "1831_Berney")
});
selectRight.classList.add("observable-container");

const mapLayerLeft = Generators.input(selectLeft);
const mapLayerRight = Generators.input(selectRight);
```

```js
// Build the WMTS tile URL for a given layer name
function wmtsUrl(layerName) {
  return `https://geo-timemachine.epfl.ch/geoserver/gwc/service/wmts/rest/TimeMachine:${layerName}/{style}/{TileMatrixSet}/{TileMatrixSet}:{z}/{y}/{x}?format=image/png`;
}
```

```js
// Create map with OSM base layer

const mapDivLeft = document.createElement("div");
mapDivLeft.style = "height: 560px; margin: 0em 0; width: 400px";

const mapDivRight = document.createElement("div");
mapDivRight.style = "height: 560px; margin: 0em 0; width: 400px";

// finish global layout

leftCol.appendChild(selectLeft);
rightCol.appendChild(selectRight);

leftCol.appendChild(mapDivLeft);
rightCol.appendChild(mapDivRight);

const histMapLeft = L.map(mapDivLeft).setView([46.55, 6.632273], 12);
const histMapRight = L.map(mapDivRight).setView([46.55, 6.632273], 12);

function syncMaps(map1, map2) {
  let isSyncing = false;

  function sync(source, target) {
    if (isSyncing) return;
    isSyncing = true;

    const center = source.getCenter();
    const zoom = source.getZoom();

    target.setView(center, zoom, { animate: false });

    isSyncing = false;
  }

  map1.on("moveend", () => sync(map1, map2));
  map2.on("moveend", () => sync(map2, map1));
}

// Activate syncing
syncMaps(histMapLeft, histMapRight);

L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  opacity: 0.5
}).addTo(histMapLeft);

L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>',
  opacity: 0.5
}).addTo(histMapRight);

invalidation.then(() => histMapLeft.remove());
invalidation.then(() => histMapRight.remove());
```

```js
// Reactively swap the historical tile layer whenever mapLayerLeft changes
{
  const tileLayerLeft = L.tileLayer(wmtsUrl(mapLayerLeft.name), {
    style: "raster",
    TileMatrixSet: "EPSG:900913x2",
    tms: false,
    attribution: '&copy; <a href="https://www.epfl.ch/schools/cdh/time-machine-unit/">EPFL Time Machine Unit</a>',
    opacity: 1
  }).addTo(histMapLeft);

  // Remove this layer when mapLayerLeft changes (cell re-runs)
  invalidation.then(() => histMapLeft.removeLayer(tileLayerLeft));

  
  const tileLayerRight = L.tileLayer(wmtsUrl(mapLayerRight.name), {
    style: "raster",
    TileMatrixSet: "EPSG:900913x2",
    tms: false,
    attribution: '&copy; <a href="https://www.epfl.ch/schools/cdh/time-machine-unit/">EPFL Time Machine Unit</a>',
    opacity: 1
  }).addTo(histMapRight);

  // Remove this layer when mapLayerRight changes (cell re-runs)
  invalidation.then(() => histMapRight.removeLayer(tileLayerRight));
}

```

</div> 