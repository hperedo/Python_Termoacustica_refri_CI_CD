# Python CI/CD Pipeline
Este repositorio implementa un pipeline de Integración Continua (CI) y Despliegue Continuo (CD) para código Python utilizando herramientas como GitHub Actions o GitLab CI. Este README explica cómo configurar y utilizar el pipeline.

```python
import numpy as np
import matplotlib.pyplot as plt

# Definición de parámetros
RM = 3.53E-3
M = 0.0045
SSUP = 2.6E4
RHO = 1.68
C = 1000
S = 1.521E-3
BL = 8
Q = 15
FO = 800
A = 7.13E-5
VOL = 2.715E-6
RG = 0
LE = 0
RE = 8
I = 1
L = 0.625

# Constantes
PI = np.pi
SVOL = (A * A * RHO * C * C) / VOL
WO = 2 * PI * FO
ALPHA = WO / (2 * Q * C)

# Rango de frecuencias
f0 = FO
frequencies = np.linspace(f0 / 4, (3 * f0) / 2, 300)
VOLTAGE = np.zeros(300)
EFFICIENCY = np.zeros(300)

for J in range(300):
    F = frequencies[J]
    W = 2 * PI * F
    K = W / C
    LCOIL = W * LE
    
    # Cálculo de las partes real e imaginaria de la impedancia mecánica ZM
    REZM = RM + ((RHO * C * S * ALPHA * np.cos(K * L) * np.sin(K * L)) + 
        (K * RHO * C * S * np.sinh(ALPHA * L) * np.cosh(ALPHA * L))) / \
        (K * (1 + (ALPHA / K)**2) * (((np.sin(K * L))**2 * (np.cosh(ALPHA * L))**2) + 
        ((np.cos(K * L))**2 * (np.sinh(ALPHA * L))**2)))

    IMZM = (W * M) - (SSUP / W) - (SVOL / W) + ((RHO * C * S * ALPHA * np.sinh(ALPHA * L) * 
        np.cosh(ALPHA * L)) - (K * RHO * C * S * np.cos(K * L) * np.sin(K * L))) / \
        (K * (1 + ALPHA / K)**2 * (((np.sin(K * L))**2 * (np.cosh(ALPHA * L))**2) + 
        ((np.cos(K * L))**2 * (np.sinh(ALPHA * L))**2)))
    
    ZM = complex(REZM, IMZM)
    
    # Impedancia total
    ZE = (BL**2) / ZM
    ZCOIL = complex(RE, LCOIL)
    ZTOT = RG + ZCOIL + ZE
    
    # Voltaje y eficiencia acústica
    V = I * ZTOT
    FORCE = BL * I
    U = FORCE / ZM
    MAGU = np.abs(U)
    
    # Potencias y eficiencia
    P = (ZTOT * U) / S
    PU = 0.5 * np.real(P * np.conj(U)) * S
    PWR = 0.5 * np.real(I * np.conj(V))
    PUPWR = PU / PWR
    MAGV = np.abs(V)
    
    # Almacenar valores
    VOLTAGE[J] = MAGV
    EFFICIENCY[J] = PUPWR

# Gráfica Voltaje vs Frecuencia
plt.figure(figsize=(10, 8))

plt.subplot(2, 1, 1)
plt.plot(frequencies, VOLTAGE, linewidth=2)
plt.xlabel('Frecuencia (Hz)')
plt.ylabel('Voltaje (V)')
plt.title('Voltaje vs Frecuencia')
plt.grid(True)

# Gráfica Eficiencia Acústica vs Frecuencia
plt.subplot(2, 1, 2)
plt.plot(frequencies, EFFICIENCY, linewidth=2)
plt.xlabel('Frecuencia (Hz)')
plt.ylabel('Eficiencia Acústica')
plt.title('Eficiencia Acústica vs Frecuencia')
plt.grid(True)

plt.tight_layout()
plt.show()
```
# Pipeline CI/CD en GitHub Actions
Para configurar un pipeline de CI/CD utilizando GitHub Actions, puedes crear el archivo .github/workflows/python-ci.yml dentro de tu repositorio.

```python
name: Python CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Check out the repository
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install matplotlib numpy

    - name: Run tests
      run: |
        python -m unittest discover tests
```
# Estructura del Proyecto
Debes estructurar tu proyecto con una carpeta de código y otra de pruebas, por ejemplo:

```bash
/src        # Código fuente
/tests      # Pruebas unitarias
.github/    # GitHub Actions configuración
```

# Pruebas Unitarias
Puedes crear un archivo de pruebas unitarias test_calculos.py dentro de la carpeta tests para validar que tus funciones se ejecutan correctamente:

```python
import unittest
import numpy as np

class TestCalculos(unittest.TestCase):
    def test_frequencies(self):
        f0 = 800
        frequencies = np.linspace(f0 / 4, (3 * f0) / 2, 300)
        self.assertEqual(len(frequencies), 300)
        self.assertAlmostEqual(frequencies[0], 200)

if __name__ == '__main__':
    unittest.main()
```
# Cómo funciona el pipeline CI/CD:
- Instalación de Dependencias: El pipeline usa GitHub Actions para configurar Python y las bibliotecas necesarias (numpy, matplotlib).
- Ejecución de Pruebas: Una vez configurado, ejecuta las pruebas definidas en la carpeta tests. Si alguna prueba falla, el pipeline reporta el error.

Con esta configuración, cada vez que se realice un push o pull_request, GitHub Actions verificará que el código funcione correctamente ejecutando el script y las pruebas.

