# Runnare ISP
## Editare server.json
```bash
{
  "host": "https://myisp.org",  # INSERIRE HOST DA CUI RAGGIUNGERE ISP ESTERNAMENTE
  "port": 443, # INSERIRE PORTA DA CUI RAGGIUNGERE ISP ESTERNAMENTE
  "useProxy": false,
  "useHttps": true,
	"httpsPrivateKey": "./config/spid-saml-check.key",
  "httpsCertificate": "./config/spid-saml-check.crt"
}
```
I campi Host e Port saranno usati nel file metadata per settare correttamente gli indirizzi di logout. Assicurarsi che host e port siano raggiungibili dall'instanza Drupal

## Editare idp.json
Nel file idp.json della repository modificare il campo entityID, inserendo lo stesso indirizzo settato nel campo host del server.json,
eventualmente indicando la porta qualora non sia convenzionale (443).
Questo campo viene usato da Drupal per comunicare con l'isp stesso ed effettuare il login, per questo deve essere raggiungibile



## Lanciare ISP
```bash
version: '3.3'
services:
    spid-saml-check:
        ports:
            - '8443:8443'
        network_mode: isp-net
        container_name: spid-isp
        volumes:
            - '$PWD/server.json:/spid-saml-check/spid-validator/config/server.json'
            - '$PWD/idp.json:/spid-saml-check/spid-validator/config/idp.json'
        image: 'italia/spid-saml-check:1.9.0'
```

# Configurare Drupal

## Impostare percorso private
Aprire il file settings.php e verificare il campo $settings['file_private_path'].
Se nessun percorso è indicato, indicarne uno fuori da Drupal, quindi creare quel percorso se non esiste ed assegnarli come proprietario www-data:www-data

## Installare modulo
```bash
composer require 'drupal/microspid:^1.0@beta'
```

## Impostare metadata ISP
1. Scaricare il file xml all'indirizzo https://myisp.org/metadata.xml
2. Backup del file web/modules/contrib/microspid/metadata/testenv2.xml
3. Copiare su web/modules/contrib/microspid/metadata/testenv2.xml il file xml scaricato al punto 1
4. Modificare il file web/modules/contrib/microspid/src/Service/SpidPaswManager.php
4.1 Alla riga 1121 copiare l'associazione 'http://localhost:8088' => 'testenv2.xml' ed inserirne un'altra come di seguito:
'https://myisp.org' => 'testenv2.xml'
4.2 Stiamo lasciando l'associazione con localhost:8088 perchè qando clicchiamo login con Spid e selezioniamo Spid di prova, il tema
invia come scelta dell'ISP http:localhost:8088. il modulo leggerà quindi testeenv2.xml e caricherà il nostro metadata corretto.
Dopo che l'ISP risponderà il modulo cercerà invece il metadata.xml usando come chiave il dominio che gli ha risposto, in questo caso
https://myisp.org, per questo motivo dobbiamo associarlo anch'esso a testenv2.xml

## Configurare Modulo
1. Andare nelle configurazioni del modulo
2. Spuntare le prime due checkbox
3. Creare i certificati
4. Copiare l'indirizzo SP Entity ID
5. Riempire i campi obbligator
6. Nei tab successivi mappare i field se desiderato

## Testare Login
1. Effettuare Login con SPID
2. una volta redirezionati su myspid.org inserire come user e password: validator validator
3. Se è la prima richiesta andare su metadada SP, ed inserire il link copiato durante la configurazione del modulo, quindi caricarlo
4. Andare sulla sezione response, quindi selezionare che tipo di response inviare (OK, failed, ecc), quindi inviare
5. Se tutto è stato fatto correttamente il login dovrebbe essere stato effettuato


