<img src="docs/banner.svg" alt="Asistente inteligente educativo con IA: proyecto de Ana Cuevas en Fundación Televisa para el programa de Cuantrix. Por WhatsApp conversa sin menús, revisa fotos de evidencia, registra grupos y acompaña el avance. Código privado." width="100%">

<p align="center">
  <img src="https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n">
  <img src="https://img.shields.io/badge/Google_Gemini-8E75B2?style=for-the-badge&logo=googlegemini&logoColor=white" alt="Google Gemini">
  <img src="https://img.shields.io/badge/AI_Agents-1F1B3A?style=for-the-badge" alt="AI agents">
  <img src="https://img.shields.io/badge/Computer_Vision-1F1B3A?style=for-the-badge" alt="Computer vision">
  <img src="https://img.shields.io/badge/WhatsApp_API-25D366?style=for-the-badge&logo=whatsapp&logoColor=white" alt="WhatsApp API">
  <img src="https://img.shields.io/badge/ManyChat-0084FF?style=for-the-badge" alt="ManyChat">
  <img src="https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL">
  <img src="https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white" alt="Google Cloud">
  <img src="https://img.shields.io/badge/AppSheet-4285F4?style=for-the-badge&logo=google&logoColor=white" alt="AppSheet">
</p>

# Asistente educativo con IA

Un asistente por **WhatsApp** que acompaña a docentes de primaria, secundaria y bachillerato que enseñan programación en sus salones, dentro de un programa de **Fundación Televisa**. El docente le escribe como le escribiría a una persona: da de alta sus grupos, manda las fotos de las actividades que hizo con sus alumnos y pregunta cómo va. El asistente entiende lo que necesita, revisa las fotos con visión por computadora, guarda todo en la base de datos y le contesta en segundos.

Lo diseñé y lo construyo yo, de punta a punta, como parte de mi trabajo de **Data & AI Analyst**.

> 🔒 El código, los prompts y los datos son privados. Aquí solo describo qué hace y cómo está pensado. Todos los ejemplos usan datos ficticios.

<p align="center">
  <img src="docs/conversacion.svg" alt="Ejemplo animado: un docente manda una foto, el asistente pregunta la lección; el docente dice 'otra actividad' y el asistente, sin adivinar, pregunta si es una lección del manual o una actividad aparte. Con la respuesta, revisa la foto, la registra y le dice que lleva 5 de 20. A la derecha se ve lo que pasa por dentro." width="100%">
</p>

## Qué hace

| | |
|---|---|
| 💬 **Conversa en lenguaje natural** | Sin menús, opciones numeradas ni palabras clave. Entiende mensajes informales, con errores o varios mensajes seguidos. |
| 🏫 **Registra escuelas y grupos** | Guía al docente para dar de alta sus grupos, valida los datos y evita duplicados. |
| 📸 **Revisa fotos de evidencia** | Un modelo de visión confirma que la foto es una actividad escolar real y la compara con el criterio de la lección. Si cumple la valida; si tiene dudas, la manda a revisión humana. |
| 📈 **Le cuenta su avance** | Cuántas lecciones lleva en cada grupo y qué le falta para su siguiente meta. |
| 🔔 **Le escribe primero** | Mensajes proactivos según el calendario del ciclo escolar, distintos según el avance de cada docente, sin repetirse. |
| 🧑‍🏫 **Trabaja con el equipo humano** | Lo que el asistente no puede decidir solo pasa a los facilitadores del programa, que lo revisan en una app interna. |

## Cómo fluye un mensaje

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#1f1b3a','primaryTextColor':'#edeafb','primaryBorderColor':'#ff9a8b','lineColor':'#7fe0c9','secondaryColor':'#15122a','tertiaryColor':'#15122a','fontFamily':'Segoe UI, Helvetica, Arial'}}}%%
flowchart LR
    D["Docente<br/>WhatsApp"] --> W["Canal de mensajería"]
    W --> O["Orquestador<br/>n8n"]
    O --> C{"¿Qué necesita?"}
    C -->|"registrar grupo"| R["Agente de registro"]
    C -->|"foto de evidencia"| F["Agente de evidencias"]
    C -->|"pregunta o avance"| P["Agente de conversación"]
    F --> V["Visión<br/>¿es evidencia? ¿cumple?"]
    R --> G["Reglas y validaciones<br/>en código"]
    F --> G
    P --> B[("Base de datos<br/>MySQL")]
    G --> B
    B --> A["Respuesta"]
    A --> D
    B -.-> E["Equipo de facilitadores<br/>revisa lo pendiente"]
```

## Cómo está construido

- **Orquestación en n8n.** Un flujo de punta a punta recibe cada mensaje, recupera el contexto de la conversación, decide qué necesita el docente y lo manda al agente indicado.
- **Agentes especializados con LLM.** Cada uno tiene un trabajo acotado: registrar, recolectar los datos de una foto o conversar y consultar el avance. Usan herramientas para leer la base de datos en lugar de suponer.
- **Visión por computadora.** Revisa cada foto en dos pasos: primero si es evidencia escolar y después si cumple el criterio de la lección que dijo el docente.
- **IA + código.** El modelo conversa y entiende, pero lo que se guarda pasa por validaciones deterministas: catálogo cerrado de lecciones, detección de duplicados y que el grupo exista entre los del docente.
- **Mensajes proactivos.** Un flujo diario calcula qué mensaje le toca a cada docente según el calendario y su avance, elige uno solo por persona y registra lo enviado para no repetir.
- **Medición.** Un tablero de métricas para el equipo: uso, lecciones registradas, cómo llegan las evidencias y los errores por tipo, con revisión de sus causas.

## Decisiones de diseño (lo que he aprendido)

1. **Conversación, no menús.** Un docente con 40 alumnos no va a aprender comandos. El asistente se adapta a cómo escribe la gente, no al revés.
2. **El modelo nunca adivina un dato que importa.** El número de lección siempre lo dice el docente. Si su respuesta es ambigua ("otra actividad"), el asistente pregunta una vez y de forma natural, en lugar de suponer.
3. **Lo que se guarda lo decide el código.** El modelo propone y el código valida. Así un error de redacción del modelo nunca termina como un dato incorrecto en la base.
4. **Detectar cuando la plática se atora.** Si el asistente está por repetir la misma pregunta, cambia de estrategia y explica qué le falta, en vez de dar vueltas.
5. **Humanos donde hace falta.** La IA valida lo que puede validar con seguridad y el resto pasa a revisión de una persona. Mejor eso que aprobar todo o rechazar todo.
6. **Medir para mejorar.** Cada error se clasifica y se revisa su causa; muchas mejoras del asistente salieron de leer esas conversaciones.

## Mi rol

Diseño de la conversación y de los prompts, arquitectura de los flujos en n8n, integración con WhatsApp, visión por computadora, modelo de datos en MySQL, mensajes proactivos, tablero de métricas y mejora continua a partir de las conversaciones reales.

## Sobre este repositorio

Este repositorio es una **vitrina del proyecto**. El código, los prompts, las credenciales y los datos de docentes son privados y no se publican. Las imágenes y conversaciones de este README son ilustrativas y usan datos inventados.

<a href="https://github.com/AnnaCUGA">
  <img src="docs/footer.svg" alt="Diseñado y construido por Ana Cuevas, Data and AI Analyst en Fundación Televisa. Ver mi perfil." width="100%">
</a>
