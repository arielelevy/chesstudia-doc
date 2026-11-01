# Arquitectura y decisiones técnicas

chesstudia combina una aplicación web, motores de ajedrez y recuperación de contenido
para construir un recorrido de estudio basado en las partidas del alumno.

[Abrir la arquitectura visual interactiva](https://claude.ai/artifact/PGzA5S1bka7vsre6u5av8M)
· [Volver a la presentación](../README.md)

## Límites entre componentes

| Componente | Contrato principal | Decisión |
|---|---|---|
| Frontend React | REST y WebSocket con FastAPI | La interfaz consume los mismos casos de uso que respaldan las herramientas del tutor. |
| API | Validación de entradas, sesión y contexto del alumno | El servidor resuelve posiciones y fija presupuestos. |
| Stockfish | Protocolo UCI en procesos separados | Las evaluaciones proceden del motor; el esfuerzo se limita por nodos. |
| Servicio de motores | API propia para la práctica contra Maia | Separar el rival de práctica del resto de la aplicación. |
| PostgreSQL | Estado transaccional y contenido persistido | Es la fuente de verdad y conserva también los vectores de los fragmentos. |
| Weaviate | Índice de recuperación híbrida | Es derivado y reconstruible; no es dueño de los datos. |
| Proveedor de IA | Llamadas mediante un adaptador y herramientas acotadas | Mantener validación, presupuestos y registro del lado del servidor. |

## De una partida a un plan

La partida se parsea como PGN. El servidor resuelve sus posiciones y solicita análisis
al motor. Las evaluaciones alimentan la clasificación de errores y la identificación
de aspectos para estudiar.

El plan conecta esos aspectos con lecturas y práctica. Las posiciones trabajadas se
incorporan al repaso con FSRS. La actividad y el estado pertenecen al alumno identificado
por la sesión.

## Recuperación de información

El contenido de ajedrez necesita búsquedas textuales y estructurales. Una descripción
de un peón aislado puede aparecer en una posición de otra apertura y con otras piezas.
La recuperación combina:

- Weaviate para búsqueda híbrida BM25 y vectorial.
- PostgreSQL para búsqueda textual en español con `tsvector`.
- Un carril posicional para coincidencias de posición, peones, material y motivos.

Los resultados se fusionan con Reciprocal Rank Fusion. Cada candidato conserva sus
fuentes; una respuesta puede referenciar el libro y la página del fragmento recuperado.
Si un carril no está disponible, la respuesta informa esa condición.

## Tutor con herramientas y validación

El tutor solicita análisis y recuperación mediante herramientas expuestas por la API.
Las posiciones se identifican por referencias de la sesión; el servidor las resuelve
antes de consultar el motor.

La respuesta pasa por un validador que relaciona las afirmaciones marcadas con las
evaluaciones registradas y las citas con la recuperación del turno. Una violación del
contrato bloquea la publicación de ese bloque.

Esto ofrece controles reproducibles sobre evaluación y procedencia. No demuestra que
el modelo haya interpretado correctamente toda una explicación del autor.

## Imágenes e ingesta

La lectura de diagramas usa procesamiento de imagen y modelos ONNX. Una posición
extraída requiere validaciones antes de usarse en el análisis. Una lectura con baja
confianza necesita revisión humana.

El pipeline de ingesta prepara fragmentos, posiciones, referencias y vectores. El OCR
corre localmente. Las secuencias de jugadas se verifican con las reglas de ajedrez antes
de incorporarlas al índice. Una línea legal todavía puede ser una transcripción
equivocada; por eso la evaluación de ingesta también requiere contraste con el original.

## Datos, identidad y costo

PostgreSQL conserva los datos del alumno y el contenido. Las consultas usan el contexto
de sesión para determinar a qué alumno pertenecen; los flujos HTTP y WebSocket requieren
controles de identidad coherentes. El producto incorpora acceso con Google OAuth.

Los cambios del esquema se gestionan con Alembic. El índice vectorial se reconstruye
desde PostgreSQL y la sincronización del corpus preserva las tablas del alumno.

Las llamadas del tutor registran consumo, modelo y costo por alumno y turno. El
presupuesto del motor y los topes del proveedor se aplican desde el servidor.

## Entrega en Azure

| Pieza | Servicio |
|---|---|
| API y análisis | Azure Container Apps |
| Rival de práctica | Servicio de motores en Azure Container Apps |
| Frontend | Azure Static Web Apps |
| Estado | Azure Database for PostgreSQL Flexible Server |
| Archivos | Azure Blob Storage |
| Imágenes de contenedores | Azure Container Registry |
| Modelo del tutor | Microsoft Foundry |
| Automatización | GitHub Actions con OIDC; infraestructura declarada parcialmente con Terraform |

La infraestructura separa la preparación del corpus de su consumo por la aplicación.
Los despliegues se seleccionan por los componentes modificados. La infraestructura,
las migraciones y la aplicación tienen pasos de operación documentados.

## Verificación

El proyecto incluye pruebas de backend, frontend y recorridos de navegador. Las pruebas
se seleccionan por el alcance del cambio; la evaluación del tutor comprueba su contrato
con el motor y las fuentes. Las trazas de HTTP, WebSocket y búsquedas del motor permiten
investigar tiempos y procedencia de resultados.

Este documento describe el diseño revisado contra el repositorio. No certifica el
estado de una instalación en vivo ni atribuye resultados nuevos a las suites existentes.
