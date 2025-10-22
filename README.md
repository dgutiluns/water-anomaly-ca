# water-anomaly-ca
GenAI for Water Data QA/QC

# Document the COMID/StreamCat enrichment 
## What is a COMID?

  - A COMID is the unique identifier for an NHDPlus catchment (stream reach + its local drainage).
  - StreamCat aggregates dozens of watershed/catchment metrics keyed by COMID (land cover, soil, dams, PRISM climate, etc).

## How we obtain COMID from sample coordinates

  - For each sample (lat, lon) in WGS84, we convert to Web Mercator (EPSG:3857) because that’s what the StreamCat web app sends to the        ArcGIS server.
  - We POST a point query to the EPA ArcGIS NHDPlus MapServer layer 6 with spatialRel=esriSpatialRelWithin, outFields=FEATUREID.
  - The server returns a featureid polygon hit. This featureid is effectively the COMID used by StreamCat endpoints.
  - We attach that COMID to the sample row.

## How we attach StreamCat metrics
  - Deduplicate COMIDs to avoid repeated network calls.
  - For each unique COMID, call https://api.epa.gov/StreamCat/streams/waters_streamcat with header comid: <COMID>.
  - Flatten the nested JSON sections (one record per section) into a single row of columns (e.g., aoi.wsareasqkm, soil.omws,                  prism_norm_8110.precip8110ws, etc.).
  - Left-join the flattened StreamCat table back to the sample table on COMID.
