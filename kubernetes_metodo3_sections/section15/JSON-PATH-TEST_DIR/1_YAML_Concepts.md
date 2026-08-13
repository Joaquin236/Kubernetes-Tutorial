## 120.1º Los ficheros YAML son ficheros de configuración con formato para guardalos estructuradamente. Los tres formatos de configuración ["YAML","XML","JSON"] comparten estructuras similares, cambia la forma de expresar la sangría y las claves.

## 120.2º XML
<Servers>
    <Server>
         <name> 101010 </name>
         <owner> 1010111 </owner>
         <created> 13-08-2000 </created>
         <status> Active </status>
    </Server>
</Servers>

## 120.3º JSON
{
    Servers: [
    {
    name: 101010,
    owner: 1010111.
    created: 13-08-2000,
    status: active,
    }
    ]
}

## 120.4º YAML
Servers:
   - name: 101010
     owner: 1010111
     created: 13-08-2000
     status: active

## 120.5º Los elementos que llevan un guión ["-"] están siendo establecidos con un array. También se puede establecer diccionarios. La cabecera no tiene sangría, las claves internas llevan el espaciado. Cuando está mal situado el espaciado, desencadena un error en la sintaxis del fichero de configuración yaml. 

## 120.6º Los diccionarios son con los contenidos desordenados, mientras las lista si están ordenadas. También se puede desarrollar una lista de diccionarios.

## 120.7º El diccionario y el mapa son identicos, el array y la lista son diferentes
