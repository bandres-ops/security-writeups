# Writeup — OverTheWire Bandit (niveles 11 al 27)

## Resumen
Continuación del wargame Bandit. Esta tanda cubrió cifrado básico (ROT13),
descompresión en capas, uso de claves SSH, fuerza bruta con Python,
cron jobs, binarios SUID, y escape de shells restringidas.

## Nivel 11 → 12 — ROT13
tr 'a-zA-Z' 'n-za-mN-ZA-M'
Cifrado de sustitución simple: cada letra se rota 13 posiciones en el abecedario.

## Nivel 12 → 13 — Descompresión en capas
Combiné xxd -r, gunzip, bunzip2 y tar -xf en cadena, usando `file` para
identificar cada capa antes de actuar. El archivo pasaba por 8 formatos
distintos antes de llegar a texto plano.

## Nivel 13 → 14 — Claves SSH
No todos los accesos usan contraseña de texto — este nivel usó una clave
privada SSH (chmod 600 + ssh -i) en vez de contraseña.

## Nivel 19 → 20 — Binarios SUID
Un archivo con permiso "s" en vez de "x" ejecuta comandos con los permisos
de su DUEÑO, no los del usuario que lo corre. Clave para escalada de privilegios.

## Nivel 20 → 21 — Netcat como listener
Usé tmux con dos paneles: uno escuchando con `nc -l puerto`, otro ejecutando
un binario SUID que se conectaba a ese puerto entregando una contraseña.

## Nivel 24 → 25 — Fuerza bruta con Python
Sin atajo lógico posible (10.000 combinaciones de PIN), escribí un script
en Python con sockets que probó cada combinación en una sola conexión,
hasta encontrar la correcta.

## Nivel 25 → 26 — Escape de shell restringida
La shell de este usuario era un script que solo mostraba un archivo y cerraba
la sesión. Logré "romperla" achicando la ventana de terminal para forzar el
paginador a modo interactivo, abriendo vim desde ahí, y cambiando la shell
interna de vim para obtener una terminal real.

## Lecciones aprendidas
- Los binarios SUID y los cron jobs mal configurados son vectores reales
  de escalada de privilegios, no solo ejercicios de práctica
- Cuando algo parece "aleatorio" (como un hash de nombre de archivo), a veces
  se puede recalcular si conocés la lógica que lo generó
- Fuerza bruta simple resuelve problemas sin atajo lógico, pero requiere
  automatización real (no se puede hacer a mano)
