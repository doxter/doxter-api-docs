---
layout: default
---

# Calendar API

## Doctena Synchronisation API v2.1
(formerly known as Doxter Synchronisation API)

#### [Introduction](#01)
#### [Calendars](#02)
#### [GET /calendars](#03)
#### [GET /calendars/{id}](#04)
#### [Timeblocks](#05)
#### [GET /calendars/{id}/timeblocks{?updated, show_deleted}](#06)
#### [GET /calendars/{id}/timeblocks/{id}](#07)
#### [Bookings](#08)
#### [GET /calendars/{id}/bookings{?updated,status}](#09)
#### [GET /calendars/{calendar_id}/bookings/{id}](#10)
#### [PUT /calendars/{calendar_id}/bookings/{id}](#11)
#### [Blockings](#12)
#### [GET /calendars/{id}/blockings/{?updated}](#13)
#### [POST /calendars/{calendar_id}/blockings](#14)
#### [PUT /calendars/{calendar_id}/blockings/{id}](#15)
#### [DELETE /calendars/{calendar_id}/blockings/{id}](#16)

## <a name="01"></a>Introduction

Welcome to the Doctena API specification for synchronisation. With the Doctena API you can access your calendars, get updates of new bookings, reschedule bookings and block available times.  

### Authentication

HTTP Basic authentication. Username and password come from a valid login on Doctena. One account can have many logins. One login can also be connected to many accounts. Passwords can be reset on the **login page**.  

API Base URL: https://www.doxter.de/api/v2/ (still on doxter.de for backwards compatibilty with existing implementations)

### Fields and Parameters

All times must be specified in **iso8601 format**.  

4 RESTful verbs are supported:  

- GET: find objects  
- POST: create new objects  
- PUT: update objects  
- DELETE: remove objects  

## <a name="02"></a>Calendars

Booked bookings are stored on the calendars. Calendars are created for the type of patients and problems a practice wants to be able to book. Furthermore calendars contain rules such as the window within patients are able to see available times. Since many practices discriminate between patients and services they offer, calendars contain an internal name to indicate what the calendar is used for.  


#### <a name="03"></a>GET /calendars

Get all calendars from all accounts the user has access rights to.  

```
curl "https://www.doxter.de/api/v2/calendars" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the response  

- **_id:** id of the calendar  
- **name:** the name of the calendar for internal discrimination  
- **profile_name:** The name of the doctor or practice the calendar is shown on  

```JSON
[
    {
        "_id": "4f02608e18cb71000100004b",
        "name": "Standardkalender PKV",
        "profile_name": "Dr. Max Mustermann"
     },
    {
        "_id": "4f02608e18cb71000100004c",
        "name": "Standardkalender GKV",
        "profile_name": "Dr. Max Mustermann"
     }
]
```

error case (wrong credentials):  

```
curl -I "https://www.doxter.de/api/v2/calendars" -u hacker@example.com:fakepwd
```

response(status): 401  


#### <a name="04"></a>GET /calendars/{id}

Retrieve a single calendar. Includes general information about the calendar not including  bookings nor timeblocks.  

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{
    "_id": "4f02608e18cb71000100004b",
    "name": "Standardkalender",
    "profile_name": "Dr. Max Mustermann"
}
```

### Error Case (Calendar does not exist):

```
curl -I "https://www.doxter.de/api/v2/calendars/doesnotexist" -u login@example.com:password
```

response status: 404  


## <a name="05"></a>Timeblocks

A timeblock defines a repeated time interval for which available times are generated. Not all generated times will be visible; valid duration, existing bookings and booking rules can prevent timeblock times from actually being shown to the user.  


#### <a name="06"></a>GET /calendars/{id}/timeblocks{?updated, show_deleted}

parameters:  

- **updated** (optional): returns only timeblocks which are modified at or after the provided time.  
- **show_deleted:** whether to include deleted timeblocks (with 'status' equals ‘deleted’) in the result. The default is False.  

Get all timeblocks for the given calendar.  

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/timeblocks" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the reponse:  

- **activated**: only if active, timeblocks will be generated  
- **begins_at**: inclusive time where the generation of available times starts  
- **ends_at**: exclusive time where generation stops. This time is not generated.  
- **beat**: An intervals in second in which times are generated.  
- **weekdays**: valid weeksdays are 0-6 . 0 is sunday, 1 is monday, 2 is tuesday.  
- **valid_forever**: Default is true, which means times are generated every week.  
- **valid_from**: Date from which the timeblock is valid  
- **valid_until**: Date to which the timeblock is valid  
- **updated_at**: Date on which the timeblock was last updated  
- **calendar_id**: calendar id,  
- **hide\_booked\_times**: if set to ‘true’, will still show the timeslots after being booked  
- **every\_n\_weeks**: timeblock will only recur every n weeks; default is 1  
- **margin\_to\_first\_timeslots**: in seconds, defines the leadtime to the first timeslot from now  
- **range\_limit\_for\_timeslots**: in seconds, defines the value until a booking can be made from now in the future  
- **status**: INACTIVE | ACTIVE | DELETED  

```JSON
[
    {
        "begins_at": "09:15",
        "ends_at": "13:30",
        "activated": true,
        "status": "ACTIVE",
        "recurrence":"FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR",
        "valid_forever": true,
        "valid_from": "2013-12-10T01:00:00+01:00",
        "valid_until": "2013-12-10T01:00:00+01:00",
        "beat": 3600,
        "weekdays": [
            1,
            2,
            3,
            4,
            5
        ]
    },
    {
        "begins_at": "14:00",
        "ends_at": "18:00",
        "activated": false,
        "status": "INACTIVE",
        "recurrence":"FREQ=WEEKLY;BYDAY=MO,TU,TH,FR",
        "valid_forever": true,
        "valid_from": "2013-09-25T00:00:00+02:00",
        "valid_until": null,
        "beat": 3600,
        "weekdays": [
            1,
            2,
            4,
            5
        ]
    }
]
```

#### <a name="07"></a>GET /calendars/{id}/timeblocks/{id}

Get a specific timeblock for a given calendar.   

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/timeblocks/4f02608e18cb71abc12341c1" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the reponse are the same as above.  

```JSON
    {
        "_id": "4f02608e18cb71abc12341c1",
        "begins_at": "09:15",
        "ends_at": "13:30",
        "activated": true,
        "status": "ACTIVE",
        "valid_forever": true,
        "valid_from": "2013-12-10T01:00:00+01:00",
        "valid_until": "2013-12-10T01:00:00+01:00",
      "recurrence":"FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR"
        "beat": 3600,
        "weekdays": [
            1,
            2,
            3,
            4,
            5
        ],
        "calendar_id": "4f02608e18cb71000100004b",
    }
```

## <a name="08"></a>Bookings

Bookings are the appointments made by patients on doxter.  

Through API booking cannot be created,  instead should be synchronized through blockings.  

#### <a name="09"></a>GET /calendars/{id}/bookings{?updated,status}

Get all updated bookings for the specified calendar. This will return all bookings since updated. Even canceled ones.  

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/bookings?updated=2014-03-08T10:54:14+01:00" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the response  

- **reason**: The problem or service (Behandlung) the patient booked an appointment  
- **confirmation_link**: Visiting the confirmation link prbookings doxter support to call the practice to verify if the booking has been seen.  
- **first_name**: Contains full_name  
- **insurance**: GKV (public), PKV (private)  
- **status**:  
  - **WAITING**: appointment was successfully booked, but practice hasn't reacted yet  
  - **HANDLED**: Event has been opened by the practice, but haven't been acted upon yet  
  - **EXPIRED**: appointment was successfully booked, but practice didn't react within Doxter's required confirmation period  
  - **CANCELLED**: Event has been explicitly cancelled by practice  
  - **CONFIRMED**: event has been confirmed by the practice  
  - **RESCHEDULED**: event has been rescheduled by the practice at least once  

```JSON
[
    {
        "_id": "5319db7acb2fa546f1000026",
        "created_at": "2014-03-07T15:45:14+01:00",
        "updated_at": "2014-03-10T10:45:43+01:00",
        "title": "Julian Hoffmann (doxter)",
        "starts": "2014-03-10T11:15:00+01:00",
        "ends": "2014-03-10T12:15:00+01:00",
        "status": "HANDLED",
        "reason": "Kontrolltermin",
        "no_show": "true",
        "contact": {
            "email": "julian.hoffmann@gmail.com",
            "first_name": "Julian Hoffmann",
            "phone_number": "015771982521"
        },
        "confirmation_link": "http://doxter.de/booking/confirmation?confirmation_token=ZYin4Kv4eTsJvbzUEQhj",
        "insurance": "GKV",
        "problem": "Vertigo / Drehschwindel",
    },
    {
        "_id": "5319db7acb2fa546f1000027",
        "created_at": "2014-03-07T15:45:14+01:00",
        "updated_at": "2014-03-10T10:45:43+01:00",
        "title": "Julian Hoffmann (doxter)",
        "starts": "2014-03-10T11:15:00+01:00",
        "ends": "2014-03-10T12:15:00+01:00",
        "status": "CONFIRMED",
        "reason": "Kontrolltermin",
        "no_show": "true",
        "contact": {
            "email": "julian.hoffmann@gmail.com",
            "first_name": "Julian Hoffmann",
            "phone_number": "015771982521"
        },
        "confirmation_link": "http://doxter.de/booking/confirmation?confirmation_token=ZYin4Kv4eTsJvbzUEQhj",
        "insurance": "GKV",
        "problem": "Vertigo / Drehschwindel",
    }
   ]
```


### Error case (update parameter is missing):

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/bookings" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{
    "error": "updated is missing"
}
```

#### <a name="10"></a>GET /calendars/{calendar_id}/bookings/{id}

Get the specified booking for the specified calendar.  

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/bookings/4f02608e18cb71000141cb11" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the response  

- **reason**: The problem or service the patient booked an appointment for  
- **confirmation_link**: Visiting the confirmation link prbookings doxter support to call the practice to verify if the booking has been seen.  
- **first_name**: Contains full_name  

```JSON
    {
        "_id": "4f02608e18cb71000141cb11",
        "created_at": "2014-03-07T15:45:14+01:00",
        "updated_at": "2014-03-10T10:45:43+01:00",
        "title": "Julian Hoffmann (doxter)",
        "starts": "2014-03-10T11:15:00+01:00",
        "ends": "2014-03-10T12:15:00+01:00",
        "status": "UNVERIFIED",
        "reason": "Kontrolltermin",
        "no_show": "true",
        "contact": {
            "email": "julian.hoffmann@gmail.com",
            "first_name": "Julian Hoffmann",
            "phone_number": "015771982521"
        },
        "confirmation_link": "http://doxter.de/booking/confirmation?confirmation_token=ZYin4Kv4eTsJvbzUEQhj",
        "insurance": "GKV",
        "problem": "Vertigo / Drehschwindel",
    }
```


### Error case (update parameter is missing):

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/bookings/4f02608e18cb71000100003c" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{
    "error": "Booking not found"
}
```

#### <a name="11"></a>PUT /calendars/{calendar_id}/bookings/{id}

Update a booking. Doesn’t have any side effects, including sending SMS or emails. All parameters are optional, but at least one of them should be provided.  

parameters:  

- **title** (optional): Title of the booking  
- **starts** (optional): Booking start time  
- **ends** (optional): Booking end time; must be after start time  
- **no_show** (optional): patient didn’t show up; can be set to true ONLY after the appointment time is passed, AND the booking status is already CANCELED or status parameter is CANCELED  
- **status** (optional): Booking status; valid values are  
  - **CONFIRMED**: signifies that the booking is confirmed  
  - **RESCHEDULED**: signifies that the booking is at least rescheduled once  
  - **CANCELED**: Booking is canceled; Canceled bookings will reopen the corresponding timeslot unless a blocking is created.  
- **send_notifications** (optional): if set to true, doxter will send sms and email notifications (for booking confirmations, cancellations and rescheduling) to users.   
  - *Notice 1*: You can force sending notifications again by only passing the send_notifications: true  
  - *Notice 2: If booking.starts is changed, and there is no status passed in, or the passed status is not cancelled, the status of the blloking will automatically change to RESCHEDULED.*  

```
curl -X PUT --data "starts=2014-04-07T11:00:00+01:00&ends=2014-04-07T12:00:00+01:00&status=CANCELED&title=’some title’" "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/bookings/4f02608e18cb71000141cb11" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the response  

- **reason**: The problem or service the patient booked an appointment for  
- **confirmation_link**: Visiting the confirmation link prbookings doxter support to call the practice to verify if the booking has been seen.  
- **first_name**: Contains full_name  

```JSON
    {
        "_id": "4f02608e18cb71000141cb11",
        "created_at": "2014-03-07T15:45:14+01:00",
        "updated_at": "2014-03-10T10:45:43+01:00",
        "title": "Julian Hoffmann (doxter)",
        "starts": "2014-03-10T11:15:00+01:00",
        "ends": "2014-03-10T12:15:00+01:00",
        "status": "CONFIRMED",
        "reason": "Kontrolltermin",
        "contact": {
            "email": "julian.hoffmann@gmail.com",
            "first_name": "Julian Hoffmann",
            "phone_number": "015771982521"
        },
        "confirmation_link": ""
    }
```


### Error case (update parameter is missing):

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/bookings/4f02608e18cb71000100003c" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{
    "error": "Booking not found"
}
```


## <a name="12"></a>Blockings

Blockings prevent times from being shown on doxter, e.g. vacations, or a specific date a practice doesn’t want to accept patients because of an equipment failure or other exceptional occurrences. Our API allows complete control over blockings from the user side. Be aware that if you create a blocking on a calendar, that calendar won’t show and allow bookings for that time frame.  


#### <a name="13"></a>GET /calendars/{id}/blockings/{?updated}

Get all updated blockings for the specified calendar.  

### Request params

- **updated**: ISO8601 time. Only gives events which where updated since from and including that time.  

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/blockings?updated=2014-03-08T10:54:14+01:00" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
[
    {
        "_id": "5319db7acb2fa546f1000026",
        "created_at": "2014-03-07T15:45:14+01:00",
        "updated_at": "2014-03-10T10:45:43+01:00",
        "title": "Julian Hoffmann (doxter)",
        "recurrence": "",
        "starts": "2014-03-10T11:15:00+01:00",
        "ends": "2014-03-10T12:15:00+01:00",
    },
    {
        "_id": "5319db7acb2fa546f1000027",
        "created_at": "2014-03-07T15:45:14+01:00",
        "updated_at": "2014-03-10T10:45:43+01:00",
        "title": "Julian Hoffmann (doxter)",
        "recurrence": "FREQ=WEEKLY;INTERVAL=3;WKST=SU;BYDAY=MO,TU,WE,TH,FR",
        "starts": "2014-03-11T13:15:00+01:00",
        "ends": "2014-03-11T18:25:00+01:00"
    }
   ]
```

### Error case (update parameter is missing):

```
curl "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/blockings" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{
    "error": "updated is missing"
}
```

#### <a name="14"></a>POST /calendars/{calendar_id}/blockings

Create a new blocking on the specified doxter calendar. Every event blocking on doxter will prevent the generation of available times during that duration.  

```
curl -X POST --data "starts=2014-03-11T13:18:33+01:00&ends=2014-03-11T15:19:22+01:00&title=BlockingTitle" "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/blockings" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

Fields in the reponse:  

- **_id**: Id of created event  
- **title**: Title of the event  
- **starts**: start time of event (inclusive)  
- **end**: end time of event (exlusive)  
- **recurrence**:  iCal compatible rrule: "RRULE:FREQ=WEEKLY;UNTIL=20150101T000000Z"  

```JSON
{
    "_id": "531ee370bb3357eb20000001",
    "created_at": "2014-03-11T11:20:32+01:00",
    "updated_at": "2014-03-11T11:20:32+01:00",
    "title": "EventTitle",
    "starts": "2014-03-11T13:18:33+01:00",
    "ends": "2014-03-11T15:19:22+01:00",
    "recurrence": "FREQ=WEEKLY;INTERVAL=3;WKST=SU;BYDAY=MO,TU,WE,TH,FR;UNTIL=20150101T000000Z",
}
```

#### <a name="15"></a>PUT /calendars/{calendar_id}/blockings/{id}

Updates an existing blocking. This can be used move an existing blocking on doxter to the another time. At least one parameter must be provided.  

**Important Notice**: _When providing a recurrence, keep in mind that the end date for the blocking is expected to be included in the recurrence rule. The end *TIME* for the event is derived from the time that is provided in the ends field. If the recurrence is infinite, the end date will be start date + 99 years, and the ends time will be appended to it. If the UNTIL is provided in the rule, the ends field will become the until date + the ends field time part._  

**parameters:**  

- title (optional)  
- starts (optional)  
- ends (optional)  
- recurrence (optional)  

```
curl -X PUT --data "starts=2014-04-07T11:00:00+01:00&ends=2014-04-07T12:00:00+01:00"  "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/blockings/5315f6493f977fb23a00005f" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{
    "_id": "5315f6493f977fb23a00005f",
    "created_at": "2014-03-11T11:20:32+01:00",
    "updated_at": "2014-03-11T11:45:32+01:00",
    "title": "EventTitle",
    "starts": "2014-04-07T11:00:00+01:00",
    "ends": "2014-04-07T12:00:00+01:00",
    "recurrence": "FREQ=WEEKLY;INTERVAL=3;WKST=SU;BYDAY=MO,TU,WE,TH,FR;UNTIL=20150101T000000Z"
}
```


#### <a name="16"></a>DELETE /calendars/{calendar_id}/blockings/{id}

Deletes an existing blocking.  

```
curl -X DELETE --data "starts=2014-04-07T11:00:00+01:00&ends=2014-04-07T12:00:00+01:00"  "https://www.doxter.de/api/v2/calendars/4f02608e18cb71000100004b/blockings/5315f6493f977fb23a00005f" -u login@example.com:password
```

### Response Headers

```
Content-Type: application/json
```

### Response Body

```JSON
{}
```

