# Report-API

Mit der Report-API lassen sich Europace-Reports erzeugen und abrufen.

![Vertrieb](https://img.shields.io/badge/-Vertrieb-lightblue)
![Produktanbieter](https://img.shields.io/badge/-Produktanbieter-lightblue)
![Baufinanzierung](https://img.shields.io/badge/-Baufinanzierung-lightblue)
![Privatkredit](https://img.shields.io/badge/-Privatkredit-lightblue)

[![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://github.com/europace/authorization-api)
[![Github](https://img.shields.io/badge/-Github-black?logo=github)](https://github.com/europace/report-api)

[![GitHub release](https://img.shields.io/github/v/release/europace/report-api)](https://github.com/europace/report-api/releases)
[![Pattern](https://img.shields.io/badge/Pattern-Tolerant%20Reader-yellowgreen)](https://martinfowler.com/bliki/TolerantReader.html)

## Dokumentation
Die aktuellste Open-Api-Specification im json-Format kann jederzeit unter `https://report.api.europace.de/documentation`
abgerufen werden.

[![YAML](https://img.shields.io/badge/OAS-HTML_Doc-lightblue)](https://europace.github.io/report-api/index.html)
[![YAML](https://img.shields.io/badge/OAS-YAML-lightgrey)](https://github.com/europace/report-api/blob/master/report-api.yml)
[![YAML](https://img.shields.io/badge/OAS-JSON-lightgrey)](https://report.api.europace.de/documentation)

## Anwendungsfälle der API
- Europace-Reports in Dataware-House ETL-Jobs einbinden
- Europace-Reports in Vertriebstools anbieten

## Übersicht Europace-Reports
Folgende Reports können mit dieser API angerufen werden:

 Name | Endpunkt | benötigter Scope | Dateityp/Encoding | Inhalts-Beschreibung
 ---- | ---- | ---- | :----: | ---
 Rohdaten-Report | ```/rohdaten``` | `report:rohdaten:lesen`  | zip/UTF-8 | alle relevanten Daten von Vorgängen, Anträgen, Bausteinen und Provisionen des Vertriebs |
Produktanbieter-Report (:construction: in Arbeit)  | ```/produktanbieter``` | `report:produktanbieter:lesen`  | csv/UTF-8 | die wesentlichen Antragsdaten mit Status und Vertriebsorganisation |

## Schnellstart

Der Abruf der Ep2_Reports erfolgt asynchron, um Netzwerk-Timeouts bei der Erstellung der Reports zu vermeiden. Auf die Fertigstellung des Reports muss aktiv getestet werden.

Die Arbeitsweise ist bei allen Europace-Reports dieselbe:

1. Report anfragen
2. Report Status abfragen
3. Report-Daten abholen

![Sequenzdiagramm](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/europace/report-api/master/resources/processing-report-api.iuml?token=AFSAZZEIDC253X3GO74BDNK7R2KOC)

![Tipp](https://img.shields.io/badge/-Tipp-yellow) Damit du unsere APIs und deinen Anwendungsfall schnellstmöglich testen kannst, haben wir eine [Postman-Collection](https://github.com/europace/api-schnellstart) für dich zusammengestellt, mit der du die folgenden Schritte einfach nachvollziehen kannst.

## Beispiel: Rohdaten-Report anfragen

### Authentifizierung
Bitte benutze [![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://github.com/europace/authorization-api), um Zugang zur Report-API zu bekommen.

Welchen Scope du für welchen Report benötigst, siehst du in der [Übersicht Europace-Reports](https://docs.api.europace.de/baufinanzierung/report/report-api/#europace-reports).

:warning: **Hinweis** \
Es wird immer der Report für die Identität des OAuth-Token geliefert. Um für eine andere Person/Orga einen Report abzurufen, sollte [impersoniert](https://docs.api.europace.de/baufinanzierung/authentifizierung/#wie-authentifiziere-ich-verschiedene-benutzer-mit-einem-client-impersionieren) oder ggf. ein weiterer Client verwendet werden.

### 1. Report anfragen
Mit der Anfrage, wird bei Europace die Erzeugung des Report gestartet. Dieser Vorgang kann je nach Komplexität und abgefragtem Zeitraum mehrere Minuten in Anspruch nehmen. Die gültigen Parameter für die Erstellung des Report findest du in der entsprechenden Report-Beschreibung.

Request für Rohdaten-Report:
```
curl --location --request POST 'https://report.api.europace.de/rohdaten' \
--header 'X-Trace-Id: ' \
--header 'Authorization: Bearer {access-token}' \
--data-raw '{
    "fromDay": "2020-01-01"
}'
```

Response:
```
Header:
  - Status: 202 ACCEPTED
  - Location: /rohdaten/{report-processing-id}
```

Die Header-Variable `Location` zeigt auf den Endpunkt für den nächsten Schritt: den Report Status abzufragen.

### 2. Report Status abfragen

Dieser Schritt ist für alle Europace-Reports gleich.

Request:
```
curl --location --request GET 'https://report.api.europace.de/rohdaten/{report-processing-id}' \
--header 'X-Trace-Id: ' \
--header 'Authorization: Bearer {access-token}'
```

**(a)** Response für den Fall, dass Erstellung des Europace-Report noch in Arbeit ist:
```
Header:
  - Status: 200 OK
Body:
{
  "status": "PROCESSING"
}
```
Bitte in 20s (Empfehlung) erneut abfragen.

**(b)** Response für den Fall, dass der Europace-Report erstellt wurde:
```
Header:
  - Status: 303 SEE OTHER
  - Location: https://{path-to-file}
```
Die Header-Variable `Location` zeigt auf die Reportdaten.

### 3. Report abholen

Dieser Schritt ist für alle Europace-Reports gleich.

Die URL, welche auf die Ergebnisdaten zeigt ist nur für einen begrenzten Zeitraum gültig. Jeder, der diese URL besitzt kann auf den Report zugreifen.

Beispiel für {path-to-file}:
```
https://greta-462912489437-eu-central-1.s3.amazonaws.com/prod/{reporttyp}/{report-processing-id}/Ep2_Reports_....zip?X-Amz-Security-Token=...&X-Amz-Signature=...
```

Request:
```
curl --location --request GET 'https://greta-462912489437-eu-central-1.s3.amazonaws.com/prod/{reporttyp}/{report-processing-id}/Ep2_Reports_....zip?X-Amz-Security-Token=...&X-Amz-Signature=...
```

Response:
```
Header:
  - Status: 200 OK
Body:
<File>
```
Der Dateityp von `<file>` ist in der [Übersicht Europace-Reports](https://docs.api.europace.de/baufinanzierung/report/report-api/#übersicht-europace-reports) beschrieben.  Um das Öffnen mit Excel zu erleichtern, ist das Encoding UTF-8 mit Byte Order Mark (BOM).

## Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zur Verfügung gestellt.

## Support
Bei Fragen oder Problemen kannst du dich an devsupport@europace2.de wenden.
