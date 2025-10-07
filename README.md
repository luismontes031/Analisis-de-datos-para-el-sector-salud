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
l conjunto de datos fue importado a Power BI Desktop, donde se realizó el modelado y la construcción de medidas DAX para obtener indicadores clave en el análisis de salud.
Estas medidas permiten realizar un análisis dinámico por paciente, enfermedad, género, seguro y facturación, facilitando la detección de patrones clínicos y financieros.

A continuación, se describen las principales medidas creadas y su propósito dentro del modelo:

🧮 Medidas Base

Casos = COUNTROWS(data_salud)
Cuenta el número total de registros en la tabla data_salud. Representa la cantidad de casos o atenciones médicas registradas.
Esta medida es fundamental para calcular frecuencias, proporciones y totales generales.

Edad Promedio = AVERAGE(data_salud[Edad])
Calcula la edad promedio de los pacientes.
Permite analizar la distribución etaria de los usuarios y detectar si ciertas enfermedades afectan grupos de edad específicos.

FacturaciónUSD = SUM(data_salud[FACTURACION]) / 1000000
Suma la facturación total y la convierte a millones de dólares.
Facilita la interpretación de montos elevados en gráficos y KPIs financieros.

🏥 Indicadores Clínicos y de Estancia

Total Estancia = SUM(data_salud[Estancia_Dias])
Suma la cantidad total de días de hospitalización de los pacientes.
Es útil para medir la demanda de camas, uso de recursos y severidad promedio de los casos.

Enfermedad Principal =
Esta medida identifica la enfermedad con mayor facturación por paciente.
Se basa en la función TOPN y CALCULATE para devolver la enfermedad más costosa o recurrente por persona.
Ayuda a detectar los diagnósticos que generan mayor carga económica en el sistema de salud.

Paciente_Estancia_Max =
Devuelve el paciente con la mayor cantidad total de días de estancia.
Incluye además la enfermedad asociada y el número de días.
Sirve para identificar casos críticos o estancias prolongadas que impactan la eficiencia hospitalaria.

💊 Indicadores por Género

Mayor_Enfermedad_Femenino =
Crea una tabla virtual que agrupa las enfermedades y calcula el número de casos en mujeres (Genero = "F").
Devuelve la enfermedad más frecuente en población femenina junto con la cantidad de casos.
Permite análisis de salud con enfoque de género.

Mayor_Enfermedad_Masculino =
Similar a la anterior, pero enfocada en el género masculino (Genero = "M").
Identifica la principal causa de atención o enfermedad en hombres, facilitando comparaciones entre géneros.

💰 Indicadores Financieros

Seguro mayor factura =
Determina cuál es el seguro o entidad que acumula la mayor facturación total.
Utiliza SUMMARIZE y TOPN para comparar el monto facturado por aseguradora.
Es esencial para priorizar relaciones con aseguradoras de alto volumen o riesgo.

FacturaciónSeguroMayorGlobal =
Calcula la facturación total del seguro identificado como el de mayor facturación global.
Ayuda a cuantificar el peso financiero de esa entidad dentro del sistema.

Paciente_Med_Facturacion_Max =
Devuelve el paciente con mayor facturación junto con su medicación principal y el valor total.
Proporciona una visión combinada entre consumo de medicamentos y costos.

📈 Indicadores Comparativos y de Proporción

Porcentaje_Enfermedad =
=DIVIDE(COUNTROWS(data_salud), CALCULATE(COUNTROWS(data_salud), ALL(data_salud[ENFERMEDAD])))
Mide el porcentaje de cada enfermedad respecto al total global.
Permite construir gráficos de distribución de enfermedades por su frecuencia relativa.

Porcentaje_Por_Resultado =
Calcula el porcentaje de casos por resultado de examen (positivo, negativo, indeterminado) dentro de cada enfermedad.
Facilita el análisis clínico de efectividad o incidencia según resultados de laboratorio.

Ranking Paciente Global =
=RANKX(ALL(data_salud[NOMBRE]), [FacturacionUSD], , DESC)
Asigna un ranking a los pacientes de acuerdo con su facturación total.
Se utiliza en dashboards para identificar los pacientes más costosos o de mayor impacto económico.

Top 5 Pacientes Global =
Filtra los pacientes que se encuentran dentro del top 5 en facturación global, permitiendo construir visualizaciones destacadas con los casos más representativos.

🧠 Resumen Analítico

Estas medidas fueron diseñadas para combinar perspectivas clínicas, demográficas y financieras dentro de un mismo modelo de Power BI.
La integración de funciones como VAR, CALCULATE, TOPN, RANKX, SUMMARIZE y CONCATENATEX permitió desarrollar un entorno analítico robusto y flexible.

En total, se implementaron 15 medidas DAX para este modelo, organizadas en categorías base, clínicas, financieras y comparativas.
Este enfoque modular facilita la ampliación futura del modelo, permitiendo incorporar predicciones de costo, prevalencia por edad o segmentación por entidad aseguradora.

💡 Conclusión:
El modelado en Power BI no solo permitió visualizar indicadores, sino transformar los datos en un sistema de inteligencia clínica-financiera, donde es posible responder preguntas como:

¿Qué enfermedad genera la mayor carga económica?

¿Cuál aseguradora concentra los costos más altos?

¿Qué pacientes requieren estancias más largas o medicación costosa?

¿Cómo varían los resultados clínicos según género o edad?

Este enfoque integral convierte el proyecto en un dashboard analítico avanzado en salud, listo para ser integrado a un portafolio profesional o entorno institucional.
En conclusión, este proceso permitió transformar un conjunto de datos crudo y con múltiples inconsistencias en una base sólida, confiable y lista para análisis descriptivos y predictivos en el ámbito de la salud.
Este flujo de trabajo combina la precisión de Excel para la limpieza detallada con el poder analítico y visual de Power BI, logrando un resultado profesional y reproducible dentro del ciclo de vida de un proyecto de ciencia de datos aplicada al sector salud.
