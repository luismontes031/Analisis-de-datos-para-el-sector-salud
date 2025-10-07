🩺 Ciencia de Datos en Salud — Limpieza, Normalización y Transformación del Dataset

El conjunto de datos utilizado, healthcare_dataset.csv, contiene más de 55 filas y 14 campos, y fue obtenido desde Kaggle.
El objetivo de este proceso fue realizar una limpieza, normalización y transformación completa en Excel, preparando los datos para un posterior análisis en Power BI.

En primer lugar, se validaron los tipos de datos de cada columna según su naturaleza: texto, numérico o fecha.
Este paso asegura que las operaciones posteriores sean coherentes y que los cálculos se ejecuten correctamente.

Antes de eliminar duplicados, fue necesario normalizar los datos para evitar que registros que en realidad son iguales sean tratados como distintos debido a diferencias en mayúsculas, tildes o espacios adicionales.
La normalización incluyó convertir todo el texto a mayúsculas, eliminar espacios innecesarios, caracteres especiales y prefijos como “Dr.” o “Mrs.”.
Para ello se aplicó la siguiente fórmula en columnas auxiliares:

=ESPACIOS(SI(DERECHA(SUSTITUIR(SUSTITUIR(SUSTITUIR(SUSTITUIR(MAYUSC(A2);"DR.";"");"MS.";"");"MRS.";"");"MR.";"");1)="."; IZQUIERDA(SUSTITUIR(SUSTITUIR(SUSTITUIR(SUSTITUIR(MAYUSC(A2);"DR.";"");"MS.";"");"MRS.";"");"MR.";""); LARGO(SUSTITUIR(SUSTITUIR(SUSTITUIR(SUSTITUIR(MAYUSC(A2);"DR.";"");"MS.";"");"MRS.";"");"MR.";""))-1); SUSTITUIR(SUSTITUIR(SUSTITUIR(SUSTITUIR(MAYUSC(A2);"DR.";"");"MS.";"");"MRS.";"");"MR.";"")))

También se validó que no existieran valores numéricos en el campo de nombre con la fórmula:
=SI(SUMA(--ESNUMERO(VALOR(EXTRAE(B2;FILA(INDIRECTO("1:"&LARGO(B2)));1))));"Contiene número";"OK")

y se verificaron las celdas vacías con:
=ESBLANCO(B2)

Posteriormente se analizó la columna de edad, verificando valores faltantes con =ESBLANCO(C2) y comprobando que las edades estuvieran dentro de un rango lógico, ya que en salud no pueden existir edades negativas ni superiores a 120 años:
=SI(Y(C2>=0;C2<=120);"OK";"Error")

En la columna género, se estandarizaron los valores para unificar distintas representaciones (“Male”, “HOMBRE”, “F”, “Female”) utilizando:
=SI(O(C2="M";C2="MALE";C2="HOMBRE");"M";SI(O(C2="F";C2="FEMALE";C2="MUJER");"F";"Revisar"))

Sin embargo, se observó que muchos registros no correspondían al género real del nombre.
Por ello se enriquecieron los datos utilizando un conjunto adicional obtenido de Kaggle con nombres y géneros (forenames dataset).
Dado que el archivo era grande, se importó mediante Power Query, filtrando únicamente registros del país USA, coherentes con el dataset principal.
Al notar bajo rendimiento en Power Query, el proceso continuó en Power BI, que cuenta con un motor más potente.

El dataset de nombres presentaba también errores y ambigüedades (nombres con género M y F simultáneamente).
Se decidió extraer únicamente el primer nombre de la tabla principal, eliminar duplicados y quedarse con valores únicos para crear un diccionario confiable.
Después de esta depuración, se redujo de más de 50.000 filas a 640 nombres únicos.
Con ayuda de inteligencia artificial (GPT) se asignó el género correcto a cada fila y se aplicó la siguiente fórmula para cruzar los datos en la tabla original:

=BUSCARV(IZQUIERDA(A33793; HALLAR(" "; A33793 & " ") - 1); Hoja1!A:B; 2; FALSO)

Esto permitió completar la columna de género con el valor correcto según el primer nombre.

En las columnas de nombres de doctores y hospitales se eliminaron símbolos innecesarios como puntos, comas y guiones, que podrían generar inconsistencias en futuros cruces de datos:
=ESPACIOS(SUSTITUIR(SUSTITUIR(A2;",";"");"-";""))
y todas las columnas de texto se convirtieron a mayúsculas con =MAYUSC(A2).

También se verificó que la fecha de ingreso fuera siempre menor a la fecha de egreso con:
=SI(A2<B2;"OK";"ERROR")

Durante la revisión de la columna facturación, se identificaron valores negativos, lo cual no tiene sentido en este tipo de datos.
Dado que las fechas de ingreso y egreso estaban completas y no existía otra columna que indicara que el servicio no se prestó, se concluyó que era un error de codificación.
Por ello se convirtieron todos los valores a positivos con:
=ABS(Facturación)

Posteriormente se realizó una limpieza final de espacios, normalización de tipos de datos y eliminación de caracteres especiales.
Al eliminar duplicados verdaderos, se suprimieron 533 registros del conjunto de datos original.
Para garantizar la unicidad lógica de los registros, se creó una clave compuesta mediante la concatenación de campos relevantes:
=NOMBRE & GENERO & TIPO_SANGRE & ENFERMEDAD & DOCTOR

Esto permitió detectar casos donde filas aparentemente diferentes (por ejemplo, con edad distinta) correspondían al mismo paciente.

Se añadió un índice para generar un identificador único por registro:
=FILAS($A$2:A2)

Este índice permite organizar, buscar y referenciar los datos de manera eficiente, incluso si existen pacientes con nombres repetidos.

Finalmente, el archivo limpio fue importado en Power BI para crear visualizaciones profesionales.
Con los datos ya transformados y validados, se generaron gráficos que muestran la distribución por género, edad y enfermedad, además del promedio de facturación por servicio, el comportamiento por hospital o doctor y la detección de anomalías en registros.

En conclusión, este proceso permitió transformar un conjunto de datos crudo y con múltiples inconsistencias en una base sólida, confiable y lista para análisis descriptivos y predictivos en el ámbito de la salud.
Este flujo de trabajo combina la precisión de Excel para la limpieza detallada con el poder analítico y visual de Power BI, logrando un resultado profesional y reproducible dentro del ciclo de vida de un proyecto de ciencia de datos aplicada al sector salud.
