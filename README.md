# Tutorial de Instalación - Remote Admin Server

## Descripción

Remote Admin Server es un sistema completo de administración remota que permite el control seguro de dispositivos móviles y computadoras a través de una interfaz web y aplicaciones móviles.

### Características Principales

- **Panel de control web responsive** - Interfaz moderna y fácil de usar
- **Aplicaciones móviles** - Compatible con Android e iOS
- **Cifrado de extremo a extremo** - Comunicaciones seguras con RSA-2048 + AES-256
- **Autenticación robusta** - JWT tokens y API keys
- **Comunicación en tiempo real** - WebSockets para comandos instantáneos
- **Protección avanzada** - Rate limiting, validación de entrada, headers de seguridad
- **Auditoría completa** - Registro detallado de todas las actividades
- **Gestión de dispositivos** - Control centralizado de múltiples dispositivos

## Requisitos del Sistema

### Distribuciones Soportadas
- Ubuntu 20.04 LTS o superior
- Debian 10 (Buster) o superior
- Linux Mint 20 o superior
- Otras distribuciones basadas en Debian/Ubuntu

### Requisitos Mínimos
- **RAM**: 1 GB (recomendado 2 GB)
- **Almacenamiento**: 500 MB de espacio libre
- **CPU**: 1 core (recomendado 2 cores)
- **Red**: Conexión a Internet para descarga de dependencias

### Dependencias Automáticas
El paquete instalará automáticamente:
- Python 3.8 o superior
- Python3-pip y python3-venv
- SQLite3
- Nginx 1.18 o superior
- UFW (Uncomplicated Firewall)

## Instalación Paso a Paso

### Paso 1: Descargar el Paquete

```bash
# Opción A: Descargar desde GitHub (cuando esté disponible)
wget https://github.com/bastardosdelsur/remote-admin-server/remote-admin-server_1.0.0_all.deb

# Opción B: Si tienes el archivo localmente
# Asegúrate de que el archivo remote-admin-server_1.0.0_all.deb esté en tu directorio actual
```

### Paso 2: Actualizar el Sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### Paso 3: Instalar el Paquete

```bash
# Instalar el paquete .deb
sudo dpkg -i remote-admin-server_1.0.0_all.deb

# Si hay dependencias faltantes, ejecutar:
sudo apt-get install -f
```

### Paso 4: Verificar la Instalación

```bash
# Verificar que el servicio está funcionando
sudo systemctl status remote-admin-server

# Verificar que Nginx está funcionando
sudo systemctl status nginx

# Verificar puertos abiertos
sudo netstat -tlnp | grep -E ':(80|443|5000)'
```

### Paso 5: Configuración Inicial

```bash
# Ver información del sistema
sudo remote-admin-ctl info

# Verificar salud del sistema
sudo remote-admin-ctl check-health

# Ver logs en tiempo real (opcional)
sudo remote-admin-ctl logs -f
```

## Acceso al Sistema

### Panel de Administración Web

1. **URL**: `https://TU_IP_DEL_SERVIDOR`
2. **Usuario por defecto**: `admin`
3. **Contraseña por defecto**: `admin123`

**⚠️ IMPORTANTE**: Cambia la contraseña por defecto inmediatamente después del primer acceso.

### Aplicación Móvil

1. **URL**: `https://TU_IP_DEL_SERVIDOR/mobile/`
2. Abre esta URL en el navegador de tu dispositivo móvil
3. Usa las mismas credenciales del panel web

### API REST

- **Base URL**: `https://TU_IP_DEL_SERVIDOR/api/`
- **Documentación**: `https://TU_IP_DEL_SERVIDOR/api/info`
- **Health Check**: `https://TU_IP_DEL_SERVIDOR/api/health`

## Configuración Post-Instalación

### 1. Cambiar Contraseña del Administrador

```bash
# Método 1: Desde la línea de comandos
sudo remote-admin-ctl reset-admin "nueva_contraseña_segura"

# Método 2: Desde el panel web
# Ir a Configuración > Usuarios > Editar admin
```

### 2. Configurar Certificados SSL Válidos (Recomendado)

```bash
# Opción A: Usar Let's Encrypt (recomendado para producción)
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d tu-dominio.com

# Opción B: Actualizar certificados autofirmados
sudo remote-admin-ctl update-ssl
```

### 3. Configurar Firewall

```bash
# Ver estado actual del firewall
sudo ufw status

# El firewall se configura automáticamente, pero puedes ajustarlo:
sudo ufw allow from TU_RED_LOCAL to any port 5000  # Solo red local
sudo ufw deny 5000  # Bloquear acceso directo al puerto 5000 desde Internet
```

### 4. Configurar Backup Automático (Opcional)

```bash
# Crear backup manual
sudo remote-admin-ctl backup /ruta/a/backup

# Configurar backup automático con cron
sudo crontab -e

# Agregar línea para backup diario a las 2 AM:
# 0 2 * * * /usr/local/bin/remote-admin-backup /var/backups/remote-admin/$(date +\%Y\%m\%d)
```

## Gestión del Servicio

### Comandos Básicos

```bash
# Ver estado
sudo remote-admin-ctl status

# Iniciar servicio
sudo remote-admin-ctl start

# Detener servicio
sudo remote-admin-ctl stop

# Reiniciar servicio
sudo remote-admin-ctl restart

# Ver logs
sudo remote-admin-ctl logs

# Ver logs en tiempo real
sudo remote-admin-ctl logs -f
```

### Gestión de Usuarios

```bash
# Listar usuarios
sudo remote-admin-ctl users list

# Crear nuevo usuario
sudo remote-admin-ctl users create usuario email@ejemplo.com contraseña user

# Crear nuevo administrador
sudo remote-admin-ctl users create admin2 admin2@ejemplo.com contraseña admin
```

### Backup y Restauración

```bash
# Crear backup
sudo remote-admin-ctl backup /ruta/destino

# Restaurar desde backup
sudo remote-admin-ctl restore /ruta/al/backup
```

## Configuración Avanzada

### Archivo de Configuración Principal

Ubicación: `/etc/remote-admin/server.conf`

```ini
[server]
host = 0.0.0.0
port = 5000
debug = false

[security]
secret_key = tu_clave_secreta_aqui
jwt_secret_key = tu_clave_jwt_aqui

[database]
url = sqlite:///var/lib/remote-admin/server/src/database/app.db

[logging]
level = INFO
file = /var/log/remote-admin/server.log
```

### Configuración de Nginx

Ubicación: `/etc/nginx/sites-available/remote-admin`

Para modificar la configuración de Nginx:

```bash
sudo nano /etc/nginx/sites-available/remote-admin
sudo nginx -t  # Verificar configuración
sudo systemctl reload nginx  # Aplicar cambios
```

### Configuración de Base de Datos Externa (Opcional)

Para usar PostgreSQL o MySQL en lugar de SQLite:

1. Instalar el driver correspondiente:
```bash
# Para PostgreSQL
sudo -u remote-admin /var/lib/remote-admin/server/venv/bin/pip install psycopg2-binary

# Para MySQL
sudo -u remote-admin /var/lib/remote-admin/server/venv/bin/pip install PyMySQL
```

2. Modificar la configuración en `/etc/remote-admin/server.conf`:
```ini
[database]
# PostgreSQL
url = postgresql://usuario:contraseña@localhost/remote_admin

# MySQL
url = mysql+pymysql://usuario:contraseña@localhost/remote_admin
```

3. Reiniciar el servicio:
```bash
sudo systemctl restart remote-admin-server
```

## Solución de Problemas

### El servicio no inicia

```bash
# Ver logs detallados
sudo journalctl -u remote-admin-server -f

# Verificar configuración
sudo remote-admin-ctl check-health

# Verificar permisos
sudo chown -R remote-admin:remote-admin /var/lib/remote-admin
```

### No se puede acceder al panel web

```bash
# Verificar que Nginx está funcionando
sudo systemctl status nginx

# Verificar configuración de Nginx
sudo nginx -t

# Verificar firewall
sudo ufw status

# Verificar puertos
sudo netstat -tlnp | grep -E ':(80|443)'
```

### Problemas de certificados SSL

```bash
# Regenerar certificados autofirmados
sudo remote-admin-ctl update-ssl

# Verificar certificados
sudo openssl x509 -in /etc/ssl/certs/remote-admin.crt -text -noout
```

### Error de base de datos

```bash
# Verificar permisos de la base de datos
sudo ls -la /var/lib/remote-admin/server/src/database/

# Recrear base de datos (⚠️ ESTO BORRARÁ TODOS LOS DATOS)
sudo systemctl stop remote-admin-server
sudo rm /var/lib/remote-admin/server/src/database/app.db
sudo systemctl start remote-admin-server
```

### Problemas de conectividad móvil

1. Verificar que el dispositivo móvil está en la misma red
2. Verificar que el firewall permite conexiones desde la red local
3. Usar la IP correcta del servidor (no localhost)
4. Verificar certificados SSL (aceptar certificado autofirmado si es necesario)

## Desinstalación

### Desinstalación Completa

```bash
# Detener y deshabilitar servicios
sudo systemctl stop remote-admin-server
sudo systemctl disable remote-admin-server

# Desinstalar paquete
sudo apt remove remote-admin-server

# Purgar datos (opcional)
sudo apt purge remote-admin-server

# Eliminar datos manualmente (si no se purgó)
sudo rm -rf /var/lib/remote-admin
sudo rm -rf /var/log/remote-admin
sudo rm -rf /etc/remote-admin
sudo userdel remote-admin
```

### Desinstalación Conservando Datos

```bash
# Solo desinstalar el paquete
sudo apt remove remote-admin-server

# Los datos se mantienen en:
# - /var/lib/remote-admin (aplicación y base de datos)
# - /var/log/remote-admin (logs)
# - /etc/remote-admin (configuración)
```

## Seguridad

### Mejores Prácticas

1. **Cambiar credenciales por defecto** inmediatamente
2. **Usar certificados SSL válidos** en producción
3. **Configurar firewall** para limitar acceso
4. **Actualizar regularmente** el sistema y dependencias
5. **Monitorear logs** regularmente
6. **Hacer backups** periódicos
7. **Usar contraseñas fuertes** para todos los usuarios

### Configuración de Firewall Avanzada

```bash
# Permitir solo desde red local
sudo ufw allow from 192.168.1.0/24 to any port 443
sudo ufw allow from 10.0.0.0/8 to any port 443

# Bloquear acceso directo al puerto 5000 desde Internet
sudo ufw deny 5000

# Permitir SSH solo desde IPs específicas
sudo ufw delete allow ssh
sudo ufw allow from TU_IP_ADMIN to any port 22
```

### Monitoreo y Alertas

```bash
# Configurar logrotate para logs
sudo nano /etc/logrotate.d/remote-admin

# Contenido del archivo:
/var/log/remote-admin/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 remote-admin remote-admin
    postrotate
        systemctl reload remote-admin-server
    endscript
}
```

## Soporte y Documentación

No hay xd

### Información del Sistema

```bash
# Ver información completa del sistema
sudo remote-admin-ctl info

# Ver configuración actual
sudo remote-admin-ctl config

# Verificar salud del sistema
sudo remote-admin-ctl check-health
```

### Logs Importantes

- **Servicio principal**: `sudo journalctl -u remote-admin-server`
- **Nginx**: `sudo tail -f /var/log/nginx/error.log`
- **Aplicación**: `sudo tail -f /var/log/remote-admin/server.log`
- **Sistema**: `sudo tail -f /var/log/syslog`

---

## Conclusión

Remote Admin Server proporciona una solución completa y segura para la administración remota de dispositivos. Siguiendo este tutorial, tendrás un sistema completamente funcional y seguro.

**¡Importante!** Recuerda cambiar las credenciales por defecto y configurar certificados SSL válidos antes de usar en producción.

Garantia hasta que te salgas de el repostorio, echo por los bastardos del sur.

