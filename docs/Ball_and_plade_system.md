#Proyecto Final

###Nombre del proyecto: Ball and Plate system

###Autores
- Barrientos Miguel Leonardo
- Zermeño Cervantes Rodrigo

###Asignatura: Introducción a la Mecatronica

###Fecha: 5 de Diciembre del 2025

###Descripción
En este proyecto final desarrollamos un Ball and Plate System, un mecanismo mecatrónico capaz de mantener una pelota equilibrada sobre una plataforma mediante la variación de ángulos en una placa superior.

El sistema utiliza cuatro servomotores, los cuales modifican la inclinación de la placa en los ejes X y Y. A través del control coordinado de estos servomotores, la plataforma es capaz de moverse en diferentes direcciones para dirigir o estabilizar la pelota en una posición deseada.

Este mecanismo es una base importante para sistemas de control avanzados, como plataformas estabilizadoras, robots equilibristas o sistemas de visión con retroalimentación.
El proyecto se encuentra documentado y versionado en GitHub, donde se registran los avances, diagramas y códigos implementados.

                         
##Objetivos


###General
Diseñar y programar un sistema mecatrónico de control de inclinación capaz de mover una plataforma en dos ejes mediante servomotores para lograr el equilibrio dinámico de una pelota.

###Especificos

- Implementar la comunicación y control de 4 servomotores utilizando un microcontrolador (Arduino/ESP32)..  
- Diseñar la estructura mecánica para mover la placa superior mediante un sistema articulado. 
- Desarrollar el código para mover la plataforma en diferentes direcciones variando los ángulos de los servos.
- Comprender la relación entre los movimientos de los servomotores y la inclinación resultante de la placa.
- Documentar el proceso mecánico, electrónico y de programación del sistema.

##Alcance y Exclusiones
 -**Incluye**:

- Montaje mecánico con una placa móvil y cuatro servos.
- Código para controlar la inclinación de la placa en ambos ejes. 
- Pruebas manuales de inclinación mediante valores programados.
- Calibración básica del rango de movimiento de los servos.


 -**No Incluye**: 

##Procedimiento

###Diseño Mecánico Del Sistema

Se construyó una plataforma con dos niveles:

1- Placa inferior fija, donde se colocan los servomotores.

2- Placa superior móvil, unida mediante rótulas o uniones articuladas.


Cada uno de los cuatro servomotores está conectado a un punto distinto de la placa móvil. Al variar el ángulo de los servos, se altera la altura relativa de cada esquina de la plataforma, generando inclinación en dos ejes.

Tambien se incluye una webcam mediante la cual recibimos la imagen del de la parte superior de la plataforma donde es colocada la pelota. El objetivo de esto es que mediante la retroaliemtación visual recibida por parte de la webcam , el sistema pueda compensar perturbaciones y mantener la pelota en la plataforma sin que se caiga de esta.

###Programación del código

Por otro lado y probablemente la parte más importante para que funionara el mecanismo, es el codigo, el cual consistio en 2 partes, el codigo realizado en python mediante Visual Studio.

```cpp
import cv2
import numpy as np
import time
 
# ====== OPCIONAL: Bluetooth ======
BT_ENABLE  = True
BT_ADDR    = "F4:65:0B:54:F7:46"
BT_PORT    = 1
BT_TIMEOUT = 5
 
sock = None
if BT_ENABLE:
    try:
        import bluetooth  # PyBluez
        sock = bluetooth.BluetoothSocket()
        sock.settimeout(BT_TIMEOUT)
        print("Conectando a ESP32 via Bluetooth...")
        sock.connect((BT_ADDR, BT_PORT))
        print("¡Conectado a ESP32!")
    except Exception as e:
        print("No se pudo conectar BT:", e)
        print("Continuaré sin BT...")
        sock = None
        BT_ENABLE = False
 
# ====== Cámara ======
video = cv2.VideoCapture(0)
w, h = 640, 480
video.set(cv2.CAP_PROP_FRAME_WIDTH, w)
video.set(cv2.CAP_PROP_FRAME_HEIGHT, h)
 
# ====== Procesamiento ======
kernel = np.ones((5, 5), np.uint8)
 
# Rojo en HSV (dos bandas)
bajo1 = np.array([ 0, 100, 100], dtype=np.uint8)
alto1 = np.array([10, 255, 255], dtype=np.uint8)
bajo2 = np.array([170, 100, 100], dtype=np.uint8)
alto2 = np.array([180, 255, 255], dtype=np.uint8)
 
def circularidad(c):
    area = cv2.contourArea(c)
    per = cv2.arcLength(c, True)
    if per == 0:
        return 0.0
    return 4.0 * np.pi * area / (per * per)
 
# Centro de referencia (centro de pantalla)
cx_ref = w // 2
cy_ref = h // 2
 
# Para evitar temblor
DEAD_X = 10
DEAD_Y = 10
 
while True:
 
    ok, frame = video.read()
    if not ok:
        break
 
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
 
    # ======== DETECCIÓN POR COLOR ========
    mask1 = cv2.inRange(hsv, bajo1, alto1)
    mask2 = cv2.inRange(hsv, bajo2, alto2)
    red_mask = cv2.bitwise_or(mask1, mask2)
 
    red_mask = cv2.morphologyEx(red_mask, cv2.MORPH_OPEN, kernel, iterations=2)
    red_mask = cv2.morphologyEx(red_mask, cv2.MORPH_CLOSE, kernel, iterations=2)
 
    contornos, _ = cv2.findContours(red_mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
 
    pelota_x = None
    pelota_y = None
 
    if contornos:
        cont = max(contornos, key=cv2.contourArea)
        area = cv2.contourArea(cont)
        circ = circularidad(cont)
 
        if area > 200 and circ > 0.5:
            (x, y), radius = cv2.minEnclosingCircle(cont)
 
            if radius > 8:
                pelota_x = int(x)
                pelota_y = int(y)
 
                cv2.circle(frame, (pelota_x, pelota_y), int(radius), (0,255,0), 2)
                cv2.putText(frame, "COLOR", (pelota_x+10, pelota_y),
                            cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0,255,0), 2)
 
    # ======== SI EL COLOR NO DETECTA → HOUGH CIRCLES ========
    if pelota_x is None:
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        gray = cv2.GaussianBlur(gray, (7,7), 0)
 
        circles = cv2.HoughCircles(gray, cv2.HOUGH_GRADIENT, 1.2, 50,
                                   param1=80, param2=25,
                                   minRadius=5, maxRadius=120)
 
        if circles is not None:
            circles = np.round(circles[0]).astype("int")
            x, y, r = circles[0]
 
            pelota_x = x
            pelota_y = y
 
            cv2.circle(frame, (x, y), r, (0,255,255), 2)
            cv2.putText(frame, "CIRCULO", (x+10, y),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0,255,255), 2)
 
    # ======== SI SE DETECTÓ PELOTA → CALCULAR ERRORES ========
    if pelota_x is not None:
 
        # Centro de la base
        cv2.drawMarker(frame, (cx_ref, cy_ref), (255,255,255),
                        cv2.MARKER_CROSS, 20, 2)
 
        # Errores
        error_x = pelota_x - cx_ref
        error_y = pelota_y - cy_ref
 
        # Zona muerta
        if abs(error_x) < DEAD_X: error_x = 0
        if abs(error_y) < DEAD_Y: error_y = 0
 
        # Enviar coordenadas NUMÉRICAS
        msg = f"{error_x},{error_y}\n"
        if BT_ENABLE and sock is not None:
            sock.send(msg.encode())
 
        print("ENVIADO →", msg.strip())
 
        # Línea visual
        cv2.line(frame, (cx_ref, cy_ref), (pelota_x, pelota_y), (0,255,0), 2)
 
    else:
        print("Pelota NO detectada")
 
    # Mostrar
    cv2.imshow("TRACKING", frame)
    cv2.imshow("MÁSCARA ROJO", red_mask)
 
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
 
# ====== Limpieza ======q
video.release()
cv2.destroyAllWindows()
if BT_ENABLE and sock is not None:
    try: sock.close()
    except: pass
```

Y el codigo en Arduino para bluetooth y de esta manera este pudiese recibir señales de la webcam.


```cpp
#include "BluetoothSerial.h"
 
BluetoothSerial SerialBT;
 
// ================================
// PID
// ================================
float kp = 0.8;
float ki = 0.01;
float kd = 0.8;
 
float t_now = 0;
float t_bef = 0;
float dt = 0;
float integralX = 0;
float errorX_ant = 0;
 
// ================================
// Servos por PWM (ESP32 LEDC)
// ================================
#define SERVO_ARRIBA     32
#define SERVO_ABAJO      33
#define SERVO_DERECHA    25
#define SERVO_IZQUIERDA  26
 
// Map de servo en ESP32: 0°=125, 180°=250
int mapServo(int ang) {
  return map(ang, 0, 180, 125, 250);
}
 
void setup() {
  Serial.begin(115200);
  SerialBT.begin("bandida");
 
  // asignar PWM
  ledcAttach(SERVO_ARRIBA, 50, 8);
  ledcAttach(SERVO_ABAJO, 50, 8);
  ledcAttach(SERVO_DERECHA, 50, 8);
  ledcAttach(SERVO_IZQUIERDA, 50, 8);
 
  t_bef = millis();
}
 
void loop() {
 
  // ==========================================
  // RECIBIR COORDENADAS: "X Y"
  // ==========================================
  if (SerialBT.available()) {
 
    String line = SerialBT.readStringUntil('\n');
 
    int spaceIndex = line.indexOf(' ');
    if (spaceIndex == -1) return; // formato incorrecto
 
    int errorX = line.substring(0, spaceIndex).toInt();
    int errorY = line.substring(spaceIndex+1).toInt();
 
    // limitar rangos
    errorX = constrain(errorX, -320, 320);
    errorY = constrain(errorY, -240, 240);
 
    // ==========================================
    // PID SOLO EN X (puedo activarte el de Y si quieres)
    // ==========================================
    t_now = millis();
    dt = (t_now - t_bef);
 
    float P = kp * errorX;
    integralX += errorX * dt;
    float I = ki * integralX;
    float D = kd * ((errorX - errorX_ant) / dt);
 
    float pidX = P + I + D;
 
    errorX_ant = errorX;
    t_bef = t_now;
 
    // ==========================================
    // MOVER SERVOS SEGÚN X Y Y (con PID en X)
    // ==========================================
 
    int servoArriba =  90 - errorY / 10;
    int servoAbajo =   90 + errorY / 10;
 
    // PID en X
    int servoDerecha =  90 + pidX / 10;
    int servoIzquierda = 90 - pidX / 10;
 
    // límites seguros
    servoArriba = constrain(servoArriba, 30, 150);
    servoAbajo = constrain(servoAbajo, 30, 150);
    servoDerecha = constrain(servoDerecha, 30, 150);
    servoIzquierda = constrain(servoIzquierda, 30, 150);
 
    // Escritura PWM
    ledcWrite(SERVO_ARRIBA, mapServo(servoArriba));
    ledcWrite(SERVO_ABAJO, mapServo(servoAbajo));
    ledcWrite(SERVO_DERECHA, mapServo(servoDerecha));
    ledcWrite(SERVO_IZQUIERDA, mapServo(servoIzquierda));
 
    Serial.println("Servo R=" + String(servoDerecha) + " L=" + String(servoIzquierda));
  }
 
  delay(15);
}
```


