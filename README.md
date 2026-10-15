# TP2
#Preparación 
#A.1. Herramientas y directorios de trabajo
$ command -v bash python 3 tar bzip2 losetup fdisk mkfs.ext4

$sudo install -d -o  "$USER" -g "$(id-gn)" -m 750 /srv/tp2
$sudo mkdir -p /srv/tmp2/{bin,lib,config,reportes,restauraciones}
$sudo mkdir -p /srv/tmp2/datos/{config,documentos,logs}

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
$sudo lbslk

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

$nano /srv/tp2/config/tp2.config
#dentro del editor:
$ORIGEN="/srv/tp2/datos/"
$PUNTO_MONTAJE="/mnt/respaldo_tp2"
$UUID_ESPERADO="${UUID_REAL}"
$DESTINO_BACKUP="\${PUNTO_MONTAJE}/copias/tp2"
$BASE_RESTAURACION=/srv/tp2/restauraciones"

#crear
$echo "Subprocesos" > /srv/tp2/datos/config/entrada_a.config
$chmod 640 /srv/tp2/datos/config/entrada_a.config

$echo "Retornos" > /srv/tp2/datos/config/entrada_b.config
$chmod 600 /srv/tp2/datos/config/entrada_b.config

#enlace simbólico
$ln -s /srv/tp2/datos/config/entrada_a.config /srv/tp2/datos/config/vigente.config

#documentos de texto
$echo "Informe Inicial" > /srv/tp2/datos/documentos/informe_inicial.txt
$echo "Reporte Final" > /srv/tp2/datos/documentos/reorte_final.txt

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


