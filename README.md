# Arquitectura de datos: RDS vs. DynamoDB · LearnWithMe

## 1. Cuadro comparativo de requisitos

| Dato | ¿Relaciones complejas? | ¿Escalabilidad masiva? | Mejor opción |
|---|---|---|---|
| Perfiles de estudiantes | Sí (inscripciones, pagos) | No, miles de usuarios | RDS |
| Catálogo de cursos | Sí (curso → módulos → lecciones) | No | RDS |
| Inscripciones | Sí (muchos a muchos) | No | RDS |
| Pagos | Sí, y requieren transacciones ACID | No | RDS |

| Criterio | Amazon RDS (MySQL) | Amazon DynamoDB |
|---|---|---|
| Modelo | Relacional (tablas) | NoSQL (clave-valor) |
| JOINs | Sí | No |
| Transacciones | ACID completas | Limitadas |
| Escalabilidad | Vertical y réplicas de lectura | Masiva y serverless |
| Ideal para | Datos relacionados y consistentes | Tráfico enorme e impredecible |

## 2. Diagrama de integración

![Diagrama de arquitectura de EduCloud](Arquitectura_Datos_RDS_EduCloud.png)

Las instancias EC2 se conectan a RDS por el puerto 3306. La base de datos está en subredes privadas con despliegue Multi-AZ: una instancia primaria en la AZ A y una standby en la AZ B.

## 3. Justificación técnica

Elegí **Amazon RDS for MySQL** (db.t3.micro, Multi-AZ) en lugar de DynamoDB por tres motivos.

**Relaciones y JOINs.** Los datos de EduCloud están muy relacionados: un estudiante se inscribe en muchos cursos, cada curso tiene muchos estudiantes y cada compra genera un pago. Con SQL, una consulta como "cursos de un estudiante y el estado de su pago" se resuelve con un JOIN. DynamoDB no admite JOINs, así que habría que duplicar datos o hacer varias consultas desde el código.

**Integridad de los pagos.** Cuando un estudiante
