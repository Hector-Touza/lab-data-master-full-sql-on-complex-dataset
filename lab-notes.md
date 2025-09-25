## Qué errores se detectaron en el dataset original
- DTE_ORDERS invalidos
- ID_CUSTOMER en blanco
- ID_PART que contienen "ERROR"
- QTY_ORDERED con valor null o valor no numérico
- CAT_ORDER_TYPE con valores sin sentido
- TYP_CUSTOMER_SEGMENT en blanco
- QTY_UNITS_BACKORDER con valores N/A
- Prácticamente todas las columnas contienen valores en blanco

## Cuáles fueron las reglas de limpieza más relevantes
- Transformar los distintos valores error (ERROR, N/A... a null)
- Transformar valores vacíos en null
- pasar payment method a mayúsculas y convertir valores vacío en null
- Validar si REF_DISCOUNT_CODE contiene 'SUMMER20' o 'BLACKFRIDAY' y convertir cualquier otro valor a null
## Cómo usaste TCL y DCL en un contexto real
No toco mucho SQL así que no tengo experiencia en un entorno productivo con ellos
## Respuestas a las preguntas de análisis del lab
### ¿Qué % de registros contenía errores en las fechas o cantidades?
<pre>
SELECT COUNT(ID_CUSTOMER)
FROM S_ORDERS_TRANSFORMED
WHERE TRY_TO_DATE(DTE_ORDER) IS NULL
OR TRY_TO_DATE(DTE_DELIVERY_EST) IS NULL
OR TRY_TO_NUMBER(QTY_ORDERED) IS NULL
OR TRY_TO_NUMBER(QTY_UNITS_BACKORDER) IS NULL;
</pre>
842 de 1001 filas contenían algun valor erróneo  
NOTA: no he considerado AMT_TOTAL al ser el prefijo amount en lugar de quantity.
### ¿Qué métodos de pago son más comunes en pedidos de +1000 EUR?
<pre>
SELECT REF_PAYMENT_METHOD,
COUNT(ID_ORDER)
FROM S_ORDERS_TRANSFORMED
WHERE AMT_TOTAL > 1000
GROUP BY REF_PAYMENT_METHOD
ORDER BY COUNT(ID_ORDER) DESC;
</pre>
NUll:	222  
CASH:	133  
CARD:	120  
CRYPTO:	115  
TRANSFER:	105
### ¿Cuál es el segmento de cliente con más backorders?
<PRE>
SELECT TYP_CUSTOMER_SEGMENT,
SUM(QTY_UNITS_BACKORDER)
FROM S_ORDERS_TRANSFORMED
GROUP BY TYP_CUSTOMER_SEGMENT
ORDER BY SUM(QTY_UNITS_BACKORDER) DESC;
</PRE>
REGULAR: 208  
VIP: 207  
NULL:	205  
B2C:	190  
B2B:	173  
### ¿Cuántos registros fueron descartados durante limpieza?
No se descarta ningún registro entre bronce y silver
