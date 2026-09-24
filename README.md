# Módulo 5 — Cobertura de Riesgo Cambiario con Swap de Divisas

**Portafolio Actuarial | Finanzas Corporativas e Instrumentos Derivados**  
Fecha de referencia: septiembre 2026

---

## Descripción

Proyecto de cobertura cambiaria para una empresa agroexportadora argentina con ingresos en USD y costos fijos en ARS. Se diseña y valúa un **currency swap plain vanilla a 12 meses**, se cuantifica el costo de la cobertura y se analiza la sensibilidad del instrumento ante shocks de tasas de interés.

El proyecto demuestra la aplicación práctica de la **Paridad de Tasas de Interés Cubierta (CIP)** y la **valuación de swaps por no arbitraje** — dos enfoques metodológicos distintos que convergen al mismo resultado, validando la consistencia del modelo.

---

## Caso de negocio

| Variable | Valor |
| Ingresos | USD 500,000 / mes |
| Costos operativos | ARS 460,500,000 / mes (fijos) |
| Horizonte | 12 meses |
| Exposición | Conversión de USD a ARS a TC incierto |
| TC spot de referencia (CCL) | ARS 1,587.52 / USD |

**Problema:** la empresa no conoce el TC al que convertirá sus exportaciones futuras. Un atraso cambiario (depreciación menor a la implícita en la curva de forwards) comprime el margen operativo en ARS.

**Solución:** un swap de divisas que fija el TC de conversión para cada período, transformando el resultado en ARS en un flujo determinístico.

---

## Estructura del Swap

| Parámetro | Valor |
| Tipo | Swap de divisas USD/ARS — pagos mensuales |
| Nocional USD | USD 500,000 por período (USD 6,000,000 total) |
| Nocional ARS | ARS 793,760,000 |
| Tasa fija ARS (empresa paga) | **25.83% TEA** |
| Tasa fija USD (empresa recibe) | 4.47% TEA (SOFR 12m) |
| TC forward a 12 meses | ARS 1,964.46 / USD |
| Depreciación implícita | 23.7% en 12 meses |
| Valor en t = 0 | ARS 0 (no arbitraje) |

---

## Metodología

### Fase 1 — Construcción de la curva de tasas ARS
- Fuente: LECAPs y BONCAPs del mercado doméstico (11 nodos, tramo 1–12 meses)
- Se excluye el nodo de 9 días (`S30S6L`) por precio distorsionado
- Interpolación sobre `log(DF)` mediante **spline cúbico** para obtener factores de descuento en los 12 nodos mensuales exactos
- Referencia USD: curva SOFR interpolada por el mismo método

### Fase 2 — Curva de tipos de cambio forward (CIP)
$$F_{0,T} = S_0 \cdot \left(\frac{1 + r_{ARS,T}}{1 + r_{USD,T}}\right)^{T/365}

- TC spot: CCL (ARS 1,587.52)
- Tasas ARS: TEA LECAP interpolada por plazo
- Tasas USD: TEA SOFR interpolada por plazo

### Fase 3 — Tasa fija del swap (`c_ARS`)
Se resuelve numéricamente con `scipy.optimize.brentq` la condición NPV = 0:

$$\sum_{t=1}^{12} c_{ARS} \cdot N_{ARS} \cdot \Delta t_t \cdot DF_{ARS,t} + N_{ARS} \cdot DF_{ARS,12} = N_{USD} \cdot S_0$$

La pata USD colapsa a par porque `c_USD = r_USD` (SOFR flat).

### Fase 4 — Valuación por dos métodos (no arbitraje)

**Método 1 — Portafolio de forwards:**
$$V_{swap} = \sum_{t=1}^{12} (F_{0,t} - K_t) \cdot N_{USD} \cdot DF_{ARS,t} = 0$$

**Método 2 — Diferencia de bonos:**
$$V_{swap} = B_{USD} \cdot S_0 - B_{ARS} = 0$$

Ambos métodos dan **ARS 0** en `t = 0` — validación de la condición de no arbitraje.

### Fase 5 — Stress test (mark-to-market)

| Shock tasas ARS | V_swap |
| −1,000 bps | −ARS 59.3 M |
| −500 bps | −ARS 28.5 M |
| Sin shock | ARS 0 |
| +500 bps | +ARS 26.4 M |
| +1,000 bps | +ARS 50.9 M |

La posición pagadora de tasa fija tiene duración implícita positiva: cuando las tasas caen, el pasivo ARS se encarece y el swap deteriora su valor de mercado para la empresa.

---

## Resultados — Análisis de escenarios (12 meses acumulado)

| Escenario | Sin cobertura | Con swap | Diferencia |
| Base (TC fijo) | ARS 3,684 M | ARS 5,128 M | **+ARS 1,444 M** |
| Moderado (+1.04%/mes) | ARS 4,230 M | ARS 5,128 M | **+ARS 898 M** |
| Fuerte (+2.6%/mes) | ARS 5,122 M | ARS 5,128 M | **+ARS 6 M** |

**Insight clave:** en el escenario de devaluación fuerte, la cobertura es prácticamente neutral porque la empresa hubiera recibido más ARS convirtiendo a spot. El valor de la cobertura está en proteger contra el **atraso cambiario**, no contra la devaluación acelerada. Para el contexto acutal argentino (depreciacion menor mes a mes) puede esperarse un resultado entre el caso base y moderado

**Costo de cobertura implícito:**
- TC forward promedio ponderado: ARS 1,767.59
- Prima total sobre spot: **11.34%** (≈ 22–23% anualizado)


## Fuentes de datos

| Dataset | Fuente |
|---|---|
| TC oficial, MEP, CCL (histórico) | BCRA / Ámbito Financiero |
| BADLAR (histórico) | BCRA — Series de tiempo |
| Curva LECAP/BONCAP | BYMA / Mercado Abierto Electrónico |
| SOFR (curva por plazos) | CME Group / FRED |

---

## Herramientas

- **Python:** `pandas`, `numpy`, `scipy` (CubicSpline, brentq), `matplotlib`
- **Excel:** modelo de flujos de caja por escenario
- **Claude AI:** redacción del informe ejecutivo

---

## Conceptos aplicados

- Paridad de Tasas de Interés Cubierta (Covered Interest Rate Parity)
- Valuación de swaps por portafolio de forwards equivalente
- Valuación de swaps por diferencia de bonos (Hull Cap. 7 / Chance 6.5)
- Interpolación de curvas de tasas sobre log-descuentos (spline cúbico)
- Análisis de sensibilidad (DV01 implícita del swap)
- Mark-to-market ante shocks paralelos de tasas
