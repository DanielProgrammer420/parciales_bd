¡Excelente idea! Analizar específicamente las restricciones `CHECK` es una de las mejores formas de prepararse, porque son la herramienta principal para implementar la **lógica de negocio** directamente en la base de datos.

A continuación, te presento una lista completa de todas las restricciones `CHECK` utilizadas o inferidas de los parciales y casos prácticos, con su script `ALTER TABLE` y una explicación clara de su propósito.

---

### **Guía de Restricciones `CHECK` para el Parcial**

Una `CONSTRAINT CHECK` es una regla que define qué valores son válidos para una o más columnas en una tabla. La base de datos verificará esta regla cada vez que se intente insertar o modificar una fila. Si la condición no se cumple, la operación es rechazada.

**Sintaxis General:**

```sql
ALTER TABLE nombre_de_la_tabla
ADD CONSTRAINT nombre_descriptivo_del_check
CHECK (condicion_logica);
```

---

### **Lista Completa de `CHECK`s de los Exámenes**

Aquí están todos los tipos de validaciones que aparecieron, con su implementación y explicación.

#### **1. Restricciones sobre Rangos Numéricos**

*   **Propósito:** Asegurar que un valor numérico se encuentre dentro de un rango lógico.
*   **Ejemplo del Parcial (Marketplace 2024):** "La calificación en la reseña solo puede asumir valores enteros de 1 a 10".
*   **Implementación:**
    ```sql
    ALTER TABLE Reseña
    ADD CONSTRAINT chk_reseña_calificacion_rango
    CHECK (calificacion >= 1 AND calificacion <= 10);
    ```
*   **Explicación:** Esta regla impide que se inserten calificaciones como 0, 11 o valores negativos. La base de datos garantiza que la calificación siempre será válida según la regla del negocio.

#### **2. Restricciones sobre Longitud de Texto**

*   **Propósito:** Validar que un campo de texto (como DNI o CUIT) tenga una longitud específica.
*   **Ejemplo del Parcial (Veterinaria 2023):** "La longitud del campo 'dni' debe ser igual o menor a 8 caracteres".
*   **Implementación:**
    ```sql
    ALTER TABLE Dueño
    ADD CONSTRAINT chk_dueño_dni_longitud
    CHECK (LEN(dni) <= 8);
    ```
*   **Explicación:** La función `LEN()` cuenta los caracteres de la cadena. Este `CHECK` rechazará cualquier intento de insertar un DNI con 9 o más caracteres, manteniendo la consistencia del formato.

#### **3. Restricciones sobre Valores de una Lista (Enumeraciones)**

*   **Propósito:** Limitar los valores de una columna a un conjunto predefinido de opciones.
*   **Ejemplo del Parcial (Institución Educativa 2023):** "en la columna 'tipo de movimiento' de la tabla 'asistencia' solo se pueden ingresar los caracteres 'I' y 'E'".
*   **Implementación:**
    ```sql
    ALTER TABLE Asistencia
    ADD CONSTRAINT chk_asistencia_tipo_movimiento_valores
    CHECK (tipo_movimiento IN ('I', 'E'));
    ```
*   **Explicación:** El operador `IN` verifica si el valor de la columna es uno de los de la lista. Esto es mucho más robusto que usar una tabla de catálogo si las opciones son fijas y nunca cambiarán. Impide que se guarden valores como 'Entrada' o 'i' (minúscula).

#### **4. Restricciones sobre Fechas y Edades**

*   **Propósito:** Validar la lógica temporal, como rangos de edad o la coherencia entre fechas.
*   **Ejemplo del Parcial (Marketplace 2024 / Gimnasio):** "La edad del vendedor al momento de darlo de alta debe ser igual o mayor a 18 años".
*   **Implementación:**
    ```sql
    ALTER TABLE Vendedor
    ADD CONSTRAINT chk_vendedor_mayoria_edad
    CHECK (DATEDIFF(year, fecha_nacimiento, GETDATE()) >= 18);
    ```
*   **Explicación:** Esta es una de las más potentes.
    *   `GETDATE()`: Obtiene la fecha y hora actual del servidor.
    *   `DATEDIFF(year, fecha_inicio, fecha_fin)`: Calcula la diferencia en años entre dos fechas.
    *   La restricción impide registrar a un vendedor si no ha cumplido los 18 años al momento del registro.

#### **5. Restricciones de Coherencia entre Columnas**

*   **Propósito:** Asegurar que la relación lógica entre dos o más columnas dentro de la misma fila sea siempre correcta.
*   **Ejemplo del Parcial (Estacionamiento 2024):** "La fecha de salida del vehículo debe ser igual o superior a la fecha de ingreso".
*   **Implementación:**
    ```sql
    ALTER TABLE Registro_Estacionamiento
    ADD CONSTRAINT chk_registro_fechas_logicas
    CHECK (fecha_hora_salida IS NULL OR fecha_hora_salida >= fecha_hora_ingreso);
    ```
*   **Explicación:**
    *   Esta regla impide registrar una salida que ocurrió antes que la entrada, lo cual es lógicamente imposible.
    *   La parte `fecha_hora_salida IS NULL OR ...` es crucial. Permite que la fila se inserte cuando el vehículo recién ingresa (y la fecha de salida es nula), pero aplica la validación solo cuando se intenta actualizar la fecha de salida.

#### **6. Restricciones para Evitar Valores "Malos"**

*   **Propósito:** Una aplicación simple de `CHECK` para evitar valores ilógicos, como números negativos donde no corresponden.
*   **Ejemplo del Caso Práctico (TechSolutions):** El cupo de una clase o la capacidad de un camión no pueden ser negativos.
*   **Implementación:**
    ```sql
    ALTER TABLE Clase
    ADD CONSTRAINT chk_clase_cupo_positivo
    CHECK (cupo_maximo > 0);
    ```
*   **Explicación:** Asegura que solo se puedan registrar cupos positivos. Es una regla de integridad de datos muy básica pero fundamental.

---

### **Consejos para el Parcial**

1.  **Crea una "Checklist de Restricciones":** Mientras lees el enunciado, haz una lista de todas las reglas de negocio. Al final, revisa tu script y asegúrate de que cada regla de esa lista esté implementada con un `CONSTRAINT` (`UNIQUE`, `CHECK`, etc.).
2.  **Nombra tus `CHECK`s:** Siempre usa `CONSTRAINT nombre_del_check CHECK (...)`. Un nombre claro como `chk_vendedor_mayoria_edad` te ayudará a ti y al profesor a entender el propósito de la regla y a depurar errores fácilmente.
3.  **No Olvides el `NULL`:** En las comparaciones de fechas (como la de salida >= ingreso), recuerda siempre contemplar el caso en que una de las fechas pueda ser `NULL` para no impedir inserciones válidas.
4.  **Usa las Funciones del Motor:** No tengas miedo de usar funciones de SQL Server como `LEN()`, `DATEDIFF()`, y `GETDATE()` dentro de tus `CHECK`s. Demuestra un conocimiento más profundo del diseño físico.
