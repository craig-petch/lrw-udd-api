# Jisc UDD LRW API

## Version 0.1.0

### Compatabile with UDD Version 1.3.0

#### Schema https://github.com/jiscdev/analytics-udd/tree/v1.3.0

### About

The new version of the UDD LRW API brings in a relational structure to the UDD. All entities now have a primary key which should be used to GET, POST, PUT, PATCH and DELETE records with.

### Available entity endpoints
```
/models/assessmentInstance
/models/course
/models/courseInstance
/models/institution
/models/module
/models/moduleInstance
/models/moduleVleMap
/models/period
/models/staff
/models/staffCourseInstance
/models/staffModuleInstance
/models/student
/models/studentAssessmentInstance
/models/studentCourseInstance
/models/studentCourseMembership
/models/studentModuleInstance
```


### Get all courses

`GET /models/course`

### Get single course
Where `COURSE_ID` is `"COURSE_1"`

`GET /models/course/COURSE_1`

### Insert new course

`POST /models/course`

```
{
  "COURSE_ID": "COURSE_2",
  "TITLE": "Test"
}
```

### Update a course

`PUT /models/course/COURSE_1`
or
`PATCH /models/course/COURSE_1`

```
{
  "TITLE": "Testing"
}
```

If using POST to update a model, the primary key _must_ be present in the request JSON body.


### Soft Deletions

All models in the UDD are soft deleted by default.

Upon a DELETE request, the model is cleared of none identifying data (fields are set to NULL unless primary key, e.g. `STUDENT_ID` in Student) and is no longer returned or amendable by standard requests.

To retrieve or update soft deleted models, use the `trashed` query parameter in your request.

`GET /student?trashed=1`

The deletion date is stored in the DELETED_AT field. This can be queried like any other date field:

`GET /student?trashed=1&query={"DELETED_AT": {"$gte": "2016-01-01 00:00"}}`

### Permanent Deletions

In order to permanently delete a model (remove it from the database), use the `force` query parameter:

`DELETE /student/:id?force=1`

If your model has previously been soft deleted, you would need to combine this with the `trashed` query parameter, otherwise it will not be found:

`DELETE /student/:id?force=1&trashed=1`

To restore a model, send a PATCH request to update the model and set the `deleted` field to null. It is *not* compulsory to remove the DELETED_AT field in case you wish to keep this field for analytical purposes.

```
PATCH /student/:id?trashed=1
{
  ... all required fields for validation
  "deleted": false,
  "DELETED_AT": null // optional
}
```

_Note that using DELETE in api versions prior to the introduction of soft deletes will cause the model to be permanently removed from the database._

### Select
`GET /student?select=PARENTS_ED`

`GET /student?select=-PARENTS_ED`

`GET /student?select={"PARENTS_ED":1}`

`GET /student?select={"PARENTS_ED":0}`

### Sort

`GET /student?sort=name`

`GET /student?sort=-name`

`GET /student?sort={"name":1}`

`GET /student?sort={"name":0}`


### Skip

`GET /student?skip=10`


### Limit
Only overrides options.limit if the queried limit is lower.

`GET /student?limit=10`


### Query
Supports all operators ($regex, $gt, $gte, $lt, $lte, $ne, etc.) as well as shorthands: ~, >, >=, <, <=, !=

`GET /student?query={"STUDENT_ID":"STUDENT_1"}`

`GET /student?query={"STUDENT_ID":{"$regex":"^(STUDENT)"}}`

`GET /student?query={"STUDENT_ID":"~^(STUDENT_1)"}`

`GET /student?query={"PARENTS_ED":{"$gt":2}}`

`GET /student?query={"PARENTS_ED":">2"}`

`GET /student?query={"PARENTS_ED":{"$gte":2}}`

`GET /student?query={"PARENTS_ED":">=2"}`

`GET /student?query={"PARENTS_ED":{"$lt":2}}`

`GET /student?query={"PARENTS_ED":"<2"}`

`GET /student?query={"PARENTS_ED":{"$lte":2}}`

`GET /student?query={"PARENTS_ED":"<=2"}`

`GET /student?query={"PARENTS_ED":{"$ne":2}}`

`GET /student?query={"PARENTS_ED":"!=2"}`


### Populate
Works with create, read and update operations.
Relations are available as a lower camel cased version of the property
e.g. Student relation to StudentModuleInstance is available as student.studentModuleInstances


`GET/POST/PUT /student?populate=tutor`

`GET/POST/PUT /student?populate={"path":"tutor"}`

`GET/POST/PUT /student?populate=[{"path":"tutor"},{"path":"studentModuleInstances"}]`

`GET/POST/PUT /student?populate=[{"path":"customer"},{"path":"products"}]`


Can be nested to fetch relations of relations

`GET/POST/PUT /student?populate={"path":"tutor","populate":{"path":"staffModuleInstances","model":"StaffModuleInstance"}}`
