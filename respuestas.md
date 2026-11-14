## A2UI

### Salida convencional

Una respuesta en texto podría resumir el hallazgo en una tabla:

| Hallazgo | Evidencia | Severidad | Acción |
|---|---|---|---|
| Consulta SQL dinámica | `buscar_usuario` concatena el nombre recibido en la consulta. | Alta | Pedir revisión (`SQL-01`) |

### Microejemplo declarativo

Para este ejemplo defino el catálogo `revision_demo`:

| Componente | Propiedades permitidas en este ejemplo |
|---|---|
| `Card` | `children`: IDs de los componentes contenidos; solo acepta `Text` y `Button`. |
| `Text` | `text`: texto plano. |
| `Button` | `child`: ID de un `Text`; `action.event.name`: solo `pedir_revision`; `action.event.context.hallazgoId`: identificador del hallazgo. |

**Ejemplo ilustrativo y didáctico; usa el catálogo definido arriba y no representa un mensaje A2UI completo de una versión específica.**

```json
{
  "catalogo": "revision_demo",
  "componentes": [
    {
      "id": "card_1",
      "component": "Card",
      "children": ["titulo", "detalle", "boton_revision"]
    },
    {
      "id": "titulo",
      "component": "Text",
      "text": "Hallazgo: consulta SQL dinámica"
    },
    {
      "id": "detalle",
      "component": "Text",
      "text": "buscar_usuario concatena el nombre recibido en la consulta."
    },
    {
      "id": "etiqueta_revision",
      "component": "Text",
      "text": "Pedir revisión"
    },
    {
      "id": "boton_revision",
      "component": "Button",
      "child": "etiqueta_revision",
      "action": {
        "event": {
          "name": "pedir_revision",
          "context": {
            "hallazgoId": "SQL-01"
          }
        }
      }
    }
  ]
}
```

### Quién describe y quién valida

El agente o backend produce la descripción declarativa: indica qué componentes permitidos mostrar, sus propiedades y las acciones disponibles. El cliente no ejecuta código de interfaz enviado por el agente; interpreta la descripción y la dibuja usando sus componentes nativos.

El cliente valida la estructura JSON contra el esquema y el catálogo que soporta. Comprueba tipos y propiedades, que existan los IDs referenciados, que la composición de componentes esté permitida y que el evento esté en la lista permitida. Si falla la validación o aparece un componente o evento desconocido, rechaza esa descripción en vez de renderizarla como confiable.

### Autorización del evento `pedir_revision`

El botón solo solicita la acción; no concede permiso para realizarla. El cliente puede ser manipulado y un evento puede fabricarse o repetirse sin pulsar el botón. Por eso el backend debe autenticar a quien lo envía y autorizar cada solicitud: verificar que ese usuario pueda pedir revisión para el `hallazgoId`, que el hallazgo exista y que su estado permita la acción. También debe validar el payload y aplicar controles contra repeticiones o abuso. La validación del cliente mejora la interfaz, pero la decisión de autorización corresponde al backend.

Referencias: [especificación A2UI v1.0](https://github.com/a2ui-project/a2ui/blob/main/specification/v1_0/docs/a2ui_protocol.md) · [guía para definir un catálogo](https://a2ui.org/guides/defining-a-catalog/).

## Preguntas finales

1. **¿Por qué dos especialistas y no uno solo? ¿Qué mejora y qué costo agrega?**  
   Separar seguridad y pruebas da una revisión más enfocada y reduce puntos ciegos. A cambio, agrega coordinación, tiempo y consumo de contexto; el orquestador también debe conciliar resultados.

2. **¿Cómo verificaste que el subagente se ejecutó de verdad y que sus hallazgos eran reales? ¿Qué harías si fallara?**  
   No puedo afirmar que se ejecutó: [la captura disponible](evidencias/revisor-seeguridad.png) muestra el TOML, no una invocación. Codex CLI no llegó a iniciar el agente y la revisión automática bloqueó el lanzamiento por el posible envío de `usuarios.py` a un servicio externo. Por eso no atribuyo hallazgos al subagente. Para verificarlo buscaría un hilo o registro con `revisor_seguridad` y cotejaría cada línea y afirmación con el archivo. Si falla, marcaría el análisis como pendiente, pediría corrección o avisaría a la persona; no inventaría una salida.

3. **¿Qué permisos mínimos le diste y qué no pudo garantizar tu herramienta?**  
   En `revisor-seguridad.toml` indiqué lectura y búsqueda, y desactivé apps, plugins, memorias y solicitudes de permisos. Codex no garantiza desde ese archivo un sandbox individual sin escritura: el subagente hereda los permisos de la sesión. La mitigación es iniciar el padre en modo de solo lectura y revisar `git status` después; la invocación de este caso no llegó a ejecutarse.

4. **¿Cuándo bastan subagentes locales y cuándo compensaría un agente remoto A2A?**  
   Un subagente local basta cuando el trabajo ocurre en el mismo entorno y bajo el mismo orquestador. A2A compensa si hay que delegar a un agente independiente de otro servicio u organización; añade descubrimiento, interfaz de red, autenticación y una frontera de confianza que validar.

5. **¿Cómo manejaría tu cliente `input-required` o un fallo de la Task?**  
   Ante `input-required`, pediría a la persona el dato faltante, lo validaría y continuaría la tarea con un mensaje. Ante `failed`, registraría el error y ofrecería un reintento acotado o revisión humana, sin presentar la tarea como completada.

6. **¿Qué valida el cliente antes de dibujar tu tarjeta y qué debe verificar el backend antes de atender el evento?**  
   El cliente valida el JSON, los datos, el catálogo, los componentes y propiedades permitidos, las referencias entre componentes, las URLs permitidas y la acción `pedir_revision`. El backend autentica al usuario y autoriza la acción para ese `hallazgoId` y estado, además de validar el payload; el botón no concede permiso.

7. **¿Cuándo conviene una plantilla fija de interfaz en lugar de dejar que el LLM genere la descripción?**  
   Una plantilla fija conviene si la pantalla debe ser estable, predecible o seguir requisitos estrictos de seguridad y accesibilidad. La generación dinámica sirve cuando la presentación varía según la tarea, siempre limitada al catálogo permitido y validada por el cliente.
