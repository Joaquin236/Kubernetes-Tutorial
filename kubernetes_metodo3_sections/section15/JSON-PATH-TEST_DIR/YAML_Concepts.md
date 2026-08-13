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
   - name:
     owner:
     created:
     status:

## 120.5º 
