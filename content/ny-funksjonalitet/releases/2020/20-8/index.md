---
title: 20.8
description: Forberedelse til FREG, mindre endringer og feilrettinger
weight: 60
type: releasenote
releasenote_info: Release 20.8, produksjonsettes 24. august 2020
--- 
**Dette er en kommende endring. Gjeldende endring ligger [her](../20-7).**

## Endringer i portal

### Tilgangsstyrer kan tilpasse operasjoner før en tilgangsforespørsel blir godkjent

Endringen er en videreutvikling av funksjonaliteten på “Be om tilgang” som ble lansert i [forrige release](../20-7). Denne gangen er det tilgangsstyrer som kan endre på operasjoner i en tilgangsforespørsel. Ved å toggle av og på operasjoner, vil tilgangsstyrer nå kunne endre på en forespørsel som kommer inn dersom dette er hensiktsmessig. Dersom tilgangsstyrer velger å endre på forespørselen, vil det komme en gul advarsel om at noen av operasjonene på forespørselen er endret. Selve forespørselen som ligger i databasen vil ikke bli endret, men selve delegeringen vil bli en annen.

### Endring av tekst som sendes ut til valg tilgangsstyrer i “Be om tilgang”

Teksten som sendes per e-post til tilgangsstyrer ved en nyopprettet tilgangssforespørsel er nå utbedret. Den nye teksten presiserer hvor den nye forespørselen kan behandles fra.

### Navneendring på tjenesteeier

Navn for tjenesteeier "Statens havarikommisjon for transport" er endret til "Statens havarikommisjon"

## Endringer i Integrasjon

### Forberedelse til oppdatert integrasjon med Folkeregisteret (FREG)

I dag er kilden for Altinns personregister datafiler overlevert fra Det Sentrale Folkeregister (DSF). Dette vil fases ut til fordel for en direkte integrasjon med FREGs hendelsesliste og personopplysninger. I denne endringen vil vi legge til rette for de nye datamodellene som FREG tilbyr og implementere systemet som kan motta hendelser og personopplysninger. Merk at det det er først i etterkant av endringen vi etterhvert vil koble oss på FREG og motta løpende oppdateringer. Det betyr at vi fortsetter å ha DSF som kilde frem til vi slår på integrasjonen.

Til informasjon henter Altinn alltid siste person dokument for hver endring og lagrer alle data i denne oppdateringen over eventuelle tidlige data.

- Datamodellen for å lagre familie relasjoner er utvidet med en mange til mange knytting mellom REG_User og Freg.FamilyRelatonship som kan knytte to personer sammen med en familerelasjon. Det er også utvidet med mulighet for å lagre mange linjer i fritekst for postadresse. Hvilket forhold de har til hverandre er bestemt av en kodetabell Freg.FamilyRelationshipType.

- Det er laget en egen RegisterFregSI med tilhørende DAL og entiteter for å lagre data fra FREG på den eksisterende register strukturen med noen små endringer.

- Det er laget en egen mapper som konverterer fra Json Modellen FReg har til en modell som ligger nær registermodellen slik at dette kan smeltes sammen rett inn i databasen. Her blir alle felter som ikke allerede eksisterer i Altinn lagt inn bortsett fra familie relasjoner og vergemål.

- Siden Freg benytter både to bokstavs og tre bokstavs landkoder i henholdsvis Statsborgerskap og Postadresse utland så måtte Altinn legge inn tre bokstavs landkoder i tabellen for land for å kunne knytte Statsborgerskap riktig. Det er også lagt på en tjeneste for å hente alle landkoder slik at disse kan benyttes i konverteringen.

- Webtjenester for å hente ut personer både interne og eksterne er oppdatert til å hente ut flere linjer med adresse for Postadresse (fritekst). GetFamily og GetCompleteFamily er også oppdatert til å forholde seg til familierelasjons strukturen fra FReg.

- Ny jobb (batch) for å hente FReg Eventer og PersonDokumenter lagre data i Altinn ved hjelp av tjenestene beskrevet over og logge feilsituasjoner, fremdrift og status.

- Enkelte felter har mer data i Freg en tilsvarende felter i Altinn Derfor har følgende felter fått utvidet kapasitet:

- Gatenavn utvidet fra 22 tegn til 840
- Postadresse linje 1-3 fra 50 tegn til 1024. Adresselinje 4..n deler på 1 000 000 tegn
- Fornavn, Mellomnavn, Etternavn fra 50 tegn til 1024
- Stedsnavn for matrikkeladresser utvides fra 25 tegn til 1024.

Ingen eksterne grensesnitt er endret for å forholde seg til at det nå kan være flere adresse-linjer i postadressene. Dette er løst med å slå sammen linje 2 og 3 til en linje for de eksterne grensesnittene med norske adresser. 
Det er lagt på tre nye varslingsmaler for å hente mer adresseinformasjon:

1. Adresseline4
2. Hele adressen som komma separert liste.
3. Hele adressen med html linjeskift.

Steder som tidligere måtte settes sammen med mellomrom eller komma er utvidet med flere linjer.

## Feilrettinger

### Varslingslisten i “Be om tilgang”

Det ble oppdaget en bug på nedtrekkslisten for valg av tilgangsstyrer som varsel om ny tilgangsforespørsel skulle sendes til. Dersom en bruker hadde to topproller i ER hvor begge hadde tilgangsstyring som barnerolle, ble denne personen listet ut to ganger i listen. Dette er nå fikset, og hver person i listen vil nå bare bli listet ut en gang.

### Feil oppsto dersom det ikke fantes noen tilgangsstyrere for hovedenheten

Det ble var en feil på “Be om tilgang” som førte til at brukeren ble møtt med gal feilmelding. Feilen oppsto da man forsøkte å hente ut tilgangsstyrere for en hovedenhet som ikke hadde noen tilgangsstyrere. Man kommer nå til riktig side. I tillegg er selve feilen for å hente ut Tilgangsstyrere nå rettet.