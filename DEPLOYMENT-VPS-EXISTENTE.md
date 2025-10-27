# 🚀 DEPLOYMENT EN VPS EXISTENTE (Con CRM corriendo)

## 🎯 TU SITUACIÓN

- ✅ VPS ya configurado con Nginx
- ✅ CRM corriendo en el mismo dominio
- ✅ SSL ya configurado
- ❓ Necesitas agregar el sitio web sin afectar el CRM

---

## 📋 OPCIONES DISPONIBLES

### **Opción A: Subdominio** (RECOMENDADA)
- URL: `www.centralpowersolutions.com`
- Ventajas: ✅ No toca el CRM, ✅ Fácil de configurar
- Ejemplo: CRM en `centralpowersolutions.com`, Web en `www.centralpowersolutions.com`

### **Opción B: Ruta específica**
- URL: `centralpowersolutions.com/web`
- Ventajas: ✅ Mismo dominio principal
- Desventaja: ⚠️ URLs más largas

### **Opción C: Dominio diferente**
- Si tienes otro dominio disponible
- Totalmente separado del CRM

---

## 🎯 OPCIÓN RECOMENDADA: SUBDOMINIO `www`

Esta es la más común y profesional. El CRM se queda en el dominio raíz y el sitio web en www.

---

## 📝 INSTRUCCIONES PARA CLAUDE CODE (NEXUS)

**Copia este prompt EXACTO:**

```
Necesito agregar el sitio web de Central Power Solutions a mi VPS existente.

SITUACIÓN ACTUAL:
- Tengo un VPS corriendo con Nginx
- Ya tengo un CRM funcionando en: centralpowersolutions.com
- Quiero agregar el sitio web sin afectar el CRM
- SSL ya está configurado en el VPS

ARCHIVOS DEL SITIO WEB:
- Ubicación local: cps-actualizado/
- Contiene: index.html, images/, Catalogos/

OPCIÓN PREFERIDA:
Quiero configurar el sitio web en: www.centralpowersolutions.com
(El CRM se queda en centralpowersolutions.com)

TAREA:
1. Crear nueva configuración Nginx para www.centralpowersolutions.com
2. Crear directorio para el sitio web
3. Dame instrucciones para subir los archivos
4. Configurar SSL para www.centralpowersolutions.com
5. Verificar que NO afecte el CRM existente

INFORMACIÓN DEL VPS:
- IP: [TU_IP_VPS]
- Usuario SSH: [TU_USUARIO]
- El CRM ya usa Nginx en este VPS

IMPORTANTE:
- No tocar la configuración del CRM
- El sitio web debe tener su propio virtual host en Nginx
- Necesito comandos paso a paso

¿Qué archivos de configuración Nginx necesitas ver primero?
```

---

## 🔧 COMANDOS RÁPIDOS (Si quieres hacerlo tú)

### 1. CREAR DIRECTORIO PARA EL SITIO WEB

```bash
# Conectar al VPS
ssh usuario@TU_IP_VPS

# Crear directorio para el sitio web
sudo mkdir -p /var/www/web-centralpowersolutions
sudo chown -R $USER:$USER /var/www/web-centralpowersolutions
sudo chmod -R 755 /var/www/web-centralpowersolutions
```

### 2. SUBIR ARCHIVOS DESDE TU PC LOCAL

```bash
# Desde tu computadora (NO desde el VPS)
rsync -avz --progress cps-actualizado/* usuario@TU_IP_VPS:/var/www/web-centralpowersolutions/
```

### 3. CREAR CONFIGURACIÓN NGINX PARA WWW

```bash
# En el VPS
sudo nano /etc/nginx/sites-available/www-centralpowersolutions
```

**Pegar esta configuración:**

```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name www.centralpowersolutions.com;
    root /var/www/web-centralpowersolutions;
    index index.html;

    # Logs separados
    access_log /var/log/nginx/www-centralpowersolutions-access.log;
    error_log /var/log/nginx/www-centralpowersolutions-error.log;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript 
               application/javascript application/xml+rss application/json;

    # Cache control para recursos estáticos
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|webp|mp4|pdf)$ {
        expires 7d;
        add_header Cache-Control "public, immutable";
    }

    # Main location
    location / {
        try_files $uri $uri/ =404;
    }

    # Seguridad
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
}
```

### 4. ACTIVAR SITIO

```bash
# Crear symlink
sudo ln -s /etc/nginx/sites-available/www-centralpowersolutions /etc/nginx/sites-enabled/

# Verificar configuración (NO debe dar errores)
sudo nginx -t

# Recargar Nginx (NO afecta el CRM)
sudo systemctl reload nginx
```

### 5. CONFIGURAR DNS

**En tu proveedor de dominio (GoDaddy, Namecheap, etc):**

Agregar registro DNS:
```
Tipo: A
Host: www
Valor: [IP_DE_TU_VPS]
TTL: 3600
```

**Esperar 5-30 minutos para propagación DNS**

### 6. AGREGAR SSL PARA WWW

```bash
# Agregar certificado para www (si Certbot ya está instalado)
sudo certbot --nginx -d www.centralpowersolutions.com

# Certbot actualizará automáticamente la configuración Nginx
```

### 7. VERIFICAR

```bash
# Verificar que el sitio carga
curl -I https://www.centralpowersolutions.com

# Ver configuraciones activas
ls -la /etc/nginx/sites-enabled/

# Ver logs si hay problemas
sudo tail -f /var/log/nginx/www-centralpowersolutions-error.log
```

---

## ✅ RESULTADO FINAL

- ✅ `centralpowersolutions.com` → CRM (sin cambios)
- ✅ `www.centralpowersolutions.com` → Sitio Web nuevo
- ✅ Ambos con HTTPS
- ✅ Completamente independientes

---

## 🔍 VERIFICACIÓN

**Después de configurar, verificar:**

```bash
# 1. Ver configuraciones Nginx activas
ls -la /etc/nginx/sites-enabled/

# 2. Ver procesos escuchando en puerto 80/443
sudo netstat -tulpn | grep nginx

# 3. Test de sintaxis Nginx
sudo nginx -t

# 4. Ver todos los server_name configurados
grep -r "server_name" /etc/nginx/sites-enabled/
```

---

## ⚠️ IMPORTANTE: NO TOCAR

**Archivos/Configuraciones del CRM:**
- NO modificar `/etc/nginx/sites-enabled/[archivo-del-crm]`
- NO tocar directorios del CRM
- NO reiniciar Nginx, solo RELOAD (`systemctl reload`)

**RELOAD vs RESTART:**
```bash
# ✅ CORRECTO - No interrumpe conexiones
sudo systemctl reload nginx

# ❌ EVITAR - Puede afectar CRM momentáneamente
sudo systemctl restart nginx
```

---

## 🆘 TROUBLESHOOTING

### Problema: "www no carga"
```bash
# Verificar DNS
dig www.centralpowersolutions.com

# Ver errores Nginx
sudo tail -50 /var/log/nginx/error.log
```

### Problema: "Conflicto de configuración"
```bash
# Ver todas las configuraciones
sudo nginx -T | grep server_name

# Verificar sintaxis
sudo nginx -t
```

### Problema: "SSL falla"
```bash
# Verificar certificados
sudo certbot certificates

# Renovar si es necesario
sudo certbot renew --dry-run
```

---

## 📊 ESTRUCTURA FINAL EN VPS

```
/var/www/
├── crm-existente/          # Tu CRM (no tocar)
│   └── [archivos del CRM]
└── web-centralpowersolutions/  # Sitio web nuevo
    ├── index.html
    ├── images/
    └── Catalogos/

/etc/nginx/sites-available/
├── crm-config              # Configuración CRM (no tocar)
└── www-centralpowersolutions  # Nueva configuración

/etc/nginx/sites-enabled/
├── crm-config -> ../sites-available/crm-config
└── www-centralpowersolutions -> ../sites-available/www-centralpowersolutions
```

---

## 🎯 ALTERNATIVA: Ruta en vez de subdominio

Si prefieres `centralpowersolutions.com/web` en vez de `www.`, modifica la configuración del CRM:

```nginx
# En la configuración EXISTENTE del CRM, agregar:
location /web {
    alias /var/www/web-centralpowersolutions;
    index index.html;
    try_files $uri $uri/ =404;
}
```

**Ventaja:** No necesitas configurar DNS  
**Desventaja:** URLs más largas

---

## ✅ CHECKLIST FINAL

- [ ] Directorio creado: `/var/www/web-centralpowersolutions`
- [ ] Archivos subidos desde `cps-actualizado/`
- [ ] Configuración Nginx creada y activada
- [ ] DNS configurado (registro A para www)
- [ ] SSL configurado con Certbot
- [ ] Sitio accesible en https://www.centralpowersolutions.com
- [ ] CRM sigue funcionando en centralpowersolutions.com
- [ ] Sin errores en logs de Nginx

---

**¿Listo para empezar? Dale el prompt a Claude Code y él te guiará paso a paso.**
