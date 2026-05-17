---
title: Analyse
---

<div class="content">

<h3>Analyse</h3>

<h4>Analyse globale</h4>
<h5>L'évolution du réseau hydrographique</h5>

<p>Analyse globale qui montre les résultats stats sur Lausanne en général</p>

<div id="visu-riviere-container" style="display: flex; flex-direction: column; gap: 15px; width: 100%; margin: 2em 0;">
  <div id="controls-div-single" style="display: flex; flex-direction: column; gap: 15px; background-color: #f8fafc; padding: 15px; border-radius: 8px; border: 1px solid #e2e8f0;"></div>
  <div id="map-single-div" style="height: 600px; width: 100%; border-radius: 8px; border: 2px solid #cbd5e1;"></div>
</div>

```js
const riversDataDict = {
  flon: {
    "1721": FileAttachment("../river_layers_by_river/flon/flon_melotte.geojson"),
    "1831": FileAttachment("../river_layers_by_river/flon/flon_berney.geojson"),
    "1888": FileAttachment("../river_layers_by_river/flon/flon_renove.geojson"),
    "1937": FileAttachment("../river_layers_by_river/flon/flon_1937.geojson"),
    "1959": FileAttachment("../river_layers_by_river/flon/flon_1959.geojson"),
    "2021": FileAttachment("../river_layers_by_river/flon/flon_contemporain.geojson")
  },
  louve: {
    "1721": FileAttachment("../river_layers_by_river/louve/louve_melotte.geojson"),
    "1831": FileAttachment("../river_layers_by_river/louve/louve_berney.geojson"),
    "1888": FileAttachment("../river_layers_by_river/louve/louve_renove.geojson"),
    "1937": FileAttachment("../river_layers_by_river/louve/louve_1937.geojson"),
    "1959": FileAttachment("../river_layers_by_river/louve/louve_1959.geojson"),
    "2021": FileAttachment("../river_layers_by_river/louve/louve_contemporain.geojson")
  },
  vuachere: {
    "1721": FileAttachment("../river_layers_by_river/vuachere/vuachere_melotte.geojson"),
    "1831": FileAttachment("../river_layers_by_river/vuachere/vuachere_berney.geojson"),
    "1888": FileAttachment("../river_layers_by_river/vuachere/vuachere_renove.geojson"),
    "1937": FileAttachment("../river_layers_by_river/vuachere/vuachere_1937.geojson"),
    "1959": FileAttachment("../river_layers_by_river/vuachere/vuachere_1959.geojson"),
    "2021": FileAttachment("../river_layers_by_river/vuachere/vuachere_contemporain.geojson")
  }
};

const erasConfig = [
  { id: "1721", label: "Cadastre Melotte (1721)", wmts: "lausanne-1721-melotte", color: "#e41a1c" },
  { id: "1831", label: "Cadastre Berney (1831)",  wmts: "1831_Berney_LZW",       color: "#377eb8" },
  { id: "1888", label: "Cadastre rénové (1888)",  wmts: "lausanne-1888-renove",  color: "#4daf4a" },
  { id: "1937", label: "Plan Parcel. (1937)",     wmts: "lausanne-1937-cadastre",color: "#984ea3" },
  { id: "1959", label: "Plan Parcel. (1959)",     wmts: "lausanne-1959-cadastre",color: "#ff7f00" },
  { id: "2021", label: "Contemporain (2021)",     wmts: "-",                     color: "#1e293b" }
];

const riverOptions = [
  {name: "Le Flon", id: "flon"}, 
  {name: "La Louve", id: "louve"}, 
  {name: "La Vuachère", id: "vuachere"}
];

const selectRiverAnalysis = Inputs.select(riverOptions, { 
  label: "Rivière analysée :", 
  format: d => d.name, 
  value: riverOptions[0] 
});

const selectYearsAnalysis = Inputs.checkbox(erasConfig, {
  label: "Époques à comparer :",
  format: d => {
    const span = document.createElement("span");
    span.innerHTML = `<span style="color:${d.color}; font-weight:bold;">■</span> ${d.label}`;
    return span;
  },
  value: [erasConfig[3], erasConfig[5]]
});

selectYearsAnalysis.style.backgroundColor = "#f0f9ff";
selectYearsAnalysis.style.border = "1px solid #bae6fd";
selectYearsAnalysis.style.color = "#0369a1";
selectYearsAnalysis.style.padding = "12px 16px";
selectYearsAnalysis.style.borderRadius = "6px";
selectYearsAnalysis.style.width = "fit-content";

const checkboxesDiv = selectYearsAnalysis.querySelector("div");
if (checkboxesDiv) {
  checkboxesDiv.style.display = "flex";
  checkboxesDiv.style.flexDirection = "column";
  checkboxesDiv.style.gap = "8px";
}

// Suppression du carré gris global pour correspondre à ton image
const parentContainer = document.getElementById("controls-div-single");
if (parentContainer) {
  parentContainer.style.backgroundColor = "transparent";
  parentContainer.style.border = "none";
  parentContainer.style.padding = "0";
}


const controlsContainer = document.getElementById("controls-div-single");
controlsContainer.innerHTML = ""; 
controlsContainer.appendChild(selectRiverAnalysis);
controlsContainer.appendChild(selectYearsAnalysis);

const mapDivSingle = document.getElementById("map-single-div");
const analysisMap = L.map(mapDivSingle).setView([46.522, 6.632], 13);

L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
  attribution: '&copy; OpenStreetMap',
  opacity: 0.5
}).addTo(analysisMap);

function wmtsUrlAnalysis(layerName) {
  return `https://geo-timemachine.epfl.ch/geoserver/gwc/service/wmts/rest/TimeMachine:${layerName}/{style}/{TileMatrixSet}/{TileMatrixSet}:{z}/{y}/{x}?format=image/png`;
}

let bgMapLayer = null;
let riverFeatureGroup = L.featureGroup().addTo(analysisMap);

async function updateAnalysisMap() {
  const currentRiver = selectRiverAnalysis.value;
  const currentYears = selectYearsAnalysis.value;

  if (bgMapLayer) {
    analysisMap.removeLayer(bgMapLayer);
    bgMapLayer = null;
  }
  riverFeatureGroup.clearLayers();

  const sortedYears = currentYears.slice().sort((a, b) => parseInt(a.id) - parseInt(b.id));
  const latestEra = sortedYears.length > 0 ? sortedYears[sortedYears.length - 1] : null;
  
  if (latestEra && latestEra.wmts !== "-") {
    bgMapLayer = L.tileLayer(wmtsUrlAnalysis(latestEra.wmts), {
      style: "raster",
      TileMatrixSet: "EPSG:900913x2",
      tms: false,
      opacity: 1
    }).addTo(analysisMap);
  }

  await Promise.all(currentYears.map(async (era) => {
    const fileRef = riversDataDict[currentRiver.id]?.[era.id];
    if (!fileRef) return;

    try {
      const geojsonData = await fileRef.json();
      L.geoJSON(geojsonData, {
        style: { color: era.color, weight: 6, opacity: 0.8 },
        onEachFeature: (feature, layer) => {
          layer.bindTooltip(`<b>${currentRiver.name}</b><br>Tracé de ${era.label}`, { sticky: true });
        }
      }).addTo(riverFeatureGroup);
    } catch (error) {
      console.error(`Erreur pour ${currentRiver.id} (${era.id})`, error);
    }
  }));

  if (riverFeatureGroup.getLayers().length > 0) {
    analysisMap.fitBounds(riverFeatureGroup.getBounds(), { padding: [40, 40] });
  }
}

selectRiverAnalysis.addEventListener("input", updateAnalysisMap);
selectYearsAnalysis.addEventListener("input", updateAnalysisMap);

updateAnalysisMap();

invalidation.then(() => analysisMap.remove());
```

<h4>Le Flon</h4>
<h5>Infrastructure et comblement du vallon</h5>

<h4>La Louve</h4>
<h5>Intégration sous le tissu urbain central</h5>

<h4>La Vuachère</h4>
<h5>Préservation du cours naturel</h5>

<h4>Les Marais</h4>
<h5>Drainage et reconversion des zones humides</h5>

<h4>Les Rives du lac</h4>
<h5>Expansion de la côte par remblais</h5>

</div>