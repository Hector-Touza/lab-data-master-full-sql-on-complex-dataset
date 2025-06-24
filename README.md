![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Lab Avanzado | SQL completo sobre Dataset Complejo

## 🎯 Objetivo

Aplicar de forma avanzada los comandos SQL (DDL, DML, DQL, TCL, DCL) sobre un dataset complejo, con múltiples columnas, errores, y estructuras heterogéneas. El objetivo es limpiar, transformar y dejar listo el dataset para consumo analítico (Silver).

## Requisitos

* Haz un ***fork*** de este repositorio.
* Clona este repositorio.

## Entrega

- Haz Commit y Push
- Crea un Pull Request (PR)
- Copia el enlace a tu PR (con tu solución) y pégalo en el campo de entrega del portal del estudiante – solo así se considerará entregado el lab

## 📁 Dataset

**Nombre:** `B_ORDERS_RAW_COMPLEX_WIDE.csv`  
**Ubicación:** `DEV_BRONZE_DB.B_LEGACY_ERP.B_ORDERS_RAW_COMPLEX_WIDE`

### 🧾 Campos destacados

- Fechas erróneas (`INVALID_DATE`)
- Códigos corruptos (`ERROR`, `None`)
- Cantidades no numéricas (`X`, `-3`, `"null"`)
- Métodos de pago inválidos
- Códigos de descuento incorrectos
- Campos con `N/A`, nulos, vacíos

## 1. 🔧 DDL – Definición de tablas

```sql
CREATE OR REPLACE TABLE DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED (
  ID_ORDER STRING,
  DTE_ORDER DATE,
  ID_CUSTOMER STRING,
  ID_PART STRING,
  QTY_ORDERED NUMBER,
  AMT_TOTAL NUMBER(12,2),
  REF_PAYMENT_METHOD STRING,
  DTE_DELIVERY_EST DATE,
  DES_ORDER_NOTE STRING,
  CAT_ORDER_TYPE STRING,
  TYP_CUSTOMER_SEGMENT STRING,
  REF_DISCOUNT_CODE STRING,
  QTY_UNITS_BACKORDER NUMBER,
  TST_INSERTION TIMESTAMP,
  AUD_USR_INSERT STRING
);
```

## 2. ✍️ DML – Limpieza y carga de datos

```sql
INSERT INTO DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED
SELECT
  ID_ORDER,
  TRY_TO_DATE(DTE_ORDER, 'YYYY-MM-DD'),
  ID_CUSTOMER,
  CASE WHEN ID_PART != 'ERROR' THEN ID_PART ELSE NULL END,
  CASE 
    WHEN IS_NUMBER(QTY_ORDERED) AND TRY_TO_NUMBER(QTY_ORDERED) > 0 THEN TRY_TO_NUMBER(QTY_ORDERED)
    ELSE NULL
  END,
  TRY_TO_NUMBER(AMT_TOTAL),
  NULLIF(UPPER(REF_PAYMENT_METHOD), ''),
  TRY_TO_DATE(DTE_DELIVERY_EST, 'YYYY-MM-DD'),
  DES_ORDER_NOTE,
  NULLIF(CAT_ORDER_TYPE, ''),
  NULLIF(TYP_CUSTOMER_SEGMENT, ''),
  CASE WHEN REF_DISCOUNT_CODE IN ('SUMMER20', 'BLACKFRIDAY') THEN REF_DISCOUNT_CODE ELSE NULL END,
  CASE WHEN IS_NUMBER(QTY_UNITS_BACKORDER) THEN TRY_TO_NUMBER(QTY_UNITS_BACKORDER) ELSE NULL END,
  TRY_TO_TIMESTAMP(TST_INSERTION),
  AUD_USR_INSERT
FROM DEV_BRONZE_DB.B_LEGACY_ERP.B_ORDERS_RAW_COMPLEX_WIDE;
```

## 3. 🔍 DQL – Consultas de validación y exploración

```sql
-- 1. Ver cuántos registros tienen al menos un campo importante nulo
SELECT COUNT(*) AS NUM_REGISTROS_NULOS
FROM DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED
WHERE DTE_ORDER IS NULL 
   OR QTY_ORDERED IS NULL
   OR AMT_TOTAL IS NULL
   OR REF_PAYMENT_METHOD IS NULL;

-- 2. Pedidos con descuento y alto importe
SELECT ID_ORDER, AMT_TOTAL, REF_DISCOUNT_CODE
FROM DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED
WHERE REF_DISCOUNT_CODE IS NOT NULL AND AMT_TOTAL > 1000;

-- 3. Clientes VIP con pedidos incompletos
SELECT *
FROM DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED
WHERE TYP_CUSTOMER_SEGMENT = 'VIP' AND QTY_UNITS_BACKORDER > 0;
```

## 4. 🔁 TCL – Transacciones

```sql
BEGIN;

UPDATE DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED
SET AMT_TOTAL = AMT_TOTAL * 0.95
WHERE REF_DISCOUNT_CODE = 'BLACKFRIDAY';

ROLLBACK;

-- Confirmar descuento si revisado
UPDATE DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED
SET AMT_TOTAL = AMT_TOTAL * 0.95
WHERE REF_DISCOUNT_CODE = 'BLACKFRIDAY';

COMMIT;
```

## 5. 🔐 DCL – Control de acceso

```sql
GRANT SELECT, INSERT ON DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED TO ROLE BI_ANALYST;

REVOKE INSERT ON DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED FROM ROLE BI_ANALYST;
```

## 🧠 Preguntas para analizar

1. ¿Qué % de registros contenía errores en las fechas o cantidades?
2. ¿Qué métodos de pago son más comunes en pedidos de +1000 EUR?
3. ¿Cuál es el segmento de cliente con más backorders?
4. ¿Cuántos registros fueron descartados durante limpieza?

## Entregables

Dentro de tu repositorio forkeado, asegúrate de incluir los siguientes archivos:

* `create.sql` – Script con las sentencias DDL (`CREATE TABLE`) para `S_ORDERS_TRANSFORMED`
* `insert.sql` – Script con las sentencias DML (`INSERT INTO ... SELECT`) y reglas de limpieza
* `queries.sql` – Script con las sentencias DQL, TCL y DCL aplicadas en el lab
* `lab-notes.md` – Documento explicativo que incluya:
  * Qué errores se detectaron en el dataset original
  * Cuáles fueron las reglas de limpieza más relevantes
  * Cómo usaste `TCL` y `DCL` en un contexto real
  * Respuestas a las preguntas de análisis del lab
* *(Opcional)* Capturas de pantalla con resultados de las consultas

## ✅ Conclusión

Este laboratorio avanzado aplica múltiples comandos SQL de forma coordinada sobre un dataset complejo. Has realizado limpieza, normalización parcial, uso de transacciones y preparado datos para análisis final o visualización.

- 📁 Dataset original: `B_ORDERS_RAW_COMPLEX_WIDE.csv`
- 📂 Silver destino: `DEV_SQL_LAB.S_SANDBOX.S_ORDERS_TRANSFORMED`