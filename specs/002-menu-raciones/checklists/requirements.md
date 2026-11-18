# Checklist de calidad de requisitos — Menú, raciones y agotados

> Evalúa la calidad de la spec. `[x]` = criterio verificado en el documento.

## Completitud

- [x] Todas las historias tienen actor, acción, beneficio y prioridad.
- [x] Cada historia tiene una prueba independiente.
- [x] Cada historia tiene escenarios `DADO/CUANDO/ENTONCES`.
- [x] Los casos borde están identificados (EC-001 a EC-015).
- [x] Las preguntas abiertas están registradas (Q-002-01 a Q-002-06).
- [x] Los estados del plato están definidos.

## Claridad y testabilidad

- [x] Cada requisito funcional usa EARS y es verificable.
- [x] No hay adjetivos ambiguos sin métrica.
- [x] Los criterios de éxito son medibles (SC-001 a SC-007).
- [x] La spec no impone una solución técnica.
- [x] Los términos del dominio están definidos.

## Consistencia

- [x] Los estados coinciden con el PDF (p. 13).
- [x] Las reglas de raciones coinciden con RN-02/RN-03/RN-04/RN-05/RN-11.
- [x] No hay requisitos duplicados o contradictorios.
- [x] Las dependencias con `001`, `003` y `005` están declaradas.
- [x] No hay costos, IGV ni pagos.

## Alcance

- [x] El feature tiene un límite claro.
- [x] Lo que queda fuera está explícito.
- [x] El alcance es realizable en la fase MVP.
- [ ] Las raciones por día o por turno están confirmadas (Q-002-01).

## Constitución

- [x] Cumple el Principio 1 (spec-first y trazabilidad).
- [x] Cumple el Principio 2 (un solo registro, tres vistas).
- [x] Cumple el Principio 3 (FIFO y una sola acción de cocina).
- [x] Cumple el Principio 4 (control de raciones y agotados).
- [x] Cumple el Principio 5 (trazabilidad y cierre de turno).
- [x] Cumple el Principio 6 (web responsive y alcance acotado).

## Resultado

| Estado | Cantidad |
|---|---|
| Verificados | 25 |
| Pendientes | 1 |
| No aplica | 0 |

**Conclusión:** spec lista para revisión. El punto pendiente es confirmar si las raciones son por día o por turno.
