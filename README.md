# Controlador TEC para ECDL

Controlador analógico de temperatura de lazo cerrado con elemento termoeléctrico (TEC / celda Peltier) para estabilizar un láser de diodo de cavidad externa con retroalimentación óptica (ECDL).

Proyecto desarrollado en el Laboratorio de Física Atómica y Molecular (LAMP), Departamento de Física, Universidad de Concepción (curso *Diseño de Prototipos I*, bajo la dirección del Dr. Pablo Solano).

## Arquitectura del circuito

```text
LM399 (6,95 V) ─► Puente de Wheatstone (NTC 10 kΩ) ─► INA114 (G ≈ 11,6) ─► Controlador PI (OP27) ─► AD711 + IRF530/IRF5305 ─► R_lim (33 Ω) + Peltier (Marlow RC 3-6)
