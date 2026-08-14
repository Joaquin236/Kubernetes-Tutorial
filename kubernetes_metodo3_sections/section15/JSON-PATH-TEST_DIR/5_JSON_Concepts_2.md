## 121.1º En las consultas del color del coche o el autobús se puede obtener con ["$.car.color"] y ["$.bus.color"]. Pero lo ofrece por separado, si queremos obtener todos los colores, ["$.*.color"] y para los precios ["$.*.color"]

## 121.2º Cuando hay una lista o diccionario, nos enfocamos en el número de índice para localizar el objetivo. $[0].model si usamos "*" muestra todos los índices $[*].model y todos los valores

## 121.3º Para mostrar el primer modelo de rueda --> $.car.wheels[0].model

## 121.4º Para mostrar todos los modelos de ruedas --> $.car.wheels[*].model

## 121.3º Para mostrar los modelos de ruedas de bus --> $.bus.wheels[*].model

## 121.4º Para mostrar todos los valores se establece un ["*"] en la clave de inicio de raíz --> $*.wheels[*].model
