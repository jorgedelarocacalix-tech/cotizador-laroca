# Cotizador La Roca — CONTEXT.md

## ¿Qué es?
Aplicación web de cotización de precios para **Corporación La Roca** (Honduras).  
Single-file HTML → `index.html`. Desplegado en GitHub Pages y Netlify.

---

## URLs
- **GitHub Pages**: https://jorgedelarocacalix-tech.github.io/cotizador-laroca/
- **Repositorio**: https://github.com/jorgedelarocacalix-tech/cotizador-laroca
- **Netlify**: https://cotizador-laroca.netlify.app

---

## Stack
- **Frontend**: `index.html` (todo en un solo archivo — HTML + CSS + JS)
- **Base de datos**: Supabase — proyecto `upaenjotkocmdvfuobii`
  - Tabla `cotizador_categorias` — márgenes por categoría (mayoreo, contado, pagos, tasa 6M, tasa 12M)
  - Tabla `cotizador_productos` — productos importados por el usuario
- **Librería Excel**: SheetJS (`xlsx@0.18.5` via unpkg CDN) — para leer y generar `.xlsx`

---

## Tabs disponibles
| Tab | Función |
|-----|---------|
| ⚡ Cotizar | Ingresa descripción + costo → genera Mayoreo / Promoción / Contado / 6M / 12M |
| 🔄 Comparar | Tabla comparativa de todos los productos REF por categoría y proveedor |
| 📥 Subir | Sube una lista `.xlsx` o `.csv` con nombre y costo → importa a Supabase |
| 🏍️ Motos | Cotizador de Motos: plan de cuotas (Contado, Prima, Tasa mensual) → cuotas 6/12/18/24/36 meses |

---

## Lógica de precios

### Tarjeta "Precios de venta" (arriba)
Calculada con márgenes de Supabase (`cotizador_categorias`):
- **Mayoreo** = costo × (1 + margen_mayoreo%)
- **Promoción** = costo × (1 + margen_cash%)
- **Contado** = costo × (1 + margen_pagos%)
- **6 Meses** = Contado × (1 + tasa_6m%)
- **12 Meses** = Contado × (1 + tasa_12m%)

Se aplica **piso mínimo**: si el precio calculado es menor que el de la lista vigente (REF), se usa el de la lista.

### Tarjeta "Precios vigentes La Roca" (abajo, azul)
Escala los precios reales del producto más cercano en el array `REF`.  
- Nunca muestra menos que los precios reales del producto de referencia.
- Si el costo ingresado > costo de referencia → precios escalan proporcionalmente hacia arriba.
- La cuota mensual (6M/12M) usa el ratio real del proveedor (no es total÷6), porque DIVECO aplica prima + cuotas.

---

## Array REF (datos de proveedores)
~110 productos de referencia con precios reales directamente en el HTML.

| Proveedor | Categorías | Productos |
|-----------|-----------|-----------|
| SAMSUNG | refrigeradora, televisión | 6 |
| DIUNSA | refrigeradora, estufa, lavadora | 14 |
| RCA | estufa | 3 |
| HONDUCAMAS | cabecera, recámara, sala | 15 |
| DIVECO | cama (unip/matri/queen/king) | 37 |

Cada entrada REF tiene: `cat, marca, nombre, costo, mayoreo, promo, contado, t6, c6, t12, c12, proveedor`

---

## Detección automática de categoría
La función `autoCategoria(texto)` detecta por palabras clave:
- Motos → Honda, Yamaha, 125cc, moto, etc.
- Electrodomésticos → refrigerador, estufa, lavadora, TV, etc.
- Dormitorio → cama, colchón, closet, recámara
- Sala y Comedor → sala, sofá, comedor, mesa

---

## Detección de tamaño en camas
Al buscar referencia para camas, filtra por tamaño si aparece en la descripción:
- "king" → Camas King
- "queen" → Camas Queen
- "matrimonial" / "matri" / "doble" → Camas Matrimoniales
- "unipersonal" / "unip" / "individual" → Camas Unipersonales

---

## Subir lista (tab 📥 Subir)
Acepta `.xlsx` y `.csv`. Detecta automáticamente la fila de encabezados (busca hasta la fila 10).  
Columnas requeridas: **nombre** (o "descripción") y **costo** (o "precio costo").  
Columnas opcionales: codigo, modelo.  
Ignora filas con "anterior" en columna A (precios viejos) y filas que empiezan con "retiro".

### Formato mínimo válido:
```
nombre                              | costo
Refrigeradora Samsung 14 pies       | 13685
Cama Matrimonial Antiestress DL     | 5672.94
Honda CB125F                        | 12000
```

---

## Cotizador de Motos (tab 🏍️ Motos)
Calcula el plan de cuotas de una motocicleta a interés fijo (add-on) sobre el saldo financiado, replicando la lógica de la hoja "PLAN DE CUOTAS INVERSIONES CALIX 2025".

**Inputs**: Precio de Contado, Prima (enganche, default L6,000), Tasa mensual % (default 4.7%).

**Fórmula** (por cada plazo `n` en {6, 12, 18, 24, 36} meses):
```
Financiado   = Contado − Prima
Interés      = Financiado × (Tasa/100) × n
Cuota Mensual = (Financiado + Interés) / n
Total a Pagar = Prima + Financiado + Interés
```

Validado contra los 5 valores reales de la hoja original (12M: L3,779.67/mes; 18M: L2,974.11/mes; 24M: L2,571.33/mes; 36M: L2,168.56/mes; 6M sin prima: L7,478.33/mes) — coincide exacto.

Función JS: `calcCuotasMoto(contado, prima, tasaPct)` en `index.html`.

**Marca y prima mínima**: el selector "Marca" carga desde la tabla Supabase `cotizador_motos_marcas` (`marca`, `prima_min_pct`, `activa`). Al cotizar, si la prima ingresada es menor al `prima_min_pct` de la marca × Contado, se ajusta automáticamente al mínimo (se muestra "(mínima aplicada)" y un toast). Marcas iniciales sembradas: Honda, Italika, Bajaj, Pulsar, Shineray, Zmoto — todas con 15% por defecto, editable en Admin.

**Admin → Marcas de Motos**: nueva sub-pestaña en el modal de administración (`showAdmTab('motos')`) para editar el % mínimo de cada marca, agregar marcas nuevas o eliminarlas. Funciones: `renderMarcasMotoList`, `dirtyMarca`, `saveMarca`, `addMarcaMoto`, `eliminarMarcaMoto`.

---

## Despliegue
```bash
# Subir cambios
git add index.html
git commit -m "descripción del cambio"
git push origin main
# → GitHub Pages actualiza en ~1-2 minutos
```

---

## Archivos locales clave
- `/Users/jorgecalix/cotizador-laroca/index.html` — toda la app
- `/Users/jorgecalix/Downloads/NUEVA LISTA DE CAMAS 2025.xlsx - TODAS LAS CAMAS.csv` — lista DIVECO original
- `/Users/jorgecalix/Downloads/LISTA DE PRECIO VARIOS JUNIO 2026.xlsx` — lista La Roca (varios productos)
