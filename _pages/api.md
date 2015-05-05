---
layout: page
title: "API"
category: doc
date: 2015-05-05 11:16:50
---


# Optinomic - API - Documentation


# Generalities

Every endpoint returns a JSON object with a key describing the data returned, e.g. `/patients` returns `{"patients": [...]}` and `/patients/1` returns `{"patient": ... }`. This is for consistency and because it must always return an object.

## Errors

In case of errors (4xx or 5xx), a JSON object is returned with an error key, e.g. 

    $ curl localhost:3001/signin -X POST -d 'email=admin@optinomics.com'
    {"error":"Parameter \"password\" is missing."}

## Entities vs objects

**Changed: all endpoints return entities**

Depending on the context, elements such as patients, modules, etc. are returned as-is or as an entity. For instance, when requesting a specific patient with `/patients/42`, it is useless to return the ID 42 so the patient is returned like this:

    {"patient":{"email":"a@b.com",...}}

On the other hand, when requesting the watchlist with `/watchlist/1`, we need to return patient IDs so the API returns entities like this:

    {
      "patients": [
        {
          "id": 42,
          "data": { "email": "a@b.com", ... }
        }, ...
      ]
    }

In the documentation thereafter, it will be shown with `ENTITY` and `OBJECT`.

## Lists as parameters

When the following documentation specifies a parameter such as `modules` as a parameter being a list, the client should set the parameter `modules_count` to the number of items in the list and `modulesN` to the value for each item. For example, if `modules = ["first", "second"]`, the corresponding query string is `modules_count=2&modules0=first&modules1=second`.

# Endpoints

The front-end should take care of all possible cases, the API might return an 500 (or other) for all endpoints. For example, if a parameter is missing, it returns a 400 Bad Request or, if not logged in or with the wrong user, it might return a 401 Unauthorized. Or even a 404 if something is not found.

## POST /signin

**Parameters:** `email`, `password`

**Responses:**
* 401 Unauthorized if the credentials are wrong
* 200 OK in case of success with a JSON like this: `{"user_id": 1, "token": "b2bb30a2-72bb-4446-be0e-1942a8d9c202"}`.

The front-end must then send this token as a HTTP header named `X-API-Token` in all following requests to the API. Also, the server does not know the current user when receiving requests, that's why it is returned when signing in so the front-end can keep it in memory and use it later on when, for example, requesting the watchlist with `POST /watchlist/USER_ID`.

## POST /signout

**Parameters:** None

**Responses:**
* 204 No Content (no JSON)

## GET /watchlist/:user_id

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"patients": [ENTITIES]}`.

## POST /watchlist/:user_id/watch/:patient_id

**Parameters:** None

**Responses:**
* 204 No Content (no JSON) in case of success.

## POST /watchlist/:user_id/unwatch/:patient_id

**Parameters:** None

**Responses:**
* 204 No Content (no JSON) in case of success.

## POST /patients

**Parameters:** `first_name`, `last_name`, `birth_name`, `gender`, `title`, `birthdate`, `address1`, `address2`, `city`, `zip_code`, `country`, `language`, `deceased`, `email`, `phone_home`, `phone_mobile`, `cis_pid`, `ahv`, `notes`, `four_letter_code`

**Responses:**
* 400 Bad Request in case of missing parameters or validation error
* 201 Created with a JSON like this: `{"patient": ENTITY}`.

## GET /patients

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"patients": [ENTITIES]}`.

## GET /patients/:patient_id

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"patient": PATIENT}`.

## PUT /patients/:patient_id

**Parameters:** `first_name`, `last_name`, `birth_name`, `gender`, `title`, `birthdate`, `address1`, `address2`, `city`, `zip_code`, `country`, `language`, `deceased`, `email`, `phone_home`, `phone_mobile`, `cis_pid`, `ahv`, `notes`, `four_letter_code`

**Responses:**
* 400 Bad Request in case of missing parameters or validation error
* 204 No Content (no JSON) in case of success

## DELETE /patients/:patient_id

**Parameters:** None

**Responses:**
* 204 No Content (no JSON)

## GET /patients/:patient_id/stays

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"stays": [ENTITIES]}`.

## PUT /stays/:stay_id

**Parameters:** `therapist_id` (this is to overwrite the therapist set by the CIS interface, leave empty to use the default one)

**Responses:**
* 400 Bad request in case of missing parameter
* 204 No Content (no JSON) in case of succes

## POST /users

**Parameters:** `email`, `password`, `gender`, `title`, `first_name`, `last_name`, `birthday`, `phone`, `description`, `role`, `initials`, `ou`, `cis_uid`

**Responses:**
* 400 Bad Request in case of missing parameters or validation error
* 401 Unauthorized when trying to create a user with admin role from an account without admin access
* 201 Created with a JSON like this: `{"user": ENTITY}`

## GET /users

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"users": [ENTITIES]}`.

## GET /users/:user_id

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"user": ENTITY}`.

## PUT /users/:user_id

**Parameters:** `email`, `gender`, `title`, `first_name`, `last_name`, `birthday`, `phone`, `description`, `initials`, `ou`, `cis_uid`

**Responses:**
* 400 Bad Request in case of missing parameter or validation error
* 204 No Content (no JSON) in case of success

## DELETE /users/:user_id

**Parameters:** None

**Responses:**
* 204 No Content (no JSON)

## GET /patients/:patient_id/modules

**Parameters:** `stay_id` (optional)

**Responses:**
* 200 OK with a JSON like this: `{"activated_patient_uses_modules": [{"entity": PUM ENTITY, "module": MODULE OBJECT}], "deactivated_modules": [MODULE OBJECTS]}` (modules are not in the DB, so not entities) with deactivated modules empty if `stay_id` is passed.

## POST /patients/:patient_id/activate_module

**Parameters:** `module_identifier` and `stay_id` (optional)

**Responses:**
* 400 Bad Request in case of missing parameter or validation error (e.g. module already activated)
* 204 No Content (no JSON) in case of success

## POST /patients/:patient_id/deactivate_module

**Parameters:** `module_identifier`, `stay_id` (optional)

**Responses:**
* 400 Bad Request in case of missing parameter
* 204 No Content (no JSON) in case of success

Note: If no `stay_id` is given, any patient_uses_module will be deleted.

## GET /patients/:patient_id/events

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this `{"events": [ENTITIES]}`

## POST /patient_uses_modules/:patient_uses_module_id/schedule_survey

**Parameters:** `survey_identifier`

**Responses:**
* 201 Created with a JSON like this `{"event": ENTITY}`

## GET /patients/:patient_id/survey_responses/:module_identifier

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this `{"survey_responses": [ENTITY]}`

## POST /run_sql

**Parameters:** `query` and `delimiter` (optional, default ';')

**Responses:**
* 200 OK with CSV as the response body

## GET /patient_groups

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"patient_groups": [ENTITIES]}`

## POST /patient_groups

**Parameters:** `name`, `sql_filter` and `modules` (list)

**Responses:**
* 400 Bad Request in case of validation error or missing parameter
* 401 Unauthorized if not admin (as the user can write SQL)
* 201 Created with a JSON like this: `{"patient_group": ENTITY}`

## PUT /patient_groups/:patient_group_id

**Parameters:** `name`, `sql_filter` and `modules` (list)

**Responses:**
* 400 Bad Request in case of validation error or missing parameter
* 401 Unauthorized if not admin
* 204 No Content (no JSON)

## DELETE /patient_groups/:patient_group_id

**Parameters:** None

**Responses:**
* 204 No Content (no JSON)

## GET /patient_groups/:patient_group_id/patients

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"patients": [ENTITIES]}`

## GET /modules

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"modules": [OBJECTS]}`

## GET /modules/disabled

Return the list of modules installed but not yet enabled in the application.

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"modules": [OBJECTS]}`

## GET /modules/errors

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"module_errors": [STRINGS]}`

## GET /module_activations

Return the list of all module activations, that is, all the entities in the DB enabling an installed module. These entities contain an optional name overwrite for the module. Deleting one will disable the module.

**Parameters:** None

**Responses:**
* 200 OK with a JSON like this: `{"module_activations": [ENTITIES]}`

## POST /module_activations

**Parameters:** `module_identifier`, `name_overwrite`

**Responses:**
* 400 Bad Request in case of validation error
* 200 OK with a JSON like this: `{"module_activation": ENTITY}`

## PUT /module_activations/:activation_id

**Parameters:** `name_overwrite`

**Responses:**
* 400 Bad Request in case of validation error
* 204 No Content (no JSON) in case of success

## DELETE /module_activations/:activation_id

**Parameters:** None

**Responses:**
* 204 No Content (no JSON) in case of success

## GET /modules/:module_identifier/view/:location

**Parameters:** None

**Responses:**
* 200 OK with HTML that can be shown in an iframe

**Notes:** 
* Available locations are specified in the module JSON. (see GET /modules for example)
* Some helpers are available to the module's JS. For them to work, some data need to be passed in the URL hash. Here's how to include a module view in a page:

```html
<iframe src="/modules/some_module/view/dashboard#patient_id=2,token=542b2a71-c6b3-424c-939d-8ab5e57c8193">
</iframe>
```

# Examples

## Patient groups

Let's create a patient group that filters only men from Switzerland and activate the module with ID "1".

```
$ curl -v -H 'X-API-Token: 79dca73c-68c4-4ace-b9a3-badf384b6e25' localhost:3001/patient_groups -X POST -d 'sql_filter=gender%20%3D%20%27Male%27%20AND%20country%20%3D%20%27CH%27&name=my_patient_group&modules_count=1&modules0=1'
* Hostname was NOT found in DNS cache
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 3001 (#0)
> POST /patient_groups HTTP/1.1
> User-Agent: curl/7.37.0
> Host: localhost:3001
> Accept: */*
> X-API-Token: 79dca73c-68c4-4ace-b9a3-badf384b6e25
> Content-Length: 118
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 118 out of 118 bytes
< HTTP/1.1 201 Created
< Transfer-Encoding: chunked
< Date: Thu, 26 Mar 2015 10:18:56 GMT
* Server Warp/3.0.5.2 is not blacklisted
< Server: Warp/3.0.5.2
< Set-Cookie: session=1w; path=/; expires=Sat, 26-May-2018 20:05:35 UTC;
< Content-Type: application/json; charset=utf-8
<
* Connection #0 to host localhost left intact
{"patient_group":{"id":1,"data":...}}
```

The patient group has been created:

```
$ curl -v -H 'X-API-Token: 79dca73c-68c4-4ace-b9a3-badf384b6e25' localhost:3001/patient_groups | jq .
* Hostname was NOT found in DNS cache
*   Trying 127.0.0.1...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to localhost (127.0.0.1) port 3001 (#0)
> GET /patient_groups HTTP/1.1
> User-Agent: curl/7.37.0
> Host: localhost:3001
> Accept: */*
> X-API-Token: 79dca73c-68c4-4ace-b9a3-badf384b6e25
>
< HTTP/1.1 200 OK
< Transfer-Encoding: chunked
< Date: Thu, 26 Mar 2015 10:20:01 GMT
* Server Warp/3.0.5.2 is not blacklisted
< Server: Warp/3.0.5.2
< Set-Cookie: session=ZA; path=/; expires=Sat, 26-May-2018 20:06:40 UTC;
< Content-Type: application/json; charset=utf-8
<
{ [data not shown]
100   130    0   130    0     0  23465      0 --:--:-- --:--:-- --:--:-- 43333
* Connection #0 to host localhost left intact
{
  "patient_groups": [
    {
      "id": 1,
      "data": {
        "name": "my_patient_group",
        "sql_filter": "gender = 'Male' AND country = 'CH'",
        "modules": [
          "1"
        ]
      }
    }
  ]
}
```

In the background, a CRON task (the executable is `therapy-server-patient-groups`) will detect patients belonging to a group and activate the specified modules for them.

When modifying a patient group, adding a module will activate for all already-detected patients in a group.