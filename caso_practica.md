¡Por supuesto! Basado en el análisis del caso práctico y en todas las pautas y mejores prácticas que los profesores han enfatizado en las clases (normalización, claves compuestas, nombrado de `constraints`, `snake_case`, etc.), he creado el script SQL completo que resuelve el problema.

Este script está diseñado para ser la "solución ideal" que cumple con todos los criterios de evaluación del parcial.

---

### **El Script SQL Completo (Solución del Caso de Práctica)**

```sql
-- =============================================================================
-- Script de Solución: Caso de Práctica - Gestión de Proyectos (TechSolutions)
-- Descripción: Este script crea la estructura de tablas y restricciones para
-- gestionar proyectos, tareas y asignaciones de empleados, aplicando
-- normalización hasta 3FN y todas las buenas prácticas de diseño físico.
-- =============================================================================

-- =============================================================================
-- SECCIÓN 1: CREACIÓN Y USO DE LA BASE DE DATOS
-- Se asegura que el script sea autocontenido y se ejecute en un entorno limpio.
-- =============================================================================
-- Si la base de datos ya existe, la elimina para una ejecución limpia
IF DB_ID('techsolutions_db') IS NOT NULL
    DROP DATABASE techsolutions_db;
GO

-- Crea la nueva base de datos
CREATE DATABASE techsolutions_db;
GO

-- Selecciona la base de datos recién creada para trabajar en ella
USE techsolutions_db;
GO

-- =============================================================================
-- SECCIÓN 2: CREACIÓN DE TABLAS DE CATÁLOGO Y MAESTRAS (DDL)
-- Se crean primero las tablas que no tienen dependencias (claves foráneas)
-- o que sirven como "listas de opciones".
-- =============================================================================

-- Tabla de catálogo para los roles que un empleado puede asumir en un proyecto
CREATE TABLE rol (
    id_rol INT IDENTITY(1,1),
    nombre_rol VARCHAR(50) NOT NULL,

    CONSTRAINT pk_rol PRIMARY KEY (id_rol),
    CONSTRAINT uq_rol_nombre UNIQUE (nombre_rol)
);
GO

-- Tabla de catálogo para los posibles estados de una tarea
CREATE TABLE estado_tarea (
    id_estado INT IDENTITY(1,1),
    nombre_estado VARCHAR(50) NOT NULL,

    CONSTRAINT pk_estado_tarea PRIMARY KEY (id_estado),
    CONSTRAINT uq_estado_tarea_nombre UNIQUE (nombre_estado)
);
GO

-- Tabla para almacenar la información de los empleados
CREATE TABLE empleado (
    id_empleado INT IDENTITY(1,1),
    nombre_empleado VARCHAR(150) NOT NULL,
    correo_electronico VARCHAR(100) NOT NULL,

    CONSTRAINT pk_empleado PRIMARY KEY (id_empleado),
    CONSTRAINT uq_empleado_correo UNIQUE (correo_electronico)
);
GO

-- Tabla para almacenar la información de los proyectos
CREATE TABLE proyecto (
    id_proyecto INT IDENTITY(1,1),
    nombre_proyecto VARCHAR(100) NOT NULL,
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE NULL, -- Opcional, por lo tanto permite NULL

    CONSTRAINT pk_proyecto PRIMARY KEY (id_proyecto),
    CONSTRAINT uq_proyecto_nombre UNIQUE (nombre_proyecto)
);
GO

-- =============================================================================
-- SECCIÓN 3: CREACIÓN DE TABLAS DEPENDIENTES Y ASOCIATIVAS (DDL)
-- =============================================================================

-- Tabla intermedia para resolver la relación N:M entre Empleado y Proyecto
-- Registra la asignación de un empleado a un proyecto con un rol específico.
CREATE TABLE asignacion_empleado_proyecto (
    id_empleado INT NOT NULL,
    id_proyecto INT NOT NULL,
    id_rol INT NOT NULL,
    fecha_inicio_participacion DATE NOT NULL,
    fecha_fin_participacion DATE NULL, -- Opcional

    -- La PK compuesta garantiza que un empleado solo pueda aparecer una vez en un proyecto.
    CONSTRAINT pk_asignacion_empleado_proyecto PRIMARY KEY (id_empleado, id_proyecto)
);
GO

-- Tabla para las tareas de un proyecto (Entidad Débil)
-- Una tarea no puede existir sin un proyecto al que pertenecer.
CREATE TABLE tarea (
    id_proyecto INT NOT NULL,      -- Parte de la PK, FK a Proyecto
    numero_orden INT NOT NULL,   -- Parte de la PK, clave parcial
    descripcion VARCHAR(500) NOT NULL,
    fecha_inicio DATE NOT NULL,
    fecha_fin DATE NOT NULL,
    id_estado INT NOT NULL,

    -- La PK compuesta identifica unívocamente una tarea DENTRO de un proyecto.
    CONSTRAINT pk_tarea PRIMARY KEY (id_proyecto, numero_orden)
);
GO

-- =============================================================================
-- SECCIÓN 4: DEFINICIÓN DE CLAVES FORÁNEAS Y RESTRICCIONES (DDL - ALTER TABLE)
-- Se definen las relaciones de integridad referencial y las reglas de negocio.
-- =============================================================================

ALTER TABLE asignacion_empleado_proyecto
ADD CONSTRAINT fk_asignacion_empleado FOREIGN KEY (id_empleado) REFERENCES empleado(id_empleado);
GO

ALTER TABLE asignacion_empleado_proyecto
ADD CONSTRAINT fk_asignacion_proyecto FOREIGN KEY (id_proyecto) REFERENCES proyecto(id_proyecto);
GO

ALTER TABLE asignacion_empleado_proyecto
ADD CONSTRAINT fk_asignacion_rol FOREIGN KEY (id_rol) REFERENCES rol(id_rol);
GO

ALTER TABLE tarea
ADD CONSTRAINT fk_tarea_proyecto FOREIGN KEY (id_proyecto) REFERENCES proyecto(id_proyecto);
GO

ALTER TABLE tarea
ADD CONSTRAINT fk_tarea_estado FOREIGN KEY (id_estado) REFERENCES estado_tarea(id_estado);
GO

-- Restricción CHECK para asegurar la lógica de las fechas de participación
ALTER TABLE asignacion_empleado_proyecto
ADD CONSTRAINT chk_asignacion_fechas_logicas
CHECK (fecha_fin_participacion IS NULL OR fecha_fin_participacion >= fecha_inicio_participacion);
GO

-- Restricción CHECK para asegurar la lógica de las fechas de las tareas
ALTER TABLE tarea
ADD CONSTRAINT chk_tarea_fechas_logicas
CHECK (fecha_fin >= fecha_inicio);
GO

-- =============================================================================
-- SECCIÓN 5: LOTE DE DATOS DE PRUEBA (CASOS VÁLIDOS)
-- =============================================================================
PRINT 'Insertando datos de prueba válidos...';

-- Catálogos
INSERT INTO rol (nombre_rol) VALUES ('Director de Proyecto'), ('Analista Funcional'), ('Desarrollador'), ('Tester'), ('Diseñador');
INSERT INTO estado_tarea (nombre_estado) VALUES ('Pendiente'), ('En progreso'), ('Finalizada');

-- Entidades Principales
INSERT INTO empleado (nombre_empleado, correo_electronico) VALUES ('Juan Pérez', 'juan.perez@tech.com'), ('Ana Martínez', 'ana.martinez@tech.com');
INSERT INTO proyecto (nombre_proyecto, fecha_inicio, fecha_fin) VALUES ('Sistema de Facturación', '2023-01-01', '2023-06-30');

-- Asignaciones (N:M)
INSERT INTO asignacion_empleado_proyecto (id_empleado, id_proyecto, id_rol, fecha_inicio_participacion)
VALUES (1, 1, 1, '2023-01-01'), -- Juan es Director en el proyecto de Facturación
       (2, 1, 3, '2023-01-15'); -- Ana es Desarrolladora en el proyecto de Facturación

-- Tareas (Entidad Débil)
INSERT INTO tarea (id_proyecto, numero_orden, descripcion, fecha_inicio, fecha_fin, id_estado)
VALUES (1, 1, 'Análisis de requerimientos', '2023-01-01', '2023-01-31', 3), -- Tarea 1 del proyecto 1, Finalizada
       (1, 2, 'Diseño de la base de datos', '2023-02-01', '2023-02-28', 2); -- Tarea 2 del proyecto 1, En progreso
GO

-- =============================================================================
-- SECCIÓN 6: LOTE DE VALIDACIÓN DE RESTRICCIONES (CASOS INVÁLIDOS)
-- =============================================================================
PRINT 'Probando restricciones (estas inserciones deben fallar)...';

-- Prueba 1: Intentar insertar un empleado con un correo duplicado
BEGIN TRY
    INSERT INTO empleado (nombre_empleado, correo_electronico) VALUES ('Otro Juan', 'juan.perez@tech.com');
END TRY
BEGIN CATCH
    PRINT '-> ÉXITO: Falló la inserción por uq_empleado_correo, como se esperaba.';
END CATCH;
GO

-- Prueba 2: Intentar asignar el mismo empleado dos veces al mismo proyecto
BEGIN TRY
    INSERT INTO asignacion_empleado_proyecto (id_empleado, id_proyecto, id_rol, fecha_inicio_participacion)
    VALUES (1, 1, 3, '2023-02-01'); -- Juan (ID=1) ya está en el proyecto (ID=1)
END TRY
BEGIN CATCH
    PRINT '-> ÉXITO: Falló la inserción por pk_asignacion_empleado_proyecto, como se esperaba.';
END CATCH;
GO

-- Prueba 3: Intentar insertar una tarea con un numero_orden que ya existe para ese proyecto
BEGIN TRY
    INSERT INTO tarea (id_proyecto, numero_orden, descripcion, fecha_inicio, fecha_fin, id_estado)
    VALUES (1, 2, 'Otra tarea con el mismo orden', '2023-03-01', '2023-03-31', 1); -- El proyecto 1 ya tiene una tarea 2
END TRY
BEGIN CATCH
    PRINT '-> ÉXITO: Falló la inserción por pk_tarea, como se esperaba.';
END CATCH;
GO

-- Prueba 4: Intentar insertar una tarea con una fecha de fin anterior a la de inicio
BEGIN TRY
    INSERT INTO tarea (id_proyecto, numero_orden, descripcion, fecha_inicio, fecha_fin, id_estado)
    VALUES (1, 3, 'Tarea con fechas inválidas', '2023-04-01', '2023-03-31', 1);
END TRY
BEGIN CATCH
    PRINT '-> ÉXITO: Falló la inserción por chk_tarea_fechas_logicas, como se esperaba.';
END CATCH;
GO
```
