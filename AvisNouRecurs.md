# 1. Introducció
Aquest és un connector conceptualment molt diferent a la resta de connectors de l'API de l'Hèstia. Es tracta d’un connector que requereix que cada ABSS que hi estigui interessada ho implementi i publiqui en el seu propi backoffice d’acord a les especificacions que aquí es detallen. Un cop estigui disponible aquest connector al backoffice de l’ABSS, s’haurà d’enregistrar a l’Hèstia per tal que es pugui consumir des de l’Hèstia de forma similar a una funció callback.

L’objectiu d’aquest connector és informar a l’ABSS que s’ha donat d’alta un nou recurs a l’Hèstia, habitualment de caràcter econòmic, i bloquejar-ho a l’Hèstia per tal que es pugui realitzar la seva resolució (actualització de l’estat) des d’una eina externa integrant aquesta eina amb el connector [`Resolució de recurs`](ResolucioRecurs.md).
Es tracta per tant d’un connector que haurà de publicar l’ABSS interessada a través d'un canal TLS 1.2 o superior, i que s'haurà d'implementar mitjançant una API REST implementada amb el llenguatge de programació que desitgi l'ABSS. L’Hèstia per la seva banda consumirà aquest connector com a client. 

En  aquest apartat es detallen els requeriments que ha de complir el connector que ha d’implementar l’ABSS per tal que l’Hèstia es pugui integrar. És important destacar però, que serà responsabilitat de l’ABSS implementar les mesures de seguretat necessàries per garantir en tot moment la confidencialitat i la integritat de les dades compartides pel connector.



# 2. Requeriments de seguretat
Per tal de garantir la confidencialitat, la integritat de les dades, així com per poder validar la identitat de l’aplicació integradora (l’Hèstia), és a dir, per tal de poder garantir una transmissió segura entre l’Hèstia i el connector **Avís de nou recurs**, l'ABSS haurà d’implementar-ho segons l’estàndard *JOSE* (*JSON Object Signing and Encryption*), que defineix un marc general per a signar i xifrar qualsevol tipus de contingut en entorns web, i més concretament fent ús de l’especificació *JWE* (*JSON Web Encryption*) que es troba definida en el següent RFC: 

https://tools.ietf.org/html/rfc7516.

El xifrat de la missatgeria s’haurà de realitzar amb l’algoritme *AES GCM* amb una contrasenya de 256-bit (*A256GC*). Aquesta contrasenya serà proporcionada a l’ABSS per l’equip tècnic de l’Hèstia a través d'un canal segur.

## 2.1 JWE serialització compacta
L'especificació JWE (JSON Web Encryption) estandarditza la manera de representar un contingut xifrat. Defineix dues formes de serialització per a representar un missatge xifrat. Una serialització compacta i una serialització en format JSON. Tots dos formats comparteixen els mateixos fonaments criptogràfics. Encara que, per a la comunicació amb el servei Hèstia ens restringirem a l'ús del format compacte. No descriurem el funcionament de JWE atès que estan àmpliament descrits en la norma. No obstant això, veurem alguns aspectes bàsics centrats en el format que genera.

La serialització compacta de JWE es basa en l'enviament de la informació en format Token JWE. Aquest token es construeix a través de cinc apartats cadascun separat per un punt (.). Aquests apartats són: capçalera JWE JOSE, clau xifrada JWE, vector d'inicialització JWE, text xifrat i etiqueta d'autenticació JWE.

![JWE.png](img/JWE.png)

A continuació detallem el contingut de cadascuna aquestes parts:

1- Capçalera JWE: conté un JSON no encriptat en format BASE64. Aquest JSON s’utilitza per transmetre informació no sensible per part de l'emissor, però també serveix per definir les operacions criptogràfiques associades al Token JWE. Per a la implementació de JWE com a mínim ha de contenir: 

* alg: indica l'algorisme criptogràfic utilitzat per a protegir la clau xifrada JWE.
* enc: indica l'algorisme de xifratge simètric utilitzat per a protegir el text xifrat.

A més, depenent de l'algorisme de codificació utilitzat, pot ser necessari incloure el camp:

* iv: indica el vector d'inicialització (*iv*) codificat en base64 per als algorismes de codificació que el requereixin. Aquest paràmetre és opcional.

A banda d'aquests camps obligatoris, es poden afegir altres camps que poden resultar molt útils com:

* jti: identificador únic (que pot servir per enregistrar la traçabilitat de la petició).

* iat: data de creació.

2- Clau xifrada JWE: és la clau que es xifra amb la clau del destinatari i el contingut xifrat resultant es registra com una matriu de bytes, que es coneix com la clau xifrada de JWE. 

3- Vector d'inicialització JWE: valor del vector d'inicialització utilitzat en xifrar el text sense format.

4- Text xifrat JWE: resultat de xifrar el contingut del missatge. Aquest text pot estar formatat o no.

5- Etiqueta d'autenticació JWE: depenent de l'algoritme de xifratge es pot generar una etiqueta d'autenticació que serveix per a garantir la integritat del text xifrat.



## 2.2 Exemple de token JWE

Un possible exemple de Token JWE seria:

```
eyJhbGciOiJBMjU2R0NNS1ciLCJpdiI6ImkzcE5kMW01NU04Ymo1cnIiLCJ0YWciOiI4VDZyZ2lnNVJwQnROblBwTWJMVmFnIiwiZW5jIjoiQTI1NkdDTSIsImp0aSI6IjMyMTU2ODc1NTYiLCJpYXQiOjE2NDAyNDE3MjV9.Q723jUVrJx85NOi8pKcsGixvELDxfyxLP9Gj_8IGhco.tNJ0Qm5UyryrlQrX.ZfBbIohyTSG5yHACeEPhgk9WiLtk2HF9p2aZ0rsZvrt6khUP8_83Yw.oyW5I78IRC8lrr79tP5ZMg
```
Si descodifiquem en base64 la capçalera JWE (que va des de l'inici fins al primer punt), veuríem:

```json
{"alg":"A256GCMKW",
 "iv":"i3pNd1m55M8bj5rr",
 "tag":"8T6rgig5RpBtNnPpMbLVag",
 "enc":"A256GCM",
 "jti":"3215687556",
 "iat":1640241725}
```


## 2.3 Implementacions suggerides
La majoria de les llibreries JOSE-JWT existents permeten generar i validar de forma senzilla els Token JWE. Podem recomanar les següents llibreries:

* Java: nimbus-jose-jwt
* .NET: jose-jwt



## 2.4 Protocol de comunicació
A continuació mostrem un diagrama de seqüència on es detalla el flux que segueix una nova petició d'avís de nou recurs:

![ProtocoloCom.png](img/ProtocoloCom.png)



**Pas 1 - ** El servei d'avisos publicat per l'ABSS rep una nova petició en format JSON.

**Pas 2 - ** El servei genera un token JWE utilitzant la contrasenya subministrada per l'integrador.

**Pas 3 - ** El servei envia una petició HTTPS a l'integrador amb el token JWE enviat.

**Pas 4 - ** L'integrador rep el token.

**Pas 5 - ** L'integrador descodifica el token JWE utilitzant la seva contrasenya.

**Pas 6 - ** L'integrador gestiona l'avís i genera un JSON de resposta.

**Pas 7 - ** L'integrador codifica el JSON en un token JWE a través de la seva contrasenya.

**Pas 8 - ** L'integrador envia el token resultant al sistema d'avisos.

**Pas 9 - ** El servei descodifica el token.

**Pas 10 - ** El servei gestiona el JSON resultant.



# 3. Avís nou recurs
Una vegada explicats els requeriments de seguretat mínims que s'hauran d'implementar en aquest connector, anem a detallar pròpiament la missatgeria d'aquest:


### 3.1. Petició d'enviament
![Aviso ENTRADA_RECURSO.png](img/Aviso ENTRADA_RECURSO.png)

| Element                                    | Descripció                                                   |
| ------------------------------------------ | :----------------------------------------------------------- |
| EntradaRecursRequest/**CodINE**            | Codi INE de l'Àrea Bàsica de Serveis Socials                 |
| EntradaRecursRequest/**IdUnicoRecurso**    | Identificador únic del recurs donat d'alta a l'Hèstia        |
| EntradaRecursRequest/**IdTipoRecurso**     | Tipus de recurs traspassat dins de l'Hèstia                  |
| EntradaRecursRequest/**IdProfesional**     | Identificador del professional que ha donat d'alta el recurs |
| EntradaRecursRequest/**NomProfesional**    | Nom del professional que ha donat d'alta el recurs           |

Exemple de petició realitzada amb [Postman](https://www.postman.com/)

![Peticio1.png](img/Peticio1.png)



### 3.1.2. Resposta
El connector implementat per l'ABSS haurà de retornar la següent missatgeria de resposta:

![RespuestaAviso.png](img/RespuestaAviso.png)

Els possibles resultats són:

|Element | Descripció|
|------- | ----------|
|EntradaRecursResponse/resultat/codiResultat | -1: La petició no és vàlida. Operació no realitzada|
| | -2: Token no vàlid. El token subministrat no és vàlid. Operació no realitzada |
| | 0: Operació completada amb èxit. L'avís ha estat correctament tractat. |
|EntradaRecursResponse/resultat/descripcio| Missatge descriptiu del resultat de l’operació. En cas d’error es detallen els motius.|


Exemple de petició realitzada amb [Postman](https://www.postman.com/)

![RespuestaAvisoPostman.png](img/RespuestaAvisoPostman.png)


### 3.1.3.  Joc de proves
El joc de proves del servei vàlid per a l’entorn de pre-producció, és el que es detalla a continuació:

|CodINE |IdUnicoRecurso | IdTipoRecurso | IdProfesional | NombreProfesional | Token | Resultat|
|------- | --------------- | --- | --- | --- | -------- |------- |
| 9821920002	| 840998044	| 370000144 | 370000126 | MARIA COLLADO MARTÍNEZ | Ok | (0)  Operació completada amb èxit|
| -1	| -1	| -1 | -1 | no vàlid | Ok | (-1) La petició no és vàlida. Operació no realitzada |
| 9821920002	| 840998044	| 370000144 | 370000126 | MARIA COLLADO MARTÍNEZ | Invàlid | (-2) Token no vàlid|



### 3.1.4.  Petició d'exemple
```json
{
    "CodINE": 9821920002,    
    "IdUnicoRecurso": 840998044,
    "IdTipoRecurso": 370000144,
    "IdProfesional": 370000126,
    "NomProfesional": "MARIA COLLADO MARTÍNEZ"
}
```


### 3.1.5.  Resposta d'exemple
```json
{
    "resultat": {
        "codiResultat": "0",
        "descripcio": "Operació completada amb èxit."
    }
}
```
