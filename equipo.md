# Equipo de revisión de `usuarios.py`

## Tabla del equipo

| Rol | Enfoque y responsabilidad | Archivo de trabajo | Entrega |
|---|---|---|---|
| Orquestador | Divide la revisión, asigna análisis en paralelo, coteja hallazgos con el código y consolida la tabla después del checkpoint humano. | `C:\flutter\flutter_deber6\deber6-agentes\usuarios.py` | Tabla final con hallazgos respaldados por evidencia. |
| Especialista 1 — Seguridad | Revisa la consulta de `buscar_usuario`, las credenciales incluidas en el código y los riesgos de acceso a la base de datos. Señala ubicación y evidencia, sin editar el archivo. | `C:\flutter\flutter_deber6\deber6-agentes\usuarios.py` | Hallazgos de seguridad con explicación y evidencia concreta. |
| Especialista 2 — Calidad y pruebas | Revisa comportamiento y casos límite de las funciones, manejo de conexiones y pruebas faltantes. Comprueba los hallazgos directamente contra el código. | `C:\flutter\flutter_deber6\deber6-agentes\usuarios.py` | Observaciones de calidad y casos de prueba sugeridos. |
| Persona — checkpoint humano | Antes de consolidar, verifica que cada hallazgo tenga evidencia, detecta falsos positivos y resuelve discrepancias relevantes entre especialistas. | `C:\flutter\flutter_deber6\deber6-agentes\usuarios.py` | Hallazgos aceptados, corregidos o descartados con criterio humano. |

## Flujo de trabajo

El orquestador entrega el mismo archivo a ambos especialistas en paralelo. Sus análisis pasan por el checkpoint humano; solo después el orquestador prepara la tabla final. El diagrama está en [`evidencias/parte1_flujo.png`](evidencias/parte1_flujo.png).

## Control humano y manejo de fallos

- La persona coteja las afirmaciones con `usuarios.py`, comprueba que la evidencia sustente el riesgo y filtra falsos positivos de seguridad. También revisa desacuerdos entre los dos análisis antes de aceptar un hallazgo.
- Si una salida llega vacía, el orquestador la marca como pendiente y solicita una nueva revisión enfocada; no inventa hallazgos para completar la tabla. Puede avanzar con la salida válida del otro especialista, dejando explícita la revisión faltante.
- Si una salida contiene afirmaciones sin sustento o contradice el archivo, el orquestador descarta o devuelve esas afirmaciones para verificación. Solo consolida los puntos respaldados por líneas o fragmentos concretos del código y los somete al checkpoint humano.

## A2A

### Agent Card ilustrativa

**Simplificación didáctica; no es JSON conforme a una versión específica de A2A.**

```json
{
  "name": "Revisor de seguridad",
  "description": "Busca claves codificadas y entradas sin validar en usuarios.py",
  "url": "https://agente.ejemplo/a2a",
  "version": "0.1",
  "capabilities": {
    "streaming": true
  },
  "skills": [
    {
      "id": "security-review",
      "name": "Revisión de seguridad"
    }
  ]
}
```

### Secuencia de comunicación

`obtener tarjeta → verificar → enviar mensaje → recibir estado/resultado`

1. El cliente obtiene la Agent Card desde una fuente de descubrimiento conocida.
2. Verifica el origen y dominio, la conexión HTTPS, los requisitos de autenticación y que la tarjeta anuncie la capacidad o habilidad necesaria.
3. Envía el mensaje de tarea al endpoint anunciado, incluyendo solo el contexto necesario.
4. Recibe una respuesta directa o consulta/escucha actualizaciones de la tarea hasta obtener su estado y resultado, que puede venir en un mensaje o artefactos.

Un recorrido típico de tarea es `submitted → working → completed`. Si falla durante el procesamiento, puede terminar en `failed`; es un estado terminal con error. El cliente puede consultar el estado periódicamente o usar streaming si la capacidad está anunciada.

### Diferencia frente al subagente local y MCP

- **Subagente local:** Codex lo delega dentro de la sesión y del entorno local. Recibe la tarea y el contexto que le proporcione el orquestador, y hereda el sandbox y los permisos de la sesión padre. A2A comunica dos sistemas de agentes mediante un endpoint y una tarjeta de capacidades. El agente remoto conserva sus propios detalles internos; el cliente interactúa con su interfaz publicada y debe comprobar la identidad del servicio antes de enviar datos.
- **MCP:** su foco es conectar un host de IA con servidores que ofrecen herramientas, recursos y prompts. A2A se enfoca en que un agente solicite trabajo a otro y reciba mensajes, estados de tarea y resultados. La diferencia principal es la función de la interacción, no simplemente si el proceso es local o remoto.

Referencias: [conceptos clave de A2A](https://a2a-protocol.org/latest/topics/key-concepts/) · [arquitectura de MCP](https://modelcontextprotocol.io/docs/learn/architecture) · [subagentes de Codex](https://learn.chatgpt.com/docs/agent-configuration/subagents).
