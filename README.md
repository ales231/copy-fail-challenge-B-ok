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