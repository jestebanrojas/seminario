README

Trabajo realizado por:
-	Juan Esteban Rojas
-	Mateo Castañeda

Para la construcción del dataset final, el cual será empleado en la aplicación de los diferentes modelos de analítica para el pronóstico de la demanda de energía eléctrica con una resolución horaria se requiere realizar en primera instancia la obtención de información de diferentes fuentes:

* 	Histórico de demanda de energía eléctrica para un mercado de comercialización específico con resolución horaria: En este caso se tomará la información para el mercado de comercialización Antioquia y se descargará la información disponible en el portal de XM para dicho mercado de comercialización desde el año 2018. XM publica en su portal la información de manera mensual, con un archivo por cada mercado de comercialización. Al descargar la información, se ubica la información de interés en la carpeta: ./Datos/historico demanda/historico la información se encuentra en un archivo .csv para cada uno de los meses entre 2018-01 hasta 2023-09

* 	Usuarios correspondientes al mercado de comercialización: Información disponible en el SUI con los usuarios de todos los comercializadores para el mercado de comercialización seleccionado (Antioquia). La información se encuentra en el portal del SUI (Sistema único de información de la superintendencia de servicios públicos domiciliarios). La información se descarga en un archivo para cada mes desde el 2018-01. Esta se dispone en la ruta: ./Datos/usuarios/Mensuales

*	Datos climatológicos:  Información relativa al histórico de temperatura, precipitaciones, humedad relativa, radiación solar, claridad del cielo, entre otros, de los municipios del departamento del mercado de comercialización elegido. La información se toma del portal de la NASA dispuesto para tal fin y se emplea la API disponible en: // https://power.larc.nasa.gov/data-access-viewer/ ; para más información consultar en https://power.larc.nasa.gov/docs/. Para descargar esta información se hace necesario contar con las coordenadas de los municipios respectivos, por lo que se toma la información disponible en el DANE y se dispone la información en la siguiente ruta: ./Datos/climatologicos/coordenadas.csv

* 	Datos Macroeconómicos: se tomará información relativa al IPP e IPC del país de manera mensual, disponible en el DANE. Esta información se almacena en la siguiente ruta ./Datos/Macroeconomicos/Evolución IPC E IPP base 2008 - Resolución_ Mensual.xlsx. Adicionalmente se incluirá información de la TRM diaria, la cual se toma del Banco de la República. Esta información se dispone en la siguiente ruta:  ./Datos/Macroeconomicos/TRM.xlsx


El repositorio presenta la siguiente estructura en su primer nivel:
*   Carpeta Datos: Contiene los diferentes archivos que sirven de información fuente y que será procesada.
*   Carpeta Codigos: Contiene los diferentes notebooks que se encargan de procesar los datos.
*   Carpeta Datasets_salida: contiene los diferentes archivos con los datasets que se cosntruyen a lo largo de la  preparación de los datos, contiene también el dataset final así como los datasets propuestos para entrenamiento y validación.

A continuación, se presentan las instrucciones para correr los códigos necesarios para la construcción del Dataset que será empleado en el desarrollo del proyecto:

La primera acción es correr el notebook Ejecutor.ipynb, el cual se encargaará de ejecutar cada una de los siguientes notebooks e intrucciones:

1. Corre el Notebook Obtener_data_demandas.ipynb el cual se encuentra en la carpeta ./Codigos/historico demanda. Este Notebook se encarga de unir los diferentes archivos xslx y xls ubicados en la ruta ./Datos/historico demanda/historico, los cuales contienen la demanda mensual por día y hora para el mercado de comercialización seleccionado (Antioquia). Al final del código del Notebook se crea un archivo csv con el nombre de demanda.csv con la información consolidada en la ruta Datasets_salida/historico demanda.

2. Corre el Notebook Obtener_usuarios.ipynb el cual se encuentra en la carpeta ./Codigos/usuarios. Este Notebook se encarga de unir los diferentes archivos csv ubicados en la ruta ./Datos/usuarios/Mensuales, los cuales contienen la información mensual de todos los usuarios, comercializadores y municipios del departamento (Antioquia). Al final del código del Notebook se crea un archivo csv con el nombre de usuarios.csv con la información consolidada en la ruta Datasets_salida/usuarios.

3. Corre el Notebook Descarga_historicos.ipynb el cual se encuentra en la carpeta ./Codigos/climatologicos. Este Notebook toma información del archivo coordenadas.csv ubicado en la misma ruta, con la finalidad de cargar en un dataset las coordenadas de los cascos urbanos de los municipios del departamento. Posteriormente se procede a realizar la descarga de los datos climatológicos deseados (temperatura ambiente, precipitaciones, humedad relativa, radiación solar, claridad del cielo) para cada una de las coordenadas, esto a través del uso de una API dispuesta por el portal de la NASA empleando un método get. Al final del código del Notebook se crea un archivo csv con el nombre de clima.csv con la información consolidada en la ruta Datasets_salida/climatologicos.

4. Corre el Notebook ponderado_clima_usuarios_macros.ipynb el cual se encuentra en la carpeta ./Codigos/union_datasets. Este notebook se encarga de consolidar la información de usuarios, clima y datos macroeconomicos, en un dataset que contendrá información para el departamento en cada día/hora del periodo 01/01/2018 al 31/10/2023. El notebook realiza las siguientes acciones:
- Primero: Cargar el dataset que proviene del documento clima.csv (resultante dle paso 3) y unir el dataset con documento usuarios.csv (Resultante del paso 2). Se realiza esta unión con método merge(left join) y teniendo como llaves las columnas municipio, año y mes. En este caso para cada día/hora en cada municipio se toma como cantidad de usuarios la correspondiente a la cantidad de usuarios del mes para dicho municipio reportado en el archivo usuarios.csv, dado que esta información no se tiene con la resolución día/hora. 
Nota: El archivo clima.csv debido a su tamaño no es posible cargarlo en repositorio de Github por lo que se debe de cargar directamente en el Notebook en la ruta /content/seminario/Datasets_salida/climatologicos.
- Se procede a realizar una ponderación de cada una de las variables climatológicas para cada municipio año, mes, día, hora, con la cantidad de usuarios para dicho periodo para posteriormente dividirlo en la cantidad total de usuarios del departamento y obtener un valor promedio ponderado de Antioquia para cada variable climatológica.
- Luego se une el dataset anterior con el dataset del archivo TRM.xlsx en la ruta ./Datos/macroeconomicos. Se realiza esta unión con método merge(left join) y teniendo como llaves las columnas año, mes y día.
- Posterior se une el dataset incluyendo la TRM con el dataset del archivo Evolución IPC E IPP base 2008 - Resolución_ Mensual.xlsx en la ruta ./Datos/macroeconomicos. Se realiza esta unión con método merge(left join) y teniendo como llaves las columnas año, mes.
- Al final del código del Notebook se crea un archivo csv con el nombre de clima_usuarios_macro.csv con la información consolidada en la ruta Datasets_salida/dataset_final. 

5. Corre el Notebook unir_fuentes.ipynb el cual se encuentra en la carpeta ./Codigos/union_datasets. Dicho notebook se encarga de poner a disposición el dataset final y carga las siguientes fuentes: clima_usuarios_macro.csv (resultante del paso 4) y demanda.csv (Resultante del paso 1). Para el dataset demanda.csv .
- El dataset proveniente del archivo demanda.csv se somete a una transformación (unpivot a través de la función melt) con la finalidad de convertir las columnas de cada hora en cada registro en registros individuales, ya que en el original un solo registro corresponde a las demandas de todo un día, cuyos valores horarios corresponden a una columna (P1 …. P24). 
- De forma análoga a los pasos anteriores, se realiza el merge (left join) con el dataset del archivo clima_usuarios_macro.csv , considerando como llaves las columnas año, mes, dia hora.
- El dataset se almacena en un nuevo archivo CSV con el nombre dataset_demada_Antioquia.csv en la ruta Datasets_salida/dataset_final.

6. Para la creación de los datasets de entrenamiento y validación se debe correr el notebook dataset_split.ipynb el cual se encuentra en la carpeta ./Codigos/Entrenamiento_Validacion_Split. Este Notebook carga el archivo dataset_demada_Antioquia.csv resultante del paso anterior y aplica 2 enfoques para crear 2 datasets de entrenamiento y 2 de validación, esto aplicando para el enfoque 1, la separación según fecha y para el enfoque 2 una separación aleatoria. estos son almacenados en la ruta Datasets_salida/Entrenamiento_Validacion_Split, carpetas enfoque 1 y enfoque 2 según corresponda.

Nota: los diferentes notebooks que cargan archivos disponibles en cualquiera de las carpetas, tienen la opción de ser sincronizados desde el repositorio de Github o ser cargados desde Google drive, para lo cual, el usuario deberá comentar y descomentar las líneas de código según corresponda y garantizar que cuente con los archivos en las rutas respectivas. Lo anterior no aplica para la carga del archivo clima.csv, el cual por su tamaño, no puede ser cargado en Github sino que se debe hacer a través de Google drive o cargar directamente en colab (para lo cual se debe ajustar la línea de código para apuntar a la ruta en la que se cargue).

