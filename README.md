#  Jeronimo Paredes- Alejandro Quenan- Andres Martinez - Ivan Lasso
#   Se importa las librerias a utlizar
from pyfirmata2 import Arduino
from time import sleep, time
import numpy as np

# Autodetectar el puerto del Arduino
port = Arduino.AUTODETECT
board = Arduino(port)

# Variable para almacenar la energía consumida
energy = 0
# Tiempo de muestreo en segundos (10 milisegundos)
T_s = 0.01

# Número de muestras a tomar para el promedio
N = 100
# Función personalizada para agregar un elemento a un índice específico en un array
def custom_append(arr, item, n):
    arr[n] = item
# Bucle infinito para la adquisición y procesamiento continuo de datos
while True:

    t_i = time()  # Tiempo inicial del bucle

#Configurar el Arduino para muestrear a una frecuencia correspondiente a T_s (10 ms)
    board.samplingOn(1000 * T_s)

# Arrays para almacenar las muestras de corriente y voltaje
    current = np.zeros(N)
    voltage = np.zeros(N)

    t0 = time()  # Tiempo inicial de la adquisición de datos

# Obtención de señales de corriente y voltaje
    for n in range(N):
# Registrar un callback para la lectura de la entrada analógica de corriente
        # y agregar el valor al array 'current' en el índice 'n'
        board.analog[0].register_callback(lambda data: custom_append(current, data * 5, n))
        board.analog[0].enable_reporting()

# Registrar un callback para la lectura de la entrada analógica de voltaje
# y agregar el valor al array 'voltage' en el índice 'n'
        board.analog[1].register_callback(lambda data: custom_append(voltage, data * 5, n))
        board.analog[1].enable_reporting()

        sleep(T_s)  # Esperar para la siguiente muestra

# Deshabilitar la generación de informes para las entradas analógicas
    board.analog[0].disable_reporting()
    board.analog[1].disable_reporting()

# Cálculo de potencia instantanea -activa - Energia
    instantaneous_power = current * voltage
    active_power = instantaneous_power.mean()
    tf = time()
    energy += active_power * (tf - t0) / 3600  # Energía en mWh
    Signal_1_rms = np.sqrt(np.mean(np.square(voltage)))
    Signal_2_rms = np.sqrt(np.mean(np.square(current)))
    apparent_power = Signal_1_rms * Signal_2_rms

# Actualizar la salida de manera eficiente
    print(f'\rActive Power (P): {active_power:.5f} Watts', end='')
    print(f' | Apparent Power (S): {apparent_power:.5f} VA', end='')
    print(f' | Energy (E): {energy:.5f} mWh', end='')
    t_0 = time()
    print(f' | Time: {t_0 - t_i:.5f} s', end="\r")
    # sleep(1)  # Se comentó para evitar retrasos en la actualización de la salida
