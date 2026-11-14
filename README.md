Sistema de Control de Acceso y Seguridad — Arduino UNO
Proyecto final | Arquitectura del Computador | 2S2026 | Variante A
Universidad Rafael Landívar — Ingeniería en Informática y Sistemas
Descripción
Prototipo de control de acceso para una puerta en miniatura. El sistema identifica una tarjeta mediante un RFID RC522 y solicita un PIN de cuatro dígitos por teclado. Si ambas credenciales coinciden con un registro almacenado en EEPROM, un servomotor simula la apertura de la puerta. La pantalla LCD, un buzzer y ocho indicadores LED permiten observar la interacción y los estados internos del sistema.

El proyecto integra entradas y salidas digitales, comunicación SPI e I²C, memoria no volátil, control de actuadores y un registro de desplazamiento 74HC595. Se trata de un prototipo académico, no de una cerradura certificada para instalaciones reales.
Componentes
- Arduino UNO y alimentación adecuada
- Módulo RFID RC522 y tarjetas o llaveros compatibles
- Teclado matricial (se emplean 4 filas y 3 columnas, aunque el kit puede incluir un teclado 4×4)
- Pantalla LCD 16×2 con interfaz I²C (dirección 0x27 en el programa)
- Servomotor SG90
- Registro de desplazamiento 74HC595 y ocho LEDs con sus resistencias
- Buzzer activo, protoboard y cableado
- Estructura impresa en 3D
Asignación de pines de la versión final
Elemento	Pines Arduino UNO
RFID RC522	SS/SDA: D10; MOSI: D11; MISO: D12; SCK: D13 (bus SPI); RST: conectado a 3.3 V según el montaje descrito en el código
Teclado, filas	D2, D3, D4, D5
Teclado, columnas	D6, D7, D8
Servomotor	Señal: D9
74HC595	DS: A0; SHCP: A1; STCP: A2
Buzzer	A3
LCD I²C	SDA: A4; SCL: A5


Alimentación: el RC522 trabaja a 3.3 V; no conectarlo directamente a 5 V. Para el servomotor puede utilizarse una fuente de alimentación adecuada e independiente, compartiendo tierra (GND) con el Arduino. El montaje debe verificarse antes de encenderlo. El archivo pinout.pdf y el código de base/ pueden corresponder a una fase anterior; prevalece la tabla de la versión final.
Uso del sistema
1. Conectar la alimentación. El LCD muestra SISTEMA ACCESO / INICIANDO... y después CONTROL ACCESO / PRESENTE TARJETA.
2. Presentar una tarjeta RFID en el lector situado al exterior de la estructura.
3. Si la tarjeta está registrada, el LCD solicita INGRESE SU PIN y muestra cuatro guiones bajos. Cada dígito ingresado aparece como *.
4. Escribir el PIN de cuatro cifras y pulsar # para confirmar. * borra todo el PIN escrito.
5. Cuando el PIN es correcto, la pantalla muestra ACCESO AUTORIZADO / * = CERRAR. El servo pasa de 0° (cerrado) a 90° (abierto). Presionar * para cerrar la puerta y volver a esperar otra tarjeta.
6. Si la tarjeta no se reconoce, se indica TARJETA INVALIDA / ACCESO DENEGADO y el sistema vuelve al inicio. Si el PIN es incorrecto, se permiten hasta tres intentos en ese proceso.
7. Después del tercer PIN incorrecto, suena una alarma continua. Pulsar # para apagarla. A continuación aparece BORRAR USUARIO? / 1:SI   2:NO: 1 elimina de EEPROM el usuario implicado y 2 lo conserva.
Tiempo de espera: durante la captura del PIN, si transcurren 15 segundos sin pulsaciones, aparece TIEMPO AGOTADO y se reinicia la espera de tarjeta. El temporizador se restablece después de cada pulsación.
Funciones de las teclas
Tecla	Acción según el contexto
0–`9`	Introducir PIN
*	Borrar PIN completo; cerrar puerta cuando se ha autorizado el acceso
#	Confirmar PIN completo; silenciar alarma
1	Confirmar eliminación de usuario después de una alarma
2	Conservar usuario después de una alarma


Indicadores de estado mediante 74HC595
El microcontrolador envía un patrón binario con shiftOut() al registro de desplazamiento, que activa el LED asociado al estado actual:
LED	Estado	Valor binario
1	Inicio	00000001
2	Esperando RFID	00000010
3	Validando RFID	00000100
4	Esperando PIN	00001000
5	Validando PIN	00010000
6	Acceso autorizado	00100000
7	Acceso denegado	01000000
8	Alarma	10000000


Administración de usuarios (Monitor Serial)
La versión final admite hasta 8 registros en EEPROM. Cada registro contiene un UID de 4 bytes representado como ocho caracteres hexadecimales y un PIN de cuatro caracteres. En la primera inicialización de la EEPROM el programa carga dos usuarios de demostración definidos en el código. Los valores deben adaptarse a las tarjetas reales utilizadas en el prototipo.

Abrir el Monitor Serial a 9600 baudios y enviar una instrucción por línea:
Comando	Descripción
ADD 2468	Prepara el registro de un usuario con PIN 2468; luego debe acercarse su tarjeta al lector
LIST	Enumera los usuarios almacenados; la implementación actual también imprime sus PIN en texto legible
CLEAR	Pone a cero el contador de usuarios; los registros anteriores dejan de estar disponibles en la lista activa


Importante: ADD comprueba el formato general y la longitud de la línea, pero no valida explícitamente que los cuatro caracteres del nuevo PIN sean numéricos ni evita por sí solo UIDs duplicados. Las herramientas de administración no deben considerarse seguras para un entorno de producción. No compartir capturas del Monitor Serial que revelen PIN reales.

Los datos almacenados permanecen en EEPROM después de desconectar la alimentación. Si se ejecuta CLEAR, el sistema no vuelve a cargar automáticamente los usuarios de ejemplo en cada reinicio, porque conserva su marca de inicialización.
Código fuente y estructura de la rama
La rama de trabajo es 4paula0r-fase-2. En la versión inspeccionada se encuentran:
arduino-proyecto-arquitectura/
├── README.md                    # Descripción e instrucciones
├── codigofuncional              # Versión funcional final compartida por el equipo
├── Fase2                        # Versión anterior de la fase 2
├── codigofase1                  # Trabajo de la fase 1
├── base/base.ino                # Plantilla académica inicial, no programa final
├── diagnostics/diagnostics.ino  # Herramientas de diagnóstico
├── 3d-printing/                 # Archivos STL de la estructura
├── pinout.pdf                   # Referencia anterior de conexiones
└── .github/workflows/compile.yaml
Para una entrega inequívoca: se recomienda copiar el contenido de codigofuncional a control_acceso_final/control_acceso_final.ino, retirando cualquier marcador Markdown de apertura o cierre (como las tres comillas invertidas que aparecen en el archivo de texto). De esta forma Arduino IDE podrá abrirlo y compilarlo como sketch. Mantener las versiones anteriores separadas como historial. El flujo de GitHub Actions existente compila base/ y diagnostics/ en pull requests hacia main, no certifica la compilación de codigofuncional.
Bibliotecas
El sketch final incluye SPI, Wire, EEPROM, Servo, MFRC522, LiquidCrystal_I2C y Keypad. Las tres primeras se incluyen normalmente con el entorno de la plataforma Arduino AVR; el resto debe estar disponible en el Arduino IDE según la placa y el gestor de bibliotecas. Seleccionar Arduino Uno y el puerto correspondiente antes de compilar y cargar.
Diseño y modificación del modelo 3D
Para construir el prototipo se partió del modelo 3D de referencia proporcionado durante el curso. Se mantuvo la estructura general y se realizaron modificaciones físicas para que el lector RFID quedara accesible desde el exterior, de modo que la tarjeta pueda leerse sin abrir la carcasa. Esta adaptación mejora la interacción con el prototipo y facilita la demostración de la autenticación.

Los archivos STL de la estructura están incluidos en 3d-printing/. No se afirma que los STL publicados incorporen necesariamente el recorte físico: la modificación realizada debe corroborarse con fotografías del montaje final y, si existen, con los STL editados.

Modelo original de referencia: memepelle/arduino-security-project.
Manual de usuario y evidencias
El entregable solicitado por el curso comprende el código fuente, el manual de usuario y los modelos 3D, además del prototipo físico funcional presentado presencialmente. Se recomienda incorporar al repositorio el manual en PDF y al menos fotografías del ensamblaje, de la ubicación exterior del lector RFID y del sistema encendido. Las imágenes son recomendables como evidencia documental, aunque la guía no exige expresamente fotografías dentro del manual.
Alcance y limitaciones
- El prototipo realiza autenticación local con tarjetas RFID y PIN, sin conexión a internet ni servidor.
- El software reconoce el UID a partir de los primeros cuatro bytes leídos de la tarjeta.
- Los PIN quedan guardados y, mediante LIST, expuestos en texto legible; por ello el diseño es de uso demostrativo/académico, no de seguridad física de producción.
- El programa no controla un sensor que confirme la posición real de la puerta; utiliza posiciones programadas del servomotor.
- La gestión por Serial requiere conexión con una computadora y permisos de acceso al dispositivo.
Referencias
- Guía del curso: Proyecto de aplicación 2S2026 — Sistema computacional de Control de Acceso y Seguridad, Variante A.
- Repositorio 3D de referencia: https://github.com/memepelle/arduino-security-project
