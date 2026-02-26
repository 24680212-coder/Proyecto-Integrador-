# 🌊 Escenario Procedural en Blender
### Animación Automatizada con Python (API bpy)

Este repositorio contiene un script de Python desarrollado para **Blender** que genera automáticamente un pasillo sinoidal con geometría dinámica y una cámara animada que recorre la trayectoria.

---

## 📖 Descripción del Proyecto

El script utiliza funciones trigonométricas para calcular una ruta curva y posicionar elementos arquitectónicos a lo largo de ella. Es ideal para entender la manipulación de datos de curvas y restricciones de cámara mediante código.

### Características principales:
* **Generación de Curva NURBS:** Creación física de una ruta guía.
* **Geometría Alternada:** Paredes con materiales y alturas que varían según la posición.
* **Cámara Inteligente:** Uso de la restricción `FOLLOW_PATH` con keyframes automáticos.

---

## 💻 Código Fuente del Script

Pega este código en la pestaña de **Scripting** dentro de Blender:

```python
import bpy
import math

def crear_material(nombre, color_rgb):
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat

def generar_escenario_final():
    # 1. Limpiar la escena
    bpy.ops.object.select_all(action="SELECT")
    bpy.ops.object.delete()

    # 2. Materiales
    mat_pared_a = crear_material("ParedOscura", (0.1, 0, 0.1))
    mat_pared_b = crear_material("ParedDetalle", (0.3, 0.1, 0.8))

    # 3. Parámetros del escenario curvo
    largo_pasillo = 40 
    ancho_pasillo = 4
    frecuencia = 0.2  
    amplitud = 5      

    # 4. Crear la CURVA física
    curva_datos = bpy.data.curves.new('CaminoDatos', type='CURVE')
    curva_datos.dimensions = '3D'
    curva_datos.use_path = True 
    
    objeto_curva = bpy.data.objects.new('CurvaGuia', curva_datos)
    bpy.context.collection.objects.link(objeto_curva)
    
    spline = curva_datos.splines.new('NURBS')
    spline.points.add(largo_pasillo - 1)

    # 5. Ciclo para construir paredes y puntos de curva
    for i in range(largo_pasillo):
        offset_x = math.sin(i * frecuencia) * amplitud
        pos_y = i * 2
        spline.points[i].co = (offset_x, pos_y, 1.5, 1)

        # Paredes
        bpy.ops.mesh.primitive_cube_add(location=(offset_x - ancho_pasillo, pos_y, 1))
        p_izq = bpy.context.active_object
        p_izq.data.materials.append(mat_pared_a if i % 2 == 0 else mat_pared_b)
        if i % 2 != 0: p_izq.scale.z = 1.5

        bpy.ops.mesh.primitive_cube_add(location=(offset_x + ancho_pasillo, pos_y, 1))
        p_der = bpy.context.active_object
        p_der.data.materials.append(mat_pared_a)

    # 6. Configurar la CÁMARA
    bpy.ops.object.camera_add()
    camara = bpy.context.active_object
    bpy.context.scene.camera = camara
    camara.rotation_euler = (0, 0, 0)

    # 7. Restricción FOLLOW PATH
    follow = camara.constraints.new(type='FOLLOW_PATH')
    follow.target = objeto_curva
    follow.use_fixed_location = True
    follow.use_curve_follow = True 
    follow.forward_axis = 'TRACK_NEGATIVE_Z' 
    follow.up_axis = 'UP_Y'

    # 8. Insertar los Keyframes
    cuadros_totales = 300
    bpy.context.scene.frame_end = cuadros_totales

    # Frame 1: Inicio
    bpy.context.scene.frame_set(1)
    follow.offset_factor = 0.0
    follow.keyframe_insert(data_path="offset_factor")

    # Frame 300: Final
    bpy.context.scene.frame_set(cuadros_totales)
    follow.offset_factor = 1.0
    follow.keyframe_insert(data_path="offset_factor")

generar_escenario_final()
