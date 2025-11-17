# Simulación de Túnel de Viento 2D

Proyecto de Modelación y Simulación - Simulación dinámica de flujo de aire usando las ecuaciones de Navier-Stokes

## Descripción del Proyecto

Este proyecto simula un túnel de viento en 2D donde el aire fluye y choca contra diferentes figuras geométricas. La simulación utiliza las ecuaciones de Navier-Stokes, que son ecuaciones matemáticas fundamentales para describir el movimiento de fluidos como el aire y el agua.

La simulación permite visualizar cómo el aire se comporta cuando encuentra obstáculos de diferentes formas: círculos, cuadrados y triángulos. Esto es útil para entender conceptos como la dinámica de fluidos, la formación de vórtices (remolinos) y la resistencia aerodinámica.

## Resultados

El proyecto genera animaciones GIF que muestran el flujo de aire alrededor de tres tipos de obstáculos:

- **Círculo**: [tunel_NavierStokes_circulo.gif](outputs/tunel_NavierStokes_circulo.gif)
- **Cuadrado**: [tunel_NavierStokes_cuadrado.gif](outputs/tunel_NavierStokes_cuadrado.gif)
- **Triángulo**: [tunel_NavierStokes_triangulo.gif](outputs/tunel_NavierStokes_triangulo.gif)

Las visualizaciones muestran:
- Mapa de colores que indica la velocidad del aire (azul claro = más rápido, azul oscuro = más lento)
- Líneas de corriente que muestran la trayectoria del flujo de aire
- El obstáculo en color rojo

## Cómo Funciona la Simulación

### 1. Configuración del Dominio

La simulación crea una malla de puntos (200 x 100) que representa el espacio físico del túnel de viento. Esta malla es donde se calculan las propiedades del fluido en cada paso de tiempo.

Parámetros principales:
- **Dominio**: 4.0 x 2.0 metros dividido en una cuadrícula
- **Viscosidad**: 0.01 m²/s (controla qué tan "espeso" es el fluido)
- **Velocidad de entrada**: 1.0 m/s (velocidad del aire entrando por la izquierda)
- **Número de Reynolds**: ~40 (indica el tipo de flujo - laminar vs turbulento)

### 2. Ecuaciones de Navier-Stokes

Las ecuaciones de Navier-Stokes describen cómo se mueve un fluido considerando:

- **Término convectivo**: Cómo el fluido se transporta a sí mismo (el aire en movimiento arrastra más aire)
- **Término difusivo**: Cómo la viscosidad "suaviza" el movimiento
- **Gradiente de presión**: Cómo las diferencias de presión empujan el fluido

El código calcula estos términos usando derivadas numéricas (diferencias finitas), que aproximan las tasas de cambio en el espacio.

### 3. Método de Proyección (Algoritmo de Chorin)

Para resolver las ecuaciones de Navier-Stokes, el código usa un algoritmo en tres pasos:

**Paso 1 - Velocidad intermedia**:
```python
u_temp = u + dt * (-conv_u + diff_u)
v_temp = v + dt * (-conv_v + diff_v)
```
Calcula cómo cambiaría la velocidad considerando solo la convección y difusión, sin la presión.

**Paso 2 - Resolver presión**:
```python
p = resolver_presion(u_temp, v_temp, p, dx, dy, dt)
```
Resuelve la ecuación de Poisson para encontrar el campo de presión que hace que el fluido sea incompresible (no se comprime ni expande).

**Paso 3 - Corrección de velocidad**:
```python
u_new = u_temp - dt / RHO * dp_dx
v_new = v_temp - dt / RHO * dp_dy
```
Ajusta la velocidad usando el gradiente de presión calculado para garantizar que el fluido mantenga su volumen.

### 4. Condiciones de Frontera

Para que la simulación sea realista, se aplican condiciones en los bordes:

- **Entrada (izquierda)**: El aire entra con velocidad constante (1.0 m/s)
- **Salida (derecha)**: El aire puede salir libremente (gradiente cero)
- **Paredes superior e inferior**: No hay deslizamiento (velocidad = 0)
- **Obstáculo**: El aire no puede atravesarlo (velocidad = 0 en su superficie)

### 5. Creación de Obstáculos

El código incluye una función que crea diferentes formas geométricas, por ejemplo:

**Círculo**:
```python
if (j - cx)**2 + (i - cy)**2 <= radius**2:
    obstacle[i, j] = True
```
Usa la ecuación de un círculo para marcar qué celdas están dentro del obstáculo.

**Cuadrado**:
```python
obstacle[y_start:y_end, x_start:x_end] = True
```
Simplemente marca un rectángulo de celdas.

**Triángulo**:
```python
width_at_row = int(base * dist_from_top / height)
obstacle[i, cx : cx + width_at_row + 1] = True
```
Crea un triángulo que se ensancha gradualmente desde la punta.

### 6. Bucle de Simulación

El programa ejecuta 3000 pasos de tiempo, donde en cada paso:

1. Calcula los términos convectivo y difusivo
2. Obtiene velocidades intermedias
3. Resuelve el campo de presión
4. Corrige las velocidades
5. Aplica condiciones de frontera
6. Cada 30 pasos, guarda una imagen para la animación

### 7. Visualización

La visualización combina varios elementos:

- **Contornos de velocidad**: Mapa de colores que muestra la magnitud de la velocidad
- **Líneas de corriente**: Muestran la dirección del flujo usando las componentes u y v de la velocidad
- **Obstáculo**: Se dibuja en rojo para distinguirlo claramente

Las imágenes se guardan como frames y al final se combinan en un GIF animado.

## Características Técnicas

- **Método numérico**: Diferencias finitas con esquema explícito en tiempo
- **Resolución de presión**: Método iterativo de Jacobi (50 iteraciones por paso)
- **Paso de tiempo**: 0.001 segundos (debe ser pequeño para estabilidad numérica)
- **Duración de simulación**: 3 segundos de tiempo físico simulado

## Archivos del Proyecto

- [tunel_v1.ipynb](tunel_v1.ipynb) - Notebook con el código completo de la simulación
- `outputs/` - Carpeta con las animaciones generadas

## Requisitos

- Python 3.x
- NumPy - cálculos numéricos
- Matplotlib - visualización
- PIL (Pillow) - creación de GIFs

## Cómo Ejecutar

1. Abrir el notebook [tunel_v1.ipynb](tunel_v1.ipynb)
2. Modificar el parámetro `GEOMETRIA` para elegir el tipo de obstáculo ('circulo', 'cuadrado', 'triangulo')
3. Ejecutar todas las celdas
4. Las animaciones se guardarán en la carpeta `outputs/`

## Conceptos Físicos Observables

Al ejecutar la simulación, se pueden observar fenómenos reales de dinámica de fluidos:

- **Aceleración del flujo**: El aire acelera al pasar alrededor del obstáculo (efecto Venturi)
- **Formación de vórtices**: Detrás del obstáculo se forman remolinos
- **Zona de baja presión**: Se crea una zona de succión detrás del obstáculo
- **Capa límite**: El fluido cerca del obstáculo se mueve más lento debido a la viscosidad

## Limitaciones

- Simulación en únicamente en 2D 
- Números de Reynolds bajos (flujo laminar, no turbulento)
- Resolución limitada por capacidad computacional
- Método explícito requiere pasos de tiempo pequeños
