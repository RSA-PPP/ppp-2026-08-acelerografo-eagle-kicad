---
titulo: "Plan de Trabajo de Prácticas Preprofesionales: Migración, Auditoría y Optimización de la PCB del Acelerógrafo ESP32 de Autodesk EAGLE a KiCad 10 con Estandarización PCBA"
proyecto: "Migración de EAGLE a KiCad 10 y Optimización de Hardware para Acelerógrafo Basado en ESP32"
codigo_proyecto: "RSA-PPP-2026-08"
area_tematica: "Diseño de Hardware Electrónico, CAD/EDA Libre (KiCad 10), Compatibilidad Firmware-Hardware y Manufactura PCBA"
estado: "Culminado"
version: "1.0"
fecha_creacion: "2026-07-15"
fecha_actualizacion: "2026-09-25"

practicante:
  nombre: "Edgar Joel Suárez Jaigua"
  cedula: "N/D"
  correo: "edgar.suarez@ucuenca.edu.ec"
  carrera: "Ingeniería en Telecomunicaciones"
  institucion: "Universidad de Cuenca"

tutoria:
  tutor_institucional: "Ing. Milton Muñoz. (Red Sísmica del Austro - RSA)"
  correo_tutor_institucional: "milton.munozc@ucuenca.edu.ec"

cronograma:
  duracion_total_horas: 96
  dedicacion_semanal_horas: 16
  duracion_semanas: 6
  fecha_inicio: "2026-07-15"
  fecha_fin_estimada: "2026-09-30"
  modalidad: "Presencial"

tecnologias:
  - "Software de Diseño Electrónico EDA: KiCad 10 (Eeschema, PCB Editor, Gestor de Paquetes PCM)"
  - "Software EDA Origen: Autodesk EAGLE (Importación de esquemático .sch y placa .brd SMD)"
  - "Entorno de Automatización y Scripts: Python 3, herramienta jlc2kicadlib (descarga automatizada de símbolos, huellas y modelos 3D STEP)"
  - "Plugins y Herramientas PCBA: Fabrication Toolkit (JLCPCB Tools) para exportación directa de Gerbers, BOM.csv y CPL.csv"
  - "Hardware y Firmware Vinculado: SoC ESP32-WROOM-32, Acelerómetro Triaxial ADXL355 (SPI 250 Hz), RTC DS3231 (I2C/SQW), GPS (UART), MicroSD (VSPI)"
  - "Estandarización de Componentes: Catálogo de componentes LCSC / JLCPCB (priorización de Basic Parts y Preferred Parts)"
  - "Control de Versiones y Librerías: Git, GitHub, librerías locales aisladas en `/libs` con variables relativas (${KIPRJMOD})"

repositorio:
  url: "https://github.com/RSA-PPP/ppp-2026-08-acelerografo-esp32-kicad"
  rama_base: "main"
---

---

## 1. Antecedentes y Justificación Técnica

1. **Contexto del Sistema y Estado Inicial del Artefacto:** 
   El desarrollo del acelerógrafo institucional de bajo costo basado en ESP32 para la Red Sísmica del Austro (RSA) ha transitado por tres hitos técnicos complementarios:
   * **Proyecto de Adquisición de Datos (Firmware y Software):** Validación del circuito en protoboard y desarrollo del firmware multihilo en C++/FreeRTOS para el ESP32, integrando el acelerómetro ADXL355 a 250 Hz por SPI, el módulo GPS por UART, el reloj RTC DS3231 por I2C (pulso SQW a 1 Hz) y el almacenamiento continuo en tarjeta MicroSD.
   * **Proyecto de Diseño Electrónico en EAGLE:** Diseño de los primeros circuitos impresos en Autodesk EAGLE, divididos en una versión modular con breakouts y una versión compacta con componentes de montaje superficial (SMD).
   * **Proyecto de Migración de EDA (Altium a KiCad):** Establecimiento del entorno de software libre KiCad 10 en la RSA, automatización de librerías locales con `jlc2kicadlib` y estandarización para manufactura automatizada PCBA en JLCPCB mediante el plugin *Fabrication Toolkit* (ejecutado previamente en [ppp-2026-07-shm-altium-kicad](https://github.com/RSA-PPP/ppp-2026-07-shm-altium-kicad)).
   A pesar de disponer del diseño SMD en EAGLE, se requiere consolidarlo en el entorno corporativo de código abierto KiCad 10 y auditarlo meticulosamente para asegurar que no existan discrepancias entre las conexiones de la placa física y la asignación de pines del firmware de producción.

2. **Limitaciones Críticas o Cuellos de Botella Identificados:** 
   * **Riesgo de Incompatibilidad Hardware-Firmware:** En proyectos previos se evidenciaron discrepancias de asignación de pines GPIO entre los esquemas circuitales y el código fuente compilado. Una divergencia en una línea SPI de selección de chip (CS) o en el pin de interrupción del pulso de sincronismo (SQW) inhabilita por completo la operatividad de la estación en campo.
   * **Fragilidad ante Ruido Eléctrico en Adquisición:** La presencia simultánea del radioenlace Wi-Fi del ESP32 (conmutaciones de potencia de RF) y buses digitales rápidos (SPI a 250 Hz) puede acoplar ruido analógico espurio a la etapa sensora del ADXL355 si no se cuenta con planos de masa masivos (Top/Bottom GND) y condensadores de desacoplo ultra cercanos ($0.1\,\mu\text{F}$ y $10\,\mu\text{F}$) a los pines de alimentación de cada integrado.
   * **Costos Adicionales en Ensamblaje SMT:** La falta de coincidencia con el catálogo de partes básicas (*Basic Parts*) de JLCPCB incrementa sustancialmente el costo por placa al requerir el cambio manual de alimentadores en las máquinas automáticas Pick & Place.
   * **Desalineación y Desfases Angulares en Visores PCBA:** La importación cruda de footprints suele generar desfases de $90^\circ$ o $180^\circ$ en las coordenadas de montaje CPL, conllevando a componentes soldados al revés si no se audita visualmente la orientación física en el visor del fabricante.

3. **Desacoplamiento del Alcance y Propuesta de Solución:** 
   Estas prácticas preprofesionales de **96 horas** se focalizan en migrar la versión SMD del acelerógrafo ESP32 desde Autodesk EAGLE hacia **KiCad 10**, aplicando una exhaustiva **auditoría de pines (Pinout Audit)** y optimización de integridad de señal:
   * Importar nativamente esquemático y placa, contrastando minuciosamente cada GPIO del ESP32 con el código fuente del firmware de adquisición de datos.
   * Reubicar e incorporar condensadores de desacoplo cerámicos de ultra baja ESR adyacentes a los pines VCC de los circuitos integrados sensibles.
   * Implementar planos de tierra continuos en capas superior e inferior para blindar las trazas frente a la antena Wi-Fi del ESP32.
   * Estandarizar el 100% de componentes pasivos y activos con números de parte LCSC, incorporando librerías locales portables en `/libs` mediante `jlc2kicadlib` y rutas relativas `${KIPRJMOD}`.
   * Incorporar puntos de prueba (*test points*) accesibles y marcas fiduciarias (*fiducials*), generando y validando el paquete de producción (Gerber, Drill, BOM.csv, CPL.csv) con el plugin *Fabrication Toolkit*.

---

## 2. Objetivos del Plan de Trabajo

### 2.1. Objetivo General
Migrar, auditar y optimizar el diseño de la placa de circuito impreso (PCB) del acelerógrafo basado en ESP32 desde Autodesk EAGLE hacia el entorno KiCad 10, asegurando compatibilidad del 100% con el firmware de adquisición de datos original, minimizando el ruido electromagnético mediante planos de masa robustos y estandarizando el diseño para ensamblaje automatizado (PCBA) en JLCPCB.

### 2.2. Objetivos Específicos
1. **[Importación y Auditoría de Pines - 16 h]:** Importar nativamente los archivos `.sch` y `.brd` de la versión SMD de EAGLE a KiCad 10 y auditar exhaustivamente la asignación de pines (GPIOs) del ESP32 frente al firmware original (ADXL355, GPS, RTC, SQW 1 Hz, MicroSD), corrigiendo cualquier discrepancia circuital.
2. **[Estandarización de Componentes y Asignación LCSC - 16 h]:** Catalogar componentes pasivos y activos a tecnología SMD estándar (0805, 0603, SOT-23, SOIC), inyectar masivamente el campo `LCSC` en todos los símbolos esquemáticos y asignar números de parte priorizando el catálogo *Basic Parts* de JLCPCB.
3. **[Gestión de Librerías Locales Relativas - 12 h]:** Descargar de forma automatizada mediante scripts de `jlc2kicadlib` los símbolos (`.kicad_sym`), huellas (`.kicad_mod`) y modelos 3D (`.step`) de componentes no nativos, estructurándolos en la carpeta local `/libs` con direccionamiento relativo `${KIPRJMOD}`.
4. **[Optimización de Ruteo, Planos GND y Test Points - 24 h]:** Rediseñar el layout de la placa optimizando desacoplos capacitivos ($0.1\,\mu\text{F}$ y $10\,\mu\text{F}$) cercanos a pines VCC, trazas directas para bus SPI a 250 Hz, planos de masa sólidos continuos (GND pour) con costura de vías, puntos de prueba (*test points*) y marcas fiduciarias (*fiducials*), aprobando el 100% de reglas DRC.
5. **[Generación y Validación de Archivos PCBA - 14 h]:** Exportar mediante el plugin *Fabrication Toolkit* el paquete de fabricación (Gerbers, NC Drill, `BOM.csv`, `CPL.csv`), validando visualmente la colocación y rotación de componentes SMD en el visor de JLCPCB y corrigiendo desfases angulares.
6. **[Cierre, Comparativa y Documentación Técnica - 14 h]:** Elaborar el informe de auditoría hardware-firmware que certifique la compatibilidad de conexiones, estructurar el archivo `README.md` operativo en la raíz del repositorio y formalizar la entrega técnica.

---

## 3. Plan de Trabajo Detallado y Checkpoints Cuantificables

### Fase 1: Importación Nativa y Auditoría de Pines de Hardware/Firmware
* **Duración:** 16 horas (Semanas 1 y 2)
* **Objetivo:** Migrar los esquemas originales de EAGLE a KiCad 10 y auditar minuciosamente las conexiones para garantizar compatibilidad directa con el firmware del sistema de adquisición.
* **Actividades técnicas:**
  1. Cargar el esquemático (`.sch`) y la placa (`.brd`) de la versión SMD de EAGLE utilizando la herramienta de importación nativa de KiCad 10.
  2. Ejecutar la **Auditoría de Asignación de Pines (Pinout Audit):**
     - Cotejar las conexiones del SoC ESP32 en el esquemático con las definiciones de GPIOs en el código fuente de adquisición (`include/ADXL355.h`, `GPS.h`, `RTC3231.h`, `SD_mod.h`).
     - Verificar líneas del bus SPI con el acelerómetro ADXL355: MOSI, MISO, SCK y Chip Select ($\overline{\text{CS}}$).
     - Verificar líneas UART con el GPS FGPMMOPA6H (TX, RX).
     - Verificar líneas del bus I2C con el RTC DS3231 (SDA, SCL) y la línea de interrupción SQW (1 Hz) vinculada al timer de hardware de 1 ms.
     - Verificar líneas del bus SPI dedicado para el módulo MicroSD (VSPI).
  3. Corregir cualquier discrepancia o pin erróneamente conectado en el esquemático para asegurar que la placa final ejecute el firmware original sin necesidad de reprogramación o parches en código.
  4. Depurar etiquetas de red globales (*global labels*), buses y puertos de alimentación para asegurar la integridad eléctrica del diseño.
* **Checkpoints medibles para revisión:**
  - [x] **CP 1.1:** Proyecto importado en KiCad 10 (`.kicad_sch` y `.kicad_pcb`) sin pérdida de elementos gráficos.
  - [x] **CP 1.2:** Matriz comparativa Pinout Hardware vs. Firmware elaborada certificando concordancia total de GPIOs.
  - [x] **CP 1.3:** Esquemático depurado superando la comprobación de reglas eléctricas (ERC) sin errores de conectividad.

---

### Fase 2: Estandarización de Componentes SMD y Asignación de Códigos LCSC
* **Duración:** 16 horas (Semanas 2 y 3)
* **Objetivo:** Adaptar y estandarizar todos los componentes para compatibilidad con la línea de montaje automatizado PCBA de JLCPCB.
* **Actividades técnicas:**
  1. Catalogar los componentes pasivos y activos del circuito para consolidar su transición definitiva a tecnología de montaje superficial.
  2. Estandarizar encapsulados: seleccionar huellas industriales (0805, 0603, SOT-23, SOIC) que soporten los niveles de corriente, voltaje y disipación térmica requeridos por el sistema.
  3. Inyectar de forma masiva el campo personalizado `LCSC` en todos los símbolos del esquemático mediante el editor de campos de KiCad.
  4. Buscar y asignar los números de parte (LCSC Part Number) en el catálogo oficial de JLCPCB/LCSC, priorizando componentes catalogados como *Basic Parts* para reducir costos de manufactura.
* **Checkpoints medibles para revisión:**
  - [x] **CP 2.1:** Inventario de componentes catalogado con encapsulados SMD normalizados.
  - [x] **CP 2.2:** 100% de los componentes del esquemático con el metadato de parte `LCSC` correctamente asignado.
  - [x] **CP 2.3:** Optimización de costos verificada priorizando partes básicas frente a partes extendidas.

---

### Fase 3: Gestión de Librerías Locales Relativas (jlc2kicadlib)
* **Duración:** 12 horas (Semana 3)
* **Objetivo:** Asegurar la portabilidad completa del proyecto mediante el uso de librerías locales aisladas y rutas relativas.
* **Actividades técnicas:**
  1. Ejecutar scripts automatizados basados en la herramienta CLI `jlc2kicadlib` para descargar símbolos (`.kicad_sym`), huellas (`.kicad_mod`) y modelos 3D (`.step`) de los componentes no estándar.
  2. Almacenar los activos descargados exclusivamente en la carpeta `/libs` en la raíz del repositorio.
  3. Configurar la tabla de librerías del proyecto de KiCad utilizando rutas relativas mediante la variable de entorno del proyecto `${KIPRJMOD}/libs/...`, garantizando que el repositorio sea 100% autónomo al clonarse en cualquier computadora.
  4. Realizar la auditoría física de huellas: verificar correspondencia geométrica milimétrica, correcta posición del Pin 1 y orientación espacial de los modelos 3D STEP respecto al footprint.
* **Checkpoints medibles para revisión:**
  - [x] **CP 3.1:** Carpeta `/libs` creada y poblada con todos los activos descargados mediante `jlc2kicadlib`.
  - [x] **CP 3.2:** Rutas de librerías y modelos 3D configuradas en modo relativo con `${KIPRJMOD}`.
  - [x] **CP 3.3:** Auditoría de Pin 1 y modelos 3D aprobada sin desalineaciones mecánicas.

---

### Fase 4: Optimización del Ruteo, Diseño Físico y Mejoras de Funcionamiento
* **Duración:** 24 horas (Semanas 4 y 5)
* **Objetivo:** Optimizar el layout físico de la PCB en KiCad para garantizar la correcta adquisición de señales y reducir el ruido electromagnético en condiciones reales.
* **Actividades técnicas:**
  1. **Planos de alimentación y desacoplo:** Añadir o reubicar condensadores de desacoplo cerámicos de montaje superficial ($0.1\,\mu\text{F}$ y $10\,\mu\text{F}$) lo más cerca posible de los pines de alimentación (VCC) del ESP32, el acelerómetro ADXL355 y el RTC para filtrar el rizado de la fuente de alimentación.
  2. **Integridad en señales críticas:** Rutar de manera directa y con el mínimo uso de vías las líneas de datos de alta velocidad (bus SPI a 250 Hz del ADXL355 y bus de la tarjeta SD) para prevenir atenuaciones de señal o pérdidas de tramas.
  3. **Plano de masa robusto:** Implementar zonas de cobre dedicadas para la masa (GND) en ambas capas (*Top* y *Bottom*), minimizando bucles de tierra y blindando el circuito frente al ruido de conmutación de la antena de RF del ESP32.
  4. **Puntos de prueba (Test Points):** Colocar puntos de prueba accesibles en las líneas críticas (GND, 3.3V, 5V, TX/RX, señal SQW a 1 Hz y líneas de control SPI) para facilitar la depuración física con osciloscopio o multímetro en laboratorio.
  5. **Layout y marcas para ensamblaje:** Colocar al menos 3 marcas fiduciarias (*Fiducials*) en las esquinas de la placa para calibración óptica de las máquinas *Pick & Place*.
  6. Ejecutar la Verificación de Reglas de Diseño (DRC) en KiCad y corregir el 100% de advertencias y violaciones.
* **Checkpoints medibles para revisión:**
  - [x] **CP 4.1:** Condensadores de desacoplo ubicados inmediatamente adyacentes a los pines VCC de integrados clave.
  - [x] **CP 4.2:** Planos de masa sólidos (Top/Bottom) con costura de vías (*via stitching*) ruteados sin islas de cobre aisladas.
  - [x] **CP 4.3:** Puntos de prueba y 3 marcas fiduciarias dispuestas en placa; reporte DRC con 0 errores.

---

### Fase 5: Generación y Validación de Archivos de Fabricación PCBA
* **Duración:** 14 horas (Semana 5)
* **Objetivo:** Generar la documentación técnica exacta de producción y validar su visualización final en la plataforma industrial de JLCPCB.
* **Actividades técnicas:**
  1. Utilizar el plugin *Fabrication Toolkit* integrado en KiCad 10 para exportar con un solo clic:
     - Archivos Gerber completos RS-274X y archivos de perforación NC Drill.
     - Lista de Materiales (`BOM.csv`) formateada con designador, cantidad, footprint, LCSC Part Number y valor.
     - Archivo de Coordenadas de Posición (`CPL.csv`) para colocación automatizada de componentes SMD.
  2. Subir el paquete de producción generado al visor web de ensamble de JLCPCB.
  3. Validar visualmente la alineación, polaridad y orientación de todos los componentes SMD sobre los pads de la placa.
  4. Corregir cualquier desvío de rotación ($90^\circ$ o $180^\circ$) aplicando el factor de corrección directamente en la configuración del *Fabrication Toolkit*.
* **Checkpoints medibles para revisión:**
  - [x] **CP 5.1:** Paquete completo de manufactura (Gerbers, NC Drill, BOM y CPL) generado de forma automatizada.
  - [x] **CP 5.2:** Inspección en visor web de JLCPCB confirmando colocación y rotación exacta del 100% de componentes SMD.
  - [x] **CP 5.3:** Archivos listos para producción sin requerir ajustes manuales en los archivos CSV.

---

### Fase 6: Cierre, Auditoría Comparativa y Documentación Técnica
* **Duración:** 14 horas (Semana 6)
* **Objetivo:** Consolidar la documentación del proyecto migrado para asegurar la reproducibilidad, el mantenimiento futuro y la entrega técnica.
* **Actividades técnicas:**
  1. Documentar formalmente las mejoras de diseño (condensadores de desacoplo, corrección de pines auditada contra firmware, planos de masa optimizados) realizadas respecto al diseño original de EAGLE.
  2. Elaborar el archivo `README.md` operativo en la raíz del repositorio [ppp-2026-08-acelerografo-esp32-kicad](https://github.com/RSA-PPP/ppp-2026-08-acelerografo-esp32-kicad) detallando:
     - Versión exacta de KiCad utilizada (v10).
     - Instrucciones detalladas de instalación y configuración de `jlc2kicadlib`.
     - Resumen de la asignación de pines auditada y validada contra el firmware en C++.
     - Guía paso a paso para regenerar los archivos de fabricación mediante el plugin.
  3. Elaborar el informe breve de auditoría técnica que certifique que la asignación de pines en la PCB coincide con el firmware original de adquisición de datos.
  4. Revisión técnica con el tutor institucional.
* **Checkpoints medibles para revisión:**
  - [x] **CP 6.1:** Repositorio institucional en GitHub organizado con código, librerías locales y archivos de release.
  - [x] **CP 6.2:** `README.md` operativo detallando el proceso de regeneración de manufactura y configuración de dependencias.
  - [x] **CP 6.3:** Documento técnico de auditoría y validación hardware-firmware concluido y entregado al tutor institucional de la RSA.

---

## 4. Resumen de Fases y Cronograma Semanal (96 Horas)

El plan de trabajo comprende **96 horas** distribuidas en **6 semanas** con una carga de **16 horas semanales**:

| Fase | Semanas | Horas | Enfoque Principal |
| :---: | :---: | :---: | :--- |
| **Fase 1** | Semanas 1–2 | **16 h** | Importación nativa de EAGLE a KiCad 10 y Auditoría exhaustiva de pines frente al firmware ESP32. |
| **Fase 2** | Semanas 2–3 | **16 h** | Estandarización de componentes SMD (0805/0603) e inyección masiva de números de parte LCSC. |
| **Fase 3** | Semana 3 | **12 h** | Descarga automatizada con `jlc2kicadlib` a `/libs` con rutas relativas portables `${KIPRJMOD}`. |
| **Fase 4** | Semanas 4–5 | **24 h** | Ruteo optimizado, desacoplos cercanos, planos de masa GND, puntos de prueba y DRC sin errores. |
| **Fase 5** | Semana 5 | **14 h** | Generación con *Fabrication Toolkit*, validación en visor web de JLCPCB y corrección de rotaciones. |
| **Fase 6** | Semana 6 | **14 h** | Informe de auditoría hardware-firmware, `README.md` operativo y entrega en repositorio GitHub. |
| **Total** | **6 Semanas** | **96 h** | **Planificación Consolidada de Prácticas Preprofesionales** |

---

## 5. Entregables y Criterios de Éxito de las Prácticas Preprofesionales

1. **Repositorio Autónomo y Portable en KiCad 10:**
   * Archivos de esquemático (`.kicad_sch`) y de circuito impreso (`.kicad_pcb`) completos, funcionales y auditados contra el firmware original de adquisición de datos en: [ppp-2026-08-acelerografo-esp32-kicad](https://github.com/RSA-PPP/ppp-2026-08-acelerografo-esp32-kicad).
   * Carpeta local `/libs` con símbolos, huellas y modelos 3D STEP asociados mediante rutas relativas (`${KIPRJMOD}`).
2. **Paquete de Fabricación Validador (Production Release):**
   * Archivos Gerber RS-274X, taladrado NC Drill, Lista de Materiales (`BOM.csv`) y archivo de coordenadas de colocación (`CPL.csv`) listos para ser procesados por JLCPCB sin modificaciones manuales intermedias.
   * Orientaciones de componentes SMD y offsets calibrados al 100% en el visor de ensamble automatizado.
3. **Documento Técnico de Auditoría Hardware-Firmware:**
   * Documento técnico que certifique que la asignación de pines (GPIOs) en la PCB coincide estrictamente con el código fuente de adquisición del ESP32, detallando las optimizaciones de bajo ruido, desacoplo capacitivo y planos de masa aplicados.
4. **README Operativo del Proyecto:**
   * Archivo `README.md` en la raíz del repositorio detallando la versión de KiCad 10, instalación de herramientas automatizadas (`jlc2kicadlib`, Fabrication Toolkit) y guía de reproducción.
