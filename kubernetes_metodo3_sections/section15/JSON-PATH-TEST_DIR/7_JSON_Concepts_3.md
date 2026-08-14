## 122.1º Cuando se está indexando las listas para obtener el resultado de consulta. Un solo número muestra un solo valor, dos números separados por coma da dos valores sin ordenar. Si usamos dos puntos, muestra el elemento desde el primer número hasta el último número asignado.

## 122.2º Si usamos el patron [n1:n2:n3] el tercer número es el número de salto --> [0:66:4] muestra desde el índice 0 hasta el índice 66 omitiendo 4 elementos.

## 122.3º Para obtener el último de la lista lo más práctico es con el número en negativo, no es correcto escribir el el índice más alto. --> [-1]. El primer índice también puede mostrarse con negativos [-66] pero para los primeros índices es mejor el número positivo. En el jpath el número negativo solo no funciona, necesita un 0 separado del caracter ':' --> $[-1:0], si establecemos $[-3:] --> devuelve los 3 últimos de la lista
