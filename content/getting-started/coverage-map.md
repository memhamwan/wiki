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
L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
}).addTo(map);
L.tileLayer('/map-tiles/sco-coverage/OUTPUT_DIRECTORY/{z}/{x}/{y}.png', {
    maxZoom: 14,
    attribution: '&copy; HamWAN Memphis Metro',
    opacity: 0.5
}).addTo(map);
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