# Practica-2

# Practica 2.1 Display Hola mundo

## Codigo

```
import machine
import ssd1306
from machine import Pin, I2C

# Configura el bus I2C y la dirección del dispositivo OLED
i2c = I2C(scl=Pin(5), sda=Pin(4))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)

# Borra la pantalla
oled.fill(0)
oled.show()

# Muestra "Hola, mundo" en la pantalla OLED
oled.text("Hola, mundo", 0, 0)
oled.show()
```

## Imagenes 

![image](https://github.com/Barapla/Practica-2/assets/89622488/bf9b3b7d-8578-4f31-b600-99f954fbee80)

# Practica 2.2 Display Hora con internet

## Codigo

```
import network
import urequests
import utime
from machine import Pin, I2C
import ssd1306

# Conectar a la red WiFi
def conectar_wifi(ssid, password):
    sta_if = network.WLAN(network.STA_IF)
    if not sta_if.isconnected():
        print('Conectándose a la red WiFi...')
        sta_if.active(True)
        sta_if.connect(ssid, password)
        while not sta_if.isconnected():
            pass
    print('Conexión WiFi exitosa')
    print('Dirección IP:', sta_if.ifconfig()[0])

# Obtener la hora desde un servicio de tiempo en línea
def obtener_hora():
    url = "http://worldtimeapi.org/api/ip"
    response = urequests.get(url)
    datos = response.json()
    return datos['datetime']

# Configurar el display OLED
def configurar_oled():
    i2c = I2C(0, sda=Pin(0), scl=Pin(1))
    oled = ssd1306.SSD1306_I2C(128, 64, i2c)
    return oled

# Mostrar la hora en la pantalla OLED
def mostrar_hora_en_oled(oled, hora):
    oled.fill(0)
    oled.text("Hora actual:", 0, 0)
    oled.text(hora, 0, 16)
    oled.show()

# Programa principal
def main():
    ssid = "TecNM-ITT-Docentes"
    password = "tecnm2022!"

    conectar_wifi(ssid, password)
    oled = configurar_oled()

    while True:
        hora = obtener_hora()
        print("Hora actual:", hora)
        mostrar_hora_en_oled(oled, hora)
        utime.sleep(60)  # Espera 60 segundos antes de actualizar nuevamente

if _name_ == '_main_':
    main()
```

## Imagenes

![image](https://github.com/Barapla/Practica-2/assets/89622488/4e00e4f2-5811-4dc0-900e-55a9b029076e)

