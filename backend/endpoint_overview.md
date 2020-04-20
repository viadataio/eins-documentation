# Backend Endpoint Overview

This document details the process of each EP of the API including the responses and requests.

### Requests
To query the API the request header must contain the access token obtained when authenticating the device.
```
“Authorization: Bearer <access token>”
```
A blanket request rate limit per IP address is set for all endpoints. If the rate limit is exceeded requests will not be successful and a 429 response code returned.

### Responses
All responses take on the standard form shown in the example below unless otherwise specified.

```JSON
{
    "data": null,
    "meta": {
        "success": true,
        "code": 200,
        "message": null
    }
}
```

## Device Authentication
### Authentication - POST: {{URL}}v1/devices
Authenticates a device using a standard key pair signature verification (RSA2048). The public key must be in the der64 format.

##### NEW DEVICE
For a new device public key, operating system, language (alpha2 format) are required fields. Push token is optional.

##### PREVIOUSLY AUTHENTICATED DEVICE
For a device that has already been authenticated the public key and operating system fields cannot not be included. This is to prevent the switching of a device's key pair which would be a security risk. Language and push token are optional. If included, it will update the device's language and or push token.

#### Requests
```JSON
{
	"AppId": "",
	"publicKey": "",
	"operatingSystem": "",
	"pushToken": "",
	"language": "",
	"Signature":{
	 	"plainTextData": "",
	 	"signedData": ""
	 }
}

```
#### Responses
Successful response
```JSON
{
    "data": {
        "accessToken": "",
        "accessTokenExpiry": "",
        "signature": {
            "plainTextData": "",
	    "signedData": ""
        }
    },
    "meta": {
        "success": true,
        "code": 201,
        "message": "Device successfully authenticated."
    }
}

```
Unsuccessful response - 401: 
* When the AppId and signature pair are invalid.
* When a public key or operating system update is attempted.

### Delete device - DELETE: {{URL}}v1/devices
Deletes all information associated with a device from the API, this includes the AppId, PublicKey, OperatingSystem, LanguageId, and PushToken.

Donated data and survey submissions can not be removed through this EP because there is no direct or indirect link between this data and a device.

### Add/Update push token - POST: {URL}}v1/devices/push_token
Adds a push token to or updates the existing push token of an already authenticated device.
#### Requests
```JSON
{
	"pushToken": ""
}
```
### Delete push token - DELETE: {{URL}}v1/devices/push_token
Deletes the push token associated with an authenticated device.

## GeoData 

### Get geo-zone specifications - GET: {{URL}}v1/geodata/zones/specifications
Gets a list of all geo zone specifications currently supported by the server along with some basic information.

#### Responses
```JSON
{
    "data": [
        {
            "shape": "Square",
            "latitudeCoordinateRange": 0.01000000,
            "longitudeCoordinateRange": 0.01000000,
            "estimatedMaxMetreRange": 1000,
            "active": true,
            "id": 3
        },
        {
            "shape": "Square",
            "latitudeCoordinateRange": 0.00010000,
            "longitudeCoordinateRange": 0.00010000,
            "estimatedMaxMetreRange": 1,
            "active": true,
            "id": 4
        }
    ],
    "meta": {
        "success": true,
        "code": 200,
        "message": null
    }
}
```

### Stream zone exposures - POST: {{URL}}v1/geodata/zones/exposure
Returns information on zones and their exposure risk based upon the time and location that is specified. Creates a file internally and then streams this to the client.
#### Requests
```JSON
{
  "geoZoneSpecId": 6,
  "exposureThreshold": 0,		\\ Percentage value between 0-100%
  "areaRequests": [
    {
      "lat": -33.25188,
      "long": 18.41853,
      "start": 1586964196,		\\ Unix
      "end": 1586965196			\\ Unix
    },
    {
      "lat": -33.88356,
      "long": 18.15303,
      "start": 1586963806,
      "end": 1586964806
    }
]
}
```
#### Responses
```JSON
{
    "Data": {
        "GeoZoneSpecId": 4,
        "latRange": 0.0001,
        "longRange": 0.0001,
        "Zones": [
            {
                "centreLat": -33.52805,
                "centreLong": 18.42845,
                "Exposures": [
                    {
                        "StartTime": 1585970874,
                        "EndTime": 1585971474,
                        "Exposure": 69.43
                    }
                ]
            },
            {
                "centreLat": -33.52335,
                "centreLong": 18.42165,
                "Exposures": [
                    {
                        "StartTime": 1586060874,
                        "EndTime": 1586061474,
                        "Exposure": 68.11
                    }
                ]
            }
        ]
    },
    "Meta": {
        "Success": true,
        "Code": 200,
        "Message": null
    }
}
```
### Log coordinates - POST: {{URL}}v1/geodata/coordinates
Donates a list of infected coordinates to the api. Once data is donated to the API it can not be deleted from the API.
#### Requests
```JSON
[
  {
    "lat": -33.58085,
    "long": 18.50831,
    "epoc": 1587204820,
    "confidence": 0.738
  },
  {
    "lat": -33.63025,
    "long": 18.55054,
    "epoc": 1587204820,
    "confidence": 0.620
  }
]
```
## Survey
### Get symptoms - GET: {{URL}}v1/symptoms
Gets a key value pair list of the symptoms in the language specified in the query parameter.
#### Responses
```JSON
{
    "data": [
        {
            "key": "question_positive_symptom-1",
            "value": "Sore throat"
        },
   ...
        {
            "key": "question_positive_symptom-12",
            "value": "Loss of smell"
        }
    ],
    "meta": {
        "success": true,
        "code": 200,
        "message": null
    }
}
```
### Suvey submission - POST: {{URL}}v1/submission/covid
Submits survey data. There is no limit to the number of submissions. Various validations on what you can submit depending on what the stats are. Symptoms must be submitted using the text copy key. Language and country must be submitted using the alpha2 format. If “IsSymptomatic” is set to false no symptoms may be provided.

Status must be one of the following; Positive, Negative, Unsure, or Recovered.
Survey submissions are not linked directly or indirectly to a device.
#### Requests
Positive
```JSON
{
	"Status":"Positive",
	"PositiveTestDate":"2020-03-29 13:00:09.1359267",
	"Country":"GB",
	"Language":"EN",
	"Age":25,
	"IsSymptomatic":false,
	"Symptoms": [
		"question_positive_symptom-1",
		"question_positive_symptom-3"
		]
	"SymptomsFrom":"2020-03-29 13:00:09.1359267"
}
```
Negative
```JSON
{
	"Status":"Negative",
	"NegativeTestDate":"2020-03-30 13:00:09.1359267",
	"Country":"GB",
	"Language":"EN",
	"Age":25,
	"Symptoms": [
		"question_positive_symptom-1",
		"question_positive_symptom-3"
		]
	"SymptomsFrom":"2020-03-29 13:00:09.1359267"
}
```
Recovered
```JSON
{
	"Status":"Recovered",
	"PositiveTestDate":"2020-03-29 13:00:09.1359267",
	"NegativeTestDate":"2020-03-30 13:00:09.1359267",
	"Country":"GB",
	"Language":"EN",
	"Age":25,
	"IsSymptomatic":false
}
```
Unsure
```JSON
{
	"Status":"Unsure",
	"Country":"GB",
	"Language":"EN",
	"Age":25,
	"IsSymptomatic":false,
	"Symptoms": [
		"question_positive_symptom-1",
		"question_positive_symptom-3"
		]
	"SymptomsFrom":"2020-03-29 13:00:09.1359267"
}
```

## Statistics
### Get statistics - GET: {{URL}}v1/statistics/covid/google?countries={Alpha2}
Gets the latest covid statistics of the requested country. Statistics are retrieved through Google’s BigQuery API from the Johns Hopkins University (JHU) Repository of aggregated coronavirus COVID-19 cases.

#### Responses
```JSON
{
    "Data": {
        "CountryName": "Global",
        "ContractedCount": 2265670,
        "DeathCount": 154249,
        "RecoveredCount": 572266,
        "Source": "GoogleBigQuery",
        "SourceCreatedAt": "2020-04-17T00:00:00",
        "SourceUrl": "https://console.cloud.google.com/marketplace/details/johnshopkins/covid19_jhu_global_cases?filter=solution-type:dataset&q=covid&id=430e16bb-bd19-42dd-bb7a-d38386a9edf5&_ga=2.196272622.-312723631.1586190188&pli=1",
        "CreatedAt": "2020-04-18T01:00:08.545734"
    },
    "Meta": {
        "Success": true,
        "Code": 200,
        "Message": null
    }
}
```
