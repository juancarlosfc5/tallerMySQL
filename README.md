# tallerMySQL

# Solución taller

## 1. Normalización

1. Crear una tabla HistorialPedidos que almacene cambios en los pedidos.

```mysql
-- Tabla historialpedido

CREATE TABLE historialpedido (
	id INT PRIMARY KEY AUTO_INCREMENT,
    
);
```



1. Evaluar la tabla Clientes para eliminar datos redundantes y normalizar hasta 3NF.
2. Separar la tabla Empleados en una tabla de DatosEmpleados y otra para Puestos .
3. Revisar la relación Clientes y UbicacionCliente para evitar duplicación de datos.
4. Normalizar Proveedores para tener ContactoProveedores en otra tabla.
5. Crear una tabla de Telefonos para almacenar múltiples números por cliente.
6. Transformar TiposProductos en una relación categórica jerárquica.
7. Normalizar Pedidos y DetallesPedido para evitar inconsistencias de precios.
8. Usar una relación de muchos a muchos para Empleados y Proveedores .
9. Convertir la tabla UbicacionCliente en una relación genérica de Ubicaciones .
