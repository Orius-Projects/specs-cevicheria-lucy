# Checklist de calidad de requisitos — Avisos en tiempo real

> Evalúa la calidad de la spec. `[x]` = criterio verificado en el documento.

## Completitud

- [x] Todas las historias tienen actor, acción, beneficio y prioridad.
- [x] Cada historia tiene una prueba independiente.
- [x] Cada historia tiene escenarios `DADO/CUANDO/ENTONCES`.
- [x] Los casos borde están identificados (EC-001 a EC-013).
- [x] Las preguntas abiertas están registradas (Q-005-01 a Q-005-05).
- [x] El contrato de eventos está definido y versionado.

## Claridad y testabilidad

- [x] Cada requisito funcional usa EARS y es verificable.
- [x] No hay adjetivos ambiguos sin métrica.
- [x] Los criterios de éxito son medibles (SC-001 a SC-007).
- [x] La spec no impone una tecnología en el “qué”; el contrato sí la detalla.
- [x] Los términos del dominio están definidos.

## Consistencia

- [x] Los estados coinciden con `001/data-model.md`.
- [x] El aviso “Listo” va a todos los mozos.
- [x] No hay requisitos duplicados o contradictorios.
- [x] Las dependencias con `003` y `004` están declaradas.
- [x] No hay eventos de tiempo, pago ni IGV.

## Alcance

- [x] El feature tiene un límite claro.
- [x] Lo que queda fuera está explícito.
- [x] El alcance es realizable en la fase MVP.
- [ ] El push futuro está confirmado como fuera del MVP (Q-005-01).

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

**Conclusión:** spec y contrato listos para revisión. El punto pendiente es la decisión futura sobre push.
