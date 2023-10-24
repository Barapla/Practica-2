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

# Practica 2.4 Elabore una solución con Pico W (oled display) + botón y ChatGTP

## Codigo

```
    import json
    import network
    import time
    import urequests
    import ssd1306
    
    # Internal libs
    i2c = machine.I2C(0, sda=machine.Pin(0), scl=machine.Pin(1), freq=400000)
    oled = ssd1306.SSD1306_I2C(128, 64, i2c)
    button = machine.Pin(14, machine.Pin.IN, machine.Pin.PULL_UP)
    
    def chat_gpt(ssid, password, endpoint, api_key, model, prompt, max_tokens):
        """
            Description: This is a functionto hit chat gpt api and get
                a response.
                
            Parameters:
            
            ssid[str]: Totalplay-5G-e9b8
            password[str]: zrSk8CQNfSEq8aFR
            endpoint[str]: API endpoint
        """
        
        wlan = network.WLAN(network.STA_IF)
        wlan.active(True)
        wlan.connect(ssid, password)
        
        max_wait = 10
        while max_wait > 0:
            if wlan.status() < 0 or wlan.status() >= 3:
                break
            max_wait -= 1
            print('waiting for connection...')
            time.sleep(1)
            
        if wlan.status() != 3:
            print(wlan.status())
            raise RuntimeError('network connection failed')
        else:
            print('connected')
            print(wlan.status())
            status = wlan.ifconfig()
            
        headers = {'Content-Type': 'application/json',
                   "Authorization": "Bearer " + api_key}
        data = { "model": model,
                 "prompt": prompt,
                 "max_tokens": max_tokens}
        
        print("Attempting to send Prompt")
        r = urequests.post("https://api.openai.com/v1/{}".format(endpoint),
                           json=data,
                           headers=headers)
        
        if r.status_code >= 300 or r.status_code < 200:
            print("There was an error with your request \n" +
                  "Response Status: " + str(r.text))
        else:
            print("Success")
            response_data = json.loads(r.text)
            completion = response_data["choices"][0]["text"]
            print(completion)
            
            oled.fill(0)
            
            max_chars_per_line = 15
            lines = [completion[i:i+max_chars_per_line] for i in range(0, len(completion), max_chars_per_line)]
            
            for i, line in enumerate(lines):
                oled.text(line, 0, i * 10)  # Usar un espaciado vertical de 10 píxeles
    
            oled.show()
            
        r.close()
        
    def button_callback(p):
        if not button.value():
            print("Button pressed. Sending prompt...")
            # Mostrar "Sending prompt" en la pantalla OLED
            oled.fill(0)  # Borra la pantalla
            oled.text("Sending prompt", 0, 0)
            oled.show()
            
            chat_gpt("Totalplay-2.4G-e9b8",
                     "zrSk8CQNfSEq8aFR",
                     "completions",
                     "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",
                     "text-davinci-003",
                     "Dame el nombre de un midlaner de league of legends. ",
                     100)
            
    # Configura una interrupción en el botón para llamar a button_callback cuando se presiona el botón
    button.irq(trigger=machine.Pin.IRQ_FALLING, handler=button_callback)
    
    while True:
        # Tu código principal puede continuar aquí
        pass
```

## Imagenes

![image](https://github.com/Barapla/Practica-2/assets/89622488/e20cd68f-b3df-48c5-a8f2-8d78929783b5)

![image](https://github.com/Barapla/Practica-2/assets/89622488/04680309-14ad-4f57-9cc0-38fa05cd0b3f)

