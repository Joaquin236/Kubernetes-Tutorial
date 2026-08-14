## 120.1º Los contenidos del JSON se establecen entre llaves y la clave lleva la comilla doble. El valor va detrás de los los puntos. Cuando se quiere hacer una lista se abre otra llave para guardar los valores. 

## 120.2º Cuando se realiza una lista más compleja, detrás de la clave se añade un corchete y una llave que contiene los valores, cada grupo de valor se cierra con la llave y una coma para separar el siguiente valor.

## 120.3º Existen muchas herramientas que permiten convertir los YAML en JSON y viceversa. 

## 120.4º El JSON_PATH es un sistema de consultas para procesar los ficheros JSON. Cuando se realizan consultas en JSON se usa las claves y subclaves para rescatar el valor que buscamos localizar. Los elementos que están dentro de la llave, son interpretados como diccionarios. En la frase de la consulta se añade el caracter ["$"] para marcar la raíz del diccionario, detrás del simbolo de raíz, separamos con un punto y escribimos las claves separadas por puntos. La salida de la consulta se muestran con corchetes en la raíz, el valor estará contenido en las llaves.

## 120.5º En las listas solo se usan los corchetes, no se interpreta como diccionarios al no llevar el símbolo de llaves. En esta situación se usa un número entre corchetes, marcando el índice del valor. el primer índice es el 0. [0,1,2,3,4,5]. --> $[2] Devuelve el valor del indice_2 (Tercer elemento)

## 120.6º Para localizar el modelo de un neumático concreto (2º modelo), necesitamos esta consulta --> ["$.car.wheels[1].model"]. 

## 120.7º En un array numérico, querermos extraer los números mayores a 40, en las consultas JSON se realiza así: ["$[Check_if_each_item_in_the_array_>_40]"], si la frase para obtener la consulta se vuelve muy larga, se puede usar otras frases más cortas para obtener la misma consulta:
[$[?(each item in the list > 40)]
$[?(each item in the list @ > 40)]
$[?(each item in the list @ < 40)]
$[?(each item in the list @ == 40)]
$[?(each item in the list @ != 40)]]

## 120.8º Si necesitamos consultar el modelo de un neumático con la localización 'rear-right' que es el tercer elemento de la lista, la consulta ["$.car.wheels[2].model"] ya no es válida en esta situación porque se ha desplazado el contenido de la lista a otro índice. $.car.wheels[?(@.localization_==_"rear-right")].model