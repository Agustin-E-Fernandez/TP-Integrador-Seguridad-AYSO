# TP Integrador – Seguridad en Sistemas
**Materia:** Arquitectura y Sistemas Operativos (AYSO)
**Tecnicatura Universitaria en Programación (TUP) – UTN**
**Alumno:** Agustín Ezequiel Fernández

## 📌 Descripción

Este trabajo práctico integrador aborda el eje temático de **Seguridad en Sistemas Operativos**, con foco en el control de acceso a archivos mediante usuarios, grupos, permisos y auditoría en un entorno Linux (Debian 12).

El objetivo es demostrar, de forma práctica, cómo un sistema operativo Linux protege la confidencialidad de la información sensible utilizando mecanismos nativos: permisos de archivos, gestión de usuarios, registro de auditoría (`auditd`) y delegación controlada de privilegios (`sudo`).

## 🎯 Objetivo del caso práctico

Configurar un entorno con dos usuarios de distinto nivel de acceso sobre un mismo archivo confidencial, demostrando:

1. Restricción de acceso mediante permisos (`chmod`, `chown`)
2. Registro de auditoría de accesos exitosos y fallidos (`auditd`)
3. Delegación de privilegios mínimos necesarios mediante `sudo` (principio de *least privilege*)

## 🛠️ Entorno utilizado

- **Sistema operativo:** Debian 12 (Bookworm), 64 bits
- **Virtualización:** Oracle VM VirtualBox
- **Recursos de la VM:** 4096 MB RAM, 2 procesadores
- **Herramientas:** `adduser`, `chmod`, `chown`, `auditd`, `audispd-plugins`, `sudo`/`visudo`

## 📂 Estructura del repositorio

```
├── 01_usuarios.png                  # Creación de usuarios autorizado/noautorizado
├── 02_permisos.png                  # Restricción de permisos sobre el archivo
├── 03_acceso_denegado.png           # Intento de acceso fallido
├── 04_acceso_autorizado.png         # Acceso exitoso y escritura en el archivo
├── 05_auditd_instalado.png          # Instalación y configuración de auditd
├── 06_log_auditoria_completo.png    # Log de auditoría con eventos fallido y exitoso
├── 07_sudoers_configurado.png       # Validación de la regla sudo (visudo -c)
├── 08_sudo_delegado.png             # Delegación de privilegio mínimo vía sudo
└── README.md
```

## 💻 Caso práctico — Pasos realizados

### 1. Creación de usuarios
Se crearon dos usuarios con distintos niveles de acceso:
```bash
sudo adduser autorizado
sudo adduser noautorizado
```

### 2. Restricción de acceso mediante permisos
Se creó un archivo confidencial y se restringió su acceso exclusivamente al usuario propietario:
```bash
sudo mkdir /seguro
sudo touch /seguro/datos_confidenciales.txt
sudo chown autorizado:autorizado /seguro/datos_confidenciales.txt
sudo chmod 600 /seguro/datos_confidenciales.txt
```

### 3. Prueba de acceso denegado
El usuario `noautorizado` no posee permisos sobre el archivo:
```bash
su - noautorizado
cat /seguro/datos_confidenciales.txt
# cat: /seguro/datos_confidenciales.txt: Permiso denegado
```

### 4. Prueba de acceso autorizado
El usuario `autorizado`, propietario del archivo, accede y escribe sin restricciones:
```bash
su - autorizado
cat /seguro/datos_confidenciales.txt
echo "informacion confidencial de prueba" >> /seguro/datos_confidenciales.txt
```

### 5. Auditoría de accesos con auditd
Se instaló y configuró el sistema de auditoría para vigilar el archivo:
```bash
sudo apt install auditd audispd-plugins
sudo systemctl enable auditd --now
sudo auditctl -w /seguro/datos_confidenciales.txt -p rwxa -k acceso_datos
```

### 6. Consulta del registro de auditoría
Se generaron eventos de acceso (fallido y exitoso) y se consultaron en el log:
```bash
sudo ausearch -k acceso_datos
```
El log muestra ambos eventos diferenciados por usuario, resultado (`success=no`/`success=yes`) y marca de tiempo.

### 7. Delegación de privilegio mínimo con sudo
Se otorgó al usuario `noautorizado` la posibilidad de ejecutar **únicamente** el comando de lectura sobre ese archivo específico, sin acceso a ningún otro recurso del sistema:
```bash
echo 'noautorizado ALL=(ALL) NOPASSWD: /bin/cat /seguro/datos_confidenciales.txt' | sudo tee /etc/sudoers.d/noautorizado_cat
sudo chmod 440 /etc/sudoers.d/noautorizado_cat
sudo visudo -c
```

Se verificó que el privilegio está correctamente acotado: el usuario puede ejecutar `cat` sobre el archivo permitido, pero no sobre ningún otro recurso del sistema (ej. `/etc/shadow`).

## ✅ Resultados obtenidos

- Se validó correctamente el bloqueo de acceso para usuarios no autorizados.
- Se registraron exitosamente en el log de auditoría tanto los intentos fallidos como los accesos exitosos, con usuario, comando y marca de tiempo.
- Se verificó la correcta delegación de privilegios mínimos mediante `sudo`, confirmando que el acceso otorgado está acotado exclusivamente al recurso autorizado.

## 🎓 Conclusiones

El trabajo permitió aplicar de forma práctica los conceptos fundamentales de seguridad en sistemas operativos Linux: control de acceso basado en permisos, auditoría de eventos y delegación de privilegios mínimos. Se comprobó que mecanismos nativos del sistema operativo, sin necesidad de herramientas externas, son suficientes para implementar una política de seguridad robusta sobre información sensible.

## 📚 Bibliografía

- Documentación oficial de Debian: https://www.debian.org/doc/
- Arch Wiki – Security: https://wiki.archlinux.org/title/Security
- Manual de `auditd`: https://man7.org/linux/man-pages/man8/auditd.8.html
- Manual de `sudoers`: https://man7.org/linux/man-pages/man5/sudoers.5.html
