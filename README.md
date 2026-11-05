# ppp-2026-08-acelerografo-esp32-kicad

> **Red Sísmica del Austro (RSA) — Universidad de Cuenca**  
> Prácticas Preprofesionales (PPP) • Diseño de Hardware Electrónico, CAD/EDA Libre y Estandarización PCBA (JLCPCB)

---

## 📋 Ficha Técnica del Proyecto

| Campo | Detalle |
| :--- | :--- |
| **Código Institucional** | `RSA-PPP-2026-08` |
| **Nombre del Proyecto** | Migración, Auditoría y Optimización de la PCB del Acelerógrafo ESP32 de Autodesk EAGLE a KiCad 10 con Estandarización PCBA |
| **Practicante** | Edgar Joel Suárez Jaigua (`edgar.suarez@ucuenca.edu.ec`) |
| **Carrera / Institución** | Ingeniería en Telecomunicaciones — Universidad de Cuenca |
| **Tutor Institucional** | Ing. Milton Muñoz (`milton.munozc@ucuenca.edu.ec`) — RSA |
| **Duración / Horas** | 96 horas (6 semanas) • Presencial |
| **Fecha de Ejecución** | 2026-07-15 al 2026-09-30 |
| **Estado** | `Culminado` |

> 📖 **Proyecto Precedente:** Este repositorio es la continuación técnica de la estandarización EDA institucional iniciada en [`RSA-PPP-2026-07` - ppp-2026-07-shm-altium-kicad](https://github.com/RSA-PPP/ppp-2026-07-shm-altium-kicad), ejecutada por el mismo practicante. Aplica la metodología institucional de librerías locales relativas (`${KIPRJMOD}`) y exportación automatizada con el plugin *Fabrication Toolkit*, integrando una auditoría rigurosa de pines de hardware frente al firmware en C++/FreeRTOS.

---

## 🎯 Descripción General y Objetivos

Este repositorio contiene el rediseño, auditoría, optimización física y estandarización para manufactura de la tarjeta de circuito impreso (PCB) del **Acelerógrafo Sísmico Basado en ESP32** de la Red Sísmica del Austro (RSA).

El proyecto migra la versión SMD original concebida en **Autodesk EAGLE** hacia el entorno de código abierto **KiCad 10**. El objetivo primordial es asegurar la compatibilidad absoluta entre las conexiones físicas de la PCB y las definiciones de GPIOs en el firmware de adquisición de datos en tiempo real (evitando desajustes en buses SPI, I2C, UART e interrupciones de 1 Hz), mitigando el ruido electromagnético mediante planos de masa sólidos y condensadores de desacoplo de ultra baja ESR, y estandarizando el diseño para ensamble automatizado (**PCBA**) en **JLCPCB**.

### Objetivos Clave
1. **[Importación y Auditoría de Pines]:** Importar esquemático y placa desde EAGLE a KiCad 10 y auditar exhaustivamente la asignación de pines (GPIOs) del ESP32 frente al firmware institucional de adquisición de datos.
2. **[Estandarización de Componentes y Asignación LCSC]:** Catalogar pasivos y activos a encapsulados SMD estándar (0805, 0603, SOT-23, SOIC) inyectando códigos de parte de *Basic Parts* de JLCPCB.
3. **[Gestión de Librerías Locales Relativas]:** Descargar símbolos, huellas y modelos 3D STEP mediante `jlc2kicadlib` en carpetas locales portables con direccionamiento relativo `${KIPRJMOD}`.
4. **[Optimización de Ruteo, Planos GND y Test Points]:** Rediseñar el layout con planos de tierra continuos (Top/Bottom GND), desacoplos capacitivos adyacentes a pines VCC, puntos de prueba accesibles y marcas fiduciarias.
5. **[Generación y Validación de Archivos PCBA]:** Generar mediante *Fabrication Toolkit* los paquetes de producción (Gerbers, NC Drill, `BOM.csv`, `CPL.csv`) y validar rotaciones en el visor web de JLCPCB.
6. **[Cierre y Documentación Técnica]:** Documentar la auditoría hardware-firmware, optimizaciones implementadas y estructurar la guía operativa.

> 📄 Para consultar el cronograma detallado semana a semana, los entregables y los checkpoints verificables, revisa el [Plan de Trabajo Oficial](docs/planificacion.md).

---

## 🔌 Hardware y Asignación de Pines Auditada (Hardware vs. Firmware)

El circuito impreso integra los siguientes periféricos clave, cuya correspondencia fue auditada pin a pin frente al firmware en C++/FreeRTOS:

* **SoC Principal:** Espressif ESP32-WROOM-32.
* **Acelerómetro Triaxial de Bajo Ruido:** Analog Devices ADXL355 (comunicación mediante bus SPI dedicado a 250 Hz: `MOSI`, `MISO`, `SCK`, $\overline{\text{CS}}$).
* **Almacenamiento Continuo:** Tarjeta MicroSD conectada por bus SPI (VSPI) mediante buffer de nivel 74LVC125A.
* **Sincronismo Temporal y RTC:** Maxim Integrated DS3231 por bus I2C (`SDA`, `SCL`) con línea de interrupción `SQW` calibrada a 1 Hz conectada a timer de hardware.
* **Geolocalización / NMEA:** Módulo GPS FGPMMOPA6H mediante interfaz UART (`TX`, `RX`).
* **Depuración e Interfaz USB:** Conversor USB-UART CH340G (`TXD0`, `RXD0`, señales de auto-reset `DTR`/`RTS`).

> 📋 Para detalles exhaustivos de componentes y valores pasivos, consulta [docs/especificacion_hardware_original.md](docs/especificacion_hardware_original.md).

---

## 📂 Estructura del Repositorio

La organización respeta fielmente la estructura desarrollada por el practicante, preservando las capturas de referencia del diseño previo en EAGLE y concentrando el nuevo diseño auditado en KiCad 10:

```text
ppp-2026-08-acelerografo-esp32-kicad/
│
├── docs/                                          # Documentación técnica de apoyo y gobernanza PPP
│   ├── especificacion_hardware_original.md        # Detalle de circuitos integrados, valores pasivos y pines
│   ├── Planificacion_Migracion_EAGLE_KiCad.pdf    # Plan de trabajo oficial previo en PDF (96 horas)
│   └── planificacion.md                           # Plan de trabajo oficial institucional, fases y checkpoints
│
├── eagle_Images/                                  # Material gráfico de referencia del diseño previo en EAGLE
│   ├── images_pcb_3d_view/                        # Vistas de renderizado 3D de la placa previa
│   └── images_schematic/                          # Capturas de los esquemas por bloques funcionales
│
├── kicad/                                         # Espacio de trabajo exclusivo para el diseño en KiCad 10
│   ├── libs/                                      # Bibliotecas locales portables (${KIPRJMOD})
│   │   ├── footprint/                             # Huellas locales auxiliares
│   │   ├── JLC2KiCad_lib/                         # Activos descargados con jlc2kicadlib
│   │   ├── LibExt.pretty/                         # Huellas externas y modelos 3D STEP
│   │   ├── RSA.pretty/                            # Huellas institucionales y paquetes 3D asociados
│   │   └── symbol/                                # Símbolos esquemáticos locales
│   │
│   └── projects/                                  # Proyecto KiCad (.kicad_pro, .kicad_sch, .kicad_pcb)
│       └── Acelerografo_ESP32/                    # Archivos de esquemático y layout de placa
│           └── production/                        # Paquete de fabricación (Gerber, Drill, BOM, CPL)
│
├── README.md                                      # Ficha técnica y guía operativa institucional
└── .gitignore                                     # Exclusiones de temporales, copias de seguridad y cachés
```

---

## 🛠️ Herramientas Utilizadas y Requisitos

| Herramienta | Versión / Tipo | Propósito |
| :--- | :--- | :--- |
| **KiCad** | 10.x | Software de diseño electrónico libre (Eeschema y PCB Editor) |
| **Python** | 3.10+ | Automatización y ejecución de scripts auxiliares |
| **JLC2KiCadLib** | CLI | Descarga automatizada de símbolos, huellas y modelos 3D STEP |
| **Fabrication Toolkit** | Plugin PCM | Generación automatizada de Gerbers, Drill, BOM.csv y CPL.csv |
| **JLCPCB PCBA** | Plataforma Industrial | Validación visual de rotación de componentes SMD y manufactura |
| **Git** | Control de versiones | Trazabilidad del repositorio con direccionamiento relativo |

---

## ⚙️ Metodología de Fabricación y Validación

El diseño fue optimizado para ensamble directo mediante el servicio **JLCPCB PCBA**:

1. **Inyección de Códigos LCSC:** Todos los componentes SMD poseen el campo `LCSC` completado con códigos de partes básicas (*Basic Parts*), minimizando los costos de calibración de alimentadores SMT.
2. **Plano de Masa y Filtrado:** Zonas de tierra en ambas capas (Top y Bottom GND) con costura de vías (*via stitching*), reduciendo el ruido de conmutación de la antena Wi-Fi del ESP32 sobre las trazas sensibles del acelerómetro ADXL355.
3. **Puntos de Prueba y Fiduciarios:** Se incorporaron 3 marcas fiduciarias en las esquinas de la placa para calibración de máquinas *Pick & Place*, junto a puntos de prueba (TP) en rieles de 3.3V, 5V, GND y señales de sincronismo.
4. **Exportación con un Clic:** El paquete de manufactura se encuentra en `kicad/projects/Acelerografo_ESP32/production/` con archivos Gerber, taladrado NC Drill, lista de materiales (`BOM.csv`) y archivo de coordenadas (`CPL.csv`).

---

## 🛡️ Reglas de Trabajo y Control de Versiones

1. **Rama Principal:** La rama activa de trabajo es **`main`**.
2. **Formato de Commits:** Convención estándar en minúsculas:
   - `feat: [nueva funcionalidad, bloque esquemático o ruteo completado]`
   - `fix: [corrección de pinout, huella o regla DRC]`
   - `docs: [actualización de esquemas, README o bitácora técnica]`
   - `refactor: [optimización de layout o desacoplos sin cambio funcional]`
3. **Control de Hitos por Git Tags:**
   Al culminar cada una de las 6 fases del plan de trabajo se registra un tag:
   ```bash
   git tag -a v0.1.0 -m "Fase 1 completada: Importación nativa de EAGLE a KiCad 10 y auditoría de pines ESP32"
   git push origin v0.1.0
   ```
   * Versión final de entrega técnica: `v1.0.0`.

---

## 📞 Contacto y Soporte Institucional

* **Practicante:** Edgar Joel Suárez Jaigua (`edgar.suarez@ucuenca.edu.ec`)
* **Tutor Institucional:** Ing. Milton Muñoz (`milton.munozc@ucuenca.edu.ec`)
* **Institución:** [Red Sísmica del Austro (RSA)](https://redsismicaaustro.github.io/RSA-Metodologias) — Universidad de Cuenca
