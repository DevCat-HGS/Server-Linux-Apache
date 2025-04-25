**Guía Completa para Desplegar un Proyecto de Git a un Servidor Kali Linux Usando Hooks - Paso a Paso**

---

# Requisitos Previos

- Windows:
  - Tener `git` instalado.
  - Tener tu proyecto clonado o inicializado.

- Kali Linux (Servidor):
  - Tener `git`, `openssh-server` y `apache2` instalados.
  - Tener configurado el servicio SSH y Apache funcionando.


# 1. En Kali Linux (Servidor)

## 1.1 Instalar Dependencias

```bash
sudo apt update
sudo apt install git openssh-server apache2
```

## 1.2 Iniciar y Habilitar el SSH

```bash
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```
(Debe decir "active (running)").


## 1.3 Crear el Repositorio Bare

```bash
cd ~
mkdir hgserver.git
cd hgserver.git
git init --bare
```

Esto crea un repositorio "bare" (sin working directory) para recibir los pushes.


# 2. Configurar el Hook post-receive en Kali

## 2.1 Crear el Hook

```bash
nano ~/hgserver.git/hooks/post-receive
```

Y pega el siguiente contenido:

```bash
#!/bin/bash

# Leer los datos del push
desplegar() {
  GIT_WORK_TREE=/var/www/html GIT_DIR=/home/devcat/hgserver.git git checkout -f main
  chown -R www-data:www-data /var/www/html
  chmod -R 755 /var/www/html
}

while read oldrev newrev refname; do
  if [[ "$refname" == "refs/heads/main" ]]; then
    echo "Desplegando cambios en /var/www/html..."
    desplegar
  fi
done
```

(Si tu usuario no es `devcat`, ajusta la ruta.)

## 2.2 Dar permisos de ejecución al Hook

```bash
chmod +x ~/hgserver.git/hooks/post-receive
```


# 3. En Windows (Cliente)

## 3.1 Configurar el Remote

```bash
cd "C:\Users\ADMIN\Documents\todo list"
git remote remove deploy   # Solo si existía antes.
git remote add deploy ssh://devcat@192.168.1.18/~/hgserver.git
```

(Reemplaza `devcat` si es otro usuario en Kali.)

## 3.2 Primer Push

```bash
git push deploy main
```

Te pedirá contraseña (la del usuario en Kali). Confirma "yes" si pregunta por la autenticidad del host.


# 4. Validar Despliegue

- Ir en navegador a:

```
http://192.168.1.18
```

Deberías ver tu sitio web desplegado.


# 5. Si no ves cambios

## 5.1 Forzar Push (si es necesario)

```bash
git commit --allow-empty -m "Redeploy forzado"
git push deploy main
```

o alternativamente:

```bash
git push deploy +main:main
```


# Resumen Visual de los Comandos en Orden

**En Kali:**
```bash
sudo apt install git openssh-server apache2
sudo systemctl enable ssh
sudo systemctl start ssh
mkdir ~/hgserver.git
cd ~/hgserver.git
git init --bare
nano hooks/post-receive
chmod +x hooks/post-receive
```

**En Windows:**
```bash
cd "C:\Users\ADMIN\Documents\todo list"
git remote add deploy ssh://devcat@192.168.1.18/~/hgserver.git
git push deploy main
```


# Notas Finales

- El hook solo ejecuta `checkout` de la rama `main`.
- Apache debe estar sirviendo `/var/www/html/`.
- Si cambias de red o IP del servidor, deberás actualizar el remote.
- Puedes agregar autenticación con llaves SSH para no pedir contraseña cada vez.

---

✅ **Con esta guía puedes desplegar proyectos a tu servidor local automáticamente al hacer push en Git.**

