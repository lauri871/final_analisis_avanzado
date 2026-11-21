## Previsión de pernoctaciones hoteleras de no residentes (E1)
### Datos

De las 552 series de la tabla, 138 son pernoctaciones de residentes en el extranjero. De esas, 97 empiezan por «Nacional.».

Series elegidas:

| Destino | Código |
|---|---|
| Benidorm | EOT2626 |
| Palma de Mallorca | EOT2658 |
| Marbella | EOT2762 |
| San Bartolomé de Tirajana | EOT2814 |
| Adeje | EOT2830 |


### Trampas encontradas y decisiones
1. **Fechas en milisegundos:** La API da las fechas en milisegundos desde 1970 (UTC). El día 1 de cada mes llega como las 22:00 del día anterior, así que una conversión directa nos movería el mes. Decisión: convertir a hora de Madrid y dejar todas las fechas en día 1 de mes (freq="MS").

2. **Último mes publicado:** Con nult=2 vimos que el último dato es septiembre de 2026. Nuestras series irán de enero de 2005 a septiembre de 2026. Los últimos meses pueden ser provisionales y revisarse después.

3. **Series sin histórico suficiente:** De las 138 series de pernoctaciones de no residentes, solo 97 son «Nacional.» y tienen datos desde 2005. Las otras 41 son puntos añadidos en 2024. Decisión: usar solo las «Nacional.». Al buscar «Palma» salió también Las Palmas de Gran Canaria; la descartamos y usamos solo Palma de Mallorca.

4. **Cierre por COVID.** En los cinco destinos abril de 2020 vale 0 y mayo y junio de 2020 están vacíos. Con un 0 no se puede usar log ni Box-Cox. Decisión: tratar estos meses como una intervención, no como datos normales. Cómo lo haremos exactamente lo decidimos al modelar.

5. **Mes que falta en Marbella.** En Marbella no existe la fila de mayo de 2020 (en los otros destinos existe, pero vacía). Ya tenemos el calendario completo de 261 meses para los cinco destinos.

6. **Secreto estadístico:** El INE no publica algunos valores por secreto estadístico y llegan vacíos con la marca secreto = True. Son mayo y junio de 2020 en todos los destinos (en Marbella solo consta junio; mayo no llega como fila) y febrero de 2021 en Benidorm. Decisión: tratarlos como dato ausente. Los de 2020 se tratan junto con el abril a 0 como intervención del COVID.

7. **Datos provisionales.** En la API cada dato trae un campo `FK_TipoDato`. Al pedir la serie con el parámetro `tip=A`, la API devuelve el nombre: el valor 1 es «Definitivo» y el 2 es «Provisional». Los 252 primeros meses son definitivos y los 9 últimos (ene-sep 2026) provisionales, en los cinco destinos (el texto lo comprobamos en Benidorm; en los demás, por el código). Decisión: guardar este campo y la fecha de descarga junto a los datos, y tener en cuenta que los últimos meses de la prueba pueden cambiar si el INE los revisa.

8. **Limpieza de huecos para los modelos.** Los modelos no admiten nulos y Box-Cox no admite ceros. Decisión: tratar abril, mayo y junio de 2020 y febrero de 2021 (Benidorm) como huecos y rellenarlos con interpolación lineal. Guardamos una columna `imputado` para saber qué meses no son datos reales. Los huecos están lejos de los cortes de la partición, así que no se usan datos del futuro. Comprobamos que quedan 1305 filas (261 meses por 5 destinos), ningún nulo y ningún cero.


## Resultados de los modelos de referencia
El seasonal naïve es el mejor de los cuatro en todas las ventanas y en el test. En el test final tiene un MASE de 0.32 (MAE de unas 23 900 pernoctaciones), frente a 2.31 del naïve y del drift y 2.80 de la media histórica. Naïve y drift dan resultados casi iguales.

El MASE del seasonal naïve baja de 0.77 (ventana 1) a 0.37 (ventana 3). Creemos que se debe a que la ventana 1 está más cerca de la recuperación del COVID, pero no lo hemos comprobado.

Para E2, el modelo a batir es el seasonal naïve: un MASE de 0.32 en el test.


## Observaciones del gráfico
Palma, Benidorm y Marbella tienen una estacionalidad muy fuerte, con máximos en verano. Adeje y San Bartolomé de Tirajana son mucho más planas durante el año. En todos los destinos se ve la caída del COVID en 2020 y la recuperación hasta 2022. Los meses imputados (abril-junio de 2020 y febrero de 2021 en Benidorm) caen en esa caída. En 2024-2026 las series vuelven a niveles parecidos o superiores a los previos al COVID.


## Resultados por destino
El seasonal naïve es el mejor modelo en los cinco destinos, tanto en la ventana rodante como en el test. En el test su MASE va de 0.15 (Adeje) a 0.59 (Marbella). Marbella es el destino más difícil y su MASE no mejora año a año como en los demás. En Benidorm, en la ventana 1 (sep-2022), el MASE del seasonal naïve fue 1.17, el único valor por encima de 1.

El naïve y el drift se defienden mejor en Adeje y San Bartolomé de Tirajana (MASE entre 0.7 y 0.85), donde la serie es más plana, y fallan mucho en Palma (MASE 5). La media de los cinco destinos esconde estas diferencias, por eso damos también los resultados por destino.

Para E2, el modelo a batir es el seasonal naïve, con especial atención a Marbella.


## Predicciones del test
Gráficamente, el seasonal naïve sigue bien la forma estacional en los cinco destinos. Los mayores errores porcentuales caen en meses de poco volumen (noviembre a marzo en Palma) y en el arranque de temporada de Marbella, donde el modelo se queda por debajo del valor real (errores de hasta el 30%). En los meses provisionales (ene-sep 2026) no vemos errores claramente mayores que en oct-dic 2025, aunque son pocos puntos y no es una prueba.