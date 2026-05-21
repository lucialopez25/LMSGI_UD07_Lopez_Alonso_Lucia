# Manual de Explotación - WillmanTech S.L.
**Norma de referencia:** ISO/IEC/IEEE 26514:2022

## 1. Introducción y Arquitectura
El sistema ERP de WillmanTech S.L. está diseñado bajo una arquitectura de microservicios contenerizada orientada a la alta disponibilidad. La topología lógica se despliega mediante **Docker Compose**, separando la capa de aplicación (ERP) de la capa de persistencia de datos (Sistema Gestor de Base de Datos).

* **Módulos activados:** Facturación (`account`), Ventas (`sale`), y Reportes QWeb (`web`).
* **Contenedor Aplicación:** Imagen basada en Odoo/Python.
* **Contenedor SGBD:** Imagen basada en PostgreSQL 15.

## 2. Guía de Instalación y Reinstalación
Para levantar el entorno desde cero, se requiere Docker y Docker Compose instalados en el nodo anfitrión.

**Pasos de despliegue:**
1. Clonar el repositorio en el servidor de producción.
2. Definir las variables de entorno en un archivo `.env` en la raíz del proyecto:
   ```env
   POSTGRES_DB=willmantech_db
   POSTGRES_USER=odoo_user
   POSTGRES_PASSWORD=secure_db_password
   ADMIN_PASSWORD=superadmin_secret
3. Ejecutar el orquestador para levantar los servicios en segundo plano:
    Bash
    docker-compose up -d
4. Para reinstalaciones, eliminar volúmenes previos (ATENCIÓN: Borrado total de datos) y levantar nuevamente:
    Bash
    docker-compose down -v
    docker-compose up -d
## 3. Seguridad y Control de Acceso
El acceso a la plataforma se rige por un modelo de Control de Acceso Basado en Roles (RBAC):

Administrador: Acceso total a la configuración del sistema, gestión de usuarios y bases de datos.

Contable: Privilegios de lectura/escritura en facturación y acceso a informes financieros (JSON/XML UBL).

Comercial: Solo lectura sobre productos e inventario, capacidad de generar presupuestos y confirmarlos a facturas.

Política de contraseñas: Mínimo 12 caracteres, alfanumérica con símbolos, expiración cada 90 días forzada por políticas de servidor.

## 4. Procedimiento de Backup y Restauración
Las copias de seguridad deben ser atómicas e incluir tanto la base de datos relacional como el almacén de archivos (Filestore).

Generar Backup (PostgreSQL):

Bash
docker exec -t willmantech_db_1 pg_dump -U odoo_user -F c -b -v -f /var/lib/postgresql/data/backup.dump willmantech_db
Restaurar Backup:

Bash
docker exec -i willmantech_db_1 pg_restore -U odoo_user -d willmantech_db -v /var/lib/postgresql/data/backup.dump
## 5. Flujo Operativo de Facturación e Informes
Creación en UI: El usuario introduce los datos de la venta mediante la interfaz gráfica del ERP.

Confirmación: Al validar el documento, la factura cambia a estado Publicado, asignándose número oficial (ej. INV/2026/05/0001).

Renderizado de Plantilla (QWeb): El motor QWeb procesa el archivo XML, inyectando los datos de la base de datos (Data Binding con t-field y t-foreach) generando HTML estático.

Conversión a PDF (wkhtmltopdf): El sistema envía este HTML al motor wkhtmltopdf para transformarlo en PDF estructurado y paginado.

Descarga: El PDF es devuelto al navegador para su descarga o impresión final.