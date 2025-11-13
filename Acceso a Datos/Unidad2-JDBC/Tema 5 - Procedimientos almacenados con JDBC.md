
## Introducción

En bases de datos relacionales, además de ejecutar consultas SQL directamente desde Java, también es posible invocar **procedimientos almacenados (Stored Procedures)** y **funciones** definidos dentro del propio SGBD.

Un **procedimiento almacenado** es un bloque de código SQL (con instrucciones de control de flujo, condiciones, bucles, etc.) que se almacena en la base de datos y puede ser reutilizado por múltiples aplicaciones.

En JDBC, los procedimientos almacenados se ejecutan mediante la interfaz **`CallableStatement`**.

## 1. Ventajas de los procedimientos almacenados

- **Reutilización**: el mismo procedimiento puede ser invocado por distintas aplicaciones.
    
- **Rendimiento**: se precompilan y el motor de la BD optimiza su ejecución.
    
- **Seguridad**: se puede restringir el acceso a las tablas y obligar a usar procedimientos.
    
- **Mantenimiento**: las reglas de negocio se mantienen en un único lugar (la BD).
    

## 2. CallableStatement en JDBC

La clase `CallableStatement` permite ejecutar procedimientos almacenados desde Java.

### Sintaxis general

```java
CallableStatement cs = con.prepareCall("{call nombre_procedimiento(?, ?, ?)}");
```

- **Parámetros de entrada (`IN`)**: se envían desde Java a la BD → `cs.setInt(1, 1001);`
    
- **Parámetros de salida (`OUT`)**: se reciben de la BD a Java → `cs.registerOutParameter(2, Types.VARCHAR);`
    
- **Parámetros de entrada/salida (`INOUT`)**: actúan como ambos.

## 3. Ejemplo práctico

### 3.1. Creación del procedimiento en MySQL

```MySQL
DELIMITER //
CREATE PROCEDURE obtener_salario(IN empId INT, OUT empSalario DECIMAL(10,2))
BEGIN
    SELECT salario INTO empSalario
    FROM empleados
    WHERE id = empId;
END //
DELIMITER ;
```

### 3.2. Invocación desde Java

```java
import java.sql.*;

public class LlamarProcedimiento {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/empresa";
        String usuario = "root";
        String password = "root";

        try (Connection con = DriverManager.getConnection(url, usuario, password)) {
            
            CallableStatement cs = con.prepareCall("{call obtener_salario(?, ?)}");
            cs.setInt(1, 2); // parámetro IN
            cs.registerOutParameter(2, Types.DECIMAL); // parámetro OUT

            cs.execute();

            double salario = cs.getDouble(2); // recuperar parámetro OUT
            System.out.println("El salario del empleado 2 es: " + salario);

            cs.close();

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```


## 4. Ejemplo con función almacenada

### 4.1. Crear la función

```MySQL
DELIMITER //
CREATE FUNCTION contar_empleados() RETURNS INT
BEGIN
    DECLARE total INT;
    SELECT COUNT(*) INTO total FROM empleados;
    RETURN total;
END //
DELIMITER ;
```


### 4.2. Llamada desde Java

```java
CallableStatement cs = con.prepareCall("{? = call contar_empleados()}");
cs.registerOutParameter(1, Types.INTEGER);
cs.execute();
int total = cs.getInt(1);
System.out.println("Número de empleados: " + total);
```

## 5. Errores comunes

- Confundir `PreparedStatement` con `CallableStatement`.
    
- No registrar correctamente parámetros de salida (`registerOutParameter`).
    
- Usar el tipo de dato equivocado en `Types` (ej.: `Types.INTEGER` vs `Types.VARCHAR`).
    
- Intentar ejecutar una función como si fuera un procedimiento y viceversa.
    

## 6. Buenas prácticas

- Usar procedimientos para operaciones críticas y repetitivas.
    
- Documentar bien los parámetros de entrada y salida.
    
- Probar el procedimiento directamente en SQL antes de integrarlo en Java.
    
- Controlar excepciones SQL tanto en la BD como en el código Java.

