# Writeup — OverTheWire Bandit (niveles 0 al 10)

## Resumen
Wargame de introducción a Linux vía SSH. Resolví los niveles 0 al 10,
cubriendo navegación básica, permisos, find, grep, sort/uniq, strings y base64.

## Nivel 0 → 1
Comando clave: cat readme

## Nivel 4 → 5
Archivos con nombres raros (guión al inicio). Usé ./archivo y file para
identificar cuál tenía texto legible entre varios binarios.

## Nivel 5 → 6
find . -type f -size 1033c ! -executable
Buscar por tamaño exacto y que no sea ejecutable, dentro de subcarpetas.

## Nivel 9 → 10
strings data.txt | grep "="
Extraer texto legible de un binario y filtrar la línea con el patrón buscado.

## Lecciones aprendidas
- Los nombres de archivo raros (guión, espacios, puntos) necesitan ./ o comillas
- file es clave para no perder tiempo abriendo archivos binarios con cat
- find combina múltiples filtros (tamaño, dueño, grupo, tipo) en un solo comando
