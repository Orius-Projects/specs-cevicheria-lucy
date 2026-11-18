# Checklist de calidad de requisitos — Plataforma base v2

> Evalúa la calidad de la spec. `[x]` = criterio verificado en el documento.

## Completitud

- [x] Todas las historias tienen actor, acción, beneficio y prioridad.
- [x] Cada historia tiene una prueba independiente.
- [x] Cada historia tiene escenarios `DADO/CUANDO/ENTONCES`.
- [x] Los casos borde están identificados (EC-001 a EC-014).
- [x] Las preguntas abiertas están registradas (Q-001-01 a Q-001-07).
- [x] Las entidades y las dos máquinas de estado están definidas.

## Claridad y testabilidad

- [x] Cada requisito funcional usa EARS y es verificable.
- [x] No hay adjetivos ambiguos sin métrica.
- [x] Los criterios de éxito son medibles (SC-001 a SC-008).
- [x] La spec no impone una solución técnica.
- [x] Los términos del dominio están definidos o referenciados.

## Consistencia

- [x] Los estados coinciden con el PDF (pp. 12-13).
- [x] Los roles coinciden con el PDF (p. 6).
- [x] No hay requisitos duplicados o contradictorios.
- [x] Las dependencias con `002`–`007` están declaradas.
- [x] No hay campos de pago, IGV ni facturación.

## Alcance

- [x] El feature tiene un límite claro.
- [x] Lo que queda fuera está explícito.
- [x] El alcance es realizable en la fase MVP.
- [ ] La apertura manual del turno está confirmada (Q-001-01).

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

**Conclusión:** spec lista para revisión. El único punto pendiente es confirmar la apertura manual del turno.
