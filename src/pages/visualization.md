---
title: Visualisation
toc: false
---

<div class="content">

<h3>Visualisation des données</h3>

<p>Pour découvrir les données sur lesquelles repose notre projet, la visualisation suivante permet de comparer l'évolution de Lausanne au fil du temps. En mettant face à face deux époques différentes, vous pouvez observer précisément comment les rivières et les rives du lac se sont transformées entre 1721 et aujourd'hui.</p>
</div>

```js
// overall layout

const container = display(document.createElement("div"));
container.style = "display: flex; flex-direction: column; gap: 10px; width: 100%; margin: 1em 0;";

const toggleDiv = document.createElement("div");
toggleDiv.style = "display: flex; gap: 1px;";

const mapsDiv = document.createElement("div");
mapsDiv.style = "display: flex; flex-direction: row; gap: 1px; width: 100%;";

const leftCol = document.createElement("div");
leftCol.style = "display: flex; flex-direction: column; gap: 5px; width: 100%;";

const rightCol = document.createElement("div");
rightCol.style = "display: flex; flex-direction: column; gap: 5px; width: 100%";

container.appendChild(toggleDiv);
container.appendChild(mapsDiv);
mapsDiv.appendChild(leftCol);
mapsDiv.appendChild(rightCol);
```

```js
// historical map layers

const lausanneLayers = [
  {label: "Cadastre Melotte 1721",       name: "lausanne-1721-melotte",  rivers: FileAttachment("../river_layers/melotte_rivers_crs84.geojson")},
  {label: "Cadastre Berney (1831)",      name: "1831_Berney_LZW",        rivers: FileAttachment("../river_layers/berney_rivers_crs84.geojson")}, //
  {label: "Cadastre rénové (1888)",      name: "lausanne-1888-renove",   rivers: FileAttachment("../river_layers/renove_rivers_crs84.geojson")},
  {label: "Lausanne Parcel Plan (1937)", name: "lausanne-1937-cadastre", rivers: FileAttachment("../river_layers/1937_rivers_crs84.geojson")}, // 
  {label: "Lausanne Parcel Plan (1959)", name: "lausanne-1959-cadastre", rivers: FileAttachment("../river_layers/1959_rivers_crs84.geojson")}, //
  {label: "Contemporain 2021",           name: "-",                      rivers: FileAttachment("../river_layers/contemporain_rivers_crs84.geojson")},
];
```

```js
// historical map selection 

const selectLeft = Inputs.select(lausanneLayers, {
  label: "Historical map",
  format: d => d.label,
  value: lausanneLayers.find(d => d.name === "lausanne-1721-melotte")
})
selectLeft.style = "width: 98%;";
selectLeft.classList.add("observable-container");

const selectRight = Inputs.select(lausanneLayers, {
  label: "Historical map",
  format: d => d.label,
  value: lausanneLayers.find(d => d.name === "-")
});
selectRight.style = "width: 98%;";
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
mapDivLeft.style = "height: 560px; margin: 0em 0; width: 100%";

const mapDivRight = document.createElement("div");
mapDivRight.style = "height: 560px; margin: 0em 0; width: 100%";

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
  var tileLayerLeft = L.tileLayer(wmtsUrl(mapLayerLeft.name), {
    style: "raster",
    TileMatrixSet: "EPSG:900913x2",
    tms: false,
    attribution: '&copy; <a href="https://www.epfl.ch/schools/cdh/time-machine-unit/">EPFL Time Machine Unit</a>',
    opacity: 1
  })
  tileLayerLeft.addTo(histMapLeft);

  // Remove this layer when mapLayerLeft changes (cell re-runs)
  invalidation.then(() => histMapLeft.removeLayer(tileLayerLeft));
  
  const tileLayerRight = L.tileLayer(wmtsUrl(mapLayerRight.name), {
    style: "raster",
    TileMatrixSet: "EPSG:900913x2",
    tms: false,
    attribution: '&copy; <a href="https://www.epfl.ch/schools/cdh/time-machine-unit/">EPFL Time Machine Unit</a>',
    opacity: 1
  })
  tileLayerRight.addTo(histMapRight);

  // Remove this layer when mapLayerRight changes (cell re-runs)
  invalidation.then(() => histMapRight.removeLayer(tileLayerRight));
}

```

```js
  const label = document.createElement("label");

  const checkbox = document.createElement("input");
  checkbox.type = "checkbox";
  checkbox.id = "toggleRivers";
  checkbox.checked = false;

  label.appendChild(checkbox);
  label.append(" Afficher les rivières");

  toggleDiv.appendChild(label);

```

```js
{
  const riverStyle = {
    color: "#2980b9",
    weight: 3,
    opacity: 1,
  }

  // Load the GeoJSON data

  const geojsonLeft = await mapLayerLeft.rivers.json();

  const geojsonLayerLeft = L.geoJSON(
    {
      type: "FeatureCollection",
      features: geojsonLeft.features, 
    }, {
      style: riverStyle
    }
  );

  invalidation.then(() => histMapLeft.removeLayer(geojsonLayerLeft));

  const geojsonRight = await mapLayerRight.rivers.json();

  const geojsonLayerRight = L.geoJSON(
    {
      type: "FeatureCollection",
      features: geojsonRight.features, 
    }, {
      style: riverStyle
    }
  );

  invalidation.then(() => histMapRight.removeLayer(geojsonLayerRight));

  if (checkbox.checked) {
    geojsonLayerLeft.addTo(histMapLeft);
    geojsonLayerRight.addTo(histMapRight);
  }

  const listener = (e) => {
    if (e.target.checked) {
      geojsonLayerLeft.addTo(histMapLeft);
      geojsonLayerRight.addTo(histMapRight);
    } else {
      histMapLeft.removeLayer(geojsonLayerLeft);
      histMapRight.removeLayer(geojsonLayerRight);
    }
  };

  checkbox.addEventListener("change", listener);

  invalidation.then(() => checkbox.removeEventListener("change", listener));
}
```

</div> 