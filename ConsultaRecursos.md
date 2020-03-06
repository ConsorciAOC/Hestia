# Consulta Expedient (HESTIA_EXPEDIENT)
El servei **Hèstia** tracta l’Expedient Familiar com l’eina principal de treball dels professionals d’Atenció Primària: dóna suport per al treball directe dels casos, permet orientar els processos del sistema d’intervenció i agrupa tota la informació generada en una mateixa unitat de convivència (domicili).

L'expedient engloba les dades descriptives del nucli familiar a gestionar, la llista de les persones vinculades i la seva relació dins l’expedient, dades de diagnòstic de la situació (demandes, problemàtiques), el registre temporalitzat de les avaluacions i actuacions dels professionals, els plans d’acció (plans d’intervenció), els tràmits (formularis de sol·licitud que cal omplir i presentar per demanar una prestació) i els recursos concedits.

Aquest connector permet obtenir les dades d'un expedient (ocasionalment si el paràmetre indicat és el DNI del ciutadà i aquest tingués més d’un expedient en estat obert en l’ABSS indicada, es retornaria la llista dels expedients). Per a realitzar la consulta se li ha d'indicar obligatòriament el codi INE de l'ABSS i un dels tres paràmetres d'identificació de l’expedient. Aquests paràmetres són: l’IdExpedient que és l’identificador intern de l'expedient dins l’**Hèstia**, el número d'expedient que és l’identificador únic utilitzat pels professionals de l’ABSS o el DNI de la persona vinculada a l'expedient. En cas d’indicar més d’un paràmetre, el connector farà la següent priorització:
1.	Si s’especifica l’identificador intern de l’**Hèstia**, el connector ignorarà la resta de paràmetres indicats i comprovarà l’existència de l'expedient únicament a partir de l’identificador intern. En aquest cas sempre es retornarà com a màxim 1 únic expedient.
2.	Si no s’indica l’identificador intern, i s’indica el número d'expedient, el connector ignorarà el DNI indicat i comprovarà l’existència de l'expedient a partir del número d'expedient. En aquest cas sempre es retornarà com a màxim 1 únic expedient
3.	Si no s’indica l’identificador intern ni tampoc el número d'expedient, el connector retornarà les dades de tots els expedients oberts en els quals la persona indicada (DNI/NIE/Passaport) estigui vinculada. Aquest és l’únic cas en el qual es pot retornar una llista d’expedients.
Si no s’indica cap dels paràmetres, el connector retornarà un codi d’error. 
S'ha de tenir en compte que actualment només es permet la consulta d’expedients de la pròpia ABSS que realitza la petició.
A continuació es detalla la missatgeria corresponent al bloc de dades específiques. 

## Petició - dades específiques
La missatgeria específica de la petició *HESTIA_EXPEDIENT* es troba definida al document [Peticio_DadesEspecifiques_Expedient.xsd](xsd/Peticio_DadesEspecifiques_Expedient.xsd)

![Peticio_consulta_expedient.png](img/Peticio_consulta_expedient.png)

|Element | Descripció|
|------- | ----------|
|DadesEspecifiques/CodiINE | Codi de l'Àrea Bàsica de Serveis Socials|
|DadesEspecifiques/IdExpedient | Identificador intern de l’expedient dins l’Hèstia. Aquest és el paràmetre que recomanem utilitzar sempre que sigui possible (especialment per consultes futures sobre un mateix expedient)|
|DadesEspecifiques/NumExpedient | Número d’expedient utilitzat pels professionals de l’ABSS|
|DadesEspecifiques/DNI | DNI, NIE o passaport de la persona vinculada a l'expedient.|

## Resposta - dades específiques
La missatgeria específica de la resposta *HESTIA_EXPEDIENT* es troba definida al document [Resposta_DadesEspecifiques_Expedient.xsd](xsd/Resposta_DadesEspecifiques_Expedient.xsd)

![Resposta_consulta_expedient.png](img/Resposta_consulta_expedient.png)

|Element | Descripció|
|------- | ----------|
|RespostaConsultaExpedients/Resultat/CodiResultat | -1: La petició no és correcta o no compleix l’esquema|
| | -2: No s'ha localitzat l'expedient. No es retorna l’apartat LListaExpedients|
| | -6: el servei Hèstia no està disponible en aquest moment. La petició s’ha de tornar a enviar més endavant|
| | -9: Només es pot accedir a informació de la pròpia ABSS que realitza la consulta|
| | 0: L’expedient s'ha localitzat correctament. Es retorna en l’apartat “LlistaExpedients” l’expedient sol·licitat|
|RespostaConsultaExpedients/Resultat/Descripcio | Missatge descriptiu del resultat de l’operació. En cas d’error es detallen els motius|
|RespostaConsultaExpedients/LlistaExpedients | Llista d’expedients localitzats, apareixerà buida en cas d'error o si no s’ha pogut localitzar l’expedient indicat. Normalment aquesta llista retornarà 1 únic expedient, ara bé, si es fa la cerca de l’expedient a partir del DNI/NIE/Passaport del ciutadà i aquest disposa de més d’un expedient obert en l’ABSS (situació que en principi no s’hauria de donar de forma habitual) llavors sí que es pot retornar una llista d’expedients|

### LlistaExpedients/Expedient
![Resposta_consulta_expedientLlistatExpedient.png](img/Resposta_consulta_expedientLlistatExpedient.png)

![Resposta_consulta_expedientLlistatExpedient2.png](img/Resposta_consulta_expedientLlistatExpedient2.png)

|Element | Descripció|
|------- | ----------|
|//Expedient/IdExpedient | Identificador intern de l’expedient dins l’Hèstia. Per a futures consultes es recomana emprar aquest identificador si es desitja consultar aquest expedient més endavant.|
|//Expedient/NumExpedient | Número d’expedient utilitzat pels professionals de l’ABSS|
|//Expedient/IdEstat | Identificador intern de l'estat de l'expedient|
|//Expedient/Estat | Estat en el qual es troba l'expedient:|
| | 1: Obert|
| | 2: Tancat|
|//Expedient/DtEstat | Data en què l'expedient va entrar en l'estat actual|
|//Expedient/IdSector | Identificador únic del sector al que pertany l’expedient (primer nivell de la divisió territorial a la que pertany l'expedient i que normalment es correspon amb el municipi)|
|//Expedient/Sector | Descripció del sector al que pertany l'expedient|
|//Expedient/IdSubsector | Identificador únic del subsector al que pertany l’expedient (segon nivell de la divisió territorial a la que pertany l'expedient i que normalment es correspon amb el barri)|
|//Expedient/Subsector | Descripció del subsector al que pertany l'expedient|
|//Expedient/NomFamiliar | Text descriptiu que utilitzen els professionals de l’ABSS per identificar l’expedient de forma complementària al número d’expedient. Sovint s’utilitzen els cognoms de la família|
|//Expedient/IdProcedenciaDelCas | Identificador únic de la procedència del cas|
|//Expedient/ProcedenciaDelCas | Descripció de la procedència del cas:|
| | 0: SENSE ASSIGNAR|
| | 1: Personalment|
| | 2: Familiars|
| | 3: Responsables del Ens locals|
| | 4: Serveis d'ensenyament|
| | 5: Serveis Sanitaris|
| | 6: Cossos de Seguretat|
| | 7: Altres serveis socials|
| | 8: Serveis d'atenció a la infància i adolescència|
| | 9: Serveis de justícia|
| | 10: Associacions/ONG/Entitats|
| | 11: Veïns/amics/altres|
|//Expedient/IdTipusFamilia | Identificador del tipus d’unitat familiar|
|//Expedient/TipusFamilia | Descripció del tipus d’unitat familiar:|
| | 0: NO CONSTA|
| | 1: Unipersonal|
| | 2: Sense nucli|
| | 3: Monoparental|
| | 4: Nuclear|
| | 5: Extensa o ampliada|
| | 6: Múltiple|
| | 7: Reconstituïda|
|//Expedient/IdResidenciaFamiliar | Identificador de la residència del nucli familiar|
|//Expedient/ResidenciaFamiliar | Descripció genèrica on viuen les persones incloses a l’expedient:|
| | 0: SENSE ASSIGNAR|
| | 1: Tot a Catalunya|
| | 2: Espanya|
| | 3: Europa|
| | 4: Fora de la Unió Europea|
|//Expedient/IdSituacioExpedient | Identificador de la situació de l'expedient|
|//Expedient/SituacioExpedient | Descripció de la situació de l’expedient que es correspon la divisió dels estadis d'actuació pels quals pot passar un expedient:|
| | 0: Sense assignar|
| | 1: Acollida|
| | 2: Seguiment|
| | 3: Tractament (Pla Intervenció)|
|//Expedient/IdGrauRisc | Identificador del grau de risc d'un expedient|
|//Expedient/GrauRisc | Descripció de l’escala que determina la situació de risc per a la seva seguretat o benestar bàsic de les persones que formen part de l'expedient:|
| | 0: Sense Valorar|
| | 1: Vulnerable|
| | 2: Risc|
| | 3: Alt Risc|
|//Expedient/IdEquipReferent | Identificador de l'equip referent de l'expedient|
|//Expedient/EquipReferent | Descripció de l’equip especialitzat o temàtic assignat a l’expedient|
|//Expedient/ProfReferent |	Professional dels Serveis Socials que canalitza les prestacions que necessita la unitat de convivència i coordina tota l’activitat que es deriva|
|//Expedient/ProfReferent/DNIUsuari | DNI/NIE del professional referent|
|//Expedient/ProfReferent/NomUsuari | Nom del professional referent|
|//Expedient/ProfCorreferent/ | Professional auxiliar que exerceix les mateixes funcions que el professional referent en absència d’aquest|
|//Expedient/ProfCorreferent/DNIUsuari | DNI/NIE del professional correferent|
|//Expedient/ProfCorreferent/NomUsuari | Nom del professional correferent|
|//Expedient/ProfAlta | Professional que va donar d'alta l'expedient|
|//Expedient/ProfAlta/DNIUsuari | DNI/NIE del professional que va donar d'alta l'expedient|
|//Expedient/ProfAlta/NomUsuari | Nom del professional que va donar d'alta l'expedient|
|//Expedient/ProfMod/ | Darrer professional que va realitzar una modificació en l'expedient|
|//Expedient/ProfMod/DNIUsuari | DNI/NIE del darrer professional que va realitzar una modificació| 
|//Expedient/ProfMod/NomUsuari | Nom del darrer professional que va realitzar una modificació|
|//Expedient/Habitatge | Recull informació sobre l’habitatge de la unitat de convivència|
|//Expedient/Habitatge/IdProvincia | Identificador de província on està situada l'habitatge|
|//Expedient/Habitatge/Provincia | Descripció de la província|
|//Expedient/Habitatge/IdMunicipi | Identificador del municipi on està situat l'habitatge|
|//Expedient/Habitatge/Municipi | Descripció del municipi|
|//Expedient/Habitatge/IdTipusVia | Identificador del tipus de via de l’habitatge|
|//Expedient/Habitatge/NomVia | Descripció del carrer de l'habitatge|
|//Expedient/Habitatge/Numero | Número del carrer|
|//Expedient/Habitatge/CP | Codi postal de l’habitatge|
|//Expedient/Habitatge/LocalitatBarri | Localitat o barri on està ubicat l’habitatge|
|//Expedient/Habitatge/Pis | Número de pis|
|//Expedient/Habitatge/Porta | Porta de l'habitatge|
|//Expedient/Habitatge/Bloc | Bloc de l’habitatge|
|//Expedient/Habitatge/Escala | Escala de ‘habitatge|
|//Expedient/PersonesVinculades | Relació de persones vinculades a l’expedient|
|//Expedient/PersonesVinculades/PersonaVinculada | Dades del ciutadà vinculat a l'expedient|
|//Expedient/PersonesVinculades/PersonaVinculada/IdPersona | Identificador intern dins l’Hèstia de la persona|
|//Expedient/PersonesVinculades/PersonaVinculada/DNI | DNI/NIE/Passaport de la persona|
|//Expedient/PersonesVinculades/PersonaVinculada/Nom | Nom del vinculat|
|//Expedient/PersonesVinculades/PersonaVinculada/Cognom1 | Primer cognom de la persona|
|//Expedient/PersonesVinculades/PersonaVinculada/Cognom2 | Segon cognom de la persona|
|//Expedient/PersonesVinculades/PersonaVinculada/CIP | Codi d’identificació personal assignat per CatSalut|
|//Expedient/PersonesVinculades/PersonaVinculada/IdTipusRelacio | Identificador del tipus de relació del ciutadà amb l'expedient|
|//Expedient/PersonesVinculades/PersonaVinculada/TipusRelacio | Descripció del tipus de relació del ciutadà amb la “Persona principal”. Tot expedient ha de tenir una única "Persona principal" i per a la resta de persones vinculades s’indica la seva relació amb la "Persona principal"|
| | 1: Altres|
| | 2: Avi/àvia, nét/a|
| | 3: Fill/a|
| | 4: Gendre/nora, sogre/a|
| | 5: Germà/ana, cunyat/ada|
| | 6: Nebot/da, cosí/ina|
| | 7: Pare/mare|
| | 8: Persona principal|
| | 9: Tiet/a|
| | 10: Espòs/osa, company/a|
|//Expedient/Tramits | Llistat de tràmits concedits a l'expedient|
|//Expedient/Tramits/IdTramit | Identificador únic del tràmit associat a l'expedient. Aquest és l’identificador que s’ha d’utilitzar en la consulta *HESTIA_TRAMITS* si es vol obtenir els detalls del tràmit|
|//Expedient/Recursos | Llista de recursos en estat concedit vinculats a l'expedient|
|//Expedient/Recursos/IdRecurs | Identificador únic del recurs associat a l'expedient. Aquest és l’identificador que s’ha d’utilitzar en la consulta *[HESTIA_RECURSOS](ConsultaRecursos.md)* si es vol obtenir els detalls del recurs|





























| | 0: NIF/NIE|
| | 1: Passaport|
| | 2: Altre|
| | 3: Cap|
| | 4: DNI (sense lletra)|
|//informacioBasica/identificacioCiutada/NIF | Número de document del ciutadà|
|//informacioBasica/identificacioCiutada/NSS | Número de seguretat Social del ciutadà|
|//informacioBasica/identificacioCiutada/CIP | Codi d’identificació personal assignat per CatSalut|
|//informacioBasica/nom | Nom del ciutadà|
|//informacioBasica/cognom1 | Primer cognom del ciutadà|
|//informacioBasica/cognom2 | Segon cognom del ciutadà|
|//informacioBasica/estat/estat | Estat de la fitxa del ciutadà:|
| | 0: Activa|
| | 1: Èxitus<a href="#noteExitus" id="noteExitusref"><sup>1</sup></a>|
|//informacioBasica/estat/dataExitus | Data èxitus|
|//informacioBasica/estat/idUsuariExitus | Identificador del professional que ha executat l’operació d’Èxitus del ciutadà|
|//informacioBasica/estat/nomUsuariExitus | Nom del professional que ha executat l’operació d’Èxitus del ciutadà|
|//informacioBasica/dadesNaixement/nasciturus | Nasciturus<a href="#noteNasciturus" id="noteNasciturusref"><sup>2</sup></a>:|
| | 0: No|
| | 1: Sí|
|//informacioBasica/dadesNaixement/dtNaixement | Data de naixement del ciutadà|
|//informacioBasica/dadesNaixement/idPais |	Identificador del [país](Paisos.md) de naixement del ciutadà|
|//informacioBasica/dadesNaixement/descPais | Nom del [país](Paisos.md) de naixement del ciutadà|
|//informacioBasica/dadesNaixement/idProvincia | Identificador de la [província](Provincies.md) de naixement del ciutadà|
|//informacioBasica/dadesNaixement/descProvincia | Nom de la [província](Provincies.md) de naixement del ciutadà|
|//informacioBasica/dadesNaixement/idMunicipi | Identificador del [municipi](Municipis.md) de naixement del ciutadà|
|//informacioBasica/dadesNaixement/descMunicipi | Nom del [municipi](Municipis.md) de naixement del ciutadà|
|//informacioBasica/dadesContacte/telefon1 | Telèfon principal del ciutadà|
|//informacioBasica/dadesContacte/telefon2 | Telèfon secundari del ciutadà|
|//informacioBasica/dadesContacte/email | Correu-e del ciutadà|
|//informacioBasica/Sexe | Gènere del ciutadà:|
| | H: Home|
| | D: Dona|
|//informacioBasica/adrecaResidencia/transeunt | Transeünt:|
| | 0: No|
| | 1: Sí|
|//informacioBasica/adrecaResidencia/idRegim | Règim de tinença:|
| | 0: No consta|
| | 1: Propietaris / Accès propietat|
| | 2: Llogaters|
| | 3: Habitatge Gratuït|
| | 4: Alberg,pensió,hotel,residència|
| | 6: Relloguer|
| | 7: Règim de masover|
| | 8: Ocupació il.legal|
| | 9: NS/NC|
| | 10: Lloguer sense contracte|
| | 11: Relloguer lloguer|
| | 12: Relloguer propietat|
|//informacioBasica/adrecaResidencia/idProvincia | Identificador de la [província](Provincies.md) on resideix el ciutadà|
|//informacioBasica/adrecaResidencia/descProvincia | Descripció de la [província](Provincies.md) on resideix el ciutadà|
|//informacioBasica/adrecaResidencia/idMunicipi | Identificador del [municipi](Municipis.md) on resideix del ciutadà|
|//informacioBasica/adrecaResidencia/descMunicipi | Nom del [municipi](Municipis.md) on resideix del ciutadà|
|//informacioBasica/adrecaResidencia/localitatBarri | Localitat o barri on resideix el ciutadà|
|//informacioBasica/adrecaResidencia/CP | Codi postal de l’adreça de residència del ciutadà|
|//informacioBasica/adrecaResidencia/idTipusVia | [Tipus de via](TipusVia.md) on resideix el ciutadà|
|//informacioBasica/adrecaResidencia/nomVia | Nom de la via on resideix el ciutadà|
|//informacioBasica/adrecaResidencia/numero | Número de la via on resideix el ciutadà|
|//informacioBasica/adrecaResidencia/bloc | Bloc de residència|
|//informacioBasica/adrecaResidencia/escala | Escala de residència|
|//informacioBasica/adrecaResidencia/pis | Pis de residència|
|//informacioBasica/adrecaResidencia/porta | Porta de residència|
|//informacioBasica/adrecaResidencia/idTipusConstruccio | Tipus de construcció de l’habitatge:|
| | 1: Pis|
| | 2: Casa|
| | 3: Mas|
| | 4: Vivenda col.lectiva|
| | 5: Barraca/Caravana|
| | 6: NS/NC|
| | 7: ESTUDI|
|//informacioBasica/nucliConvivencia/idTipusFamilia	| Tipus de família:|
| | 0: No consta|
| | 1: Unipersonal|
| | 2: Sense nucli|
| | 3: Monoparental|
| | 4: Nuclear|
| | 5: Extensa o ampliada|
| | 6: Múltiple|
| | 7: Reconstituïda|
|//informacioBasica/nucliConvivencia/idNucliConvivencia	| Tipus de nucli de convivència:|
| | 0: No consta|
| | 1: Viu sol|
| | 2: Comparteix pis|
| | 3: Viu amb família|
| | 4: Viu amb amics|
| | 5: Centre o residència|
|//informacioBasica/nucliConvivencia/numMembres | Número de membres del nucli de convivència|
|//informacioBasica/nucliConvivencia/numMenors | Número de menors del nucli de convivència|
|//informacioBasica/nucliConvivencia/numFills | Número de fills del ciutadà|
|//informacioBasica/nucliConvivencia/numACarrec | Número de persones dependents a càrrec del ciutadà|
|//informacioBasica/dependencia/dependent | Dependent:|
| | 0: No|
| | 1: Sí|
|//informacioBasica/dependencia/grauDependencia	| Grau i nivell de dependència:|
| | 0: No consta|
| | 1: GRAU I - Nivell 1|
| | 2: GRAU II - Nivell 1|
| | 3: GRAU III - Nivell 1|
| | 4: GRAU I - Nivell 2|
| | 5: GRAU II - Nivell 2|
| | 6: GRAU III - Nivell 2|
| | 7: GRAU I|
| | 8: GRAU II|
| | 9: GRAU III|
|//informacioBasica/dependencia/expeDP | Número d’expedient de dependència|
|//informacioBasica/discapacitat/discapacitat | Discapacitat:|
| | 0: Sense discapacitat|
| | 1: Amb discapacitat|
|//informacioBasica/discapacitat/grauDiscapacitat | Grau de discapacitat (en percentatge)|
|//informacioBasica/idEstatCivil | Identificador de l’estat civil del ciutadà:|
| | 1: No consta|
| | 2: Parella estable|
| | 3: Casat/ada|
| | 4: Divorciat/ada|
| | 5: Separat/ada de fet|
| | 6: Solter/a|
| | 7: Vidu/a|
| | 8: Separat/ada de dret|
|//informacioBasica/consentiment | Consentiment:|
| | 0: el ciutadà no ha signat el consentiment d’utilització de dades personals|
| | 1: el ciutadà ha signat el consentiment d’utilització de dades personals|
|//informacioBasica/observacions | Observacions|
|//informacioBasica/dadesRegistre/dataAlta | Data i hora d’alta del ciutadà a l’Hèstia|
|//informacioBasica/dadesRegistre/idUsuariAlta | Identificador del professional que va donar d’alta el ciutadà a l’Hèstia|
|//informacioBasica/dadesRegistre/nomUsuariAlta | Nom del professional que va donar d’alta el ciutadà a l’Hèstia|
|//informacioBasica/dadesRegistre/dataModificacio | Data i hora de la darrera modificació de la fitxa del ciutadà|
|//informacioBasica/dadesRegistre/idUsuariModificacio | Identificador del darrer professional que va modificar la fitxa del ciutadà|
|//informacioBasica/dadesRegistre/nomUsuariModificacio | Nom del darrer professional que va modificar la fitxa del ciutadà|
|//informacioBasica/dadesRegistre/idUsuariResponsable | Identificador del professional referent assignat al ciutadà|
|//informacioBasica/dadesRegistre/nomUsuariResponsable | Nom del professional referent assignat al ciutadà|

<a id="noteExitus" href="#noteExitusref"><sup>1</sup></a> Terme latí que s'utilitza en medicina per tancar les històries clíniques d'aquells pacients en els quals l'enfermetat ha finalitzat amb la mort.

<a id="noteNasciturus" href="#noteNasciturusref"><sup>2</sup></a> Terme jurídic per designar al ser humà des de que és concebut fins al seu naixement. Es refereix per tant al ser humà concebut i no nascut.

### ciutada/informacioAmpliada

![Resposta_consulta_ciutada_informacioBasica1.png](img/Resposta_consulta_ciutada_informacioBasica1.png)

![Resposta_consulta_ciutada_informacioBasica2.png](img/Resposta_consulta_ciutada_informacioBasica2.png)

|Element | Descripció|
|------- | ----------|
|//informacioAmpliada/activitatLaboral/idActivitatLaboral | Identificador de l’[activitat laboral](ActivitatLaboral.md) del ciutadà|
|//informacioAmpliada/activitatLaboral/permisTreball | Permís de treball:|
| | 0: el ciutadà no disposa de permís de treball|
| | 1: el ciutadà disposa de permís de treball|
|//informacioAmpliada/activitatLaboral/dataCaducitatPermis | Data de caducitat del permís de treball|
|//informacioAmpliada/activitatLaboral/idMunicipi |	Identificador del [municipi](Municipis.md) on treballa el ciutadà|
|//informacioAmpliada/activitatLaboral/descMunicipi | Nom del municipi on treballa el ciutadà|
|//informacioAmpliada/activitatLaboral/idSectorLaboral | Identificador del sector laboral del ciutadà:|
| | 0: No consta|
| | 1: Sector primari|
| | 2: Sector secundari|
| | 3: Sector Terciari|
|//informacioAmpliada/activitatLaboral/ocupacioComptePropi | Ocupació per compte propi:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/activitatLaboral/ocupacioCompteAltre | Ocupació per compte d’altre:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/activitatLaboral/ingressos | Import de la font d’ingressos del ciutadà|
|//informacioAmpliada/dadesImmigrants/idSituacioLegal | Identificador de la situació legal del ciutadà:|
| | 0: No consta|
| | 1: Refugiats|
| | 2: Residencia sense permís de treball|
| | 3: Residència en tràmit|
| | 4: Expedient d'expulsió|
| | 5: Es desconeix|
| | 6: Irregular|
| | 7: Residencia i treball|
| | 8: Règim comunitari|
| | 9: Expedient de retornat|
| | 10: Residència llarga durada|
| | 11: Nacionalitat espanyola reconeguda|
|//informacioAmpliada/dadesImmigrants/dataArribadaEspanya | Data d’arribada a Espanya del ciutadà|
|//informacioAmpliada/dadesImmigrants/dataArribadaMunicipi | Data d’arribada al municipi del ciutadà|
|//informacioAmpliada/dadesImmigrants/dataCaducitatNIE | Data de caducitat del NIE del ciutadà|
|//informacioAmpliada/dadesImmigrants/pendentReagrupament/pares | El ciutadà està pendent de reagrupar els pares:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/dadesImmigrants/pendentReagrupament/fills | El ciutadà està pendent de reagrupar els fills:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/dadesImmigrants/pendentReagrupament/conjuge | El ciutadà està pendent de reagrupar el cònjuge:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/idHabilitatComunicativa | Identificador de l’habilitat comunicativa del ciutadà:|
| | 0: No consta|
| | 1: Bona capacitat de comunicació|
| | 2: Nul·la capacitat de comunicació|
| | 3: Dificultats de comunicació|
|//informacioAmpliada/estudisIdiomes/idLlenguaMaterna | Identificador de la [llengua materna](LlenguaMaterna.md) del ciutadà|
|//informacioAmpliada/estudisIdiomes/descLlenguaMaterna | Nom de la llengua materna del ciutadà|
|//informacioAmpliada/estudisIdiomes/idiomes/idioma/idioma | Nom de l’idioma|
|//informacioAmpliada/estudisIdiomes/idiomes/idioma/enten | El ciutadà entén l’idioma:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/idiomes/idioma/parla | El ciutadà parla l’idioma:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/idiomes/idioma/llegeix | El ciutadà llegeix l’idioma:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/idiomes/idioma/escriu | El ciutadà escriu l’idioma:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/idNivellFormacio | Identificador del nivell de formació del ciutadà:|
| | 0: No consta|
| | 1: NS/NC|
| | 2: Analfabetisme|
| | 3: Escola especial|
| | 4: Estudis primaris inacabats|
| | 5: ESO / Graduat escolar|
| | 6: Batxillerat, CFGM / Equivalent|
| | 7: Estudis universitaris grau mig|
| | 8: Estudis universitaris superiors|
| | 9: Programes de dep. treball|
|//informacioAmpliada/estudisIdiomes/estudisHomologats | El ciutadà té homologats els seus estudis:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/associacions/pertanyAssociacio | El ciutadà pertany a associacions:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/associacions/tipus | Identificador del tipus d’associació a la que pertany el ciutadà:|
| | 0: Sense assignar|
| | 1: Agrupacions veïnals|
| | 2: Agrupacions esportives|
| | 3: Agrupacions de dones|
| | 4: Agrupacions juvenils|
| | 5: Agrupacions culturals|
| | 6: Comunitats religioses|
| | 7: Sindicats|
|//informacioAmpliada/estudisIdiomes/associacions/nom | Nom de l’associació a la que pertany el ciutadà|
|//informacioAmpliada/estudisIdiomes/suport/ONG | El ciutadà rep el suport d’alguna ONG:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/suport/administracioPublica | El ciutadà rep el suport d’alguna Administració Pública:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/suport/associacionsImmigrants | El ciutadà rep el suport d’alguna associació d’immigrants:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/estudisIdiomes/suport/personesAltres | El ciutadà rep el suport de persones o d’altres:|
| | 0: No|
| | 1: Sí|
|//informacioAmpliada/dadesRegistre/dataAlta | Data i hora d’alta de la fitxa ampliada del ciutadà a l’Hèstia|
|//informacioAmpliada/dadesRegistre/idUsuariAlta | Identificador del professional que va donar d’alta la fitxa ampliada del ciutadà a l’Hèstia|
|//informacioAmpliada/dadesRegistre/nomUsuariAlta | Nom del professional que va donar d’alta la fitxa ampliada del ciutadà a l’Hèstia|
|//informacioAmpliada/dadesRegistre/dataModificacio | Data i hora de la darrera modificació de la fitxa ampliada del ciutadà|
|//informacioAmpliada/dadesRegistre/idUsuariModificacio | Identificador del darrer professional que va modificar la fitxa ampliada del ciutadà|
|//informacioAmpliada/dadesRegistre/nomUsuariModificacio | Nom del darrer professional que va modificar la fitxa ampliada del ciutadà|
|//informacioAmpliada/dadesRegistre/idUsuariResponsable | Identificador del professional referent assignat al ciutadà|
|//informacioAmpliada/dadesRegistre/nomUsuariResponsable | Nom del professional referent assignat al ciutadà|
