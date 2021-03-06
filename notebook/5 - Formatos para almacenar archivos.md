# Formatos para almacenar archivos

Un formato de archivo es solo una forma de definir cómo se almacena la información en el sistema de archivos HDFS . Esto generalmente es impulsado por el caso de uso o los algoritmos de procesamiento para un dominio específico. El formato del archivo debe ser bien definido y expresivo. Debe ser capaz de manejar una variedad de estructuras de datos específicamente estructuras, registros, mapas, matrices junto con cadenas, números, etc. El formato del archivo debe ser simple, binario y comprimido.

Al tratar con el sistema de archivos de Hadoop, no solo tiene a su disposición todos estos formatos de almacenamiento tradicionales (como puede almacenar imágenes PNG y JPG en HDFS si lo desea), sino que también tiene algunos formatos de archivo enfocados a Hadoop para usarlos estructurados y datos no estructurados. Un gran cuello de botella para aplicaciones habilitadas para HDFS como MapReduce y Spark es el tiempo que lleva encontrar datos relevantes en una ubicación particular y el tiempo que lleva escribir los datos en otra ubicación. Estos problemas se ven agravados por las dificultades para administrar grandes conjuntos de datos, como esquemas en evolución o restricciones de almacenamiento. Los diversos formatos de archivo de Hadoop han evolucionado como una forma de aliviar estos problemas en una serie de casos de uso.

* Formato de texto

    Los archivos simples basados ​​en texto son comunes en el mundo que no pertenece a Hadoop, y también son muy comunes en el mundo de Hadoop. Los datos se presentan en líneas, con cada línea siendo un registro. Las líneas son terminadas por un carácter de nueva línea \ n en la moda típica de UNIX.

    Los archivos de texto son intrínsecamente divisibles (¡solo se divide en \ n caracteres!), Pero si desea comprimirlos tendrá que usar un códec de compresión de nivel de archivo que admita la división, como BZIP2.

    Debido a que estos archivos son solo archivos de texto, puede codificar cualquier cosa que desee en una línea del archivo. Un ejemplo común es hacer que cada línea sea un documento JSON para agregar alguna estructura. Si bien esto puede desperdiciar espacio con encabezados de columna innecesarios, es una forma sencilla de comenzar a utilizar datos estructurados en HDFS.

    * Los formatos predeterminados, JSON , CSV están disponibles
    * Lento para leer y escribir
    * No se pueden dividir los archivos comprimidos (conduce a enormes mapas) 
    * Necesita leer / descomprimir todos los campos.
    
    Un formato de entrada para archivos de texto sin formato. Los archivos están divididos en líneas. El avance de línea o el retorno de carro se utilizan para señalar el final de la línea. Las claves son la posición en el archivo, y los valores son la línea de texto.

    Ventajas: Peso ligero

    Desventajas: Lento para leer y escribir, No se pueden dividir archivos comprimidos (conduce a enormes mapas)

* Formato de archivo de secuencia

    Los archivos de secuencia se diseñaron originalmente para MapReduce, por lo que la integración es fluida. Codifican una clave y un valor para cada registro y nada más. Los registros se almacenan en un formato binario que es más pequeño de lo que sería un formato basado en texto. Al igual que los archivos de texto, el formato no codifica la estructura de las claves y valores, por lo que si realiza migraciones de esquema, deben ser aditivos.

    Normalmente, si necesita almacenar datos complejos en un archivo de secuencia, lo hace en la parte de valor mientras codifica la identificación en la clave. El problema con esto es que si agrega o cambia campos en su clase de escritura, no será retrocompatible con los datos almacenados en el archivo de secuencia.

    Una de las ventajas de los archivos de secuencia es que admiten la compresión a nivel de bloque, por lo que puede comprimir el contenido del archivo y, al mismo tiempo, mantener la capacidad de dividir el archivo en segmentos para múltiples tareas de mapa.

    * El mapa tradicional reduce el formato de archivo binario 
    * Almacena claves y valores como una clase
    * No es bueno para Hive , que tiene tipos sql
    * Hive siempre almacena toda la línea como un valor
    * El tamaño de bloque predeterminado es 1 MB
    * Necesita leer y descomprimir todos los campos

Además de los archivos de texto, Hadoop también brinda soporte para archivos binarios. Fuera de estos formatos de archivo binarios, Hadoop Sequence Files es uno de los formatos de archivo específicos de Hadoop que almacena pares clave / valor serializados.

Ventajas: compacto en comparación con archivos de texto, soporte de compresión opcional. Procesamiento en paralelo. Contenedor para una gran cantidad de archivos pequeños.

Desventajas: no es bueno para Hive, se agrega solo como otros formatos de datos, no se proporciona soporte multilingüe

* Formato de archivo RC (Row-Columnar)

    RCFILE es el acronimo para Record Columnar File, que es otro tipo de formato de archivo binario que ofrece una alta tasa de compresión en la parte superior de las filas utilizadas cuando queremos realizar operaciones en múltiples filas a la vez.

    Los RCFILE son archivos planos que constan de pares clave / valor binarios, que comparten mucha similitud con el ARCHIVO DE SECUENCIA. RCFILE almacena columnas de una tabla en forma de registro de forma columnar. En primer lugar, divide las filas horizontalmente en divisiones de fila y luego divide verticalmente cada división de fila de forma columnar.

    RCFILE primero almacena los metadatos de una división de fila, como la parte clave de un registro, y todos los datos de una fila se dividen como la parte de valor. Esto significa que RCFILE fomenta el almacenamiento orientado a columnas en lugar del almacenamiento orientado a filas. Este almacenamiento orientado a columnas es muy útil al realizar análisis. Es fácil realizar análisis cuando "colgamos" un tipo de almacenamiento orientado a columnas. No podemos cargar datos en RCFILE directamente. Primero, necesitamos cargar datos en otra tabla y luego tenemos que sobrescribirlos en nuestro RCFILE recién creado.

    * columnas almacenadas por separado
    * Leído y descomprimido solo necesitó uno.
    * Mejor compresión
    * Columnas almacenadas como Blobs binarios 
    * Depende de Meta store para suministrar tipos de datos
    * Bloques grandes: 4 MB predeterminados
    * Aún archivo de búsqueda de límite dividido

* Formato de entrada ORC (Optimized Row Columnar)

    ORC significa Columna de fila optimizada, lo que significa que puede almacenar datos de forma optimizada en comparación con otros formatos de archivo. ORC reduce el tamaño de los datos originales hasta en un 75%. Como resultado, la velocidad de procesamiento de datos también aumenta y muestra un mejor rendimiento que los formatos de archivo de Texto, Secuencia y RC.

    Un archivo ORC contiene datos de filas en grupos llamados como Stripes junto con un pie de página de archivo. El formato ORC mejora el rendimiento cuando Hive está procesando los datos. No podemos cargar datos en ORCFILE directamente. Primero, necesitamos cargar datos en otra tabla y luego tenemos que sobrescribirlos en nuestro ORCFILE recién creado.

    El formato completo de archivo ORC es una fila optimizada Formato de archivo en columna.ORC El formato de archivo proporciona una forma muy eficiente de almacenar datos relacionales luego en un archivo RC. Al usar el formato de archivo ORC podemos reducir el tamaño de los datos originales hasta en un 75%. Comparar con texto, secuencia , Formatos de archivo Rc ORC es mejor

    * Columna almacenada por separado
    * Knows Types - Usos Tipos encriptores específicos 
    * Estadísticas de tiendas (Min, Max, Sum, Count)
    * Tiene un índice de peso ligero
    * Omita bloques de filas que no importan 
    * Bloques más grandes: 256 MB de forma predeterminada, tiene un índice para los límites de los bloques

* Formato  AVRO

    Apache Avro es un sistema de serialización de datos independiente del idioma. Fue desarrollado por Doug Cutting, el padre de Hadoop. Dado que las clases editables de Hadoop carecen de portabilidad de idioma, Avro se vuelve bastante útil, ya que trata con formatos de datos que pueden procesarse en múltiples idiomas. Avro es una herramienta preferida para serializar datos en Hadoop.

    Avro es un formato obstinado que entiende que los datos almacenados en HDFS generalmente no son un combo simple de clave / valor como int / string. El formato codifica el esquema de sus contenidos directamente en el archivo, lo que le permite almacenar objetos complejos de forma nativa. Honestamente, Avro no es realmente un formato de archivo, es un formato de archivo más un marco de serialización y deserialización con viejos archivos de secuencia normales, puede almacenar objetos complejos, pero tiene que administrar el proceso.

    Avro maneja esta complejidad al tiempo que proporciona otras herramientas para ayudar a administrar datos a lo largo del tiempo y es un formato bien pensado que define esquemas de datos de archivos en JSON (para interoperabilidad), permite evoluciones de esquema (elimina una columna, agrega una columna) y serialización múltiple / casos de uso de deserialización. También es compatible con la compresión de nivel de bloque. Para la mayoría de los casos de uso basados ​​en Hadoop, Avro se convierte en una buena opción.

* Formato Parquet

    El último hotness en formatos de archivo para Hadoop es  el almacenamiento de archivos en columnas . Básicamente, esto significa que en lugar de simplemente almacenar  filas  de datos adyacentes entre sí, también almacena  valores de columnas  adyacentes entre sí. Por lo tanto, los conjuntos de datos se dividen tanto horizontal como verticalmente. Esto es particularmente útil si su marco de procesamiento de datos solo necesita acceder a un subconjunto de datos que está almacenado en el disco, ya que puede acceder a todos los valores de una sola columna muy rápidamente sin leer registros completos.

    * Diseño basado en Google Dreamel paper
    * Esquema segregado en pie de página
    * Formato principal de columna con rayas
    * Modelo de tipo simple con tipos lógicos
    * Todos los datos empujados a las hojas del árbol
    * Compresión integrada e índices

El formato de archivo de parquet es también un formato columnar. En lugar de simplemente almacenar filas de datos adyacentes entre sí, también almacena valores de columnas adyacentes entre sí. Por lo tanto, los conjuntos de datos se dividen tanto horizontal como verticalmente. Esto es particularmente útil si su marco de procesamiento de datos solo necesita acceder a un subconjunto de datos que está almacenado en el disco, ya que puede acceder a todos los valores de una sola columna muy rápidamente sin leer registros completos. Al igual que el archivo ORC, es excelente para la compresión con un gran rendimiento de consulta, especialmente eficiente al consultar datos de columnas específicas. El formato de parquet es computacionalmente intenso en el lado de la escritura, pero reduce mucho el costo de E / S para lograr un excelente rendimiento de lectura. Disfruta de más libertad que el archivo ORC en la evolución del esquema, que puede agregar nuevas columnas al final de la estructura.

---

Dado que Avro y Parquet tienen tanto en común al elegir un formato de archivo para usar con HDFS, debemos considerar el rendimiento de lectura y el rendimiento de escritura. Debido a que la naturaleza de HDFS es almacenar datos que se escriben una sola vez, leer varias veces, queremos enfatizar en el rendimiento de lectura. La diferencia fundamental en términos de cómo usar cualquier formato es esta: Avro es un formato basado en Fila. Si se desea recuperar los datos como un todo, se puede usar Avro. Parquet es un formato basado en columnas. Si los datos constan de muchas columnas pero se está interesado en un subconjunto de columnas, se puede usar Parquet.