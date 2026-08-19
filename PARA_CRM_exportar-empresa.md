# Para el CRM (Seguimiento Rotoplast): incluir la razón social en "Pasar al cotizador"

**Problema detectado:** el botón **"🧮 Pasar al cotizador"** de la ficha de contacto arma el
payload `simpleclima: "cliente-v1"` con `empresa: ""` aunque el contacto tenga cargada una
empresa / razón social en el CRM.

Caso real: contacto **Eduardo Portillo**, arquitecto de **Hiper del Pollo**. El JSON exportado
trae `"nombre": "Eduardo Portillo", "empresa": ""` — la empresa se pierde y el presupuesto
sale a nombre de la persona sola.

## Qué corregir

En la función que arma el payload del botón "Pasar al cotizador":

1. **`empresa` debe salir del campo empresa/razón social del contacto** en Supabase (el dato
   ya existe en el CRM), no solo del parseo "Persona — Empresa" del nombre.
2. Mantener `nombre` = persona de contacto (nombre y apellido).
3. Si el CRM tiene la empresa como entidad relacionada (tabla aparte), mandar su razón social
   en `empresa` igual.

## Formato esperado (sin cambios de estructura, solo llenar bien `empresa`)

```json
{
  "simpleclima": "cliente-v1",
  "crmId": "6bf23936-8f35-4760-9d6d-c880966b11fa",
  "nombre": "Eduardo Portillo",
  "empresa": "Hiper del Pollo",
  "tel": "5492215993309",
  "ciudad": "Corrientes",
  "provincia": "Corrientes",
  "direccion": "RP5 Km 1.7, W3400 Corrientes",
  "rubro": "Deportivo",
  "producto": "",
  "responsable": "Seba"
}
```

## Cómo lo usa el cotizador (ya implementado — no requiere cambios allá)

- El campo "Razón Social / Nombre" del presupuesto se arma como
  **"Hiper del Pollo — At.: Eduardo Portillo"** cuando hay empresa, o solo la persona si no hay.
- Defensivo: si `empresa` viene vacía pero `nombre` trae "Persona — Empresa", el cotizador
  los separa igual. Pero lo correcto es que el CRM mande el dato en su campo.
- Dedup por `crmId`: reexportar el mismo contacto actualiza el cliente, no lo duplica; si la
  empresa venía vacía de antes, al reimportar con empresa cargada se completa.
