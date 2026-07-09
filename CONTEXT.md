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
