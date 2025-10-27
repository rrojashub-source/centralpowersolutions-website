# 🚀 INSTRUCCIONES PARA DEPLOYMENT - Central Power Solutions

## Comparación de Opciones

### **Opción A: VPS (Recomendada para producción)**
✅ Mayor control y flexibilidad  
✅ Mejor rendimiento  
✅ SSL gratis con Let's Encrypt  
✅ Escalabilidad futura  
❌ Requiere configuración técnica  

### **Opción B: Hostinger (Más rápida y simple)**
✅ Configuración muy simple  
✅ Panel visual fácil de usar  
✅ SSL incluido automáticamente  
✅ Soporte técnico disponible  
❌ Menos control técnico  

---

## 🎯 RECOMENDACIÓN

**Para lanzamiento rápido → HOSTINGER**  
**Para máximo control y rendimiento → VPS**

Como ya tienes ambos, te sugiero:
1. **PRIMERO → Hostinger** (lanzar rápido, 10-15 minutos)
2. **DESPUÉS → VPS** (cuando tengas tiempo para configurar bien)

---

## 📋 OPCIÓN A: DEPLOYMENT EN VPS

### Requisitos Previos:
- VPS con Ubuntu/Debian
- Acceso SSH
- Dominio apuntando a IP del VPS

### Instrucciones para Claude Code (Nexus):

```markdown
# TAREA: Deploy sitio web Central Power Solutions en VPS

## CONTEXTO:
- Sitio web estático HTML en carpeta `cps-actualizado/`
- Necesito configurar Nginx + SSL
- Dominio: [TU_DOMINIO.com]
- VPS IP: [TU_IP_VPS]

## INSTRUCCIONES DETALLADAS:

### 1. CONECTAR AL VPS
```bash
ssh root@[TU_IP_VPS]
# o
ssh usuario@[TU_IP_VPS]
```

### 2. ACTUALIZAR SISTEMA
```bash
sudo apt update && sudo apt upgrade -y
```

### 3. INSTALAR NGINX
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 4. CONFIGURAR FIREWALL
```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
```

### 5. CREAR DIRECTORIO DEL SITIO
```bash
sudo mkdir -p /var/www/centralpowersolutions.com
sudo chown -R $USER:$USER /var/www/centralpowersolutions.com
sudo chmod -R 755 /var/www/centralpowersolutions.com
```

### 6. SUBIR ARCHIVOS AL VPS
Desde tu computadora local:
```bash
# Opción A: Con SCP
scp -r cps-actualizado/* usuario@[TU_IP_VPS]:/var/www/centralpowersolutions.com/

# Opción B: Con RSYNC (mejor)
rsync -avz --progress cps-actualizado/* usuario@[TU_IP_VPS]:/var/www/centralpowersolutions.com/
```

### 7. CONFIGURAR NGINX
Crear archivo de configuración:
```bash
sudo nano /etc/nginx/sites-available/centralpowersolutions.com
```

Pegar esta configuración:
```nginx
server {
    listen 80;
    listen [::]:80;
    
    server_name centralpowersolutions.com www.centralpowersolutions.com;
    root /var/www/centralpowersolutions.com;
    index index.html;

    # Logs
    access_log /var/log/nginx/centralpowersolutions-access.log;
    error_log /var/log/nginx/centralpowersolutions-error.log;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

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

### 8. ACTIVAR SITIO
```bash
sudo ln -s /etc/nginx/sites-available/centralpowersolutions.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 9. INSTALAR SSL (Let's Encrypt)
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d centralpowersolutions.com -d www.centralpowersolutions.com
```

### 10. VERIFICAR
Abrir navegador: https://centralpowersolutions.com

## RESULTADO ESPERADO:
✅ Sitio accesible por HTTPS
✅ Redirección automática HTTP → HTTPS
✅ Certificado SSL válido
✅ Todas las imágenes y PDFs cargando
```

---

## 📋 OPCIÓN B: DEPLOYMENT EN HOSTINGER

### Instrucciones para Claude Code (Nexus):

```markdown
# TAREA: Subir sitio web Central Power Solutions a Hostinger

## MÉTODO 1: Panel hPanel (MÁS FÁCIL)

### PASOS:

1. **Acceder a Hostinger:**
   - Ir a: https://hostinger.com
   - Login con tus credenciales
   - Ir a "Hosting" → Seleccionar tu plan

2. **Administrador de Archivos:**
   - Click en "Administrador de archivos" (File Manager)
   - Navegar a carpeta `public_html`
   - **IMPORTANTE**: Eliminar archivos por defecto (index.html de Hostinger)

3. **Subir Archivos:**
   - Click en "Subir archivos" (Upload)
   - Seleccionar TODA la carpeta `cps-actualizado/`:
     - index.html
     - Carpeta images/ (completa)
     - Carpeta Catalogos/ (completa)
   - Esperar a que termine la subida

4. **Verificar Estructura:**
   ```
   public_html/
   ├── index.html
   ├── images/
   │   ├── logo.jpg
   │   ├── LogoCPSanimado.mp4
   │   └── (todos los logos)
   └── Catalogos/
       └── (todos los PDFs)
   ```

5. **Configurar Dominio:**
   - Ir a "Dominios" en el panel
   - Verificar que tu dominio apunte a este hosting
   - Si es nuevo: Esperar 24-48h propagación DNS

6. **Activar SSL (Automático):**
   - Ir a "Avanzado" → "SSL"
   - Click en "Instalar SSL"
   - Hostinger lo hace automáticamente

7. **Verificar:**
   - Abrir: https://[TU_DOMINIO].com
   - Verificar que todo carga correctamente

## MÉTODO 2: FTP (Alternativo)

### Credenciales FTP (obtener de Hostinger):
- Host: ftp.tudominio.com
- Usuario: [de Hostinger]
- Puerto: 21
- Password: [de Hostinger]

### Con FileZilla:
1. Conectar con credenciales FTP
2. Ir a carpeta `public_html`
3. Arrastrar archivos de `cps-actualizado/` al servidor
4. Esperar transferencia completa

## MÉTODO 3: Git Deploy (Avanzado)

Si Hostinger tiene Git Deploy habilitado:
```bash
# En Hostinger panel
1. Ir a "Git" → "Create Repository"
2. Conectar con tu repo GitHub
3. Branch: main
4. Deploy automático
```

## RESULTADO ESPERADO:
✅ Sitio visible en tu dominio
✅ HTTPS activado automáticamente
✅ Todas las imágenes cargando
✅ PDFs descargables
✅ Formulario funcional
```

---

## 🎯 PROMPT COMPLETO PARA CLAUDE CODE

**Copia esto textual a Claude Code / Nexus:**

```
Necesito que me ayudes a deployar mi sitio web de Central Power Solutions.

CONTEXTO:
- Tengo un sitio web estático HTML en la carpeta `cps-actualizado/`
- Tengo un dominio: [TU_DOMINIO.com]
- Tengo un VPS con Ubuntu
- También tengo cuenta en Hostinger

ARCHIVOS DEL PROYECTO:
- cps-actualizado/index.html (archivo principal)
- cps-actualizado/images/ (logos y video)
- cps-actualizado/Catalogos/ (3 PDFs)

OPCIÓN PREFERIDA: [ELIGE UNA]
- [ ] VPS con Nginx (máximo control)
- [ ] Hostinger (más rápido)

INFORMACIÓN NECESARIA:
- IP del VPS: [TU_IP]
- Dominio: [TU_DOMINIO.com]
- Usuario SSH: [TU_USUARIO]

TAREA:
1. Dame instrucciones paso a paso para subir el sitio
2. Configura Nginx si es VPS
3. Configura SSL/HTTPS
4. Verifica que todo funcione

IMPORTANTE:
- Los archivos están en: cps-actualizado/
- El sitio debe ser accesible por HTTPS
- Todos los PDFs deben ser descargables
- El formulario debe funcionar

¿Qué necesitas saber primero para comenzar?
```

---

## 📊 COMPARACIÓN TIEMPO DE SETUP

| Tarea | Hostinger | VPS |
|-------|-----------|-----|
| Subir archivos | 5 min | 10 min |
| Configurar servidor | Automático | 15 min |
| SSL/HTTPS | Automático | 5 min |
| **TOTAL** | **~10-15 min** | **~30-40 min** |

---

## ✅ VERIFICACIÓN POST-DEPLOYMENT

Después del deployment, verificar:

```bash
# 1. Sitio accesible
curl -I https://tudominio.com

# 2. SSL válido
curl -vI https://tudominio.com 2>&1 | grep "SSL certificate verify"

# 3. Archivos estáticos
curl -I https://tudominio.com/images/logo.jpg
curl -I https://tudominio.com/Catalogos/Transformadores%20Pad-mounted.pdf

# 4. HTML cargando
curl https://tudominio.com | grep "Central Power Solutions"
```

### Checklist Visual:
- [ ] Logo visible en header
- [ ] Video animado reproduce
- [ ] Menú funciona
- [ ] Secciones expandibles funcionan
- [ ] Formulario se envía
- [ ] PDFs se descargan
- [ ] Responsive en móvil
- [ ] HTTPS activo (candado verde)

---

## 🆘 TROUBLESHOOTING

### Problema: Imágenes no cargan
```bash
# Verificar permisos
chmod -R 755 /var/www/centralpowersolutions.com
```

### Problema: 404 Error
```bash
# Verificar ruta correcta en Nginx config
sudo nginx -t
sudo systemctl reload nginx
```

### Problema: SSL no funciona
```bash
# Renovar certificado
sudo certbot renew --dry-run
```

---

## 💡 RECOMENDACIÓN FINAL

**Para lanzamiento HOY:**
→ Usa **Hostinger** (15 minutos, sin errores)

**Para producción óptima:**
→ Usa **VPS** (mejor rendimiento, más control)

**Lo ideal:**
1. Lanzar en Hostinger HOY
2. Configurar VPS perfectamente esta semana
3. Migrar cuando esté listo

---

**¿Necesitas las credenciales exactas o ayuda con algún paso específico?**
