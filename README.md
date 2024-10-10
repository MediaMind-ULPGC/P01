# Práctica 1: Procesamiento de Imágenes, audio y vídeo.

## 1a. Hacer un programa que cargue una imagen y que obtenga y muestre el valor RGB de cada píxel de la imagen sobre el que esté el cursor del ratón.
Este programa carga una imagen y permite obtener y visualizar el valor RGB de los píxeles de la imagen cuando el cursor del ratón pasa sobre ellos. La información de color (valores RGB) se muestra en pantalla junto al cursor.

**Flujo del programa**:
1. **Cargar la imagen**
```python
img = cv.imread('./images/colores.jpg', cv.IMREAD_COLOR)
img_original = img.copy()
```
2. **Detectar movimiento del ratón**: Se define una función mouse_event que detecta el evento de movimiento del ratón (`cv.EVENT_MOUSEMOVE`). Cada vez que el ratón se mueve sobre la imagen, la posición del cursor se utiliza para obtener el valor de color del píxel (en formato BGR).
```python
def mouse_event(event, x, y, flags, param):
    if event == cv.EVENT_MOUSEMOVE:
        img[:] = img_original.copy()
        bgr = img[y, x]
        ...
```
3. **Mostrar el valor RGB** en el cuadro de texto al lado del ratón y posteriormente volver a mostrar la imagen original con la finalidad de que el texto no se quede permanente en la imagen.
```python
        text = f'RGB ({bgr[2]}, {bgr[1]}, {bgr[0]})'
        cv.rectangle(img, (x, y + 5), (x + 170, y - 15), (0, 0, 0), -1)
        cv.putText(img, text, (x, y), cv.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1, cv.LINE_AA)   

    cv.imshow('colores', img)
```
4. **Mostrar la imagen y gestionar los eventos**: Se muestra la imagen en una ventana llamada colores y se establece la función de callback para manejar los eventos del ratón. El programa espera indefinidamente hasta que el usuario cierre la ventana.
```python
cv.imshow('colores', img)
cv.setMouseCallback('colores', mouse_event)
cv.waitKey(0)
cv.destroyAllWindows()
```

## 1b. Hacer un programa para dibujar las siguientes primitivas con el ratón.
1. **Configuración inicial**: El programa define varias variables clave:
- `drawing`: Un indicador que detecta si el usuario está dibujando.
- `mode`: El tipo de figura geométrica a dibujar inicial, que es 'line'.
- `start_point`: El punto donde se hace clic para empezar a dibujar.
- `color`: Color de la figura en formato RGB (inicialmente negro).
- `thickness`: El grosor de la línea de la figura.
2. **Dibujar formas con el ratón**: Se utiliza una función de callback, `draw_shape`, para gestionar los eventos del ratón. Cuando se hace clic con el botón izquierdo del ratón, comienza el dibujo de la forma seleccionada (línea, rectángulo o círculo). Mientras se mueve el ratón, la figura se va actualizando visualmente, y al soltar el botón del ratón, la forma queda dibujada de forma permanente sobre el lienzo.
```python
def draw_shape(event, x, y, flags, param):
    if event == cv.EVENT_LBUTTONDOWN:
        drawing = True
        start_point = (x, y)
    ...
    elif event == cv.EVENT_LBUTTONUP:
        drawing = False
        ...
```
3. **Cambiar color y grosor**: Se implementan barras deslizantes (Trackbars) para que el usuario pueda ajustar los valores de los canales RGB (para cambiar el color) y el grosor de la línea. Estos valores se actualizan dinámicamente mientras el programa está corriendo.
```python
cv.createTrackbar('R','image',0,255,nothing)
cv.createTrackbar('G','image',0,255,nothing)
cv.createTrackbar('B','image',0,255,nothing)
cv.createTrackbar('Thickness', 'image', 1, 5, nothing)
```
4. **Cambiar de forma y borrar el lienzo:**
- Cambiar de forma: Pulsando la tecla 'm', el usuario puede cambiar entre las distintas primitivas geométricas: línea, rectángulo o círculo.
- Borrar el lienzo: Pulsando la tecla 'd', el lienzo se restablece a su estado original (blanco).
- Salir del programa: Pulsando la tecla 'q', se cierra la ventana del programa.
```python
if key == ord('m'):
    mode = 'line' if mode == 'circle' else 'rectangle' if mode == 'line' else 'circle'
elif key == ord('q'):
    break
elif key == ord('d'):
    img[:] = img_orig
```

## 2. Mejoras del programa:

1. Permitir crear **figuras rellenas** (seleccionar a través de la interfaz). Para ello se ha creado una variable llamada `filled`.
```python
cv.rectangle(img_copy, start_point, (x, y), color, thickness if not filled else -1)
...
cv.circle(img_copy, start_point, radius, color, thickness if not filled else -1)
...
cv.ellipse(img_copy, start_point, axes, 0, 0, 360, color, thickness if not filled else -1)
...
change_filled(cv.getTrackbarPos('Filled','image'))
...
```
2. Poder **crear otras primitivas** (construir polilíneas y polígonos, elipses…).
- Elipses: Similar a los círculos, pero con ejes diferentes.
- Polilíneas y polígonos: Se dibuja una serie de puntos conectados, pudiendo cerrar el polígono al completar la forma. Como funcionamiento, cada vez que se pulsa el botón, se añade a la lista de puntos el punto y en el caso de los polígonos, se cierra el polígono pulsando la letra 'c'.
```python
def draw_shape(event, x, y, flags, param):
    # Dibujar la figura al hacer click
    if event == cv.EVENT_LBUTTONDOWN:
        ...
        if mode in ['polyline', 'polygon']:
            points.append((x, y))
    # Dibujar la figura mientras se mueve el mouse
    elif event == cv.EVENT_MOUSEMOVE:
        if drawing:
            ...
            elif mode == 'ellipse':
                axes = (abs(x - start_point[0]), abs(y - start_point[1]))
                cv.ellipse(img_copy, start_point, axes, 0, 0, 360, color, thickness if not filled else -1)
            elif mode in ['polyline', 'polygon'] and len(points) > 1:
                cv.polylines(img_copy, [np.array(points)], False, color, thickness)
            ...
    # Dibujar la figura al soltar el click
    elif event == cv.EVENT_LBUTTONUP:
        ...
        elif mode == 'ellipse':
            axes = (abs(x - start_point[0]), abs(y - start_point[1]))
            cv.ellipse(img, start_point, axes, 0, 0, 360, color, thickness if not filled else -1)
        elif mode in ['polyline', 'polygon']:
            cv.polylines(img, [np.array(points)], False, color, thickness)
```
3. **Guardar el dibujo como un vídeo** para ver cómo se realizó paso a paso.
```python
# Crear la ventana del video
height, width, _ = img.shape
video = cv.VideoWriter('drawing.mp4', cv.VideoWriter_fourcc(*'MP4V'), 160, (width, height))
while True:
  ...
  video.write(img)
  ...
video.release()
```
4. **Aportaciones propias**.
- `display_mode`: Función que muestra sobre la imagen el modo en el que se encuentra el usuario cada vez que se cambia el modo.
```python
def display_mode(image):
    font = cv.FONT_HERSHEY_SIMPLEX
    cv.putText(image, f'Mode: {mode}', (10, 30), font, 0.75, (0, 0, 0), 2, cv.LINE_AA)
```
- Permitir **retroceder al estado anterior** del lienzo pulsando la tecla 'r'. Esto se logra creando una lista vacía (`actions`) a la cual se le agrega una imagen cada vez que se edita el lienzo.
```python
...
elif key == ord('r') and actions:
  actions.pop()
  if actions:
    img = actions[-1].copy()
  else:
    img = img_orig.copy()
  cv.imshow('image', img)
...
```
- Permitir **limpiar el lienzo** mediante la tecla d.
```python
elif key == ord('d'):
  img = img_orig.copy()
  cv.imshow('image', img)
  actions = []
```
## Trabajo futuro
Existen diversas oportunidades para mejorar y expandir las funcionalidades del programa. Algunas de las propuestas incluyen:
- Adición de más tipos de figuras: Implementar una variedad más amplia de formas geométricas para enriquecer la experiencia del usuario.
- Captura y almacenamiento de dibujos: Permitir a los usuarios capturar su dibujo actual y guardarlo como una imagen en formatos estándar (por ejemplo, PNG o JPEG).
- Carga y edición de imágenes: Incorporar la opción de cargar una imagen existente como lienzo, brindando la posibilidad de editarla y realizar anotaciones directamente sobre ella.
- Mejora de la interfaz gráfica: Explorar el uso de diferentes bibliotecas o extensiones para desarrollar una interfaz gráfica más intuitiva y visualmente atractiva.
Estas mejoras no solo aumentarán la funcionalidad del programa, sino que también mejorarán la usabilidad y la satisfacción del usuario.
