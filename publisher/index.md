---
layout: default
---

# The doxter Practices API for publishers

The doxter Practice API is a service that returns information about doctor practices using HTTP requests. The service is accessed as an HTTP request and returns JSON only. The Pratice API uses a doxter ID to uniquely identify each entity (practice or doctor). The following api requests are available:

- [Practice Search](#practice-search) returns a list of practices
- [Doctor Search](#doctor-search) returns a list of doctors
- [Doctor Details](#doctor-details) requests return more details about a specific doctor, including available times

### <a name="authentication"></a>Authentication

The doxter Practice API uses BASIC AUTH to authenticate valid consumers. Contact Doxter to receive your credentials.  


## <a name="practice-search"></a>Practice Search

A Practice Search returns a list of practices along with summary information about each practice. The default is to return all active practices on doxter. The results may be paginated, in which case you find a `pageToken` in the metadata.  

A Search request is an HTTP URL of the following form:  

```
GET https://practices.doxterapis.com/v1/search?parameters
```

**Optional parameters**  

_(token explanation is taken verbatim from google calendar api)_  

- `sync_token` - Token obtained from the `next_sync_token` field returned on the last page of results from the previous request. It makes the result of this list request contain only entries that have changed since then. All events deleted since the previous list request will always be in the result set  

- `page_token` - Token obtained from the `next_page_token` field that is returned if the results are paginated. Optional, but always together with `sync_token`.  


## <a name="search-responses"></a>Search Responses

Search responses are returned JSON format. The following example shows a search result. The response has different fields depending on the type of practice.  

```JSON
{
    "results": [
        {
            "id": "54dde78366323021eb550000",
            "type": "GROUP",
            "formatted_name": "HNO Praxis Fuhrmann, Weikert, Hake",
            "name_components": {
                "company": "HNO Praxis Fuhrmann, Weikert, Hake"
            },
            "formatted_address": "Paracelsusstraße 1, 93053 Regensburg",
            "international_phone_number": null,
            "url": "http://www.doxter.de/practices/54dde78366323021eb550000",
            "job_titles": [
                "Hals-Nasen-Ohrenheilkunde"
            ],
            "focuses": [
                "Gleichgewichtsstörungen",
                "Schlechte Nasenatmung",
                "Verzögerte Sprachentwicklung"
            ],
            "photos": [
                "https://doxter.s3.amazonaws.com/assets/images/doctors/54dde7b46632380011bb0000.jpeg"
            ],
            "insurances": [
                "GKV",
                "PKV"
            ],
            "geometry": {
                "location": {
                    "lat": 49.01040279999999,
                    "lng": 12.096912599999996
                }
            },
            "address_components": {
                "street": "Paracelsusstraße 1",
                "zip": "93053",
                "city": "Regensburg"
            },
            "doctors":  [
               "54ddef6d3030312a5d420000",
               "54ddef6d3030312a5d420001",
            ]
            "status": "LISTED"
        },
        {
            "id": "54ddef6d3030312a5d420000",
            "type": "SINGLE",
            "formatted_name": "Dr. Marc Lamek",
            "name_components": {
                "gender": "Herr",
                "title": "Dr.",
                "first_name": "Marc",
                "last_name": "Lamek"
            },
            "formatted_address": "Rehmstr. 1a, 49080 Osnabrück",
            "international_phone_number": "+49 30 1234 456",
            "url": "http://www.doxter.de/doctors/54ddef6d3030312a5d420000",
            "rating": 4.9,
            "job_titles": [
                "Implantologe",
                "Ästhetischer Zahnarzt",
                "Endodontologe",
                "Parodontologe"
            ],
            "focuses": [
                "Implantologie zertifiziert",
                "Endodontologie zertifiziert"
            ],
            "photos": [
                "https://doxter.s3.amazonaws.com/assets/images/doctors/54ddef6d3030312a5d420000.png"
            ],
            "insurances": [
                "GKV",
                "PKV"
            ],
            "geometry": {
                "location": {
                    "lat": 52.2694645,
                    "lng": 8.041485100000045
                }
            },
            "address_components": {
                "street": "Rehmstr. 1a",
                "zip": "49080",
                "city": "Osnabrück"
            },
            "status": "LISTED",
            "available_times": [
                "2015-06-01T14:30:00+02:00",
                "2015-06-03T10:00:00+02:00",
                "2015-06-08T08:30:00+02:00",
                "2015-06-08T14:30:00+02:00",
                "2015-06-10T10:00:00+02:00",
                "2015-06-15T08:30:00+02:00",
                "2015-06-15T14:30:00+02:00",
                "2015-06-17T10:00:00+02:00"
            ],
            "doctors":  [
               "54ddef6d3030312a5d420000"
            ]

        }],
      "next_sync_token": "T1FNcjB4Nlc3OU1QczVBdFRqUGFmemxiSWU2alV6b2pnZ29ZQ2xSdElNdEJTNnBnWENNWHRiYStMZUJ0ZXpFTy0tcEhZVHd3eUwxeTBuQmpRcXdyUFA4dz09--c2e9930792560c97e6fd22c91a6ae3ba123232",
      "status": "OK"
}
```
The JSON response contains two root elements:  

`meta` - contains metadata on the request. See Status and Synchronization below.  
`results` - contains an array of practices with information about each. See Search Results for information about these results.  

### <a name="metadata"></a>Metadata

#### Status Codes

- `OK` indicates that no errors occurred  
- `INVALID_TOKEN` indicates that either the sync or page token is invalid.  

#### Synchronization

Using the metadata from the Practice Service "you can keep data for all [practice] collections in sync while saving bandwidth" ([_Synchronize Resources Efficiently, Google Calendar API_](https://developers.google.com/google-apps/calendar/v3/sync)).  

- `next_sync_token` - Token used at a later point in time to retrieve only the entries that have changed since this result was returned. Omitted if further results are available, in which case next_page_token is provided.  

- `next_page_token` - Token used to access the next page of this result. Omitted if no further results are available, in which case next_sync_token is provided.  

## <a name="search-results"></a>Search Results

When the Practice service returns JSON results from a search, it places them within a results array. Even if the service returns no results it still returns an empty results array.



Each result within the results array may contain the following fields:  

- `id` - contains a unique stable identifier denoting this practice  
- `type` - describing the given practice. Possible types are:  
  - `"SINGLE"` an individual practice with one doctor  
  - `"GROUP"` for group practices.  
- `industries[]` generalizes all practice's fields into one or several of:  
  - `"ZAHNMEDIZIN"` - All dentist related fields  
  - `"FACHMEDIZIN"` - All regular doctor fields  
  - `"PRIVATMEDIZIN"` - Privat fields not covered by public insurance  
- `formatted_name` is a string containing the human-readable name of this practice  
- `name_components[]` is an array of separate name components. It can contain:  
  - `gender` can be either: `"Herr"` or `"Frau"`  
  - `title` contains the academic title like `"Dr."` or `"Prof. Dr."`  
  - `first_name` is the first name of the person  
  - `last_name` is the last name of the person  
  - `company` the name of the practice  
- `formatted_address` is a string containing the human-readable address of this place.  
- `international_phone_number` contains the place's phone number in international format.International format includes the country code, and is prefixed with the plus (+) sign. For example, the `international_phone_number` for Doxters's Berlin, Germany office is `"+49 30 609 849 65"`  
- `url` contains the URL of the official doxter page for this practice  
- `rating` the user's overall rating for this practice. Optional.  
- `rating_count` the number of ratings for this practice. Optional, but always given when `rating` is.  
- `job_titles[]` contains an array of practice type or gender inflected specialties. For example: dentist will inflect to `"Zahnarzt"` for a single male practice, `"Zahnärztin"` for a single female practice and `"Zahnmedizin"` for a group practice.  
- `focuses[]` an array of focus areas of the practice. In contrast to the `job_titles` this is not a curated list. The list may contain entries like `"Ästhetische Zahnmedizin`", `"Prophylaxe"` or `"Invisalign"`  
- `photos[]` an array of photo urls. A Practice Search will return at most one photo object.  
- `insurances[]` is a list of accepted insurances. It can contain `GKV` and `PKV`.  
- `geometry` contains geometry information about the result, including the location (geocode) of the place.  
- `address_components[]` is an array of separate address components used to compose a given address. It contains:  
  - `street`  
  - `zip`  
  - `city`  
- `status`: Status of the practice. Possible values are:  
  - `"DELETED"` - the practice has been deleted from doxter  
  - `"LISTED"` - the practice is listed on doxter  
- `doctors` - a list of doxter ids of doctors that work in that practice
- `available_times[]`: An array of times in ISO8601 format with available times of this practice or this doctor. Optional.  

## <a name="doctor-search"></a>Doctor search

A doctor search returns a list of doctors along with summary information about each doctor; additional information is available via a [doctor-details](#doctor-details) query.

```
GET https://practices.doxterapis.com/v1/doctors
```

**Optional parameters**  

_(token explanation is taken verbatim from google calendar api)_  

- `sync_token` - Token obtained from the `next_sync_token` field returned on the last page of results from the previous request. It makes the result of this list request contain only entries that have changed since then. All events deleted since the previous list request will always be in the result set  

- `page_token` - Token obtained from the `next_page_token` field that is returned if the results are paginated. Optional, but always together with `sync_token`.  


### Search Responses

The following example shows a doctor search response.

```JSON
{
  "status": "OK",
  "next_page_token": "ZFlDUkFiaXJxS29qb0g2cFpwQkk5Zz09LS1lT2hVdUhyd0paMW10TU9tVXRlbzhBPT0=--3baee41914192c4819f9bdf9705fdd4ed840373d",
  "results": [
    {
      "id": "5099327ddf34d500020061f3",
      "url": "https://www.doxter.de/frauenarzt-hannover/dr-gabriele-toboll-jacob",
      "gender": "Frau",
      "title": "Dr.",
      "first_name": "Gabriele",
      "last_name": "Toboll-Jacob",
      "job_titles": [
              "Frauenärztin",
              "Psychotherapie"
              ],
      "photo": "https://doxter-de.s3.amazonaws.com/assets/images/doctors/5099327ddf34d500020061f3.jpeg",
      "rating_count": 0,
      "status": "LISTED"
    }
  ]
}
```

#### Status Code

- `OK` indicates that no errors occurred  
- `INVALID_TOKEN` indicates that either the sync or page token is invalid.  

#### Pagination and Synchronization

Using the metadata from the Practice Service "you can keep data for all [doctor profile] collections in sync while saving bandwidth" ([_Synchronize Resources Efficiently, Google Calendar API_](https://developers.google.com/google-apps/calendar/v3/sync)).  

- `next_sync_token` - Token used at a later point in time to retrieve only the profiles that have changed since this result was returned. Omitted if further results are available, in which case next_page_token is provided.  

- `next_page_token` - Token used to access the next page of this result. Omitted if no further results are available, in which case next_sync_token is provided.  

## Results

Each result within the results array may contain the following fields:  

- `id` - contains a unique stable identifier denoting this profile
- `url` contains the URL of the official doxter page for this practice
- `gender` can be either: `"Herr"` or `"Frau"`  
- `title` contains the academic title like `"Dr."` or `"Prof. Dr."`  
- `first_name` is the first name of the person  
- `last_name` is the last name of the person  
- `job_titles[]` contains an array of practice type or gender inflected specialties. For example: dentist will inflect to `"Zahnarzt"` for a male doctor, `"Zahnärztin"` for a female doctor.
- `photo` an photo photo url of the doctor profile
- `rating` the patients's overall rating for this practice. Optional.  
- `rating_count` the number of ratings for this profile. Optional, but always given when `rating` is.  
- `status`: Status of the practice. Possible values are:  
  - `"DELETED"` - the practice has been deleted from doxter  
  - `"LISTED"` - the practice is listed on doxter  

## <a name="doctor-details"></a>Doctor details

This service returns detailed information about a doctor. A request is an HTTP URL of the following form:  

```
GET https://practices.doxterapis.com/v1/doctors/{id}
```

**required parameters**  

- `id` - doxter doctor id obtained from a doctor or practice search

### Search Responses

```JSON
{
    "id": "4dc96eb2dc6e9a0af3000045",
    "gender": "Herr",
    "title": "Dr. ",
    "first_name": "Dietrich",
    "last_name": "Gericke",
    "job_titles": [
        "Zahnarzt"
    ],
    "focuses": [
        "Master of Science Endodontie",
        "Wurzelkanalbehandlung unter dem Dentalmikroskop"
    ],
    "photo": "https://doxter-de.s3.amazonaws.com/assets/images/doctors/4dc96eb2dc6e9a0af3000045.jpeg",
    "rating": 4.6,
    "rating_count": 9,
    "available_times": [
        "2015-11-16T07:00:00.000Z",
        "2015-11-16T07:30:00.000Z"
    ],
    "url": "https://www.doxter.de/doctors/4dc96eb2dc6e9a0af3000045",
    "status": "LISTED"
}
```
