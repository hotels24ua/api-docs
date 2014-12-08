#General API Convention

##The Service
api.hotels24.ua - the service provides HTTP interface to access hotels24.ua project data

##Requests
- preferable schema - `https`
- current version - `v1`
- version prefix - `api.hotels24.ua/{version_prefix}/query/path?query=parameters`
- default prefix for current version - could be ommited - `api.hotels24.ua/query/path?query=parameters`

###Format
###Access
###

##Responses

###JSON Response
#####Examples:

######Example1 (Simple Ok)
```JSON
{
  "ok":1,
  "data": "...mixed value",
  "msg":"ok"
}
```
######Example2 (Simple Error)
```JSON
{
  "ok":0,
  "data":null,
  "msg":"Underflow Exception Message"
  "code":500
}
```
