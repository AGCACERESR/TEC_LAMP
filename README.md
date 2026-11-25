# Controlador TEC para ECDL

Controlador de temperatura con celda Peltier para un láser de diodo de cavidad externa (ECDL).
Proyecto del curso *Diseño de Prototipos I*, Universidad de Concepción.
## Diseño

NTC 10 kΩ en puente de Wheatstone → INA114 → control PI (OP27) → AD711 + IRF530/IRF5305 (push-pull) → Peltier.

* Referencia: LM399 (6,95 V). Alimentación: ±15 V (7815/7915).
* Set point: 25 °C con el puente en 10 kΩ.
* PI: Kp = 10, Ki = 1000 s⁻¹.

## Estado

Validado en protoboard: la cadena INA114 → PI → MOSFET → Peltier calienta y enfría con el signo correcto.

Pendiente: limitación de corriente (resistencia en serie con el Peltier), disipación de los MOSFET y sintonía del PI.

##

