# Estimación de Tareas – Proyecto Integración SAP / Dynamics CRM

## Resumen Ejecutivo

| Bloque | Horas Estimadas |
|--------|-----------------|
| 1. Capa de Persistencia (Azure SQL) | 9 h |
| 2. Capa de Orquestación (WS) | 24 h |
| 3. Procesamiento Diferido (WebJob) | 26 h |
| 4. Adaptación en CRM y QA | 18 h |
| **TOTAL** | **77 h** |

---

## Detalle por Bloque

### Bloque 1 – Capa de Persistencia (Azure SQL)

| ID | Historia de Usuario (HU) | Descripción Técnica | Estimación (h) |
|----|--------------------------|---------------------|---------------|
| HU 1.1 | Creación de Tabla SQL en Azure | Ejecutar el script para la tabla `SapIntegrationQueue`. | 1 |
| HU 1.2 | Setup de Conexión SQL y Seguridad | Configurar permisos de Firewall en Azure SQL, Managed Identity/Key Vault para el WS y WebJob. | 4 |
| HU 1.3 | Integración T-SQL en WS | Mapear e integrar la consulta transaccional (evaluación síncrona/asíncrona) proporcionada en el Anexo en el acceso a datos del WS. | 2 |
| HU 1.4 | Integración T-SQL en WebJob | Mapear e integrar la consulta CTE proporcionada en el Anexo en el Worker Service. | 2 |
| | **Subtotal Bloque 1** | | **9** |

---

### Bloque 2 – Capa de Orquestación (WS)

| ID | Historia de Usuario (HU) | Descripción Técnica | Estimación (h) |
|----|--------------------------|---------------------|---------------|
| HU 2.1 | Enrutador Síncrono/Asíncrono | Refactorizar el endpoint principal. Conectar con SQL para decidir si llamar a SAP o insertar como 'Pendiente'. | 16 |
| HU 2.2 | Nuevo Endpoint de Callback | Crear endpoint interno en el WS que reciba la respuesta de SAP desde el WebJob y finalice actualización en Dynamics. | 8 |
| | **Subtotal Bloque 2** | | **24** |

---

### Bloque 3 – Procesamiento Diferido (WebJob)

| ID | Historia de Usuario (HU) | Descripción Técnica | Estimación (h) |
|----|--------------------------|---------------------|---------------|
| HU 3.1 | Setup y Configuración Base | Creación del proyecto .NET Worker Service, configuración del Timer Trigger (20 s) y despliegue inicial en Azure. | 6 |
| HU 3.2 | Motor de Procesamiento (Worker) | Implementar bucle principal: recuperar ID pendiente, marcar 'En Curso' sincrónicamente, instanciar `HttpClient` hacia SAP. | 12 |
| HU 3.3 | Manejo de Respuestas y Errores | Lógica de parseo tras llamar a SAP: actualizar a 'Ejecutada' o 'Error' en SQL, y lanzar petición al Callback del WS. | 8 |
| | **Subtotal Bloque 3** | | **26** |

---

### Bloque 4 – Adaptación en CRM y QA

| ID | Historia de Usuario (HU) | Descripción Técnica | Estimación (h) |
|----|--------------------------|---------------------|---------------|
| HU 4.1 | Ajuste Plugin/Action Dynamics | Modificar el código en Dynamics para interpretar la nueva respuesta HTTP 202 (Accepted) y mostrar mensaje de 'Actualización encolada'. | 8 |
| HU 4.2 | Pruebas E2E y Pruebas de Estrés | Simulación de condiciones de carrera, validación del Throttling y revisión de logs. | 10 |
| | **Subtotal Bloque 4** | | **18** |

---

## Análisis de la Estimación

### Distribución del esfuerzo

```
Bloque 1 – Persistencia   :  9 h  ( 11.7 %)
Bloque 2 – Orquestación   : 24 h  ( 31.2 %)
Bloque 3 – WebJob         : 26 h  ( 33.8 %)
Bloque 4 – CRM y QA       : 18 h  ( 23.4 %)
────────────────────────────────────────────
TOTAL                      : 77 h  (100.0 %)
```

### Observaciones y riesgos

1. **HU 2.1 (16 h) y HU 3.2 (12 h)** concentran el 36 % del total del proyecto. Son las historias de mayor riesgo técnico:
   - HU 2.1 requiere refactorizar un endpoint productivo y añadir lógica de decisión síncrona/asíncrona; cualquier error puede afectar el flujo existente.
   - HU 3.2 implementa el bucle central del WebJob con control de concurrencia; una estimación ajustada puede ser insuficiente si aparecen condiciones de carrera.

2. **HU 1.2 (4 h – Seguridad)** puede extenderse si el proceso de aprobación de Managed Identity o Key Vault en el entorno de producción requiere coordinación con el equipo de infraestructura/seguridad.

3. **HU 4.2 (10 h – Pruebas E2E y Estrés)** depende de que todos los bloques anteriores estén finalizados y estables. Si existen retrasos en bloques previos, el tiempo de QA podría incrementarse.

4. La estimación total de **77 horas** no incluye:
   - Gestión/coordinación del proyecto.
   - Revisiones de código (code reviews).
   - Despliegues en entornos intermedios (DEV → QA → PRD).
   - Documentación técnica adicional.

### Recomendación

Considerar agregar un **colchón de contingencia del 15–20 %** (~12–15 h adicionales) para absorber imprevistos en las historias de mayor complejidad (HU 2.1, HU 3.2 y HU 3.3), llevando la estimación total a un rango de **89–92 horas**.
