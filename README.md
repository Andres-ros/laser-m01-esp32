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
4.	GND – masa
5.	RXD – entrada UART del módulo
6.	TXD – salida UART del módulo
7.	NC
8.	
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
Si tu mazo solo saca 4 hilos (rojo/negro/verde/amarillo), ENA suele ir integrado a alto debes conectarlo.
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

