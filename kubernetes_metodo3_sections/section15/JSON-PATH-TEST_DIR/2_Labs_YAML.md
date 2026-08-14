## ¿Que caracter ASCCI se usa par delimitar la clave y el valor?
Bettween the key and value in YAML can use the colon character [":"]

## Se ha creado una lista. ¿Cuantos array keys hay en la lista?
Fruits:
  - Orange
  - Apple
  - Banana
Vegetables:
  - Carrot
  - CauliFlower
  - Tomato
--------------
This yaml have two array keys

## ¿Cual es la afirmación correcta?
Dictionary is an unordered collection whereas list is an ordered collection --> TRUE

## Crea un array básico de ensayo
nano practice1.yaml
property1: value1
property2: value2

## Crea un array básico con valores informativos
nano practice2.yaml
name: apple
color: red
weight: 90g

## Crea un array básico con valores de un empleado
nano practice3.yaml
employee:
  name: 101010
  gender: male
  age: 24

## Crea un array intermedio con valores informativos
nano practice4.yaml
employee:
  name: 101010
  gender: male
  age: 24
  address:
    city: edison
    state: new jersey
    country: united states

## Crea un array con 4 valores
nano practice5.yaml
- apple
- apple
- apple
- apple

## Crea un array con 6 valores
nano practice6.yaml
- apple
- apple
- apple
- apple
- apple
- apple

##  Crea un array informativo sobre frutas y sus propiedades
nano practice7.yaml
- name: apple
  color: red
  weight: 100g
- name: orange
  color: orange
  weight: 90g
- name: mango
  color: yellow
  weight: 150g
  
## Crea un array informativo sobre empleados
nano practice8.yaml
employees:
 - employee:
    name: 1010101
    gender: BOT
    age: 2
 - employee:
    name: 10101
    gender: BOT
    age: 4
 - employee:
    name: 101001
    gender: BOT
    age: 6

## Crea un array informativo sobre empleados_2
nano practice8.yaml
employees:
  - name: john
    gender: male
    age: 24
  - name: sarah
    gender: female
    age: 28

## Crea un array informativo sobre empleados_3
nano practice9.yaml
employee:
  name: john
  gender: male
  age: 24
  address:
    city: edison
    state: new jersey
    country: united states
  payslips:
    - month: june
      amount: 1400
    - month: july
      amount: 2400
    - month: august
      amount: 3400
