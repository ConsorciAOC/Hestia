

# 1. Introducció
Aquest és un connector conceptualment molt diferent de la resta de connectors de l'API de l'Hèstia. Es tracta d'un connector que requereix que cada ABSS que hi estigui interessada ho implementi i publiqui en el seu propi backoffice d'acord amb les especificacions que aquí es detallen. Un cop estigui disponible aquest connector al backoffice de l'ABSS, s'haurà d'enregistrar a l'Hèstia per tal que es pugui consumir des de l'Hèstia de forma similar a una funció callback.

L'objectiu d'aquest connector és informar a l'ABSS que s'ha donat d'alta un nou recurs a l'Hèstia, habitualment de caràcter econòmic, i bloquejar-ho a l'Hèstia per tal que es pugui realitzar la seva resolució (actualització de l'estat) des d'una eina externa integrant aquesta eina amb el connector ['Resolució de recurs'](ResolucioRecurs.md).
Es tracta, per tant, d'un connector que haurà de publicar l'ABSS interessada a través d'un canal TLS 1.2 o superior, i que s'haurà d'implementar mitjançant una API REST implementada amb el llenguatge de programació que desitgi l'ABSS. L'Hèstia per la seva banda consumirà aquest connector com a client.

En aquest apartat es detallen els requisits que ha de complir el connector que ha d'implementar l'ABSS per tal que l'Hèstia es pugui integrar. És important destacar, però, que serà responsabilitat de l'ABSS implementar les mesures de seguretat necessàries per garantir en tot moment la confidencialitat i la integritat de les dades compartides pel connector.



# 2. Requisits de seguretat
Per tal de garantir la confidencialitat, la integritat de les dades, així com per poder validar la identitat de l'aplicació integradora (l'Hèstia), és a dir, per tal de poder garantir una transmissió segura entre l'Hèstia i el connector **Avís de nou recurs**, l'ABSS haurà d'implementar-ho segons l'estàndard *JOSE* (*JSON Object Signing and Encryption*), que defineix un marc general per a signar i xifrar qualsevol mena de contingut en entorns web, i més concretament fent ús de l'especificació *JWT* (*JSON Web Token*) que es troba definida en el següent RFC:

[https://datatracker.ietf.org/doc/rfc7519/](https://datatracker.ietf.org/doc/rfc7519/)

El xifratge de la missatgeria s'haurà de realitzar amb l'algoritme *HMAC SHA-256* amb una contrasenya de 256-bit (*H256*). Aquesta contrasenya serà proporcionada a l'ABSS per l'equip tècnic de l'Hèstia a través d'un canal segur.



## 2.1 JWT serialització compacta
L'especificació JWT (JSON Web Token) estandarditza la manera de representar un contingut firmat. Defineix dues formes de serialització per a representar un missatge firmat. Una serialització compacta i una serialització en format JSON. Tots dos formats comparteixen els mateixos components bàsics del token (capçalera, càrrega útil i firma). Encara que, per a la comunicació amb el servei Hèstia ens restringirem a l'ús del format compacte. No descriurem el funcionament de JWT atès que estan àmpliament descrits en la norma. Obstant això, veurem alguns aspectes bàsics centrats en el format que genera

La serialització compacta de JWT és una forma de representar un token JWT de forma més senzilla i eficient que la serialització completa. En lloc d'incloure el token complet com una cadena JSON llegible, la serialització compacta utilitza una cadena codificada en Base64URL que consta de tres parts separades per un punt".".

Les tres parts d'un token JWT serialitzat de forma compacta són:

**1- Capçalera JWT**: definieix una cadena JSON codificada en Base64URL que conté informació sobre l'algoritme d'encriptació utilitzat i el tipus de token. Per exemple:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

Per a la implementació de JWT com a mínim ha de contenir:

* **alg**: indica l'algoritme de xifratge utilitzat per a la firma del token.
* **typ**: indica el tipus de token.

**2- Càrrega útil**: defineix una cadena JSON codificada en Base64URL que conté les dades específiques del token.
Per exemple:
```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
```

**3- Firma**: una cadena codificada en Base64URL que s'utilitza per verificar l'autenticitat del token i assegurar que no ha sigut alterat. La firma es genera utilitzant un algoritme d'encriptació específic (H256 en el nostre cas) i una clau secreta compartida entre l'emissor i el receptor del token. Per exemple:
```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

## 2.2 Exemple de token JWT
Un possible exemple de Token JWT seria:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZFJlY3VycyI6MzcwMzkwMTA5LCJpZFByb2Zlc3Npb25hbCI6NTUzLCJpZFRpcHVzUmVjdXJzIjo0ODQsIm51bWVyb0V4cGVkaWVudCI6IjIwMTUvNjE4NyIsIm5vbVByb2Zlc3Npb25hbCI6Ik0uSm9zXHUwMEU5IFNcdTAwRTFuY2hleiBQXHUwMEU5cmV6IiwiY29kaUlORSI6MX0.93JOnIyjbuIwvE49DCKEfo1UawhS6Pcl-APi7sLxIsI
```

Si descodifiquem en base64 la capçalera JWT (que va des de l'inici fins al primer punt), veuríem:

```json
{
"alg": "HS256",
"typ": "JWT"
}
```

El payload seria:
```json
{
"idRecurs": 370390109,
"idProfessional": 553,
"idTipusRecurs": 484,
"numeroExpedient": "2015/6187",
"nomProfessional": "M.José Sánchez Pérez",
"codiINE": 1
}
```
I la firma seria:
```
S2V5LU11c3QtQmUtYXQtbGVhc3QtMzItYnl0ZXMtaW4tbGVuZ3RoIQ==
```

## 2.3 Implementacions suggerides
La majoria de les llibreries JOSE-JWT existents permeten generar i validar de forma senzilla els Token JWT. Podem recomanar les següents llibreries:

* **Java**: nimbus-jose-jwt
* **.NET**: jose-jwt

## 2.4 Protocol de comunicació
A continuació mostrem un diagrama de seqüència on es detalla el flux que segueix una nova petició d'avís de nou recurs:

![Protocol_JWT.png](img/Protocol_JWT.png)

**Pas 1 -** Un professional de l'ABSS *A* dona d'alta un nou recurs a l'Hèstia, aquest recurs s'afegeix a la cua de recursos pendents d'avís i arriba el moment de processar aquest recurs.

**Pas 2 -** L'Hèstia genera un token JWT utilitzant la contrasenya acordada amb l'ABSS *A*.

**Pas 3 -** L'Hèstia envia la petició d'avís de nou recurs per un canal TLS 1.2, o superior, al backoffice de l'ABSS *A* incloent el token JWT generat al pas 2.

**Pas 4 -** El backoffice de l'ABSS *A* rep la petició que inclou el token JWT.

**Pas 5 -** L'ABSS *A* valida el token JWT utilitzant la seva contrasenya (la mateixa que l'Hèstia ha fet servir al pas 2).

**Pas 6 -** L'ABSS *A* processa l'avís i genera un JSON de resposta.

**Pas 7 -** L'ABSS *A* codifica el JSON i genera un token JWT de resposta fent servir la seva contrasenya (la mateixa dels passos 2 i 5).

**Pas 8 -** L'ABSS *A* retorna la resposta a l'Hèstia.

**Pas 9 -** L'Hèstia valida el token JWT fent servir la contrasenya de l'ABSS A (la mateixa dels passos 2, 5 i 7).

**Pas 10 -** Si tot és correcte, l'Hèstia elimina el recurs de la cua de pendents d'avís. En cas contrari, s'executarà la política de reintents en uns minuts.

# 3. Avís nou recurs
Una vegada explicats els requisits de seguretat mínims que s'hauran d'implementar en aquest connector, detallem pròpiament la missatgeria d'aquest:

### 3.1. Petició d'enviament
Exemple de petició realitzada amb [Postman](https://www.postman.com/)

![Peticio_AvisNouRecurs.png](img/Peticio_AvisNouRecurs.png)

| Element | Descripció |
| ------------------------------------------ | :----------------------------------------------------------- |
| EntradaRecursRequest/**CodINE** | Codi INE de l'Àrea Bàsica de Serveis Socials |
| EntradaRecursRequest/**IdUnicoRecurso** | Identificador únic del recurs donat d'alta a l'Hèstia |
| EntradaRecursRequest/**IdTipoRecurso** | Tipus de recurs traspassat dins de l'Hèstia |
| EntradaRecursRequest/**IdProfesional** | Identificador del professional que ha donat d'alta el recurs |
| EntradaRecursRequest/**NomProfesional** | Nom del professional que ha donat d'alta el recurs |
| EntradaRecursRequest/**NomExpediente** | Expedient en el qual s'ha donat d'alta el recurs |

### 3.1.2. Resposta
El connector implementat per l'ABSS haurà de retornar la següent missatgeria de resposta:

Exemple de resposta realitzada amb [Postman](https://www.postman.com/)

![Resposta_AvisNouRecurs.png](img/Resposta_AvisNouRecurs.png)

Els possibles resultats són:

|Element | Descripció|
|------- | ----------|
|EntradaRecursResponse/resultat/**codiResultat** | -1: La petició no és vàlida. Operació no realitzada|
| | -2: Token no vàlid. El token subministrat no és vàlid. Operació no realitzada |
| | 0: Operació completada amb èxit. L'avís ha estat correctament tractat. |
|EntradaRecursResponse/resultat/**descripcio**| Missatge descriptiu del resultat de l'operació. En cas d'error es detallen els motius.|

### 3.1.3. Joc de proves
El joc de proves del servei vàlid per a l'entorn de pre-producció, és el que es detalla a continuació:

|CodINE |IdUnicoRecurso | IdTipoRecurso | IdProfesional | NombreProfesional | Token | Resultat|
|------- | --------------- | --- | --- | --- | -------- |------- |
| 9821920002 | 840998044 | 370000144 | 370000126 | MARIA COLLADO MARTÍNEZ | Ok | (0) Operació completada amb èxit|
| -1 | -1 | -1 | -1 | no vàlid | Ok | (-1) La petició no és vàlida. Operació no realitzada |
| 9821920002 | 840998044 | 370000144 | 370000126 | MARIA COLLADO MARTÍNEZ | Invàlid | (-2) Token no vàlid|

### 3.1.4. Petició d'exemple
```json
{
"CodINE": 9821920002,
"IdUnicoRecurso": 840998044,
"IdTipoRecurso": 370000144,
"IdProfesional": 370000126,
"NomProfesional": "MARIA COLLADO MARTÍNEZ"
}
```

### 3.1.5. Resposta d'exemple
```json
{
"resultat": {
"codiResultat": "0",
"descripcio": "Operació completada amb èxit."
}
}
```