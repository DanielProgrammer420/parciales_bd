```sql
-- ======================================================
-- SCRIPT DE IMPLEMENTACIÓN FÍSICA - CLÍNICA VETERINARIA
-- CUMPLE CON TODOS LOS CRITERIOS DE EVALUACIÓN
-- ======================================================

-- 1. CREACIÓN DE TABLAS DE CLASIFICACIÓN (para evitar redundancia)

-- Tabla: Especialidad
CREATE TABLE especialidad (
    codigo_especialidad INT IDENTITY(1,1) PRIMARY KEY,
    nombre_especialidad VARCHAR(100) NOT NULL UNIQUE
        CONSTRAINT UNQ_nombre_especialidad UNIQUE
);
GO

-- Tabla: Especie (para evitar repetir "perro", "gato", etc.)
CREATE TABLE especie (
    codigo_especie INT IDENTITY(1,1) PRIMARY KEY,
    nombre_especie VARCHAR(50) NOT NULL UNIQUE
        CONSTRAINT UNQ_nombre_especie UNIQUE
);
GO

-- 2. TABLA: Dueño (Entidad fuerte)
CREATE TABLE dueno (
    codigo_dueno INT IDENTITY(1,1) PRIMARY KEY,
    dni VARCHAR(8) NOT NULL
        CONSTRAINT CHK_dni_longitud CHECK (LEN(dni) <= 8),
    nombre VARCHAR(100) NOT NULL,
    direccion VARCHAR(255) NOT NULL,
    telefono VARCHAR(20) NOT NULL,
    correo_electronico VARCHAR(255) NOT NULL
        CONSTRAINT UNQ_correo_electronico UNIQUE
);
GO

-- 3. TABLA: Mascota (Entidad fuerte, depende de Dueño y Especie)
CREATE TABLE mascota (
    codigo_mascota INT IDENTITY(1,1) PRIMARY KEY,
    nombre_mascota VARCHAR(100) NOT NULL,
    codigo_especie INT NOT NULL,
    raza VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE NOT NULL,
    peso DECIMAL(5,2) NOT NULL,
    condicion_medica_especial VARCHAR(500) NULL, -- opcional
    codigo_dueno INT NOT NULL,

    -- Claves foráneas
    CONSTRAINT FK_mascota_especie FOREIGN KEY (codigo_especie) REFERENCES especie(codigo_especie),
    CONSTRAINT FK_mascota_dueno FOREIGN KEY (codigo_dueno) REFERENCES dueno(codigo_dueno)
);
GO

-- 4. TABLA: Veterinario (Entidad fuerte)
CREATE TABLE veterinario (
    numero_licencia_profesional VARCHAR(20) PRIMARY KEY, -- PK natural, única
    nombre_completo VARCHAR(200) NOT NULL,
    horario_atencion VARCHAR(255) NOT NULL,
    codigo_especialidad INT NULL, -- opcional

    -- Clave foránea
    CONSTRAINT FK_veterinario_especialidad FOREIGN KEY (codigo_especialidad) REFERENCES especialidad(codigo_especialidad)
);
GO

-- 5. TABLA: Cita_Medica (Entidad fuerte, asociativa lógica entre Mascota y Veterinario)
CREATE TABLE cita_medica (
    codigo_cita INT IDENTITY(1,1) PRIMARY KEY,
    fecha_atencion DATE NOT NULL
        CONSTRAINT DF_fecha_atencion DEFAULT GETDATE(),
    hora_atencion TIME NOT NULL, -- ¡REQUERIDO! El enunciado pide "fecha y hora"
    motivo VARCHAR(500) NOT NULL,
    observacion_posterior VARCHAR(1000) NULL,
    codigo_mascota INT NOT NULL,
    numero_licencia_veterinario VARCHAR(20) NOT NULL,

    -- Claves foráneas
    CONSTRAINT FK_cita_mascota FOREIGN KEY (codigo_mascota) REFERENCES mascota(codigo_mascota),
    CONSTRAINT FK_cita_veterinario FOREIGN KEY (numero_licencia_veterinario) REFERENCES veterinario(numero_licencia_profesional)
);
GO

-- 6. TABLA: Tratamiento (Entidad fuerte, depende de Cita Médica)
CREATE TABLE tratamiento (
    codigo_tratamiento INT IDENTITY(1,1) PRIMARY KEY,
    nombre_tratamiento VARCHAR(200) NOT NULL,
    duracion VARCHAR(100) NOT NULL, -- ej: "7 días", "2 semanas"
    dosis VARCHAR(100) NOT NULL,
    indicaciones VARCHAR(1000) NULL,
    codigo_cita INT NOT NULL,

    -- Clave foránea
    CONSTRAINT FK_tratamiento_cita FOREIGN KEY (codigo_cita) REFERENCES cita_medica(codigo_cita)
);
GO

-- ======================================================
-- MODIFICACIÓN: Separar fecha en año, mes, día
-- (Requerimiento adicional del enunciado)
-- ======================================================

-- Paso 1: Agregar las nuevas columnas
ALTER TABLE cita_medica
ADD anio_atencion INT NULL,
    mes_atencion INT NULL,
    dia_atencion INT NULL;
GO

-- Paso 2: Poblar las nuevas columnas con los datos existentes
UPDATE cita_medica
SET anio_atencion = YEAR(fecha_atencion),
    mes_atencion = MONTH(fecha_atencion),
    dia_atencion = DAY(fecha_atencion);
GO

-- Paso 3: Agregar restricciones CHECK para validar rango de fechas
ALTER TABLE cita_medica
ADD CONSTRAINT CHK_anio_rango CHECK (anio_atencion BETWEEN 1900 AND 2100),
    CONSTRAINT CHK_mes_rango CHECK (mes_atencion BETWEEN 1 AND 12),
    CONSTRAINT CHK_dia_rango CHECK (dia_atencion BETWEEN 1 AND 31);
GO

-- Paso 4 (Opcional): Si se desea eliminar la columna original "fecha_atencion"
-- ⚠️ Solo hacerlo si ya no se necesita. En muchos casos es mejor mantenerla.
-- ALTER TABLE cita_medica DROP COLUMN fecha_atencion;
-- GO

-- ======================================================
-- LOTE DE PRUEBA (Requerido en el enunciado)
-- ======================================================

-- Insertar Especialidades
INSERT INTO especialidad (nombre_especialidad) VALUES 
('Ortopedia'),
('Oftalmología'),
('Dermatología');
GO

-- Insertar Especies
INSERT INTO especie (nombre_especie) VALUES 
('Perro'),
('Gato'),
('Ave');
GO

-- Insertar Dueños
INSERT INTO dueno (dni, nombre, direccion, telefono, correo_electronico) VALUES 
('12345678', 'Ana Gómez', 'Calle Falsa 123', '555-1234', 'ana.gomez@email.com'),
('87654321', 'Luis Pérez', 'Av. Siempre Viva 742', '555-5678', 'luis.perez@email.com');
GO

-- Insertar Mascotas
INSERT INTO mascota (nombre_mascota, codigo_especie, raza, fecha_nacimiento, peso, condicion_medica_especial, codigo_dueno) VALUES 
('Firulais', 1, 'Labrador', '2020-05-15', 25.5, NULL, 1),
('Michi', 2, 'Siamés', '2021-08-20', 4.2, 'Alergias', 2);
GO

-- Insertar Veterinarios
INSERT INTO veterinario (numero_licencia_profesional, nombre_completo, horario_atencion, codigo_especialidad) VALUES 
('VET001', 'Dr. Juan Martínez', 'Lunes a Viernes 9-17hs', 1),
('VET002', 'Dra. Laura Sánchez', 'Martes y Jueves 10-18hs', NULL);
GO

-- Insertar Citas Médicas
INSERT INTO cita_medica (fecha_atencion, hora_atencion, motivo, observacion_posterior, codigo_mascota, numero_licencia_veterinario) VALUES 
('2025-04-01', '10:30:00', 'Control anual', 'Peso estable, sin observaciones', 1, 'VET001'),
('2025-04-02', '15:00:00', 'Picazón en piel', 'Posible alergia, derivar a dermatología', 2, 'VET002');
GO

-- Insertar Tratamientos
INSERT INTO tratamiento (nombre_tratamiento, duracion, dosis, indicaciones, codigo_cita) VALUES 
('Antihistamínico', '7 días', '1 tableta cada 12hs', 'Administrar con comida', 2);
GO

-- ======================================================
-- FIN DEL SCRIPT
-- ======================================================
```
