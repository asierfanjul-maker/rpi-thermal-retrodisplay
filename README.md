# rpi-thermal-retrodisplay

Es un controlador de bajo nivel para la visualización de la temperatura del SoC (System on Chip) de la Raspberry Pi en tiempo real, utilizando hardware de visualización retro.
Hecho en C y con componentes reciclados de otros dispositivos para darles una segúnda vida.

## Especificaciones Técnicas

El proyecto consiste en un servicio en segundo plano (daemon) gestionado por **Systemd** que interactúa directamente con los registros GPIO mediante la librería **WiringPi**.

### Arquitectura de Hardware
* **Visualización:** 2 Displays de 7 segmentos **HP 5082-7613** (tecnología de arseniuro de galio-fósforo).
* **Decodificación:** Integrado **CD4511** (BCD-to-7-Segment Latch/Decoder/Driver).
* **Multiplexación:** Compuerta **CD40107BE** (Dual NAND Buffer/Driver) en configuración de colector abierto para la conmutación de cátodos/ánodos.
* **Niveles Lógicos:** Adaptación de impedancias para compatibilidad entre lógica TTL (5V) y CMOS (3.3V) de la Raspberry Pi.
* **Alimentación:** A 5V.

## Conexiones (Pinout)

| Señal | Función | GPIO (BCM) | Pin Físico (Header) | Destino |
| :--- | :--- | :--- | :--- | :--- |
| **BCD A** | Bit 0 (Peso 1) | **21** | 40 | CD4511 Pin 7 |
| **BCD B** | Bit 1 (Peso 2) | **12** | 32 | CD4511 Pin 1 |
| **BCD C** | Bit 2 (Peso 4) | **16** | 36 | CD4511 Pin 2 |
| **BCD D** | Bit 3 (Peso 8) | **20** | 38 | CD4511 Pin 6 |
| **MUX CTRL** | Selección Dígito | **25** | 22 | CD40107 Pin 1/2 |

### Lógica de Software
* **Lenguaje:** C (ISO C99).
* **Algoritmo:** Bucle infinito con control de tiempos de asentamiento (*settling time*) para evitar condiciones de carrera y *ghosting* en el bus BCD.
* **Lectura Térmica:** Acceso directo al sistema de archivos virtual `/sys/class/thermal/thermal_zone0/temp`.

## Compilación e Instalación

1. **Compilar el código fuente:**
   ```bash
   gcc -Wall -o tempdisplay tempdisplay.c -lwiringPi
2. **Ejecución de prueba (Manual):**
   Se requieren privilegios de superusuario (root) para acceder a los registros físicos de memoria (`/dev/mem`) que controlan los GPIO.
   ```bash
   sudo ./tempdisplay
3. **Despliegue como Servicio (Systemd):**
    Para garantizar la persistencia y el arranque automático tras el inicio del sistema (boot), se debe registrar la unidad en el gestor de servicios.
     # 1. Copiar el archivo de unidad al directorio del sistema
      sudo cp tempdisplay.service /etc/systemd/system/

    # 2. Recargar el demonio de systemd para reconocer la nueva configuración:
      sudo systemctl daemon-reload

    # 3. Habilitar el servicio para el arranque (crea el symlink en multi-user.target)
      sudo systemctl enable tempdisplay.service

   # 4. Iniciar el servicio inmediatamente
     sudo systemctl start tempdisplay.service
4. **Verificación de estado:**
   systemctl status tempdisplay.service
