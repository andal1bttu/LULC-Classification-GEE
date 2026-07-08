// =====================================================
// HIGH DEM + SLOPE + ASPECT + HILLSHADE (Export) from ROI
// ROI asset from your screenshot:
// projects/peaceful-crane-486406-v9/assets/Export_Output
// =====================================================

// 1) ROI
var roiFC = ee.FeatureCollection('projects/peaceful-crane-486406-v9/assets/Export_Output');
var roi = roiFC.geometry();
Map.centerObject(roiFC, 10);
Map.addLayer(roiFC, {}, 'ROI');

// 2) High-quality DEM (Copernicus GLO-30 -> ImageCollection so mosaic)
var dem = ee.ImageCollection('COPERNICUS/DEM/GLO30')
  .select('DEM')
  .mosaic()
  .clip(roi);

// Mask NoData/invalid (clean edges in ArcGIS/QGIS)
dem = dem.updateMask(dem.neq(-32768)).updateMask(dem.gt(-10));

// 3) Terrain products
var terrain = ee.Terrain.products(dem);
var slope = terrain.select('slope');     // degrees
var aspect = terrain.select('aspect');   // degrees (0-360)

// Hillshade (you can change azimuth/zenith if you want)
var hillshade = ee.Terrain.hillshade(dem, 315, 45).clip(roi);

// 4) Visualize
// Optional: print min/max for DEM to choose good vis
var demStats = dem.reduceRegion({
  reducer: ee.Reducer.minMax(),
  geometry: roi,
  scale: 30,
  maxPixels: 1e13
});
print('DEM min/max:', demStats);

Map.addLayer(dem, {min: 0, max: 300, palette: ['0b3d91','2aa9d6','ffffbf','fdae61','d7191c']}, 'DEM (GLO-30)');
Map.addLayer(slope, {min: 0, max: 30}, 'Slope (deg)');
Map.addLayer(aspect, {min: 0, max: 360}, 'Aspect (deg)');
Map.addLayer(hillshade, {min: 0, max: 255}, 'Hillshade');

// 5) Export helpers
function exportToDrive(img, name, scale) {
  Export.image.toDrive({
    image: img,
    description: name,
    folder: 'GEE_Exports',
    fileNamePrefix: name,
    region: roi,
    scale: scale,
    crs: 'EPSG:4326',
    maxPixels: 1e13
  });
}

// 6) Exports (GeoTIFF)
exportToDrive(dem, 'DEM_Copernicus_GLO30_ROI', 30);
exportToDrive(slope, 'Slope_deg_GLO30_ROI', 30);
exportToDrive(aspect, 'Aspect_deg_GLO30_ROI', 30);
exportToDrive(hillshade, 'Hillshade_GLO30_ROI', 30);

// =====================================================
// If Copernicus is not available for you, use ALOS AW3D30:
// var dem = ee.ImageCollection('JAXA/ALOS/AW3D30/V3_2')
//   .select('DSM')
//   .mosaic()
//   .clip(roi)
//   .updateMask(ee.ImageCollection('JAXA/ALOS/AW3D30/V3_2').select('DSM').mosaic().gt(-10));
// =====================================================
