#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Configuración de la pantalla I2C (Dirección habitual: 0x27 o 0x3F, 16 columnas, 2 filas)
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Configuración de pines del Joystick KY-023
const int pinVRx = A0; // Eje X (Izquierda / Derecha)
const int pinVRy = A1; // Eje Y (Arriba / Abajo)
const int pinSW  = 2;  // Botón integrado en el Joystick

// Posición inicial del carácter en la pantalla (Columna 0-15, Fila 0-1)
int col = 0;
int fila = 0;

// Umbrales de lectura para detectar movimiento del Joystick
const int UMBRAL_ALTO = 700;
const int UMBRAL_BAJO = 300;

// Carácter a mostrar
const char CARACTER = '1';

void setup() {
  // Inicialización de la comunicación I2C y el LCD
  lcd.init();
  lcd.backlight(); // Enciende la luz de fondo del LCD
  
  // Configuración del botón del Joystick como entrada con resistencia Pull-up interna
  pinMode(pinSW, INPUT_PULLUP);

  // Mostrar el carácter en la posición de inicio (0,0)
  lcd.clear();
  lcd.setCursor(col, fila);
  lcd.print(CARACTER);
}

void loop() {
  // Lectura de los potenciómetros del Joystick (Valores entre 0 y 1023)
  int valorX = analogRead(pinVRx);
  int valorY = analogRead(pinVRy);
  int boton  = digitalRead(pinSW);

  // Variable para detectar si ocurrió un movimiento
  bool huboMovimiento = false;

  // 1. REINICIO AL PRESIONAR EL BOTÓN (Pin SW)
  if (boton == LOW) { // Al usar INPUT_PULLUP, presionar el botón devuelve LOW
    col = 0;
    fila = 0;
    huboMovimiento = true;
    delay(200); // Evitar rebotes del pulsador
  } 
  else {
    // 2. MOVIMIENTO EN EJE X (Izquierda / Derecha)
    if (valorX < UMBRAL_BAJO) {
      if (col > 0) {
        col--; // Mover a la izquierda
        huboMovimiento = true;
      }
    } 
    else if (valorX > UMBRAL_ALTO) {
      if (col < 15) {
        col++; // Mover a la derecha
        huboMovimiento = true;
      }
    }

    // 3. MOVIMIENTO EN EJE Y (Arriba / Abajo)
    if (valorY < UMBRAL_BAJO) {
      if (fila > 0) {
        fila--; // Mover hacia arriba
        huboMovimiento = true;
      }
    } 
    else if (valorY > UMBRAL_ALTO) {
      if (fila < 1) {
        fila++; // Mover hacia abajo
        huboMovimiento = true;
      }
    }
  }

  // 4. ACTUALIZAR PANTALLA LCD SOLO SI HUBO MOVIMIENTO
  if (huboMovimiento) {
    lcd.clear();              // Limpiar pantalla
    lcd.setCursor(col, fila); // Establecer nueva posición
    lcd.print(CARACTER);      // Imprimir el carácter
    delay(200);               // Pausa para movimiento fluido
  }
}
