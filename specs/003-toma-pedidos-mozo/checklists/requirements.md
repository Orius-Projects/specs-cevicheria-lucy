# Checklist de calidad de requisitos — Toma de pedidos (Mozo)

> Evalúa la calidad de la spec. `[x]` = criterio verificado en el documento.

## Completitud

- [x] Todas las historias tienen actor, acción, beneficio y prioridad.
- [x] Cada historia tiene una prueba independiente.
- [x] Cada historia tiene escenarios `DADO/CUANDO/ENTONCES`.
- [x] Los casos borde están identificados (EC-001 a EC-017).
- [x] Las preguntas abiertas están registradas (Q-003-01 a Q-003-06).
- [x] Los estados y permisos están referenciados.

## Claridad y testabilidad

- [x] Cada requisito funcional usa EARS y es verificable.
- [x] No hay adjetivos ambiguos sin métrica.
- [x] Los criterios de éxito son medibles (SC-001 a SC-007).
- [x] La spec no impone una solución técnica.
- [x] Los términos del dominio están definidos.

## Consistencia

- [x] Los estados coinciden con `001/data-model.md`.
- [x] La regla de cancelación coincide con la decisión de negocio.
- [x] No hay requisitos duplicados o contradictorios.
- [x] Las dependencias con `001`, `002` y `005` están declaradas.
- [x] No hay pagos, IGV, tiempos ni asignación de mozo.

## Alcance

- [x] El feature tiene un límite claro.
- [x] Lo que queda fuera está explícito.
- [x] El alcance es realizable en la fase MVP.
- [ ] La cancelación por cualquier mozo está confirmada (Q-003-02).

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

**Conclusión:** spec lista para revisión. El punto pendiente es confirmar si cualquier mozo puede cancelar.
