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


Las instancias EC2 se conectan a RDS por el puerto 3306. La base de datos está en subredes privadas con despliegue Multi-AZ: una instancia primaria en la AZ A y una standby en la AZ B.

## 3. Justificación técnica

Elegí **Amazon RDS for MySQL** (db.t3.micro, Multi-AZ) en lugar de DynamoDB por tres motivos.

**Relaciones y JOINs.** Los datos de la app están muy relacionados: un estudiante se inscribe en muchos cursos, cada curso tiene muchos estudiantes y cada compra genera un pago. Con SQL, una consulta como "cursos de un estudiante y el estado de su pago" se resuelve con un JOIN. DynamoDB no admite JOINs, así que habría que duplicar datos o hacer varias consultas desde el código.

**Integridad de los pagos.** Cuando un estudiante compra un curso, el pago y la inscripción deben guardarse juntos o no guardarse. RDS garantiza esto con transacciones ACID y claves foráneas.

**Volumen moderado.** La plataforma tendrá miles de estudiantes, no millones de operaciones por segundo. La escalabilidad masiva de DynamoDB no compensa sus limitaciones de consulta. Además, el costo de una instancia db.t3.micro es bajo y predecible, ideal para una startup que recién empieza.

RDS sigue siendo un servicio gestionado: AWS aplica los parches, hace backups automáticos y, con Multi-AZ, cambia a la réplica si la base de datos principal falla.

No elegí Aurora porque una instancia pequeña de RDS cubre la carga a menor costo. Si el tráfico crece, se puede aumentar el tamaño de la instancia o añadir réplicas de lectura.

## 4. Seguridad entre cómputo y datos

- **Subredes privadas:** RDS no tiene acceso público.
- **Security Group `SG-DB`:** solo permite el puerto 3306 desde `SG-Web-Servers`.
- **Rol IAM y Secrets Manager:** las EC2 obtienen las credenciales de la base de datos sin contraseñas en el código.
- **Cifrado:** en reposo con KMS y en tránsito con TLS.
