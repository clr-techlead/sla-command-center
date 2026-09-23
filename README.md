# Case Management System - Proyecto de Portafolio

Dashboard ejecutivo en Power BI Service para un sistema sintetico de gestion de casos de soporte (tickets), con modelado dimensional, medidas DAX y una vista operativa de backlog/riesgo de SLA. Proyecto de portafolio (no un sistema en produccion), en la misma linea que [Contoso Retail SQL & BI](https://github.com/clr-techlead/contoso-retail-sql-bi).

## Objetivo

Simular el sistema de tickets de un equipo de soporte y responder tres preguntas de negocio:

- Cuantos casos hay abiertos y cerrados?
- Se esta cumpliendo el SLA acordado por prioridad?
- Como esta distribuida la carga de trabajo entre agentes?

## Arquitectura de datos

Modelo semantico en esquema estrella con 8 tablas: 2 de hechos y 6 dimensiones.

| Tabla | Tipo | Contenido |
|---|---|---|
| fact_caso | Hecho | Un registro por caso: fechas de creacion/cierre, horas transcurridas, prioridad, categoria, estado, agente asignado |
| fact_comentario | Hecho | Interacciones/comentarios asociados a cada caso |
| fact_historial_estado | Hecho | Historial de cambios de estado por caso |
| dim_estado | Dimension | Estados del caso (nuevo, en progreso, cerrado, etc.) con bandera de estado final |
| dim_prioridad | Dimension | Niveles de prioridad y su SLA en horas |
| dim_categoria | Dimension | Categorias de los casos |
| dim_usuario | Dimension | Agentes y usuarios del sistema |
| dim_adjunto | Dimension | Adjuntos vinculados a los casos |

### Diagrama entidad-relacion

```
dim_estado ----\
dim_prioridad ---\
dim_categoria -----\
dim_usuario ---------> fact_caso <----- fact_historial_estado
dim_adjunto ----/                \----- fact_comentario
```

fact_caso se relaciona 1 a N con cada dimension por clave subrogada (estado_id, prioridad_id, categoria_id, usuario_asignado_id). Todas las relaciones son de direccion unica, sin relaciones circulares.

## Medidas DAX clave

```dax
Total Casos = COUNTROWS(fact_caso)

Casos Activos = COUNTROWS(
    FILTER(fact_caso, RELATED(dim_estado[es_estado_final]) = "f")
)

Casos Cerrados = COUNTROWS(
    FILTER(fact_caso, RELATED(dim_estado[es_estado_final]) = "t")
)
```

Sobre esa misma base se calculan % Cumplimiento SLA (casos cerrados dentro del tiempo definido por dim_prioridad[sla_horas]) y % SLA Vencido.

## Dashboard

**Resumen Ejecutivo** (Pagina 1): 4 tarjetas KPI (Total Casos, Casos Activos, Casos Cerrados, % Cumplimiento SLA con semaforo rojo/ambar/verde), tendencia de casos por ano, casos por categoria y carga de trabajo por agente. Tema oscuro ejecutivo.

![Dashboard - Resumen Ejecutivo](dashboard-resumen-ejecutivo.png)

**Backlog SLA** (Pagina 2): tabla operativa filtrada a casos activos (estado no final), con estado, prioridad, agente asignado y horas transcurridas - para que un supervisor priorice los casos mas antiguos.

![Dashboard - Backlog SLA](backlog-sla.png)

## Retos tecnicos resueltos

| Problema | Diagnostico | Solucion |
|---|---|---|
| Error al crear el reporte ("Sorry, we couldn't find that semantic model") | Bug conocido del boton "New report" en Power BI Service | Ruta alterna: My workspace > + New item > Report > Pick a published semantic model |
| Comparacion de tipos incompatibles en DAX (Text vs True/False) | es_estado_final importada como texto en vez de booleano | Consulta DAX exploratoria (EVALUATE DISTINCT) + medidas reescritas comparando contra texto ("f"/"t") |
| Error de relacion "needs to be recalculated" tras editar una medida | Estado transitorio del modelo tras un cambio de esquema via TMDL | Refresh manual del semantic model (no basta el refresh a nivel de reporte) |

## Roadmap

Migrar el origen de datos a Azure SQL con captura de casos via Power Apps, habilitando refresco incremental y captura en tiempo real. Pendiente de resolver el acceso a una suscripcion Azure propia (ver notas del proyecto).

## Stack

Power BI Service (TMDL, DAX, Power Query), modelado dimensional, SQL/logica relacional.

## Documento completo del proyecto

Escritura extendida (resumen ejecutivo, contexto, arquitectura, medidas, retos, habilidades): ver el documento de portafolio.

---

Camilo Andres Leon Rubriche - linkedin.com/in/caleru
