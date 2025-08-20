# laser-m01-esp32
Integración de un Módulo de medición de distancia láser de alta presión en una placa de desarrollo ESP32 Wemos Lolin32. 
Módulo Laser Range (M01, 6 pines) con ESP32 (LOLIN32)
Integración de un Módulo de medición de distancia láser de alta presión en una placa de desarrollo ESP32 Wemos Lolin32. 
Materiales: Módulo Laser ranging sensor de 50m de Liancheng Electronics (Aliespress).
Ejemplo mínimo y funcional para leer distancias por UART desde un módulo láser M01 (placa de 6 pines) usando un ESP32 (Wemos LOLIN32).
Teclas: Q (medir), L (laser ON), K (laser OFF), R (reset del módulo).
Hardware
•	Módulo láser M01 (placa 6 pines). Liancheng Electronics (Shenzhen) Co., Ltd. Store) AliExpress
•	ESP32 Wemos LOLIN32 (cualquier ESP32 sirve).
•	Alimentación 3.3 V (mín. ~150 mA recomendados).
•	Cables Dupont / soldadura.
Pinout del módulo (de arriba → abajo)
1.	MIN (3V3) – alimentación 3.3 V
2.	ENA – enable (activo en alto)
3.	GND – masa
4.	RXD – entrada UART del módulo
5.	TXD – salida UART del módulo
6.	NC
Colores del mazo (según fabricante)
•	Rojo = MIN (3V3)
•	Negro = GND
•	Verde = RXD (del módulo)
•	Amarillo = TXD (del módulo)
Ojo: TX del módulo → RX del ESP32, RX del módulo ← TX del ESP32 (cruzado).
Conexión recomendada (ESP32 LOLIN32)
•	Rojo (MIN) → 3V3
•	Negro (GND) → GND
•	Amarillo (TXD módulo) → GPIO33 (RX del ESP32)
•	Verde (RXD módulo) ← GPIO32 (TX del ESP32)
•	ENA (si está accesible) → GPIO5 en HIGH (o directo a 3V3)
Si tu mazo solo saca 4 hilos (rojo/negro/verde/amarillo), ENA suele ir integrado a alto y puedes ignorarlo.
________________________________________
Sketch mínimo (Q/L/K/R)
•	Monitor Serie: 115200 baudios.
•	UART del módulo a 9600 (cambiar a 115200 si tu unidad viene así).
•	Imprime solo metros con 3 decimales cuando llega un frame válido.
•	// ===== M01 (6 pines) – Minimal Q/L/K/R =====
•	// SPDX-License-Identifier: MIT 
•	// Copyright (c) 2025 Tu Nombre
•	// ESP32 Wemos Lolin32
•	// Laser ranging sensor 50m (Liancheng Electronics (Shenzhen) Co., Ltd. Store) AliExpress
•	// LOLIN32/ESP32: TX del módulo (amarillo) -> RX ESP32 (GPIO33)
•	//                RX del módulo (verde)    <- TX ESP32 (GPIO32)
•	//                ENA -> GPIO5 (alto), MIN->3V3, GND->GND
•	// L=Enciende Laser, Q= lectura rápida, K= Apaga Laser, R= reinicia módulo.
•	// Medidas metros y dos decimales
•	
•	#include <Arduino.h>
•	
•	// --- Pines (ajusta si usas otros) ---
•	static const int PIN_RX  = 33;   // Módulo TXD -> RX ESP32
•	static const int PIN_TX  = 32;   // Módulo RXD <- TX ESP32
•	static const int PIN_ENA = 5;    // ENA (HIGH = activo)
•	
•	// --- UART ---
•	HardwareSerial LZR(2);
•	
•	// --- Comandos ---
•	const uint8_t CMD_LASER_ON[]  = {0xAA,0x00,0x01,0xBE,0x00,0x01,0x00,0x01,0xC1};
•	const uint8_t CMD_LASER_OFF[] = {0xAA,0x00,0x01,0xBE,0x00,0x01,0x00,0x00,0xC0};
•	const uint8_t CMD_QUICK[]     = {0xAA,0x00,0x00,0x22,0x00,0x01,0x00,0x00,0x23};
•	const uint8_t CMD_READ_RES[]  = {0xAA,0x80,0x00,0x22,0xA2};
•	
•	// --- Estado ---
•	float lastMeters = NAN;
•	
•	// --- Utilidades ---
•	static inline void enaHigh(){ pinMode(PIN_ENA,OUTPUT); digitalWrite(PIN_ENA,HIGH); }
•	static inline void enaLow(){  pinMode(PIN_ENA,OUTPUT); digitalWrite(PIN_ENA,LOW);  }
•	static inline void powerCycle(){ enaLow(); delay(120); enaHigh(); delay(400); }
•	
•	static inline bool csumOK(const uint8_t* f,int n){
•	  if(n<3) return false; uint32_t s=0; for(int i=1;i<n-1;i++) s+=f[i]; return ((uint8_t)s)==f[n-1];
•	}
•	static inline uint32_t bcd32(const uint8_t* b){
•	  uint32_t v=0; for(int i=0;i<4;i++){ v=v*100 + ((b[i]>>4)&0x0F)*10 + (b[i]&0x0F); } return v;
•	}
•	
•	// Lee durante 'ms' y, si encuentra frame válido (13B), imprime metros y devuelve true
•	bool readMeters(unsigned long ms){
•	  uint8_t f[16]; int pos=0; unsigned long t0=millis(), last=0; bool any=false;
•	  while(millis()-t0 < ms){
•	    while(LZR.available()){
•	      uint8_t x=LZR.read();
•	      if(pos==0 && x!=0xAA) continue;
•	      f[pos++]=x; last=millis();
•	      if(pos==13){
•	        if(f[0]==0xAA && f[4]==0x00 && f[5]==0x04 && csumOK(f,13)){
•	          uint8_t func=f[3];
•	          if(func==0x20 || func==0x21 || func==0x22){
•	            lastMeters = bcd32(&f[6]) / 1000.0f;
•	            Serial.println(lastMeters, 2);
•	            any = true;
•	          }
•	        }
•	        pos=0;
•	      }
•	      if(pos>=13) pos=0;
•	    }
•	    if(pos>0 && last && millis()-last>80){ pos=0; last=0; }
•	  }
•	  return any;
•	}
•	
•	// --- Acciones ---
•	void doLaserOn(){  LZR.write(CMD_LASER_ON,  sizeof(CMD_LASER_ON));  LZR.flush(); }
•	void doLaserOff(){ LZR.write(CMD_LASER_OFF, sizeof(CMD_LASER_OFF)); LZR.flush(); }
•	
•	void doQuick(){
•	  while(LZR.available()) LZR.read();           // limpia RX
•	  LZR.write(CMD_QUICK, sizeof(CMD_QUICK)); LZR.flush();
•	  if(!readMeters(2000)){                       // espera frame
•	    LZR.write(CMD_READ_RES, sizeof(CMD_READ_RES)); LZR.flush(); // intenta último resultado
•	    readMeters(800);
•	  }
•	}
•	
•	void doReset(){                                // R = "borrar/puesta a cero"
•	  powerCycle();                                 // reinicia solo el módulo
•	  while(LZR.available()) LZR.read();            // limpia RX
•	  lastMeters = NAN;
•	  Serial.println("OK RESET");
•	}
•	
•	// --- Setup / Loop ---
•	void setup(){
•	  Serial.begin(115200);
•	  enaHigh();
•	  LZR.begin(9600, SERIAL_8N1, PIN_RX, PIN_TX);
•	
•	  Serial.println("\nM01 listo: Q=medir  L=laser ON  K=laser OFF  R=reset");
•	}
•	
•	void loop(){
•	  if(Serial.available()){
•	    char c = toupper((unsigned char)Serial.read());
•	    if(c=='Q') doQuick();
•	    else if(c=='L') doLaserOn();
•	    else if(c=='K') doLaserOff();
•	    else if(c=='R') doReset();
•	  }
•	  // por si llegan frames (p.ej., continuo activado desde fuera)
•	  if(LZR.available()) readMeters(200);
•	}
•	
________________________________________

Uso
1.	Carga el sketch y abre Monitor Serie a 115200.
2.	Conecta el láser a una pared mate (2–4 m) con fondo oscuro.
3.	Pulsa:
o	L → enciende el láser.
o	Q → lectura rápida (muestra X.XXX).
o	K → apaga el láser.
o	R → reinicia el módulo (corta y restablece ENA).
Nota: el emisor suele ser infrarrojo; el “punto” no se ve a simple vista. Con la cámara del móvil sí se aprecia.
________________________________________
Diagnóstico rápido (si no hay datos en RX)
•	Cables cruzados: Amarillo (TXD módulo) → RX del ESP32; Verde (RXD módulo) ← TX del ESP32.
•	GND común entre módulo y ESP32.
•	ENA alto (GPIO5 HIGH o 3V3).
•	Eco por cable (prueba exprés): Puentea verde↔amarillo en el conector del módulo; lo que envíe el ESP32 debe volver idéntico por RX. Si no hay eco → revisar soldaduras/continuidad en GPIO33/GPIO32 ↔ conector.
•	Si tu unidad viene a 115200, cambia el LZR.begin(9600, …) por 115200.
________________________________________
Seguridad
•	Es IR (≈905–940 nm). No apuntar a ojos ni a superficies reflectantes cercanas.
________________________________________
Agradecimientos
Agradecemos especialmente la colaboración y soporte de Liancheng Electronics (Shenzhen) Co., Ltd. Store, que nos facilitó el esquema de colores del mazo y ayudó a resolver los problemas de conexión RX/TX.
Enlaces de compras: https://www.aliexpress.com/store/1104805174?spm=a2g0o.order_list.order_list_main.8.21ef194dfoSHNu

