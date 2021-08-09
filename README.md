# Python and QGIS 2

## Geocodes

Primero, para obtener información de los puntos de las capitales departamentales buscamos información en Google y dimos con el siguiente [link](https://es.wikipedia.org/wiki/Anexo:Departamentos_de_la_provincia_de_Corrientes). En la carpeta Geocode, se encuentra el código correspondiente en python haciendo scrapping para obtener las capitales departamentales y guardarlas en un csv para poder usarla en QGIS. Para obtener las coordenadas de las capitales departamentales seguimos tres procedimientos.

1) MMQGIS.

Luego de hacer scrapping (hasta la línea 70 del código) armamos una base de datos que contuviera los campos requeridos en MMQGIS, a saber, Country, State y City. Exportamos esta base como csv para usarla en QGIS y pedimos a MMQGIS que realizara las búsquedas. En este caso, los resultados fueron insatisfactorios porque el algoritmo ponía puntos agrupados en un mismo municipio.

Para mejorar la precisión era necesario cargar los address. Sin embargo, no era conveniente. Por ejemplo, para el municipio de Santa Lucía, había que buscar, en Google Maps, "Argentina, Corrientes, Lavalle, Municipalidad de Santa Lucía".

2) "A mano".

Lo anterior nos llevó al método 2. Al hacer la búsqueda de las municipalidades manualmente, Google Maps devolvía la dirección de los edificios en donde estaba ubicada la municipalidad capital de cada localidad. Luego, habría que copiar y pegar la dirección en el csv para luego abrirla en MMQGIS. No obstante, esto no era útil porque haciendo click derecho sobre el edificio de la municipalidad, se podía copiar directamente las coordenadas con muy alto grado de precisión. Es decir, en el método 2, buscamos los edificios de cada una de las capitales y se copiaron y pegaron las coordenadas en un csv.

3) Python.

Entendiendo que el método 2 fue posible porque se trataba de relativamente pocos municipios, tratamos de hacer las búsquedas automáticamente (y lo logramos). Creamos una cuenta en Google Cloud y obtuvimos una API key. Desde la línea 86 del código está la búsqueda. Usando las bases creadas anteriormente logramos obtener los mismos puntos que bajando la información a mano. Nos quedamos con este procedimiento al final. El código fue adaptado desde el [Github de Google Maps](https://github.com/googlemaps/google-maps-services-python).

En la carpeta, están los archivos csv. El archivo Corrientes2.csv es el que contiene los puntos de las capitales municipales.

## Shapfiles

Los archivos shape que usamos como boundaries de Corrientes están [aquí](https://datos.gob.ar/dataset/ign-unidades-territoriales/archivo/ign_01.02.02). La provincia de Corrientes tiene como IN1 = 18. Es decir, 18 para provincia, 18... para municipios.

Los archivos shape de rutas nacionales se encuentran [aquí](https://datos.transporte.gob.ar/dataset/rutas-nacionales).

Los archivos shape de rutas provinciales se encuentran [aquí](https://datos.transporte.gob.ar/dataset/rutas-provinciales).

No encontramos archivos shape sobre calles. Esto es importante porque implica que, a la hora de calcular costos, éstos serán mayores. Además, porque los puntos de las capitales departamentales no están conectados usando únicamente las rutas.

Para obtener las rutas nacionales y provinciales en un mismo shape en QGIS: Vectorial (Vector) >> Herramientas de gestión de datos (Data Management Tools) >> Unir capas vectoriales (Merge Shapefiles to one), ahí seleccionamos las capas de rutas nacionales y provinciales, y la exportamos como shapefile.

## Networks

Creamos esta layer con el objetivo de unir los puntos de las capitales con las rutas para que pueda conectarlas ([aquí](AIzaSyAslDtZJHYRYZvfDTrPzlJlBmxTlMOi3YM)). Para esto usamos v.distance desde capitales hacia rutas y arrojó los puntos más próximos y la distancia. Luego unimos rutas con distancia en una sola layer.

Tomamos los vértices y calculamos la distancia entre los puntos más próximos y las capitales. A esta layer le añadimos una variable cost que mide el costo en pesos por kilómetro. Para calcularla tomamos el consumo de nafta de un auto promedio ([ver aquí](https://siomaa.com:8082/Documents/Reports/informe_Parque_junio_2020.pdf?name=Parque%20Automotor%20jun%202020&date=01-07-2020)), que utiliza 6.2 litros de nafta cada 100 km. Así, este auto promedio consume un litro de nafta cada 16.13 kilómetros.

Por otro lado, el precio promedio de nafta (premium) en Corrientes, a agosto de 2021, es de $ 103,78 por litro de nafta ([ver aquí](http://datos.minem.gob.ar/dataset/precios-en-surtidor/archivo/80ac25de-a44a-4445-9215-090cf55cfda5?filters=provincia%3ACORRIENTES%7Cproducto%3ANafta+(premium)+de+m%C3%A1s+de+95+Ron)), por lo que el precio de nafta por kilómetro es $ 6.43 ($ 103.78 / 16.13).

## Matriz OD

Teniendo los puntos de las capitales departamentales, las rutas que los unen y el costo promedio por kilómetro, usamos QNEAT para obtener la matriz OD. Se seteó default speed en 90 (km/h) y Topology Tolerance en 0.0001. Es una archivo .csv en la carpeta Matriz OD.
