---
weight: 2
---
_This is a work-in-progress and is not yet complete._
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.3/dist/leaflet.css"
    integrity="sha256-kLaT2GOSpHechhsozzB+flnD+zUyjE2LlfWPgU04xyI="
    crossorigin=""/>
 <!-- Make sure you put this AFTER Leaflet's CSS -->
 <script src="https://unpkg.com/leaflet@1.9.3/dist/leaflet.js"
     integrity="sha256-WBkoXOwTeyKclOHuWtc+i2uENFpDZ9YPdf5Hf+D7ewM="
     crossorigin=""></script>
 <div id="map" style="height: 600px"></div>
<script type="text/javascript">
var map = L.map('map').setView([35.0800, -89.9303], 9);
var osm = L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
});
var terrain = L.tileLayer('https://stamen-tiles.a.ssl.fastly.net/terrain/{z}/{x}/{y}.jpg', {
    // maxZoom: 19,
    attribution: 'Map tiles by <a href="http://stamen.com">Stamen Design</a>, under <a href="http://creativecommons.org/licenses/by/3.0">CC BY 3.0</a>. Data by <a href="http://openstreetmap.org">OpenStreetMap</a>, under <a href="http://www.openstreetmap.org/copyright">ODbL</a>.'
}).addTo(map);
var sco_coverage = L.tileLayer('/map-tiles/sco-coverage/OUTPUT_DIRECTORY/{z}/{x}/{y}.png', {
    maxZoom: 14,
    minZoom: 8,
    attribution: '&copy; HamWAN Memphis Metro',
    opacity: 0.5
}).addTo(map);
var hil_coverage = L.tileLayer('/map-tiles/hil-coverage/{z}/{x}/{y}.png', {
    maxZoom: 14,
    minZoom: 8,
    attribution: '&copy; HamWAN Memphis Metro',
    opacity: 0.5
}).addTo(map);
var mno_coverage = L.tileLayer('/map-tiles/mno-coverage/{z}/{x}/{y}.png', {
    maxZoom: 14,
    minZoom: 8,
    attribution: '&copy; HamWAN Memphis Metro',
    opacity: 0.5
}).addTo(map);
var crw_coverage = L.tileLayer('/map-tiles/crw-coverage/{z}/{x}/{y}.png', {
    maxZoom: 14,
    minZoom: 8,
    attribution: '&copy; HamWAN Memphis Metro',
    opacity: 0.5
}).addTo(map);
var azo_coverage = L.tileLayer('/map-tiles/azo-coverage/{z}/{x}/{y}.png', {
    maxZoom: 14,
    minZoom: 8,
    attribution: '&copy; HamWAN Memphis Metro',
    opacity: 0.5
}).addTo(map);
var baseLayers = {
    "Terrain": terrain,
    "Streets": osm
};
var overlays = {
    "SCO Coverage": sco_coverage,
    "HIL Coverage": hil_coverage,
    "MNO Coverage": mno_coverage,
    "CRW Coverage": crw_coverage,
    "AZO Coverage": azo_coverage
};
L.control.layers(baseLayers, overlays, { collapsed: false, hideSingleBase: true }).addTo(map);
function onEachFeature(feature, layer) {
    let popupContent = `<p>${feature.properties.name}</p>`;
    if (feature.properties && feature.properties.popupContent) {
        popupContent += feature.properties.popupContent;
    }
    layer.bindPopup(popupContent);
}
// Want to add a new point to the map? Look at /static/sites.geojson which is the source for the list of sites.
let xhr = new XMLHttpRequest();
xhr.open('GET', '/sites.geojson');
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.responseType = 'json';
xhr.onload = function() {
    if (xhr.status !== 200) return
    L.geoJSON(xhr.response, {
		style(feature) {
			return feature.properties && feature.properties.style;
		},
		onEachFeature,
		pointToLayer(feature, latlng) {
			return L.circleMarker(latlng, {
				radius: 5,
				fillColor: '#ff0000',
				color: '#000',
				weight: 1,
				opacity: 1,
				fillOpacity: 0.5
			});
		}
	}).addTo(map);
};
xhr.send();
</script>
Red means that a signal of at least -80dBb with a 30 dBi dish at 10m is theoretically possible. We find that in practice this map is untrustworthy. Don't buy hardware based on the map alone: ask for help from a friend with a survey.
