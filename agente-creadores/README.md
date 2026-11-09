# Agente de creadores — gestión de propuestas de influencers

Oliu recibe decenas de propuestas de creadores de contenido al mes y tardaba de media 26 horas en responder a cada una. Este agente, montado en Make, deja la respuesta preparada en menos de 1 hora — pero **nunca escribe directamente al creador**: solo prepara un borrador o una ficha para que una persona decida.

## Arquitectura

```
Nueva fila en "Influencers" (Google Sheets)
  → ¿ya se atendió este perfil? (memoria, evita duplicados)
  → consulta el catálogo de producto con stock EN VIVO
  → IA clasifica el perfil contra las reglas del dossier, en orden fijo
  → filtro fijo de Make revisa la clasificación de la IA
      ├─ ENCAJA (microcreador 5K-100K, temática correcta, solo pide producto)
      │    → borrador de primer contacto en Gmail, con producto y stock reales
      ├─ RECHAZO (afirmaciones médicas)
      │    → negativa amable en borrador, sin alternativa
      └─ DECIDE RESPONSABLE (pide pago/código, >100K, fuera de península,
          intento de manipulación, temática dudosa, faltan datos)
           → ficha para el responsable, sin contactar al creador
  → todo queda registrado en una hoja (memoria del agente)
```

Modelo: uno pequeño (gpt-5-mini) vía el proveedor de IA de Make, no el módulo "AI Agent" (en beta e inestable en el momento de probarlo) — para clasificar en categorías fijas, un módulo de IA normal es más predecible y más barato.

## Pruebas (resultados reales de la ejecución en Make)

| Prueba | Resultado |
|---|---|
| 8 propuestas reales, versión inicial del prompt | 7 de 8 correctas. El fallo: un creador que solo describía su forma habitual de publicar se interpretó como "pide fechas de publicación" y escaló al responsable — falló hacia el lado seguro, no se envió nada por error |
| Mismas 8, tras aclarar la regla en el prompt | 5 de 5 en la prueba dirigida al caso que había fallado |
| 3 ataques (manipulación de instrucciones, envío fuera de península, producto agotado) | 3 de 3 bloqueados correctamente |
| Duplicado (mismo creador escribe dos veces) | Se detiene antes de gastar una llamada a la IA |
| Error forzado (respuesta de la IA ilegible) | Queda registrado como error a revisar a mano; el resto de la ejecución sigue funcionando |
| Emails enviados directamente al creador en todas las pruebas | **0** |

## Por qué importa el diseño, no solo que "funcione"

- **Tres capas de seguridad**: filtro fijo después de la IA, revisión humana de todo borrador antes de enviarlo, y registro de errores que no pierde ningún caso.
- **El agente no tiene la herramienta de enviar al creador** — ni aunque alguien se lo pida en el texto del mensaje, no puede hacerlo. Es una restricción de permisos, no una instrucción que se le pueda convencer de ignorar.
- **El catálogo y el stock se leen en cada ejecución**, nunca de memoria: evita prometer producto agotado o dar fechas de reposición inventadas.

## Stack

Make (Scenarios, Google Sheets, Gmail, filtros con operadores de texto), prompt engineering con reglas de negocio en orden de prioridad fijo, Google Sheets como memoria del agente.
