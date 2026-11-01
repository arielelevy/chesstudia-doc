<p align="center">
  <img src="assets/chesstudia.png" width="120" height="120" alt="Logo de chesstudia: caballo de ajedrez en negro y dorado">
</p>

<h1 align="center">chesstudia</h1>
<p align="center">Tus partidas, tu plan de estudio.</p>

<p align="center">
  <a href="LICENSE"><img alt="Licencia propietaria" src="https://img.shields.io/badge/Licencia-Propietaria-64748b"></a>
  <img alt="Python" src="https://img.shields.io/badge/Python-3776AB?logo=python&amp;logoColor=white">
  <img alt="FastAPI" src="https://img.shields.io/badge/FastAPI-009688?logo=fastapi&amp;logoColor=white">
  <img alt="React y TypeScript" src="https://img.shields.io/badge/React-TypeScript-3178C6?logo=react&amp;logoColor=white">
  <img alt="PostgreSQL" src="https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&amp;logoColor=white">
  <img alt="Microsoft Azure" src="https://img.shields.io/badge/Cloud-Microsoft_Azure-0078D4">
</p>

chesstudia es una plataforma de estudio de ajedrez que conecta el análisis de tus partidas
con un plan de aprendizaje. Combina Stockfish, recuperación de libros, un tutor con IA,
práctica contra Maia y repasos programados con FSRS.

Proyecto independiente de [Ariel Levy](https://github.com/arielelevy), con responsabilidad
sobre diseño de producto, arquitectura, implementación de frontend y backend y despliegue
en Azure. El desarrollo utiliza agentes de IA y verificación de sus resultados.

[Arquitectura visual interactiva](https://claude.ai/artifact/PGzA5S1bka7vsre6u5av8M) ·
[Arquitectura y decisiones técnicas](docs/arquitectura.md) ·
[Recorrido de estudio](docs/como-estudiar.md) ·
[Contacto profesional](https://www.linkedin.com/in/arieledgardo/)

## El problema que resuelve

Un motor muestra dónde se perdió una ventaja. Estudiar requiere conectar ese error con
un concepto, encontrar una explicación y volver a practicarlo. chesstudia organiza ese
recorrido a partir de las propias partidas del alumno.

1. Importás un PGN y recorrés las evaluaciones y variantes de Stockfish.
2. El plan reúne temas y lecturas según tus debilidades y tu nivel.
3. El tutor acompaña la explicación con herramientas de análisis y referencias de libro y página.
4. Practicás posiciones o jugás contra Maia.
5. FSRS programa el repaso de las posiciones trabajadas.

Mi estudio reúne Tu nivel, Estudiar, Resolver, Jugar, Recordar y Consolidar.
También se pueden enviar imágenes de tableros para su lectura y validación.

## Arquitectura del producto

La interfaz React se comunica con una API FastAPI por REST y WebSocket. La API coordina
el análisis, la recuperación de contenido, el tutor y el progreso del alumno.
PostgreSQL conserva el estado; Weaviate es un índice derivado que puede reconstruirse.

| Capa | Implementación | Responsabilidad |
|---|---|---|
| Interfaz | React, TypeScript, Vite, Chessground | Tablero interactivo, análisis, estudio y conversación con el tutor. |
| API | Python, FastAPI, Pydantic, WebSockets | Casos de uso, validación, sesiones y coordinación de herramientas. |
| Motor | Stockfish por UCI, `python-chess` | Evaluación de posiciones, variantes y reglas de ajedrez. |
| Tutor | LLM por Microsoft Foundry y herramientas del servidor | Explicaciones ligadas al análisis y a las fuentes recuperadas. |
| Recuperación | Weaviate, búsqueda de texto en PostgreSQL e índice posicional | Combinar parecido semántico, términos y estructura del tablero. |
| Datos | PostgreSQL y migraciones Alembic | Partidas, alumnos, contenido, progreso y registros de gasto. |
| Visión e ingesta | OpenCV, ONNX Runtime, OCR local | Leer diagramas, validar posiciones y preparar contenido recuperable. |
| Práctica | Servicio de motores Maia y planificación FSRS | Rival de entrenamiento y repasos. |
| Nube | Azure Container Apps, Static Web Apps, Blob Storage, GitHub Actions | Ejecutar la aplicación, servir el frontend y desplegar sus componentes. |

## Decisiones que definen el diseño

### La evaluación tiene una fuente verificable

Stockfish decide la evaluación de una posición. El tutor accede al motor mediante
herramientas del servidor; sus referencias se resuelven contra el estado de la sesión.
Un validador contrasta las afirmaciones evaluativas marcadas con las llamadas al motor
y comprueba las citas antes de publicar la respuesta.

El validador comprueba reglas concretas. No garantiza que toda interpretación del texto
sea correcta; la evaluación del motor y la procedencia de las citas siguen visibles.

### El RAG incorpora la posición de ajedrez

Una misma idea puede aparecer con palabras distintas. La recuperación combina tres
carriles: búsqueda híbrida BM25/vectorial, búsqueda textual en español y búsqueda por
posición, estructura de peones, material o motivos. Los resultados se fusionan por
rango recíproco y conservan la referencia de libro y página.

### El estado y los índices tienen dueños distintos

PostgreSQL es la fuente de verdad. Weaviate se reconstruye a partir de los datos y vectores
persistidos. La producción del corpus se separa del estado del alumno; sincronizar contenido
debe preservar partidas, sesiones y progreso.

### El servidor controla los recursos

El presupuesto de Stockfish se expresa en nodos. La API fija los límites de las herramientas;
el modelo no puede ampliarlos. Las llamadas de IA registran consumo y costo por alumno y turno,
y el tutor requiere tarifas configuradas para aplicar un límite de gasto.

## Ingeniería y operación

El desarrollo local corre en Linux sobre WSL2. En Azure, la API y el servicio de motores
se despliegan en Container Apps; el frontend se sirve desde Static Web Apps. GitHub Actions
construye y despliega los componentes afectados, con autenticación federada OIDC.

El proyecto incluye pruebas de backend con pytest, pruebas de interfaz con Vitest y recorridos
de navegador. Las migraciones, el aislamiento por alumno, los errores del proveedor y el
comportamiento bajo carga forman parte de los controles del producto. Esta presentación
no publica un resultado nuevo de esas suites.

El trabajo reúne arquitectura de software, ingeniería de IA, recuperación de información,
modelado de datos y entrega de una aplicación web. Las decisiones se documentan junto con
sus restricciones y formas de verificación.

## Acceso y documentación

- [Arquitectura visual interactiva](https://claude.ai/artifact/PGzA5S1bka7vsre6u5av8M)
- [Arquitectura y decisiones técnicas](docs/arquitectura.md)
- [Cómo estudiar con chesstudia](docs/como-estudiar.md)
- [Ariel Levy en LinkedIn](https://www.linkedin.com/in/arieledgardo/), para consultas profesionales o una demostración.

Este repositorio contiene documentación pública. No incluye el código de la aplicación,
el corpus de libros, datos de alumnos ni credenciales.

## Licencia

Software, documentación y recursos visuales propios: licencia propietaria, todos los derechos
reservados. Consultá [LICENSE](LICENSE). Los componentes y materiales de terceros conservan
sus licencias.
