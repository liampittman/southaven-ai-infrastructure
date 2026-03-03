---
layout: default
title: Musk/xAI Data Center Sites
---

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<style>
.map-container {
  margin: 0 auto 40px auto;
  max-width: 1200px;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.map {
  height: 600px;
  margin-top: 1em;
  border-radius: 4px;
}
.legend {
  background: white !important;
  padding: 10px !important;
  border-radius: 4px !important;
  box-shadow: 0 1px 3px rgba(0,0,0,0.3) !important;
  line-height: 24px !important;
  color: #555 !important;
  min-width: 190px !important;
}
.legend h4 {
  margin: 0 0 10px 0 !important;
  color: #333 !important;
}
.legend-row {
  display: flex !important;
  align-items: center !important;
  margin-bottom: 5px !important;
  font-size: 13px !important;
}
.legend-poly {
  width: 18px !important;
  height: 18px !important;
  flex-shrink: 0 !important;
  margin-right: 8px !important;
  border: 2px solid #6c0175 !important;
  background: rgba(168,57,198,0.12) !important;
}
.legend-divider {
  border: none !important;
  border-top: 1px solid #ddd !important;
  margin: 7px 0 !important;
}
.leaflet-popup-content {
  font-size: 13px !important;
  line-height: 1.5 !important;
  max-width: 260px !important;
}
.popup-title {
  font-weight: bold;
  font-size: 14px;
  margin-bottom: 2px;
}
.popup-subtitle {
  font-size: 12px;
  color: #6c0175;
  font-weight: 600;
  margin-bottom: 4px;
}
.popup-divider {
  border: none;
  border-top: 1px solid #eee;
  margin: 6px 0;
}
.popup-description {
  font-size: 12px;
  color: #333;
}
</style>

<div class="map-container">
  <div id="xai-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
document.addEventListener("DOMContentLoaded", function () {

  var map = L.map('xai-map', {
    scrollWheelZoom: false,
    zoomControl: true,
    zoomSnap:0.5,
    zoomDelta:0.5
  }).setView([35.02, -90.06], 12);

  L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', {
    attribution: '&copy; OpenStreetMap contributors &copy; CARTO | Map by William Pittman for Mississippi Free Press'
  }).addTo(map);

  // ── Polygon style ────────────────────────────────────────────────────────
  var polygonStyle = {
    color: '#6c0175',
    weight: 2,
    opacity: 0.85,
    fillColor: '#a839c6',
    fillOpacity: 0.12
  };

  // ── SVG pin icon factory ─────────────────────────────────────────────────
  function makeIcon(color) {
    var svg = '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="36" viewBox="0 0 24 36">'
      + '<path fill="' + color + '" stroke="#fff" stroke-width="1.5" '
      + 'd="M12 0C5.373 0 0 5.373 0 12c0 9 12 24 12 24S24 21 24 12C24 5.373 18.627 0 12 0z"/>'
      + '<circle fill="#fff" cx="12" cy="12" r="4"/>'
      + '</svg>';
    return L.divIcon({
      html: svg,
      className: '',
      iconSize: [12, 18],
      iconAnchor: [6, 18],
      popupAnchor: [0, -20]
    });
  }

  // Icon colors keyed to type values in xAI_facilities.geojson
  var iconColors = {
    'xAI data center':            '#6c0175',
    'xAI data center (proposed)': '#a839c6',
    'xAI power plant':            '#c27a00'
  };

  function iconFor(type) {
    return makeIcon(iconColors[type] || '#555555');
  }

  // ── Popup builder ────────────────────────────────────────────────────────
  function buildPopup(title, subtitle, description) {
    var html = '<div class="popup-title">' + title + '</div>';
    if (subtitle) {
      html += '<div class="popup-subtitle">' + subtitle + '</div>';
    }
    if (description) {
      html += '<hr class="popup-divider">'
            + '<div class="popup-description">' + description + '</div>';
    }
    return html;
  }

  // ── Layer tracking for fitBounds ─────────────────────────────────────────
  var layers = [];
  var loaded = 0;
  var total  = 3;

  function onLoaded(layer) {
    if (layer) layers.push(layer);
    loaded++;
    if (loaded === total) {
      var bounds = L.latLngBounds([]);
      layers.forEach(function(l) { bounds.extend(l.getBounds()); });
      map.fitBounds(bounds, { padding: [30, 30] });
    }
  }

  // ── Whitehaven polygon ───────────────────────────────────────────────────
  fetch('assets/data/whitehaven.geojson')
    .then(function(r) { return r.json(); })
    .then(function(data) {
      var layer = L.geoJSON(data, {
        style: polygonStyle,
        onEachFeature: function(feature, layer) {
          layer.bindPopup(buildPopup(
            'Whitehaven',
            'Neighborhood in Memphis, Tenn.',
            'The neighborhood in Memphis, Tenn., where Colossus 2 is located.'
          ));
        }
      }).addTo(map);
      onLoaded(layer);
    })
    .catch(function(e) { console.error('whitehaven.geojson:', e); onLoaded(null); });

  // ── Colonial Hills polygon ───────────────────────────────────────────────
  fetch('assets/data/colonialhills.geojson')
    .then(function(r) { return r.json(); })
    .then(function(data) {
      var layer = L.geoJSON(data, {
        style: polygonStyle,
        onEachFeature: function(feature, layer) {
          layer.bindPopup(buildPopup(
            'Colonial Hills',
            'Neighborhood in Southaven, Miss.',
            'A neighborhood in Southaven, Miss., near the unpermitted turbines. Read more <a href="https://www.mississippifreepress.org/as-musks-xai-data-centers-encroach-on-southaven-north-mississippi-residents-push-back/" target="_blank">here</a>.'
          ));
        }
      }).addTo(map);
      onLoaded(layer);
    })
    .catch(function(e) { console.error('colonialhills.geojson:', e); onLoaded(null); });

  // ── Point markers ────────────────────────────────────────────────────────
  fetch('assets/data/xAI_facilities.geojson')
    .then(function(r) { return r.json(); })
    .then(function(data) {
      var layer = L.geoJSON(data, {
        pointToLayer: function(feature, latlng) {
          return L.marker(latlng, { icon: iconFor(feature.properties.type) });
        },
        onEachFeature: function(feature, layer) {
          var p = feature.properties;
          var subtitle = p.city + ', ' + p.state_code;
          layer.bindPopup(buildPopup(p.point_of_interest, subtitle, p.description));
        }
      }).addTo(map);
      onLoaded(layer);
    })
    .catch(function(e) { console.error('xAI_facilities.geojson:', e); onLoaded(null); });

  // ── Legend ───────────────────────────────────────────────────────────────
  function pinSVG(color) {
    return '<svg xmlns="http://www.w3.org/2000/svg" width="14" height="21" viewBox="0 0 24 36"'
      + ' style="flex-shrink:0;margin-right:8px;">'
      + '<path fill="' + color + '" stroke="#fff" stroke-width="1.5" '
      + 'd="M12 0C5.373 0 0 5.373 0 12c0 9 12 24 12 24S24 21 24 12C24 5.373 18.627 0 12 0z"/>'
      + '<circle fill="#fff" cx="12" cy="12" r="4"/>'
      + '</svg>';
  }

  var legend = L.control({ position: 'bottomleft' });
  legend.onAdd = function() {
    var div = L.DomUtil.create('div', 'legend');
    div.innerHTML =
      '<h4>Sites</h4>'
      + '<div class="legend-row"><div class="legend-poly"></div>Neighborhood/Subdivision</div>'
      + '<hr class="legend-divider">'
      + '<div class="legend-row">' + pinSVG('#6c0175') + 'xAI data center</div>'
      + '<div class="legend-row">' + pinSVG('#a839c6') + 'xAI data center (proposed)</div>'
      + '<div class="legend-row">' + pinSVG('#c27a00') + 'xAI power plant</div>';
    return div;
  };
  legend.addTo(map);

});
</script>
