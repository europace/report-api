# Report-API

The Report API can be used to generate and retrieve Europace reports.

![advidor](https://img.shields.io/badge/-advidor-lightblue)
![loanProvider](https://img.shields.io/badge/-loanProvider-lightblue)
![mortgageLoan](https://img.shields.io/badge/-mortgageLoan-lightblue)
![consumerLoan](https://img.shields.io/badge/-consumerLoan-lightblue)

[![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://github.com/europace/authorization-api)
[![Github](https://img.shields.io/badge/-Github-black?logo=github)](https://github.com/europace/report-api)

[![GitHub release](https://img.shields.io/github/v/release/europace/report-api)](https://github.com/europace/report-api/releases)
[![Pattern](https://img.shields.io/badge/Pattern-Tolerant%20Reader-yellowgreen)](https://martinfowler.com/bliki/TolerantReader.html)

## Documentation
The latest OpenApi specification in json format is available at any time at `https://report.api.europace.de/documentation`.
can be downloaded.

[![YAML](https://img.shields.io/badge/OAS-HTML_Doc-lightblue)](https://europace.github.io/report-api/index.html)
[![YAML](https://img.shields.io/badge/OAS-YAML-lightgrey)](https://github.com/europace/report-api/blob/master/report-api.yml)
[![YAML](https://img.shields.io/badge/OAS-JSON-lightgrey)](https://report.api.europace.de/documentation)

## API use cases
- use Europace reports in dataware house ETL jobs
- provide Europace reports in sales tools

## Overview Europace Reports
The following reports can be called with this API:

Name | Endpoint                         | Required Scope                     | File Type/Encoding | Content Description.                                                                                                                                                                                                              
---- |----------------------------------|------------------------------------| :----: |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Vertriebs-Rohdaten-Report | ```/rohdaten```                  | `report:rohdaten:lesen`            | zip/UTF-8 | all relevant data of [Vorgänge, Anträge, Bausteine](https://docs.api.europace.de/common/glossary/) and [Provisionen](https://docs.api.europace.de/common/glossary/) of the advisor<br>Data older than 2014 will not be delivered. |
Produktanbieter-Report | ```/produktanbieter```           | `report:produktanbieter:lesen`     | csv/UTF-8 | the essential data of [Anträge](https://docs.api.europace.de/common/glossary/) with state and [Vertriebsorganisation](https://docs.api.europace.de/common/glossary/)                                                              |
Vertriebsreport | ```/vertrieb```                  | ```report:rohdaten:lesen```        | csv/UTF-8 | the "EUROPACE Report Vertrieb".                                                                                                                                                                            | 
Smart Facts Vertrieb | ```/smartfactsvertrieb```        | ```report:rohdaten:lesen```        | XLSM | the “EUROPACE Report Vertrieb” with a graphical interface<br>Data older than 2 years (relative to calling date) will not be delivered.
Smart Facts Produktanbieter | ```/smartfactsproduktanbieter``` | ```report:produktanbieter:lesen``` | XLSM | the “EUROPACE Report Produktanbieter” with a graphical interface<br>Data older than 2 years (relative to calling date) will not be delivered.
Konditionsmeldungen | ```konditionsmeldungen``` | N/A | csv/UTF-8 | Konditionsmeldungen that are currently known in BaufiSmart

## Quickstart

The report creation is asynchronous to avoid network timeouts during the report creation. Active testing is required for the completion of the report.

The procedure is the same for all Europace reports:

1. request report
2. query report status
3. retrieve report data

![Sequenzdiagramm](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/europace/report-api/master/resources/processing-report-api.iuml?token=AFSAZZEIDC253X3GO74BDNK7R2KOC)

![Tip](https://img.shields.io/badge/-Tip-yellow) To help you test our APIs and your use case as quickly as possible, we've put together a [Postman Collection](https://github.com/europace/api-schnellstart) for you to easily follow along.

The previous day's data will be available around 5:00 am.

## Example: Requesting a raw sales data report

### Authentication
Please use [![Authentication](https://img.shields.io/badge/Auth-OAuth2-green)](https://github.com/europace/authorization-api) to get access to the report API.

Which scope you need for which report you can see in the [Overview Europace-Reports](https://docs.api.europace.de/baufinanzierung/report/report-api/#europace-reports).

:warning: **Note** \.
The report for the identity of the OAuth token is always supplied. To retrieve a report for another person/organisation, [impersonated](https://docs.api.europace.de/baufinanzierung/authentifizierung/#wie-authentifiziere-ich-verschiedene-benutzer-mit-einem-client-impersionieren) or another client should be used if necessary. For the Produktanbieter-Report, the [Produktanbieter](https://docs.api.europace.de/common/glossary/) must be authenticated as the client.

### 1. request report
With the request, the generation of the report is started at Europace. This process can take several minutes depending on the complexity and the requested time period. The valid parameters for the report creation can be found in the corresponding report description.

Request for Vertriebs-Rohdaten-Report:
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

The header variable `Location` points to the endpoint for the next step: to query the report status.

### 2. query report status

This step is the same for all Europace reports.

Request:
```
curl --location --request GET 'https://report.api.europace.de/rohdaten/{report-processing-id}' \
--header 'X-Trace-Id: ' \
--header 'Authorization: Bearer {access-token}'
```

**(a)** Response if the report is still in progress:
```
Header:
  - Status: 200 OK
Body:
{
  "status": "PROCESSING"
}
```
Please query again in 20s (recommendation).

**(b)** Response if the report has been created:
```
Header:
  - Status: 303 SEE OTHER
  - Location: https://{path-to-file}
```
The header variable 'Location' points to the report data.

### 3. retrieve report-data

This step is the same for all Europace reports.

The URL pointing to the result data is only valid for a limited period of time. Anyone who has this URL can access the report.

Example for {path-to-file}:
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
The file type of `<file>` is described in the [Overview Europace Reports](https://docs.api.europace.de/baufinanzierung/report/report-api/#übersicht-europace-reports).  To facilitate opening with Excel, the encoding is UTF-8 with Byte Order Mark (BOM).

## Terms of use
The APIs are made available under the following [Terms of Use](https://docs.api.europace.de/nutzungsbedingungen/).

## Support
If you have any questions or problems, pls contact devsupport@europace2.de.
