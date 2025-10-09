<!DOCTYPE html>
<html>
<head>
  <title>Site Location Picker</title>
  <meta charset="utf-8" />
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px auto;
      max-width: 800px;
      padding: 0 20px;
    }
    h2 {
      color: #0078d4;
    }
    #map {
      height: 500px;
      border: 2px solid #ccc;
      border-radius: 8px;
      margin-top: 20px;
    }
    #output {
      margin-top: 15px;
      font-size: 16px;
    }
    #submitBtn {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 16px;
      background-color: #0078d4;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    #submitBtn:hover {
      background-color: #005a9e;
    }
    #siteSelect {
      margin-top: 15px;
      padding: 10px;
      font-size: 16px;
      width: 100%;
    }
    #loadingBox {
      margin-top: 10px;
      font-size: 16px;
      color: #666;
    }
  </style>
</head>
<body>
  <h2>📍 Select Site Location</h2>
  <p>Start by typing a site name below, then click on the map to drop a pin. When ready, click "Submit Location" to send it to SharePoint.</p>

  <!-- Combobox -->
  <input list="siteList" id="siteSelect" placeholder="🔍 Search and select a site..." />
  <datalist id="siteList"></datalist>
  <div id="loadingBox">⏳ Loading site list, please wait…</div>

  <!-- Map -->
  <div id="map"></div>
  <div id="output">No location selected yet.</div>
  <button id="submitBtn" disabled>Submit Location</button>

  <script>
    let selectedLat = null;
    let selectedLng = null;
    let marker;

    // Fetch site list from Power Automate
    fetch("https://default33b73c32b09b4b50bffd9aabd482cc.50.environment.api.powerplatform.com:443/powerautomate/automations/direct/workflows/943ab9f77b5c4b78ac03ed669c0d8b9f/triggers/manual/paths/invoke?api-version=1&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=C61dZxg4VhdVG3nck0tbbiJ7VgaEoOkYX9MvfGvEdBg", {
      method: "GET",
    })
    .then(response => response.json())
    .then(data => {
      const siteListElement = document.getElementById("siteList");
      data.forEach(site => {
        const option = document.createElement("option");
        option.value = site.Title;
        siteListElement.appendChild(option);
      });
      document.getElementById("loadingBox").style.display = "none";
    })
    .catch(error => {
      console.error("Error loading site names:", error);
      document.getElementById("loadingBox").innerText = "⚠️ Failed to load site names.";
    });

    // Map setup
    const map = L.map('map').setView([51.5, -0.12], 10);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

    map.on('click', function(e) {
      if (marker) map.removeLayer(marker);
      marker = L.marker(e.latlng).addTo(map);
      selectedLat = e.latlng.lat;
      selectedLng = e.latlng.lng;
      document.getElementById("output").innerText =
        `📌 Latitude: ${selectedLat.toFixed(6)}, Longitude: ${selectedLng.toFixed(6)}`;
      checkReadyToSubmit();
    });

    document.getElementById("siteSelect").addEventListener("input", checkReadyToSubmit);

    function checkReadyToSubmit() {
      const siteSelected = document.getElementById("siteSelect").value.trim() !== "";
      const locationSelected = selectedLat !== null && selectedLng !== null;
      document.getElementById("submitBtn").disabled = !(siteSelected && locationSelected);
    }

    // Submit to SharePoint
    document.getElementById("submitBtn").addEventListener("click", function() {
      const selectedSite = document.getElementById("siteSelect").value;

      fetch("https://default33b73c32b09b4b50bffd9aabd482cc.50.environment.api.powerplatform.com:443/powerautomate/automations/direct/workflows/5152a2aa37294bebb9abcc8a508d6ed0/triggers/manual/paths/invoke?api-version=1&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=ATn_AIfGGppbiuqcLWVL9tcKe-b9z8qyiXNSYcjavEU", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          siteName: selectedSite,
          latitude: selectedLat,
          longitude: selectedLng
        })
      })
      .then(response => {
        if (response.ok) {
          alert("✅ Location submitted successfully!");
        } else {
          alert("❌ Failed to submit location.");
        }
      })
      .catch(error => {
        console.error("Error submitting location:", error);
        alert("⚠️ Error submitting location.");
      });
    });
  </script>
</body>
</html>
