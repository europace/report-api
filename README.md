# Report-API

Mit der Report-API lassen sich Europace-Reports erzeugen und abrufen.

![Vertrieb](https://img.shields.io/badge/-Vertrieb-lightblue)
![Produktanbieter](https://img.shields.io/badge/-Produktanbieter-lightblue)
![Baufinanzierung](https://img.shields.io/badge/-Baufinanzierung-lightblue)
![Privatkredit](https://img.shields.io/badge/-Privatkredit-lightblue)

[![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://github.com/europace/authorization-api)
[![YAML](https://img.shields.io/badge/{}-YAML-green)](https://github.com/europace/report-api/report-api.yaml)
[![Github](https://img.shields.io/badge/-Github-black?logo=github)](https://github.com/europace/report-api)

[![GitHub release](https://img.shields.io/github/v/release/europace/authorization-api)](https://github.com/europace/report-api/releases)
[![Pattern](https://img.shields.io/badge/Pattern-Tolerant%20Reader-yellowgreen)](https://martinfowler.com/bliki/TolerantReader.html)

## Europace-Reports

 Name | Endpunkt | benötigter Scope | Inhalts-Beschreibung
 ---- | ---- | ---- | ----
 Rohdaten-Report | ```/rohdaten``` | `report:rohdaten:lesen`  | alle relevanten Daten von Vorgängen, Anträgen, Bausteinen und Provisionen des Vertriebs |
Produktanbieter-Report (:construction: in Arbeit)  | ```/produktanbieter``` | `report:produktanbieter:lesen`  | die wesentlichen Antragsdaten mit Status und Vertriebsorganisation |

## Anwendungsfälle
- Europace-Reports in Dataware-House ETL-Jobs einbinden
- Europace-Reports in Vertriebstools anbieten

## Schnellstart

Die Bearbeitung der Reportanfragen erfolgt asynchron, um Netzwerk-Timeouts bei der Erstellung der Reports zu vermeiden. Auf die Fertigstellung des Reports muss aktiv getestet werden.

Die Arbeitsweise ist bei allen Europace-Reports dieselbe:

1. Report anfragen
2. Report Status abfragen
3. Report-Daten abholen

![Sequenzdiagramm](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/europace/report-api/master/resources/processing-report-api.iuml?token=AFSAZZEIDC253X3GO74BDNK7R2KOC)

![Tipp](https://img.shields.io/badge/-Tipp-yellow) Damit du unsere APIs und deinen Anwendungsfall schnellstmöglich testen kannst, haben wir eine [Postman-Collection](https://github.com/europace/api-schnellstart) für dich zusammengestellt, mit der du die folgenden Schritte einfach nachvollziehen kannst.

## 1. Report anfragen
Hier am Beispiel des Rohdaten-Report.

### Authentifizierung
Bitte benutze [![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://github.com/europace/authorization-api), um Zugang zur Report-API zu bekommen.

Welchen Scope du für welchen Report benötigst, siehst du unter [Europace-Reports](#markdown-header-Europace-Reports).

Das Token muss auf die angefragte `partnerId` zugreifen dürfen (Hierarchie der Plakette)

### Anfrage
Mit der Anfrage, wird bei Europace die Erzeugung des Report gestartet. Dieser Vorgang kann je nach Komplexität und abgefragtem Zeitraum mehrere Minuten in Anspruch nehmen. Die gültigen Parameter für die Erstellung des Report findest du in der entsprechenden Report-Beschreibung.

Request für Rohdaten-Report:
```
POST https://report.api.europace.de/rohdaten

Header:
  * Authentication: "Bearer {token}"
  * Content-Type: "application/json"
  * Trace-Id: "{traceId}" (Optional)

Body:
{
    "partnerId": "ABC12",
    "fromDay": "2020-10-01"
}
```

Response:
```
Header:
  * Status: 202 ACCEPTED
  * Location: /rohdaten/{report-processing-id}
```

Die Header-Variable `Location` zeigt auf den Endpunkt für den nächsten Schritt: den Report Status abzufragen.

## 2. Report Status abfragen

Dieser Schritt ist für alle Europace-Reports gleich.

Request:
```
GET https://report.api.europace.de/rohdaten/{report-processing-id}

Header:
  * Authentication: "Bearer {token}"
  * Trace-Id: "{traceId}" (Optional)
```

**(a)** Response für den Fall, dass Erstellung des Europace-Report noch in Arbeit ist:
```
Header:
  * Status: 200 OK

Body:
{
  "status": "PROCESSING"
}
```
Bitte in 20s (Empfehlung) erneut abfragen.

**(b)** Response für den Fall, dass der Europace-Report erstellt wurde:
```
Header:
  * Status: 303 SEE OTHER
  * Location: https://{path-to-file}
```
Die Header-Variable `Location` zeigt auf die Reportdaten.

## 3. Report abholen

Dieser Schritt ist für alle Europace-Reports gleich.

Die URL, welche auf die Ergebnisdaten zeigt ist nur für einen begrenzten Zeitraum gültig. Jeder, der diese URL besitzt kann auf den Report zugreifen.

Beispiel für {path-to-file}:
```
https://greta-462912489437-eu-central-1.s3.amazonaws.com/prod/{reporttyp}/{report-processing-id}/Ep2_Reports_....zip?X-Amz-Security-Token=...&X-Amz-Signature=...
```

Request:
```
GET https://greta-462912489437-eu-central-1.s3.amazonaws.com/prod/{reporttyp}/{report-processing-id}/Ep2_Reports_....zip?X-Amz-Security-Token=...&X-Amz-Signature=...
```

Response:
```
Header:
  * Status: 200 OK

Body:
<File>
```

## Nutzungsbedingungen
Die APIs werden unter folgenden [Nutzungsbedingungen](https://docs.api.europace.de/nutzungsbedingungen/) zur Verfügung gestellt.

## Support
Bei Fragen oder Problemen kannst du dich an devsupport@europace2.de wenden.
