---
title: "Vineer Lab - Kgofa maps TEST"
layout: textlay
excerpt: "Vineer Lab -- Kgofa maps TEST"
sitemap: false
permalink: /kgofa_maps/
---

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Leaflet + GeoTIFF</title>
    <!-- Leaflet CSS -->
    <link
      rel="stylesheet"
      href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
      integrity="sha512-sA+qmdqYXUcU4gnuhfFM7tKVb/c/jMAasTXmL2JYSlkDMEGS6d6CuYuEuT1bG1Ic5fQ2G1FtbTSfjdC7Cr7zKg=="
      crossorigin=""
    />
    <style>
      body {
        margin: 0;
        padding: 0;
      }
      #map {
        width: 100%;
        height: 90vh;
      }
      .description {
        padding: 1em;
      }
    </style>
  </head>
  <body>
    <div id="map"></div>
    <div class="description">
      <h2>About this map</h2>
      <p>
        This map loads a GeoTIFF in the browser using <strong>GeoRaster</strong>
        and <strong>Leaflet</strong>. We keep nodata values without forcing
        8-bit conversion. 
      </p>
    </div>

    <!-- Leaflet JS -->
    <script
      src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
      integrity="sha512-v2kP4XJ6f+VcnAR2BeW25NUp17wxdBlOYdydALwTE/vp4vB+DUXLvuB1KcRHPyEPE1r7z6kW8fT4OfnG8MjFQw=="
      crossorigin=""
    ></script>

    <!-- GeoRaster libraries -->
    <!-- georaster parses GeoTIFF arrays in browser -->
    <script src="https://unpkg.com/georaster/dist/georaster.browser.min.js"></script>
    <!-- georaster-layer-for-leaflet displays georaster on Leaflet -->
    <script src="https://unpkg.com/georaster-layer-for-leaflet/dist/georaster-layer-for-leaflet.min.js"></script>

    <script>
      // 1. Initialize Leaflet map
      const map = L.map('map').setView([0, 0], 2);

      // 2. Add an OSM basemap (optional)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19,
        attribution: '&copy; OpenStreetMap contributors'
      }).addTo(map);

      // 3. Load your GeoTIFF in the browser
      const tiffUrl = 'https://github.com/hannahvineer/kgofa_maps/blob/main/rhipicephalus_appendiculatus_2021-2040_suitability_mean_GCMs_cog.tif'; // <-- Replace with your actual URL

      fetch(tiffUrl)
        .then(response => {
          if (!response.ok) {
            throw new Error(`Network response was not ok. Status: ${response.status}`);
          }
          return response.arrayBuffer();
        })
        .then(arrayBuffer => {
          // Parse the GeoTIFF into a georaster object
          return parseGeoraster(arrayBuffer);
        })
        .then(georaster => {
          console.log("GeoTIFF parsed:", georaster);

          // 4. Create a georaster layer with nodata handling if needed
          // By default, georaster-layer-for-leaflet tries to read noDataValue from georaster
          // But you can customize color mapping:
          const layerOptions = {
            georaster,
            opacity: 1.0,
            pixelValuesToColorFn: values => {
              // If single-band with a known nodata
              if (values[0] === georaster.noDataValue) return null; // Transparent
              // Example grayscale mapping:
              const val = values[0];
              return `rgb(${val}, ${val}, ${val})`;
            }
          };

          // If multi-band (RGB), skip pixelValuesToColorFn or do something like:
          //   pixelValuesToColorFn: values => `rgb(${values[0]}, ${values[1]}, ${values[2]})`;

          const geoRasterLayer = new GeoRasterLayer(layerOptions);
          geoRasterLayer.addTo(map);

          // 5. Zoom the map to the raster bounds
          map.fitBounds(geoRasterLayer.getBounds());
        })
        .catch(err => console.error('Error loading GeoTIFF:', err));
    </script>
  </body>
</html>

