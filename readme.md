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

7. **Datos provisionales.** En la API cada dato trae un campo `FK_TipoDato`. Los 252 primeros meses tienen el valor 1 y los 9 últimos (ene-sep 2026) el valor 2. Interpretamos que el 2 es provisional, y que el INE puede revisarlo. Decisión: guardar este campo y la fecha de descarga junto a los datos, y tener en cuenta que los últimos meses de la prueba pueden cambiar. Comprobamos que los 9 meses de tipo 2 son los mismos en los cinco destinos.

