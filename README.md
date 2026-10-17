# TP2
#Registro del dispositivo
$lsb_release -a
$arch
$id
$whoami

#Preparación 
#A.1. Herramientas y directorios de trabajo
$ command -v bash python 3 tar bzip2 losetup fdisk mkfs.ext4

$sudo install -d -o  "$USER" -g "$(id-gn)" -m 750 /srv/tp2
$sudo mkdir -p /srv/tp2/{bin,lib,config,reportes,restauraciones}
$sudo mkdir -p /srv/tp2/datos/{config,documentos,logs}

#A.2. Imagen y dispositivo loop
#Toma de Snapshot
$sudo mkdir -p /var/lib/tp2
$sudo dd if=/dev/zero of=/var/lib/tp2/respaldo.img bs=1M count=512 conv=excl
$LOOP_TP2=$(sudo losetup --find --show --partscan /var/lib/tp2/respaldo.img)
$printf ("%s\n""$LOOP_T2"
$sudo losetup "$LOOP_TP2"
$lbslk -o NAME,SIZE,TYPE,FSTYPR,MOUNTPOINTS"$LOOP_TP2"

#A.3. Particionado
$sudo fdisk "$LOOP_TP2" <<EOF
n
p
1


p
w
EOF

#verificación 
$sudo lsblk

#A.4. Formateo
$PART_TP2="${LOOP_TP2}P1"
$lbslk -o NAME,SIZE,TYPE,FSTYPE "$LOOP_TP2"
$sudo mkfs.ext4 -L TP2_BACKUP "$PART_TP2"
$sudo mkdir -p /mnt/respaldo_tp2
$sudo mount -o noexec,nosuid,nodev "$PART_TP2" /mnt/respaldo_tp2
$sudo install -d -o "$USER" -g "$(id-gn)" -m 750 /mnt/respaldo_tp2
$findmnt --mountpoint /mnt/respaldo_tp2
$sudo blkid "$PART_TP2"

#A.5. Configuración
$UUID_REAL=""

$nano /srv/tp2/config/tp2.conf
#dentro del editor:
$ORIGEN="/srv/tp2/datos/"
$PUNTO_MONTAJE="/mnt/respaldo_tp2"
$UUID_ESPERADO="${UUID_REAL}"
$DESTINO_BACKUP="${PUNTO_MONTAJE}/copias/tp2"
$BASE_RESTAURACION="/srv/tp2/restauraciones"

#crear
$echo "Subprocesos" > /srv/tp2/datos/config/entrada_a.config
$chmod 640 /srv/tp2/datos/config/entrada_a.config

$echo "Retornos" > /srv/tp2/datos/config/entrada_b.config
$chmod 600 /srv/tp2/datos/config/entrada_b.config

#enlace simbólico
$cd /srv/tp2/datos/config
$ln -s entrada_a.config vigente.config

#verificacion
$ls -la /srv/tp2/datos/config

#documentos de texto
$echo "Informe Inicial" > /srv/tp2/datos/documentos/informe_inicial.txt
$echo "Reporte Final" > /srv/tp2/datos/documentos/reporte final.txt

#con identificador de prueba
$echo "ID PRUEBA TP2" > /srv/tp2/datos/.control

#logs
$nano /srv/tp2/datos/logs/operacion_01.log
#dentro del editor
2026-09-30T08:00:00|INFO|auth|Inicio de sesion correcto 
2026-09-30T08:01:00|WARN|auth|Intento fallido 
2026-09-30T08:02:00|INFO|repo|Archivo recibido 
2026-09-30T08:03:00|ERROR|repo|Escritura rechazada 
2026-09-30T08:04:00|INFO|backup|Respaldo iniciado 
2026-09-30T08:05:00|WARN|backup|Espacio reducido 

$nano /srv/tp2/datos/logs/operacion_02.log
#dentro del editor
2026-09-30T09:00:00|INFO|auth|Sesion finalizada 
2026-09-30T09:01:00|ERROR|auth|Cuenta bloqueada 
2026-09-30T09:02:00|INFO|repo|Archivo consultado 
2026-09-30T09:03:00|WARN|repo|Cuota proxima al limite 
2026-09-30T09:04:00|ERROR|backup|Copia interrumpida 
2026-09-30T09:05:00|INFO|auth|Inicio de sesion correcto 

$ls -la /srv/tp2/datos

#Respaldo de datos
$sudo tar -czvpf /backups/respaldo_datos_$(date +%Y%m%d_%H%M%S).tar.gz /srv/tp2

#Herramienta bash
$#!/bin/bash
$set -euo pipefail
$RUTA_A="$(cd "(dirname "${BASH_SOURCE[0]}")" && pwd)" //obtener ruta absoluta
$DIRECTORIO_R="$(cd "${RUTA_A}//.." && pwd)" //obtener directorio raiz
$source "${DIRECTORIO_R}/config/tp2.conf"
$source "${DIRECTORIO_R}/lib/comunes.sh"

$accion="$1"
$case "$accion" in
$  reporte)
$     "generando reporte..."
$      hacer_reporte
$      ;;
$  backup)
$     "generando respaldo..."
$      hacer_backup
$      ;;
$  restaurar)
$     "generando restauración..."
$      hacer_restauracion
$      ;;
$  *)
$     "opción no reconocida: $accion" >&2
$      exit 1
$      ;;
$ esac

$#!/bin/bash
$set -euo pipefail
$validar_montaje(){
$find --mountpoint "$1" > /dev/null || {echo "ERROR: $1 no esta montado" >&2; return 1; }
$[[ "$(findmnt -n -o UUID --mountpoint "$1")" == "$2" ]] || {echo "ERROR: UUID no coincide" >&2; return 1; }
$}

$hacer_reporte(){
$local csv="${DIRECTORIO_R}/reportes/reporte_bash.csv"
$mkdir -p "${DIRECTORIO_R}/reportes"
$echo "servicio,nivel,cantidad" > "$csv"
$for i in auth backup repo; do
$for j in ERROR INFO; do
$echo "${i},${j},$(grep -c "|${j}|${i}|"
"${ORIGEN}/logs"/*.log || true) >> "$csv"
$done
$done
$echo "reporte listo: $csv"
$}

$hacer_backup(){
$
