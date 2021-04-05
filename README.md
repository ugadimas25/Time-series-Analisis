# Time-series-Analisis
Analisis NDVI secara time series menggunakan citra landsat

```javascript

// TIME SERIES

// Buat polygon (geometry) sebagai batas untuk analisis time series.
// Buat koleksi citra S-2 (Sentinel 2) untuk periode 2016-2019.
var S2 = ee.ImageCollection('COPERNICUS/S2')

// filter tanggal mulai dan berakhir
.filterDate('2016-01-01', '2019-01-01')

// filter sesuai batas yang ditentukan
.filterBounds(geometry);

// Function untuk mask cloud dari built-in informasi kualitas saluran
// dari tutupan awan
var maskcloud1 = function(image) {
    var QA60 = image.select(['QA60']);
    return image.updateMask(QA60.lt(1));
};

// Fungsi untuk menghitung dan menambahkan saluran NDVI
var addNDVI = function(image) {
    return image.addBands(image.normalizedDifference(['B8', 'B4']));
};

// Menambahkan saluran NDVI ke koleksi citra
var S2 = S2.map(addNDVI);
// Ekstrak saluarn NDVI dan membuat citra komposit median NDVI
var NDVI = S2.select(['nd']);
var NDVImed = NDVI.median(); //Hanya mengubah nama variabel ini ;)

// Membuat palettes untuk menampilkan NDVI
var ndvi_pal = ['#d73027', '#f46d43', '#fdae61', '#fee08b', '#d9ef8b',
    '#a6d96a'
];

// Membuat time series chart.
var plotNDVI = ui.Chart.image.seriesByRegion(S2, geometry, ee.Reducer.mean(),
        'nd', 500, 'system:time_start', 'system:index')
    .setChartType('LineChart').setOptions({
        title: 'NDVI short-term time series',
        hAxis: { title: 'Date' },
        vAxis: { title: 'NDVI' }
    });

// Display.
print(plotNDVI);

// Display hasil NDVI ke peta
Map.centerObject(geometry, 12);
Map.addLayer(NDVImed.clip(geometry), { min: -0.5, max: 0.9, palette: ndvi_pal }, 'NDVI');

// Parameter visualisasi.
var args = {
    crs: 'EPSG:3857', // Maps Mercator
    dimensions: '300',
    region: geometry,
    min: -1,
    max: 1,
    palette: ndvi_pal,
    framesPerSecond: 4,
};

// Buat thumbnail video dan tambahkan ke peta.
var thumb = ui.Thumbnail({
    // Menentukan koleksi "image", membuat animasi urutan "images".
    image: NDVI,
    params: args,
    style: {
        position: 'bottom-right',
        width: '320px'
    }
});
Map.add(thumb);
