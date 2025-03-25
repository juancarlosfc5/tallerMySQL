# Solución taller

### 0. Base de datos

```mysql
-- Creación de la base de datos
CREATE DATABASE vtaszfs;
USE vtaszfs;

-- Tabla Clientes
CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Tabla UbicacionCliente
CREATE TABLE UbicacionCliente (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Tabla Empleados
CREATE TABLE Empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    puesto VARCHAR(50),
    salario DECIMAL(10, 2),
    fecha_contratacion DATE
);

-- Tabla Proveedores
CREATE TABLE Proveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    contacto VARCHAR(100),
    telefono VARCHAR(20),
    direccion VARCHAR(255)
);

-- Tabla TiposProductos
CREATE TABLE TiposProductos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    tipo_nombre VARCHAR(100),
    descripcion TEXT
);

-- Tabla Productos
CREATE TABLE Productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    precio DECIMAL(10, 2),
    proveedor_id INT,
    tipo_id INT,
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
    FOREIGN KEY (tipo_id) REFERENCES TiposProductos(id)
);

-- Tabla Pedidos
CREATE TABLE Pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    fecha DATE,
    total DECIMAL(10, 2),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Tabla DetallesPedido
CREATE TABLE DetallesPedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    producto_id INT,
    cantidad INT,
    precio DECIMAL(10, 2),
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES Productos(id)
);
```

## 1. Normalización

1. Crear una tabla HistorialPedidos que almacene cambios en los pedidos.

```mysql
-- Creación de la tabla HistorialPedidos
CREATE TABLE HistorialPedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    cliente_id INT,
    fecha_modificacion DATETIME DEFAULT CURRENT_TIMESTAMP,
    total_anterior DECIMAL(10, 2),
    total_nuevo DECIMAL(10, 2),
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
```

2. Evaluar la tabla Clientes para eliminar datos redundantes y normalizar hasta 3NF.

```mysql
-- Tabla Clientes
CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    email VARCHAR(100) UNIQUE
);
```

3. Separar la tabla Empleados en una tabla de DatosEmpleados y otra para Puestos.

```mysql
CREATE TABLE DatosEmpleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    puesto_id INT,
    fecha_contratacion DATE,
    FOREIGN KEY (puesto_id) REFERENCES Puestos(id)
);

CREATE TABLE Puestos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre_puesto VARCHAR(50) UNIQUE,
    salario DECIMAL(10,2)
);

```

4. Revisar la relación Clientes y UbicacionCliente para evitar duplicación de datos.

```mysql
CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE Direcciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50),
    UNIQUE (direccion, ciudad, estado, codigo_postal, pais)
);

CREATE TABLE ClienteDireccion (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    direccion_id INT,
    tipo_direccion VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (direccion_id) REFERENCES Direcciones(id)
);
```

5. Normalizar Proveedores para tener ContactoProveedores en otra tabla.

```mysql
CREATE TABLE Proveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    direccion VARCHAR(255)
);

CREATE TABLE ContactoProveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proveedor_id INT,
    nombre_contacto VARCHAR(100),
    telefono VARCHAR(20),
    email VARCHAR(100),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);
```

6. Crear una tabla de Telefonos para almacenar múltiples números por cliente.

```mysql
CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE Telefonos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    numero VARCHAR(20),
    tipo_telefono VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
```

7. Transformar TiposProductos en una relación categórica jerárquica.

```mysql
CREATE TABLE TiposProductos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    descripcion TEXT,
    padre_id INT NULL,
    FOREIGN KEY (padre_id) REFERENCES TiposProductos(id)
);
```

8. Normalizar Pedidos y DetallesPedido para evitar inconsistencias de precios.

```mysql
CREATE TABLE Pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    fecha DATE,
    total DECIMAL(10,2),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

CREATE TABLE PreciosProductos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    producto_id INT,
    precio DECIMAL(10,2),
    fecha_inicio DATE, 
    fecha_fin DATE NULL,
    FOREIGN KEY (producto_id) REFERENCES Productos(id)
);

CREATE TABLE DetallesPedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    producto_id INT,
    precio_id INT,
    cantidad INT,
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES Productos(id),
    FOREIGN KEY (precio_id) REFERENCES PreciosProductos(id)
);
```

9. Usar una relación de muchos a muchos para Empleados y Proveedores.

```mysql
CREATE TABLE Empleados_Proveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    empleado_id INT,
    proveedor_id INT,
    rol VARCHAR(50),
    FOREIGN KEY (empleado_id) REFERENCES Empleados(id),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);
```

10. Convertir la tabla UbicacionCliente en una relación genérica de Ubicaciones.

```mysql
CREATE TABLE Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50)
);

CREATE TABLE Clientes_Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    ubicacion_id INT,
    tipo_ubicacion_cliente VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id)
);

CREATE TABLE Proveedores_Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proveedor_id INT,
    ubicacion_id INT,
    tipo_ubicacion_proveedor VARCHAR(50),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
    FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id)
);
```

### 1.1 Base de datos Normalizada

```mysql
-- Creación de la base de datos
CREATE DATABASE vtaszfs;
USE vtaszfs;

-- Tabla Clientes
CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Tabla Direcciones
CREATE TABLE Direcciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50),
    UNIQUE (direccion, ciudad, estado, codigo_postal, pais)
);

-- Relación Cliente-Dirección
CREATE TABLE ClienteDireccion (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    direccion_id INT,
    tipo_direccion VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (direccion_id) REFERENCES Direcciones(id)
);

-- Tabla Telefonos
CREATE TABLE Telefonos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    numero VARCHAR(20),
    tipo_telefono VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Tabla Puestos
CREATE TABLE Puestos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre_puesto VARCHAR(50) UNIQUE,
    salario DECIMAL(10,2)
);

-- Tabla Empleados
CREATE TABLE DatosEmpleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    puesto_id INT,
    fecha_contratacion DATE,
    FOREIGN KEY (puesto_id) REFERENCES Puestos(id)
);

-- Tabla Proveedores
CREATE TABLE Proveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    direccion VARCHAR(255)
);

-- Contacto de Proveedores
CREATE TABLE ContactoProveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proveedor_id INT,
    nombre_contacto VARCHAR(100),
    telefono VARCHAR(20),
    email VARCHAR(100),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);

-- Tabla TiposProductos con jerarquía
CREATE TABLE TiposProductos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    descripcion TEXT,
    padre_id INT NULL,
    FOREIGN KEY (padre_id) REFERENCES TiposProductos(id)
);

-- Tabla Productos
CREATE TABLE Productos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(100),
    proveedor_id INT,
    tipo_id INT,
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
    FOREIGN KEY (tipo_id) REFERENCES TiposProductos(id)
);

-- Historial de precios de productos
CREATE TABLE PreciosProductos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    producto_id INT,
    precio DECIMAL(10,2),
    fecha_inicio DATE, 
    fecha_fin DATE NULL,
    FOREIGN KEY (producto_id) REFERENCES Productos(id)
);

-- Tabla Pedidos
CREATE TABLE Pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    fecha DATE,
    total DECIMAL(10,2),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Historial de cambios en Pedidos
CREATE TABLE HistorialPedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    cliente_id INT,
    fecha_modificacion DATETIME DEFAULT CURRENT_TIMESTAMP,
    total_anterior DECIMAL(10,2),
    total_nuevo DECIMAL(10,2),
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- Tabla DetallesPedido
CREATE TABLE DetallesPedido (
    id INT PRIMARY KEY AUTO_INCREMENT,
    pedido_id INT,
    producto_id INT,
    precio_id INT,
    cantidad INT,
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (producto_id) REFERENCES Productos(id),
    FOREIGN KEY (precio_id) REFERENCES PreciosProductos(id)
);

-- Relación muchos a muchos entre Empleados y Proveedores
CREATE TABLE Empleados_Proveedores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    empleado_id INT,
    proveedor_id INT,
    rol VARCHAR(50),
    FOREIGN KEY (empleado_id) REFERENCES DatosEmpleados(id),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id)
);

-- Tabla Ubicaciones Genérica
CREATE TABLE Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    direccion VARCHAR(255),
    ciudad VARCHAR(100),
    estado VARCHAR(50),
    codigo_postal VARCHAR(10),
    pais VARCHAR(50)
);

-- Relación entre Clientes y Ubicaciones
CREATE TABLE Clientes_Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    ubicacion_id INT,
    tipo_ubicacion_cliente VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id)
);

-- Relación entre Proveedores y Ubicaciones
CREATE TABLE Proveedores_Ubicaciones (
    id INT PRIMARY KEY AUTO_INCREMENT,
    proveedor_id INT,
    ubicacion_id INT,
    tipo_ubicacion_proveedor VARCHAR(50),
    FOREIGN KEY (proveedor_id) REFERENCES Proveedores(id),
    FOREIGN KEY (ubicacion_id) REFERENCES Ubicaciones(id)
);
```

### 1.3 Inserts

```mysql
-- Inserts para la tabla Clientes
INSERT INTO Clientes (nombre, email) VALUES
('Juan Perez', 'juan@example.com'),
('Maria Gomez', 'maria@example.com'),
('Carlos Ramirez', 'carlos@example.com'),
('Ana Torres', 'ana@example.com'),
('Pedro Jimenez', 'pedro@example.com'),
('Laura Méndez', 'laura@example.com'),
('Luis Martínez', 'luis@example.com'),
('Sofia Herrera', 'sofia@example.com'),
('Jorge Pineda', 'jorge@example.com'),
('Diana Rojas', 'diana@example.com');

-- Inserts para la tabla Direcciones
INSERT INTO Direcciones (direccion, ciudad, estado, codigo_postal, pais) VALUES
('Calle 1 #123', 'Bogotá', 'Cundinamarca', '110111', 'Colombia'),
('Carrera 5 #45', 'Medellín', 'Antioquia', '050001', 'Colombia'),
('Avenida 30 #67', 'Cali', 'Valle', '760001', 'Colombia'),
('Calle 20 #10', 'Barranquilla', 'Atlántico', '080001', 'Colombia'),
('Carrera 40 #90', 'Cartagena', 'Bolívar', '130001', 'Colombia'),
('Calle 12 #21', 'Bucaramanga', 'Santander', '680001', 'Colombia'),
('Carrera 17 #33', 'Pereira', 'Risaralda', '660001', 'Colombia'),
('Avenida 9 #55', 'Manizales', 'Caldas', '170001', 'Colombia'),
('Calle 80 #100', 'Santa Marta', 'Magdalena', '470001', 'Colombia'),
('Carrera 25 #70', 'Villavicencio', 'Meta', '500001', 'Colombia');

-- Inserts para ClienteDireccion
INSERT INTO ClienteDireccion (cliente_id, direccion_id, tipo_direccion) VALUES
(1, 1, 'Residencial'),
(2, 2, 'Comercial'),
(3, 3, 'Residencial'),
(4, 4, 'Comercial'),
(5, 5, 'Residencial'),
(6, 6, 'Comercial'),
(7, 7, 'Residencial'),
(8, 8, 'Comercial'),
(9, 9, 'Residencial'),
(10, 10, 'Comercial');

-- Inserts para Telefonos
INSERT INTO Telefonos (cliente_id, numero, tipo_telefono) VALUES
(1, '3001234567', 'Móvil'),
(2, '3109876543', 'Móvil'),
(3, '3201112233', 'Móvil'),
(4, '3305556677', 'Fijo'),
(5, '3407778899', 'Móvil'),
(6, '3503332211', 'Fijo'),
(7, '3608887766', 'Móvil'),
(8, '3706665554', 'Fijo'),
(9, '3802221110', 'Móvil'),
(10, '3904443332', 'Fijo');

-- Inserts para Puestos
INSERT INTO Puestos (nombre_puesto, salario) VALUES
('Gerente', 5000000),
('Analista', 3000000),
('Desarrollador', 3500000),
('Diseñador', 2800000),
('Soporte Técnico', 2500000),
('Administrador', 4000000),
('Contador', 4200000),
('Vendedor', 2200000),
('Asistente', 2000000),
('Recepcionista', 1800000);

-- Inserts para DatosEmpleados
INSERT INTO DatosEmpleados (nombre, puesto_id, fecha_contratacion) VALUES
('Andrés Pérez', 1, '2023-01-15'),
('Marta Rodríguez', 2, '2022-06-20'),
('Camilo Suárez', 3, '2021-11-05'),
('Verónica Torres', 4, '2022-03-10'),
('Javier Gómez', 5, '2020-09-14'),
('Lucía Morales', 6, '2019-08-01'),
('Sergio Duarte', 7, '2018-12-23'),
('Tatiana Ríos', 8, '2023-07-18'),
('Oscar Herrera', 9, '2020-02-29'),
('Nadia Beltrán', 10, '2021-05-17');

-- Inserts para Proveedores
INSERT INTO Proveedores (nombre, direccion) VALUES
('Distribuidora XYZ', 'Calle 23 #12'),
('Suministros ABC', 'Carrera 10 #45'),
('Almacén El Éxito', 'Avenida 30 #67'),
('Comercializadora LMN', 'Calle 15 #89'),
('Mayoristas 123', 'Carrera 40 #90'),
('GlobalTech', 'Calle 50 #200'),
('Electroventas', 'Carrera 17 #33'),
('Soluciones JJ', 'Avenida 9 #55'),
('TecnoMarket', 'Calle 80 #100'),
('Distribuciones RS', 'Carrera 25 #70');

-- Inserts para ContactoProveedores
INSERT INTO ContactoProveedores (proveedor_id, nombre_contacto, telefono, email) VALUES
(1, 'Carlos Mendoza', '3123456789', 'carlos.mendoza@proveedor.com'),
(2, 'Ana Torres', '3156784321', 'ana.torres@proveedor.com'),
(3, 'Juan Pérez', '3145678901', 'juan.perez@proveedor.com'),
(4, 'Luis Gómez', '3167890123', 'luis.gomez@proveedor.com'),
(5, 'Marta Díaz', '3178901234', 'marta.diaz@proveedor.com'),
(6, 'Sofía Ramírez', '3189012345', 'sofia.ramirez@proveedor.com'),
(7, 'Pedro Sánchez', '3190123456', 'pedro.sanchez@proveedor.com'),
(8, 'Laura Herrera', '3201234567', 'laura.herrera@proveedor.com'),
(9, 'David Castro', '3212345678', 'david.castro@proveedor.com'),
(10, 'Gabriela López', '3223456789', 'gabriela.lopez@proveedor.com');

-- Inserts para TiposProductos
INSERT INTO TiposProductos (nombre, descripcion, padre_id) VALUES
('Electrónica', 'Dispositivos electrónicos', NULL),
('Móviles', 'Teléfonos y accesorios', 1),
('Computadoras', 'Laptops y accesorios', 1),
('Hogar', 'Electrodomésticos y muebles', NULL),
('Cocina', 'Electrodomésticos de cocina', 4),
('Ropa', 'Vestimenta y accesorios', NULL),
('Calzado', 'Zapatos y sandalias', 6),
('Deportes', 'Equipamiento deportivo', NULL),
('Ciclismo', 'Bicicletas y accesorios', 8),
('Camping', 'Equipo de camping', 8);

-- Inserts para Productos
INSERT INTO Productos (nombre, precio, proveedor_id, tipo_id) VALUES
('Smartphone X', 1200.00, 1, 2),
('Laptop Pro', 2500.00, 2, 3),
('Horno Eléctrico', 300.00, 3, 5),
('Zapatillas Running', 150.00, 4, 7),
('Bicicleta Montaña', 800.00, 5, 9),
('Tienda de Campaña', 250.00, 6, 10),
('Cámara Digital', 450.00, 7, 1),
('Reloj Inteligente', 200.00, 8, 2),
('Lavadora', 600.00, 9, 4),
('Sofá', 1200.00, 10, 4);

-- Inserts para PreciosProductos
INSERT INTO PreciosProductos (producto_id, precio, fecha_inicio, fecha_fin) VALUES
(1, 1200.00, '2024-01-01', NULL),
(2, 2500.00, '2024-02-01', NULL),
(3, 300.00, '2024-03-01', NULL),
(4, 150.00, '2024-04-01', NULL),
(5, 800.00, '2024-05-01', NULL),
(6, 250.00, '2024-06-01', NULL),
(7, 450.00, '2024-07-01', NULL),
(8, 200.00, '2024-08-01', NULL),
(9, 600.00, '2024-09-01', NULL),
(10, 1200.00, '2024-10-01', NULL);

-- Inserts para Pedidos
INSERT INTO Pedidos (cliente_id, fecha, total) VALUES
(1, '2024-01-10', 1500.00),
(2, '2024-02-15', 3200.00),
(3, '2024-03-20', 800.00),
(4, '2024-04-25', 1200.00),
(5, '2024-05-30', 250.00),
(6, '2024-06-05', 600.00),
(7, '2024-07-10', 300.00),
(8, '2024-08-15', 450.00),
(9, '2024-09-20', 1200.00),
(10, '2024-10-25', 900.00);

-- Inserts para HistorialPedidos
INSERT INTO HistorialPedidos (pedido_id, cliente_id, fecha_modificacion, total_anterior, total_nuevo) VALUES
(1, 1, '2024-01-11 10:00:00', 1500.00, 1600.00),
(2, 2, '2024-02-16 14:00:00', 3200.00, 3100.00),
(3, 3, '2024-03-21 09:00:00', 800.00, 750.00),
(4, 4, '2024-04-26 16:00:00', 1200.00, 1300.00),
(5, 5, '2024-05-31 11:00:00', 250.00, 270.00),
(6, 6, '2024-06-15 12:30:00', 500.00, 520.00),
(7, 7, '2024-07-20 15:45:00', 2200.00, 2100.00),
(8, 8, '2024-08-10 08:00:00', 1450.00, 1500.00),
(9, 9, '2024-09-05 17:20:00', 3100.00, 3050.00),
(10, 10, '2024-10-01 13:10:00', 600.00, 590.00);

-- Inserts para DetallesPedido
INSERT INTO DetallesPedido (pedido_id, producto_id, precio_id, cantidad) VALUES
(1, 1, 1, 2),
(2, 2, 2, 1),
(3, 3, 3, 1),
(4, 4, 4, 2),
(5, 5, 5, 1),
(6, 6, 6, 3),
(7, 7, 7, 2),
(8, 8, 8, 4),
(9, 9, 9, 1),
(10, 10, 10, 5);

-- Inserts para Empleados_Proveedores
INSERT INTO Empleados_Proveedores (empleado_id, proveedor_id, rol) VALUES
(1, 1, 'Comprador'),
(2, 2, 'Gerente'),
(3, 3, 'Asistente'),
(4, 4, 'Supervisor'),
(5, 5, 'Encargado'),
(6, 6, 'Coordinador'),
(7, 7, 'Logística'),
(8, 8, 'Administrador'),
(9, 9, 'Analista'),
(10, 10, 'Director');

-- Inserts para Ubicaciones
INSERT INTO Ubicaciones (direccion, ciudad, estado, codigo_postal, pais) VALUES
('Calle 1 #10-20', 'Bogotá', 'Cundinamarca', '110111', 'Colombia'),
('Av. Siempre Viva 742', 'Medellín', 'Antioquia', '050001', 'Colombia'),
('Cra 15 #100-50', 'Cali', 'Valle del Cauca', '760001', 'Colombia'),
('Calle 50 #20-30', 'Barranquilla', 'Atlántico', '080001', 'Colombia'),
('Av. El Dorado #90-10', 'Bogotá', 'Cundinamarca', '110921', 'Colombia'),
('Calle 8 #22-15', 'Cartagena', 'Bolívar', '130001', 'Colombia'),
('Av. Caracas #12-40', 'Bogotá', 'Cundinamarca', '111221', 'Colombia'),
('Calle 23 #56-78', 'Bucaramanga', 'Santander', '680002', 'Colombia'),
('Calle 45 #78-90', 'Pereira', 'Risaralda', '660001', 'Colombia'),
('Av. Boyacá #45-67', 'Bogotá', 'Cundinamarca', '111041', 'Colombia');

-- Inserts para Clientes_Ubicaciones
INSERT INTO Clientes_Ubicaciones (cliente_id, ubicacion_id, tipo_ubicacion_cliente) VALUES
(1, 1, 'Residencial'),
(2, 2, 'Oficina'),
(3, 3, 'Residencial'),
(4, 4, 'Oficina'),
(5, 5, 'Residencial'),
(6, 6, 'Oficina'),
(7, 7, 'Residencial'),
(8, 8, 'Oficina'),
(9, 9, 'Residencial'),
(10, 10, 'Oficina');

-- Inserts para Proveedores_Ubicaciones
INSERT INTO Proveedores_Ubicaciones (proveedor_id, ubicacion_id, tipo_ubicacion_proveedor) VALUES
(1, 1, 'Bodega'),
(2, 2, 'Sucursal'),
(3, 3, 'Centro de Distribución'),
(4, 4, 'Almacén Central'),
(5, 5, 'Planta de Producción'),
(6, 6, 'Bodega'),
(7, 7, 'Sucursal'),
(8, 8, 'Centro de Distribución'),
(9, 9, 'Almacén Central'),
(10, 10, 'Planta de Producción');
```

## 2. Joins

1. Obtener la lista de todos los pedidos con los nombres de clientes usando INNER JOIN

```mysql
SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente, Pedidos.fecha, Pedidos.total
FROM Pedidos
INNER JOIN Clientes ON Pedidos.cliente_id = Clientes.id;
```

2. Listar los productos y proveedores que los suministran con INNER JOIN.

```mysql
SELECT Productos.nombre AS producto, Proveedores.nombre AS proveedor
FROM Productos
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id;
```

3. Mostrar los pedidos y las ubicaciones de los clientes con LEFT JOIN.

```mysql
SELECT Pedidos.id AS pedido_id, Clientes.nombre AS cliente, Ubicaciones.direccion, Ubicaciones.ciudad
FROM Pedidos
LEFT JOIN Clientes ON Pedidos.cliente_id = Clientes.id
LEFT JOIN Clientes_Ubicaciones ON Clientes.id = Clientes_Ubicaciones.cliente_id
LEFT JOIN Ubicaciones ON Clientes_Ubicaciones.ubicacion_id = Ubicaciones.id;
```

4. Consultar los empleados que han registrado pedidos, incluyendo empleados sin pedidos
   ( LEFT JOIN ).

```mysql
SELECT Empleados.id AS empleado_id, Empleados.nombre AS empleado, Pedidos.id AS pedido_id
FROM Empleados
LEFT JOIN Pedidos ON Empleados.id = Pedidos.id;
```

5. Obtener el tipo de producto y los productos asociados con INNER JOIN .

```mysql
SELECT TiposProductos.nombre AS tipo_producto, Productos.nombre AS producto
FROM Productos
INNER JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;
```

6. Listar todos los clientes y el número de pedidos realizados con COUNT y GROUP BY.

```mysql
SELECT Clientes.nombre AS cliente, COUNT(Pedidos.id) AS numero_pedidos
FROM Clientes
LEFT JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
GROUP BY Clientes.nombre;
```

7. Combinar Pedidos y Empleados para mostrar qué empleados gestionaron pedidos
   específicos.

```mysql
SELECT Pedidos.id AS pedido_id, Empleados.nombre AS empleado
FROM Pedidos
INNER JOIN Empleados ON Pedidos.id = Empleados.id;
```

8. Mostrar productos que no han sido pedidos ( RIGHT JOIN ).

```mysql
SELECT Productos.nombre AS producto, DetallesPedido.pedido_id
FROM Productos
RIGHT JOIN DetallesPedido ON Productos.id = DetallesPedido.producto_id
WHERE DetallesPedido.pedido_id IS NULL;
```

9. Mostrar el total de pedidos y ubicación de clientes usando múltiples JOIN .

```mysql
SELECT Clientes.nombre AS cliente, COUNT(Pedidos.id) AS total_pedidos, Ubicaciones.direccion, Ubicaciones.ciudad
FROM Clientes
LEFT JOIN Pedidos ON Clientes.id = Pedidos.cliente_id
LEFT JOIN Clientes_Ubicaciones ON Clientes.id = Clientes_Ubicaciones.cliente_id
LEFT JOIN Ubicaciones ON Clientes_Ubicaciones.ubicacion_id = Ubicaciones.id
GROUP BY Clientes.nombre, Ubicaciones.direccion, Ubicaciones.ciudad;
```

10. Unir Proveedores , Productos , y TiposProductos para un listado completo de inventario.

```mysql
SELECT Productos.nombre AS producto, Proveedores.nombre AS proveedor, TiposProductos.nombre AS tipo_producto
FROM Productos
INNER JOIN Proveedores ON Productos.proveedor_id = Proveedores.id
INNER JOIN TiposProductos ON Productos.tipo_id = TiposProductos.id;
```

## 3. Consultas Simples

1. Seleccionar todos los productos con precio mayor a $50.

```mysql
SELECT nombre FROM Productos WHERE precio > 50;
```

2. Consultar clientes registrados en una ciudad específica.

```mysql
SELECT c.nombre FROM Clientes c
JOIN Clientes_Ubicaciones cu ON c.id = cu.cliente_id
JOIN Ubicaciones u ON cu.ubicacion_id = u.id
WHERE u.ciudad = 'Bogotá';
```

3. Mostrar empleados contratados en los últimos 2 años.

```mysql
SELECT nombre FROM DatosEmpleados 
WHERE fecha_contratacion >= DATE_SUB(CURDATE(), INTERVAL 2 YEAR);
```

4. Seleccionar proveedores que suministran más de 5 productos.

```mysql
SELECT p.id, p.nombre, COUNT(prod.id) AS cantidad_productos
FROM Proveedores p
JOIN Productos prod ON p.id = prod.proveedor_id
GROUP BY p.id, p.nombre
HAVING COUNT(prod.id) > 5;
```

5. Listar clientes que no tienen dirección registrada en UbicacionCliente .

```mysql
SELECT nombre FROM Clientes c
LEFT JOIN Clientes_Ubicaciones cu ON c.id = cu.cliente_id
WHERE cu.cliente_id IS NULL;
```

6. Calcular el total de ventas por cada cliente.

```mysql
SELECT c.id, c.nombre, SUM(p.total) AS total_ventas
FROM Clientes c
JOIN Pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre;
```

7. Mostrar el salario promedio de los empleados.

```mysql
SELECT AVG(p.salario) AS salario_promedio FROM Puestos p;
```

8. Consultar el tipo de productos disponibles en TiposProductos .

```mysql
SELECT DISTINCT nombre FROM TiposProductos;
```

9. Seleccionar los 3 productos más caros.

```mysql
SELECT nombre FROM Productos ORDER BY precio DESC LIMIT 3;
```

10. Consultar el cliente con el mayor número de pedidos.

```mysql
SELECT c.id, c.nombre, COUNT(p.id) AS total_pedidos
FROM Clientes c
JOIN Pedidos p ON c.id = p.cliente_id
GROUP BY c.id, c.nombre
ORDER BY total_pedidos DESC
LIMIT 1;
```

## 4.  Consultas Multitabla

1. Listar todos los pedidos y el cliente asociado.

```mysql
SELECT p.id AS pedido_id, c.id AS cliente_id, c.nombre AS cliente_nombre
FROM Pedidos p
JOIN Clientes c ON p.cliente_id = c.id;
```

2. Mostrar la ubicación de cada cliente en sus pedidos.

```mysql
SELECT p.id AS pedido_id, c.nombre AS cliente_nombre, u.ciudad, u.direccion
FROM Pedidos p
JOIN Clientes c ON p.cliente_id = c.id
JOIN Clientes_Ubicaciones cu ON c.id = cu.cliente_id
JOIN Ubicaciones u ON cu.ubicacion_id = u.id;
```

3. Listar productos junto con el proveedor y tipo de producto.

```mysql
SELECT prod.id AS producto_id, prod.nombre AS producto_nombre, 
       p.nombre AS proveedor_nombre, tp.nombre AS tipo_producto
FROM Productos prod
JOIN Proveedores p ON prod.proveedor_id = p.id
JOIN TiposProductos tp ON prod.tipo_producto_id = tp.id;
```

4. Consultar todos los empleados que gestionan pedidos de clientes en una ciudad específica.

```mysql
SELECT DISTINCT e.id, e.nombre
FROM DatosEmpleados e
JOIN Pedidos p ON e.id = p.empleado_id
JOIN Clientes c ON p.cliente_id = c.id
JOIN Clientes_Ubicaciones cu ON c.id = cu.cliente_id
JOIN Ubicaciones u ON cu.ubicacion_id = u.id
WHERE u.ciudad = 'Bogotá';
```

5. Consultar los 5 productos más vendidos.

```mysql
SELECT dp.producto_id, prod.nombre, SUM(dp.cantidad) AS total_vendido
FROM DetallesPedido dp
JOIN Productos prod ON dp.producto_id = prod.id
GROUP BY dp.producto_id, prod.nombre
ORDER BY total_vendido DESC
LIMIT 5;
```

6. Obtener la cantidad total de pedidos por cliente y ciudad.

```mysql
SELECT c.id AS cliente_id, c.nombre AS cliente_nombre, u.ciudad, COUNT(p.id) AS total_pedidos
FROM Clientes c
JOIN Pedidos p ON c.id = p.cliente_id
JOIN Clientes_Ubicaciones cu ON c.id = cu.cliente_id
JOIN Ubicaciones u ON cu.ubicacion_id = u.id
GROUP BY c.id, c.nombre, u.ciudad;
```

7. Listar clientes y proveedores en la misma ciudad.

```mysql
SELECT c.id AS cliente_id, c.nombre AS cliente_nombre, 
       p.id AS proveedor_id, p.nombre AS proveedor_nombre, u.ciudad
FROM Clientes c
JOIN Clientes_Ubicaciones cu ON c.id = cu.cliente_id
JOIN Ubicaciones u ON cu.ubicacion_id = u.id
JOIN Proveedores_Ubicaciones pu ON u.id = pu.ubicacion_id
JOIN Proveedores p ON pu.proveedor_id = p.id;
```

8. Mostrar el total de ventas agrupado por tipo de producto.

```mysql
SELECT tp.nombre AS tipo_producto, SUM(dp.cantidad * pp.precio) AS total_ventas
FROM DetallesPedido dp
JOIN Productos prod ON dp.producto_id = prod.id
JOIN TiposProductos tp ON prod.tipo_producto_id = tp.id
JOIN PreciosProductos pp ON prod.id = pp.producto_id
GROUP BY tp.nombre;
```

9. Listar empleados que gestionan pedidos de productos de un proveedor específico.

```mysql
SELECT DISTINCT e.id, e.nombre
FROM DatosEmpleados e
JOIN Pedidos p ON e.id = p.empleado_id
JOIN DetallesPedido dp ON p.id = dp.pedido_id
JOIN Productos prod ON dp.producto_id = prod.id
JOIN Proveedores prov ON prod.proveedor_id = prov.id
WHERE prov.nombre = 'Distribuidora XYZ'
```

10. Obtener el ingreso total de cada proveedor a partir de los productos vendidos.

```mysql
SELECT p.id AS proveedor_id, p.nombre AS proveedor_nombre, 
       SUM(dp.cantidad * pp.precio) AS ingreso_total
FROM Proveedores p
JOIN Productos prod ON p.id = prod.proveedor_id
JOIN DetallesPedido dp ON prod.id = dp.producto_id
JOIN PreciosProductos pp ON prod.id = pp.producto_id
GROUP BY p.id, p.nombre;
```

## 5. Subconsultas

1. Consultar el producto más caro en cada categoría.

```mysql
SELECT p.id, p.nombre, p.tipo_producto_id, p.precio
FROM Productos p
WHERE p.precio = (
    SELECT MAX(p2.precio)
    FROM Productos p2
    WHERE p2.tipo_producto_id = p.tipo_producto_id
);
```

2. Encontrar el cliente con mayor total en pedidos.

```mysql
SELECT c.id, c.nombre, 
       (SELECT SUM(dp.cantidad * pp.precio) 
        FROM Pedidos p 
        JOIN DetallesPedido dp ON p.id = dp.pedido_id
        JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
        WHERE p.cliente_id = c.id) AS total_pedidos
FROM Clientes c
ORDER BY total_pedidos DESC
LIMIT 1;
```

3. Listar empleados que ganan más que el salario promedio.

```mysql
SELECT id, nombre, salario
FROM DatosEmpleados
WHERE salario > (SELECT AVG(salario) FROM DatosEmpleados);
```

4. Consultar productos que han sido pedidos más de 5 veces.

```mysql
SELECT p.id, p.nombre
FROM Productos p
WHERE (SELECT COUNT(nombre) FROM DetallesPedido dp WHERE dp.producto_id = p.id) > 5;
```

5. Listar pedidos cuyo total es mayor al promedio de todos los pedidos.

```mysql
SELECT p.id, SUM(dp.cantidad * pp.precio) AS total_pedido
FROM Pedidos p
JOIN DetallesPedido dp ON p.id = dp.pedido_id
JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
GROUP BY p.id
HAVING total_pedido > (SELECT AVG(total_pedido) 
                       FROM (SELECT SUM(dp.cantidad * pp.precio) AS total_pedido
                             FROM Pedidos p
                             JOIN DetallesPedido dp ON p.id = dp.pedido_id
                             JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
                             GROUP BY p.id) AS pedidos_promedio);
```

6. Seleccionar los 3 proveedores con más productos.

```mysql
SELECT proveedor_id, COUNT(nombre) AS total_productos
FROM Productos
GROUP BY proveedor_id
ORDER BY total_productos DESC
LIMIT 3;
```

7. Consultar productos con precio superior al promedio en su tipo.

```mysql
SELECT p.id, p.nombre, p.tipo_producto_id, p.precio
FROM Productos p
WHERE p.precio > (SELECT AVG(p2.precio) 
                  FROM Productos p2 
                  WHERE p2.tipo_producto_id = p.tipo_producto_id);
```

8. Mostrar clientes que han realizado más pedidos que la media.

```mysql
SELECT c.id, c.nombre
FROM Clientes c
WHERE (SELECT COUNT(nombre) FROM Pedidos p WHERE p.cliente_id = c.id) > 
      (SELECT AVG(pedidos_por_cliente) 
       FROM (SELECT COUNT(nombre) AS pedidos_por_cliente FROM Pedidos GROUP BY cliente_id) AS pedidos_media);
```

9. Encontrar productos cuyo precio es mayor que el promedio de todos los productos.

```mysql
SELECT p.id, p.nombre, p.precio
FROM Productos p
WHERE p.precio > (SELECT AVG(precio) FROM Productos);
```

10. Mostrar empleados cuyo salario es menor al promedio del departamento.

```mysql
SELECT e.id, e.nombre, e.salario, e.departamento_id
FROM DatosEmpleados e
WHERE e.salario < (SELECT AVG(e2.salario) 
                   FROM DatosEmpleados e2 
                   WHERE e2.departamento_id = e.departamento_id);
```

## 6. Procedimientos almacenados

1. Crear un procedimiento para actualizar el precio de todos los productos de un proveedor.

```mysql
DELIMITER $$
CREATE PROCEDURE ActualizarPrecioProveedor(IN proveedorID INT, IN porcentaje DECIMAL(5,2))
BEGIN
    UPDATE Productos 
    SET precio = precio * (1 + porcentaje / 100) 
    WHERE proveedor_id = proveedorID;
END $$
DELIMITER ;
```

2. Un procedimiento que devuelva la dirección de un cliente por ID.

```mysql
DELIMITER $$
CREATE PROCEDURE ObtenerDireccionCliente(IN clienteID INT)
BEGIN
    SELECT u.direccion
    FROM Ubicaciones u
    JOIN Clientes_Ubicaciones cu ON u.id = cu.ubicacion_id
    WHERE cu.cliente_id = clienteID;
END $$
DELIMITER ;
```

3. Crear un procedimiento que registre un pedido nuevo y sus detalles.

```mysql
DELIMITER $$
CREATE PROCEDURE RegistrarPedido(
    IN p_cliente_id INT,
    IN p_fecha_pedido DATETIME,
    IN p_total DECIMAL(10,2),

    -- Datos del primer producto
    IN p_producto1_id INT, IN p_precio1_id INT, IN p_cantidad1 INT,
    -- Datos del segundo producto (opcional)
    IN p_producto2_id INT, IN p_precio2_id INT, IN p_cantidad2 INT,
    -- Datos del tercer producto (opcional)
    IN p_producto3_id INT, IN p_precio3_id INT, IN p_cantidad3 INT,
    -- Datos del cuarto producto (opcional)
    IN p_producto4_id INT, IN p_precio4_id INT, IN p_cantidad4 INT,
    -- Datos del quinto producto (opcional)
    IN p_producto5_id INT, IN p_precio5_id INT, IN p_cantidad5 INT
)
BEGIN
    DECLARE v_pedido_id INT;

    -- Insertar el pedido en la tabla Pedidos
    INSERT INTO Pedidos (cliente_id, fecha_pedido, total)
    VALUES (p_cliente_id, p_fecha_pedido, p_total);

    -- Obtener el ID del pedido recién creado
    SET v_pedido_id = LAST_INSERT_ID();

    -- Insertar detalles del pedido solo si los valores no son NULL
    IF p_producto1_id IS NOT NULL THEN
        INSERT INTO DetallesPedido (pedido_id, producto_id, precio_id, cantidad)
        VALUES (v_pedido_id, p_producto1_id, p_precio1_id, p_cantidad1);
    END IF;

    IF p_producto2_id IS NOT NULL THEN
        INSERT INTO DetallesPedido (pedido_id, producto_id, precio_id, cantidad)
        VALUES (v_pedido_id, p_producto2_id, p_precio2_id, p_cantidad2);
    END IF;

    IF p_producto3_id IS NOT NULL THEN
        INSERT INTO DetallesPedido (pedido_id, producto_id, precio_id, cantidad)
        VALUES (v_pedido_id, p_producto3_id, p_precio3_id, p_cantidad3);
    END IF;

    IF p_producto4_id IS NOT NULL THEN
        INSERT INTO DetallesPedido (pedido_id, producto_id, precio_id, cantidad)
        VALUES (v_pedido_id, p_producto4_id, p_precio4_id, p_cantidad4);
    END IF;

    IF p_producto5_id IS NOT NULL THEN
        INSERT INTO DetallesPedido (pedido_id, producto_id, precio_id, cantidad)
        VALUES (v_pedido_id, p_producto5_id, p_precio5_id, p_cantidad5);
    END IF;

    -- Devolver el ID del pedido creado
    SELECT v_pedido_id AS nuevo_pedido_id;
END $$
DELIMITER ;
```

4. Un procedimiento para calcular el total de ventas de un cliente.

```mysql
DELIMITER $$
CREATE PROCEDURE TotalVentasCliente(IN clienteID INT)
BEGIN
    SELECT SUM(dp.cantidad * pp.precio) AS total_ventas
    FROM Pedidos p
    JOIN DetallesPedido dp ON p.id = dp.pedido_id
    JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
    WHERE p.cliente_id = clienteID;
END $$
DELIMITER ;
```

5. Crear un procedimiento para obtener los empleados por puesto.

```mysql
DELIMITER $$
CREATE PROCEDURE ObtenerEmpleadosPorPuesto(IN puesto VARCHAR(50))
BEGIN
    SELECT id, nombre, salario 
    FROM DatosEmpleados 
    WHERE cargo = puesto;
END $$
DELIMITER ;
```

6. Un procedimiento que actualice el salario de empleados por puesto.

```mysql
DELIMITER $$
CREATE PROCEDURE ActualizarSalarioPorPuesto(IN puesto VARCHAR(50), IN incremento DECIMAL(5,2))
BEGIN
    UPDATE DatosEmpleados 
    SET salario = salario * (1 + incremento / 100) 
    WHERE cargo = puesto;
END $$
DELIMITER ;
```

7. Crear un procedimiento que liste los pedidos entre dos fechas.

```mysql
DELIMITER $$
CREATE PROCEDURE ListarPedidosEntreFechas(IN fechaInicio DATE, IN fechaFin DATE)
BEGIN
    SELECT id 
    FROM Pedidos 
    WHERE fecha_pedido BETWEEN fechaInicio AND fechaFin;
END $$
DELIMITER ;
```

8. Un procedimiento para aplicar un descuento a productos de una categoría.

```mysql
DELIMITER $$
CREATE PROCEDURE AplicarDescuentoCategoria(IN tipoProductoID INT, IN descuento DECIMAL(5,2))
BEGIN
    UPDATE Productos 
    SET precio = precio * (1 - descuento / 100) 
    WHERE tipo_producto_id = tipoProductoID;
END $$
DELIMITER ;
```

9. Crear un procedimiento que liste todos los proveedores de un tipo de producto.

```mysql
DELIMITER $$
CREATE PROCEDURE ListarProveedoresPorTipoProducto(IN tipoProductoID INT)
BEGIN
    SELECT DISTINCT p.id, p.nombre
    FROM Proveedores p
    JOIN Productos prod ON p.id = prod.proveedor_id
    WHERE prod.tipo_producto_id = tipoProductoID;
END $$
DELIMITER ;
```

10. Un procedimiento que devuelva el pedido de mayor valor.

```mysql
DELIMITER $$
CREATE PROCEDURE PedidoMayorValor()
BEGIN
    SELECT p.id, SUM(dp.cantidad * pp.precio) AS total_pedido
    FROM Pedidos p
    JOIN DetallesPedido dp ON p.id = dp.pedido_id
    JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
    GROUP BY p.id
    ORDER BY total_pedido DESC
    LIMIT 1;
END $$
DELIMITER ;
```

## 7. Funciones definidas por el usuario

1. Crear una función que reciba una fecha y devuelva los días transcurridos.

```mysql
DELIMITER $$
CREATE FUNCTION DiasTranscurridos(fecha DATE) RETURNS INT DETERMINISTIC
BEGIN
    RETURN DATEDIFF(CURDATE(), fecha);
END $$
DELIMITER ;
```

2. Crear una función para calcular el total con impuesto de un monto.

```mysql
DELIMITER $$
CREATE FUNCTION CalcularTotalConImpuesto(monto DECIMAL(10,2), impuesto DECIMAL(5,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN monto * (1 + impuesto / 100);
END $$
DELIMITER ;
```

3. Una función que devuelva el total de pedidos de un cliente específico.

```mysql
DELIMITER $$
CREATE FUNCTION TotalPedidosCliente(clienteID INT) RETURNS INT DETERMINISTIC
BEGIN
    DECLARE totalPedidos INT;
    SELECT COUNT(id) INTO totalPedidos FROM Pedidos WHERE cliente_id = clienteID;
    RETURN totalPedidos;
END $$
DELIMITER ;
```

4. Crear una función para aplicar un descuento a un producto.

```mysql
DELIMITER $$
CREATE FUNCTION AplicarDescuento(precio DECIMAL(10,2), descuento DECIMAL(5,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN precio * (1 - descuento / 100);
END $$
DELIMITER ;
```

5. Una función que indique si un cliente tiene dirección registrada.

```mysql
DELIMITER $$
CREATE FUNCTION ClienteTieneDireccion(clienteID INT) RETURNS BOOLEAN DETERMINISTIC
BEGIN
    DECLARE tieneDireccion BOOLEAN;
    SELECT EXISTS (
        SELECT 1 FROM Clientes_Ubicaciones WHERE cliente_id = clienteID
    ) INTO tieneDireccion;
    RETURN tieneDireccion;
END $$
DELIMITER ;
```

6. Crear una función que devuelva el salario anual de un empleado.

```mysql
DELIMITER $$
CREATE FUNCTION SalarioAnualEmpleado(salarioMensual DECIMAL(10,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN salarioMensual * 12;
END $$
DELIMITER ;
```

7. Una función para calcular el total de ventas de un tipo de producto.

```mysql
DELIMITER $$
CREATE FUNCTION TotalVentasPorTipoProducto(tipoProductoID INT) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    DECLARE totalVentas DECIMAL(10,2);
    SELECT SUM(dp.cantidad * pp.precio) INTO totalVentas
    FROM DetallesPedido dp
    JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
    JOIN Productos p ON dp.producto_id = p.id
    WHERE p.tipo_producto_id = tipoProductoID;
    RETURN COALESCE(totalVentas, 0);
END $$
DELIMITER ;
```

8. Crear una función para devolver el nombre de un cliente por ID.

```mysql
DELIMITER $$
CREATE FUNCTION NombreCliente(clienteID INT) RETURNS VARCHAR(100) DETERMINISTIC
BEGIN
    DECLARE nombre VARCHAR(100);
    SELECT nombre_cliente INTO nombre FROM Clientes WHERE id = clienteID;
    RETURN nombre;
END $$
DELIMITER ;
```

9. Una función que reciba el ID de un pedido y devuelva su total.

```mysql
DELIMITER $$
CREATE FUNCTION TotalPedido(pedidoID INT) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);
    SELECT SUM(dp.cantidad * pp.precio) INTO total
    FROM DetallesPedido dp
    JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
    WHERE dp.pedido_id = pedidoID;
    RETURN COALESCE(total, 0);
END $$
DELIMITER ;
```

10. Crear una función que indique si un producto está en inventario.

```mysql
DELIMITER $$
CREATE FUNCTION ProductoEnInventario(productoID INT) RETURNS BOOLEAN DETERMINISTIC
BEGIN
    DECLARE enInventario BOOLEAN;
    SELECT EXISTS (
        SELECT 1 FROM Productos WHERE id = productoID AND stock > 0
    ) INTO enInventario;
    RETURN enInventario;
END $$
DELIMITER ;
```

## 8. Triggers

1. Crear un trigger que registre en HistorialSalarios cada cambio de salario de empleados.

```mysql
DELIMITER $$
CREATE TRIGGER tr_ActualizarHistorialSalarios
AFTER UPDATE ON Empleados
FOR EACH ROW
BEGIN
    IF OLD.salario <> NEW.salario THEN
        INSERT INTO HistorialSalarios (empleado_id, salario_anterior, nuevo_salario, fecha_cambio)
        VALUES (NEW.id, OLD.salario, NEW.salario, NOW());
    END IF;
END $$
DELIMITER ;
```

2. Crear un trigger que evite borrar productos con pedidos activos.

```mysql
DELIMITER $$
CREATE TRIGGER tr_PrevenirBorradoProductos
BEFORE DELETE ON Productos
FOR EACH ROW
BEGIN
    IF EXISTS (SELECT 1 FROM DetallesPedido WHERE producto_id = OLD.id) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede eliminar un producto con pedidos activos.';
    END IF;
END $$
DELIMITER ;
```

3. Un trigger que registre en HistorialPedidos cada actualización en Pedidos .

```mysql
DELIMITER $$
CREATE TRIGGER tr_ActualizarHistorialPedidos
AFTER UPDATE ON Pedidos
FOR EACH ROW
BEGIN
    INSERT INTO HistorialPedidos (pedido_id, estado_anterior, nuevo_estado, fecha_actualizacion)
    VALUES (NEW.id, OLD.estado, NEW.estado, NOW());
END $$
DELIMITER ;
```

4. Crear un trigger que actualice el inventario al registrar un pedido.

```mysql
DELIMITER $$
CREATE TRIGGER tr_ActualizarInventario
AFTER INSERT ON DetallesPedido
FOR EACH ROW
BEGIN
    UPDATE Productos
    SET stock = stock - NEW.cantidad
    WHERE id = NEW.producto_id;
END $$
DELIMITER ;
```

5. Un trigger que evite actualizaciones de precio a menos de $1.

```mysql
DELIMITER $$
CREATE TRIGGER tr_ValidarPrecio
BEFORE UPDATE ON PreciosProductos
FOR EACH ROW
BEGIN
    IF NEW.precio < 1 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El precio no puede ser menor a $1.';
    END IF;
END $$
DELIMITER ;
```

6. Crear un trigger que registre la fecha de creación de un pedido en HistorialPedidos .

```mysql
DELIMITER $$
CREATE TRIGGER tr_RegistrarPedido
AFTER INSERT ON Pedidos
FOR EACH ROW
BEGIN
    INSERT INTO HistorialPedidos (pedido_id, estado_anterior, nuevo_estado, fecha_actualizacion)
    VALUES (NEW.id, NULL, NEW.estado, NOW());
END $$
DELIMITER ;
```

7. Un trigger que mantenga el precio total de cada pedido en Pedidos .

```mysql
DELIMITER $$
CREATE TRIGGER tr_CalcularTotalPedido
AFTER INSERT ON DetallesPedido
FOR EACH ROW
BEGIN
    UPDATE Pedidos
    SET total = (SELECT SUM(dp.cantidad * pp.precio)
                 FROM DetallesPedido dp
                 JOIN PreciosProductos pp ON dp.producto_id = pp.producto_id
                 WHERE dp.pedido_id = NEW.pedido_id)
    WHERE id = NEW.pedido_id;
END $$
DELIMITER ;
```

8. Crear un trigger para validar que UbicacionCliente no esté vacío al crear un cliente.

```mysql
DELIMITER $$
CREATE TRIGGER tr_ValidarUbicacionCliente
BEFORE INSERT ON Clientes
FOR EACH ROW
BEGIN
    IF NEW.ubicacion_id IS NULL THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'El cliente debe tener una ubicación registrada.';
    END IF;
END $$
DELIMITER ;
```

9. Un trigger que registre en LogActividades cada modificación en Proveedores .

```mysql
DELIMITER $$
CREATE TRIGGER tr_LogModificacionProveedores
AFTER UPDATE ON Proveedores
FOR EACH ROW
BEGIN
    INSERT INTO LogActividades (tabla_afectada, id_registro, accion, fecha)
    VALUES ('Proveedores', NEW.id, 'Actualización', NOW());
END $$
DELIMITER ;
```

10. Crear un trigger que registre en HistorialContratos cada cambio en Empleados .

```mysql
DELIMITER $$
CREATE TRIGGER tr_HistorialContratos
AFTER UPDATE ON Empleados
FOR EACH ROW
BEGIN
    IF OLD.puesto <> NEW.puesto OR OLD.salario <> NEW.salario THEN
        INSERT INTO HistorialContratos (empleado_id, puesto_anterior, nuevo_puesto, salario_anterior, nuevo_salario, fecha_cambio)
        VALUES (NEW.id, OLD.puesto, NEW.puesto, OLD.salario, NEW.salario, NOW());
    END IF;
END $$
DELIMITER ;
```

## 9. Ejercicios Combinados de Funciones y Consultas

### Función de Descuento por Categoría de Producto

Objetivo: Crear una función que aplique un descuento sobre el precio de un producto si
pertenece a una categoría específica.
Pasos:

1. Crear una función CalcularDescuento que reciba el tipo_id del producto y el precio
   original, y aplique un descuento del 10% si el tipo es "Electrónica".
2. Realizar una consulta para mostrar el nombre del producto, el precio original y el precio
   con descuento.

```mysql
-- Función de Descuento por Categoría de Producto
DELIMITER //
CREATE FUNCTION CalcularDescuento(tipo_id INT, precio DECIMAL(10,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    IF tipo_id = 'Electrónica' THEN
        RETURN precio * 0.9;
    ELSE
        RETURN precio;
    END IF;
END //
DELIMITER ;

-- Consulta de productos con descuento
SELECT nombre, precio, CalcularDescuento(tipo_id, precio) AS precio_con_descuento
FROM productos;
```

### Función para Obtener la Edad de un Cliente y Filtrar Clientes Mayores de Edad

Objetivo: Crear una función que calcule la edad de un cliente en función de su fecha de
nacimiento y luego usarla para listar solo los clientes mayores de 18 años.
Pasos:

1. Crear la función CalcularEdad que reciba la fecha de nacimiento y calcule la edad.
2. Consultar todos los clientes y mostrar solo aquellos que sean mayores de 18 años.

```mysql
-- Función para calcular la edad de un cliente
DELIMITER //
CREATE FUNCTION CalcularEdad(fecha_nacimiento DATE) RETURNS INT DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, fecha_nacimiento, CURDATE());
END //
DELIMITER ;

-- Consulta de clientes mayores de edad
SELECT nombre, fecha_nacimiento, CalcularEdad(fecha_nacimiento) AS edad
FROM clientes
WHERE CalcularEdad(fecha_nacimiento) >= 18;
```

### Función de Cálculo de Impuesto y Consulta de Productos con Precio Final

Objetivo: Crear una función que calcule el precio final de un producto aplicando un
impuesto del 15% y luego mostrar una lista de productos con el precio final incluido.
Pasos:

1. Crear la función CalcularImpuesto que reciba el precio del producto y aplique el
   impuesto.
2. Mostrar el nombre del producto, el precio original y el precio final con impuesto.

```mysql
-- Función para calcular el impuesto
DELIMITER //
CREATE FUNCTION CalcularImpuesto(precio DECIMAL(10,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN precio * 1.15;
END //
DELIMITER ;

-- Consulta de productos con precio final
SELECT nombre, precio, CalcularImpuesto(precio) AS precio_final
FROM productos;
```

### Función para Calcular el Total de Pedidos de un Cliente

Objetivo: Crear una función que calcule el total de los pedidos de un cliente y usarla para
mostrar los clientes con total de pedidos mayor a $1000.
Pasos:

1. Crear la función TotalPedidosCliente que reciba el ID de un cliente y calcule el total
   de todos sus pedidos.
2. Realizar una consulta que muestre el nombre del cliente y su total de pedidos, y filtrar
   clientes con un total mayor a $1000.

```mysql
-- Función para calcular total de pedidos de un cliente
DELIMITER //
CREATE FUNCTION TotalPedidosCliente(cliente_id INT) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);
    SELECT SUM(total) INTO total FROM pedidos WHERE cliente_id = cliente_id;
    RETURN IFNULL(total, 0);
END //
DELIMITER ;

-- Consulta de clientes con total de pedidos mayor a $1000
SELECT c.nombre, TotalPedidosCliente(c.id) AS total_pedidos
FROM clientes c
WHERE TotalPedidosCliente(c.id) > 1000;
```

### Función para Calcular el Salario Anual de un Empleado

Objetivo: Crear una función que calcule el salario anual de un empleado y usarla para listar
todos los empleados con un salario anual mayor a $50,000.
Pasos:

1. Crear la función SalarioAnual que reciba el salario mensual y lo multiplique por 12.
2. Realizar una consulta que muestre el nombre del empleado y su salario anual, filtrando
   empleados con salario mayor a $50,000.

```mysql
-- Función para calcular salario anual
DELIMITER //
CREATE FUNCTION SalarioAnual(salario_mensual DECIMAL(10,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN salario_mensual * 12;
END //
DELIMITER ;

-- Consulta de empleados con salario anual mayor a $50,000
SELECT nombre, SalarioAnual(salario_mensual) AS salario_anual
FROM empleados
WHERE SalarioAnual(salario_mensual) > 50000;
```

### Función de Bonificación y Consulta de Salarios Ajustados

Objetivo: Crear una función que calcule la bonificación de un empleado (10% de su salario) y
mostrar el salario ajustado de cada empleado.
Pasos:

1. Crear una función Bonificacion que reciba el salario y calcule el 10%.
2. Realizar una consulta que muestre el salario ajustado (salario + bonificación).

```mysql
-- Función para calcular bonificación
DELIMITER //
CREATE FUNCTION Bonificacion(salario DECIMAL(10,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN salario * 1.10;
END //
DELIMITER ;

-- Consulta de salarios ajustados
SELECT nombre, salario, Bonificacion(salario) AS salario_ajustado
FROM empleados;
```

### Función para Calcular Días Transcurridos Desde el Último Pedido

Objetivo: Crear una función que calcule los días desde el último pedido de un cliente y
mostrar clientes que hayan hecho un pedido en los últimos 30 días.
Pasos:

1. Crear la función DiasDesdeUltimoPedido que reciba el ID de un cliente y calcule los
   días desde su último pedido.
2. Realizar una consulta que muestre solo a los clientes con pedidos en los últimos 30 días.

```mysql
-- Función para calcular días desde último pedido
DELIMITER //
CREATE FUNCTION DiasDesdeUltimoPedido(cliente_id INT) RETURNS INT DETERMINISTIC
BEGIN
    DECLARE dias INT;
    SELECT DATEDIFF(CURDATE(), MAX(fecha_pedido)) INTO dias FROM pedidos WHERE cliente_id = cliente_id;
    RETURN IFNULL(dias, 9999);
END //
DELIMITER ;

-- Consulta de clientes con pedidos en los últimos 30 días
SELECT nombre, DiasDesdeUltimoPedido(id) AS dias
FROM clientes
WHERE DiasDesdeUltimoPedido(id) <= 30;
```

### Función para Calcular el Total en Inventario de un Producto

Objetivo: Crear una función que calcule el total en inventario (cantidad x precio) de cada
producto y listar productos con inventario superior a $500.
Pasos:

1. Crear la función TotalInventarioProducto que multiplique cantidad y precio de un
   producto.
2. Realizar una consulta que muestre el nombre del producto y su total en inventario,
   filtrando los productos con inventario superior a $500.

```mysql
-- Función para calcular total en inventario
DELIMITER //
CREATE FUNCTION TotalInventarioProducto(cantidad INT, precio DECIMAL(10,2)) RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    RETURN cantidad * precio;
END //
DELIMITER ;

-- Consulta de productos con inventario mayor a $500
SELECT nombre, TotalInventarioProducto(cantidad, precio) AS total_inventario
FROM productos
WHERE TotalInventarioProducto(cantidad, precio) > 500;
```

### Creación de un Historial de Precios de Productos

Descripción: Crear un trigger y una tabla para mantener un historial de precios de
productos. Cada vez que el precio de un producto cambia, el trigger debe guardar el ID del
producto, el precio antiguo, el nuevo precio y la fecha de cambio.
Pasos:

1. Crear la tabla HistorialPrecios .
2. Crear el trigger RegistroCambioPrecio en la tabla Productos para registrar los
   cambios de precio.

```mysql
-- Creación de tabla para historial de precios
CREATE TABLE HistorialPrecios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    producto_id INT,
    precio_anterior DECIMAL(10,2),
    precio_nuevo DECIMAL(10,2),
    fecha_cambio TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger para registrar cambios de precio
DELIMITER //
CREATE TRIGGER RegistroCambioPrecio
BEFORE UPDATE ON productos
FOR EACH ROW
BEGIN
    INSERT INTO HistorialPrecios (producto_id, precio_anterior, precio_nuevo)
    VALUES (OLD.id, OLD.precio, NEW.precio);
END //
DELIMITER ;
```

### Procedimiento para Generar Reporte de Ventas Mensuales por Empleado

Descripción: Crear un procedimiento almacenado que genere un reporte de ventas mensual
para cada empleado. El procedimiento debe recibir como parámetros el mes y el año, y
devolver una lista de empleados con el total de ventas que gestionaron en ese periodo.
Pasos:

1. Crear el procedimiento ReporteVentasMensuales .
2. Usar una subconsulta que agrupe las ventas por empleado y que filtre por el mes y el
   año.

```mysql
-- Procedimiento para reporte de ventas mensuales
DELIMITER //
CREATE PROCEDURE ReporteVentasMensuales(IN mes INT, IN anio INT)
BEGIN
    SELECT e.nombre, SUM(v.total) AS total_ventas
    FROM empleados e
    JOIN ventas v ON e.id = v.empleado_id
    WHERE MONTH(v.fecha) = mes AND YEAR(v.fecha) = anio
    GROUP BY e.id;
END //
DELIMITER ;

-- Consulta del producto más vendido por proveedor
SELECT p.nombre AS proveedor, pr.nombre AS producto, MAX(v.cantidad) AS cantidad_vendida
FROM ventas v
JOIN productos pr ON v.producto_id = pr.id
JOIN proveedores p ON pr.proveedor_id = p.id
GROUP BY p.id;
```

### Subconsulta para Obtener el Producto Más Vendido por Cada Proveedor

Descripción: Realizar una consulta compleja que devuelva el producto más vendido para
cada proveedor, mostrando el nombre del proveedor, el nombre del producto y la cantidad
vendida.
Pasos:

1. Utilizar una subconsulta para calcular la cantidad total de ventas por producto.
2. Filtrar en la consulta principal para obtener el producto más vendido de cada
   proveedor.

```mysql
-- Subconsulta para obtener el producto mas vendido por cada proveedor
SELECT p.proveedor_id, pr.nombre AS proveedor, p.producto_id, p.nombre AS producto, p.total_vendido
FROM (
    SELECT d.producto_id, pr.proveedor_id, p.nombre, SUM(d.cantidad) AS total_vendido,
           RANK() OVER (PARTITION BY pr.proveedor_id ORDER BY SUM(d.cantidad) DESC) AS ranking
    FROM DetallesPedido d
    JOIN Productos p ON d.producto_id = p.producto_id
    JOIN Proveedores pr ON p.proveedor_id = pr.proveedor_id
    GROUP BY d.producto_id, pr.proveedor_id, p.nombre
) p
JOIN Proveedores pr ON p.proveedor_id = pr.proveedor_id
WHERE p.ranking = 1;
```

#### Función para Calcular el Estado de Stock de un Producto

Descripción: Crear una función que calcule el estado de stock de un producto y lo clasifique
en “Alto”, “Medio” o “Bajo” en función de su cantidad. Usar esta función en una consulta para
listar todos los productos y su estado de stock.
Pasos:

1. Crear la función EstadoStock que reciba la cantidad de un producto.
2. En la consulta principal, utilizar la función para clasificar el estado de stock de cada
   producto.

```mysql
-- Función para calcular estado de stock
DELIMITER //
CREATE FUNCTION EstadoStock(cantidad INT) RETURNS VARCHAR(10) DETERMINISTIC
BEGIN
    RETURN CASE 
        WHEN cantidad > 100 THEN 'Alto'
        WHEN cantidad BETWEEN 50 AND 100 THEN 'Medio'
        ELSE 'Bajo'
    END;
END //
DELIMITER ;

-- Consulta de estado de stock
SELECT nombre, cantidad, EstadoStock(cantidad) AS estado_stock
FROM productos;
```

### Trigger para Control de Inventario en Pedidos

Descripción: Crear un trigger que, al insertar un nuevo pedido, disminuya automáticamente
la cantidad en stock del producto. El trigger debe también prevenir que se inserte el pedido si
el stock es insuficiente.
Pasos:

1. Crear el trigger ActualizarInventario en la tabla DetallesPedido .
2. Controlar que no se permita la inserción si la cantidad es mayor que el stock disponible.

```mysql
-- Trigger para actualizar inventario al hacer un pedido
DELIMITER //
CREATE TRIGGER ActualizarInventario
BEFORE INSERT ON DetallesPedido
FOR EACH ROW
BEGIN
    IF (SELECT cantidad FROM productos WHERE id = NEW.producto_id) < NEW.cantidad THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Stock insuficiente';
    ELSE
        UPDATE productos SET cantidad = cantidad - NEW.cantidad WHERE id = NEW.producto_id;
    END IF;
END //
DELIMITER ;
```

### Procedimiento para Generar Informe de Clientes Inactivos

Descripción: Crear un procedimiento almacenado que genere un informe de clientes
inactivos (aquellos que no han realizado pedidos en los últimos 6 meses).
Pasos:

1. Crear el procedimiento ClientesInactivos .
2. Filtrar clientes que no tengan pedidos recientes usando una subconsulta.

```mysql
-- Procedimiento para identificar clientes inactivos
DELIMITER //
CREATE PROCEDURE ClientesInactivos()
BEGIN
    SELECT nombre FROM clientes WHERE id NOT IN (
        SELECT DISTINCT cliente_id FROM pedidos WHERE fecha_pedido >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
    );
END //
DELIMITER ;
```

