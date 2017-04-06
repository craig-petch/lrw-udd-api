# Jisc UDD LRW API

## Version 0.1.0

### Compatabile with UDD Version 1.3.0

#### Schema https://github.com/jiscdev/analytics-udd/tree/v1.3.0

### About

The new version of the UDD LRW API brings in a relational structure to the UDD. All entities now have a primary key which should be used to GET, POST, PUT, PATCH and DELETE records with.

### Available entity endpoints
The model names below are *not* case sensitive and can contain underscores. The convention is to use all lower case letters without underscores as discussed in #47.

```
/models/assessmentinstance
/models/course
/models/courseinstance
/models/institution
/models/module
/models/moduleinstance
/models/modulevlemap
/models/period
/models/staff
/models/staffcourseinstance
/models/staffmoduleinstance
/models/student
/models/studentassessmentinstance
/models/studentcourseinstance
/models/studentcoursemembership
/models/studentidmap
/models/studentmoduleinstance
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

Primary keys must be passed to a POST unless the model contains composite keys (e.g. `moduleVleMap`), in which case the LRW will generate a [UUID (v4)](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_.28random.29) automatically.


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

### Extensions

To add an extension to a model:

`PUT/PATCH /models/course/COURSE_2`
```
{
  "extensions": {
  	"ht2": {
  		"foo": "bar"
  	}
  }
}
```

Returns:

```
{
  "COURSE_ID": "COURSE_2",
  "SUBJECT": null,
  "TITLE": "Test 1",
  "COURSE_AIM": null,
  "INST_TIER_1": null,
  "INST_TIER_2": null,
  "INST_TIER_3": null,
  "TENANT_ID": null,
  "AWARDING_BODY": null,
  "DELETED_AT": null,
  "CREATED_AT": null,
  "UPDATED_AT": "2017-03-23T17:00:17.000Z",
  "deleted": false,
  "extensions": {
    "ht2": {
      "foo": "bar"
    }
  }
}
```

Additional PATCHs will add to the existing extension set:

`PUT/PATCH /models/course/COURSE_2`
```
{
  "extensions": {
  	"unicon": {
  		"hello": "world"
  	}
  }
}
```

Returns:

```
{
  "COURSE_ID": "COURSE_2",
  "SUBJECT": null,
  "TITLE": "Test 1",
  "COURSE_AIM": null,
  "INST_TIER_1": null,
  "INST_TIER_2": null,
  "INST_TIER_3": null,
  "TENANT_ID": null,
  "AWARDING_BODY": null,
  "DELETED_AT": null,
  "CREATED_AT": null,
  "UPDATED_AT": "2017-03-23T17:01:06.000Z",
  "deleted": false,
  "extensions": {
    "ht2": {
      "foo": "bar"
    },
    "unicon": {
      "hello": "world"
    }
  }
}
```

Extension PATCHs may be combined with other PATCH data, and multiple `CONTROLLER`'s extensions can be updaetd in a single PATCH.

Currently there is no method to delete an extension, although one may be set to `null` through a PATCH. This is pending the outcome of [issue #46](https://github.com/jiscdev/lrw-udd-api/issues/46)

### Delete all models

Currently this feature is disabled to prevent accidental mass deletion. We are planning to introduce a new scope that will allow individual clients to FLUSH an entity for an organisation.

### Soft Deletions

All models in the UDD are soft deleted by default.

Upon a DELETE request, the model is cleared of none identifying data (fields are set to NULL unless primary key, e.g. `STUDENT_ID` in Student) and is no longer returned or amendable by standard requests.

To retrieve or update soft deleted models, use the `trashed` query parameter in your request.

`GET /student?trashed=1`

The deletion date is stored in the DELETED_AT field. This can be queried like any other date field:

`GET /student?trashed=1&query={"DELETED_AT": {"$gte": "2016-01-01 00:00"}}`

To restore a model, send a PATCH request to update the model and set the `deleted` field to null. It is *not* compulsory to remove the DELETED_AT field in case you wish to keep this field for analytical purposes.

```
PATCH /student/:id?trashed=1
{
  ... all required fields for validation
  "deleted": false,
  "DELETED_AT": null // optional
}
```

### Permanent Deletions

In order to permanently delete a model (remove it from the database), use the `force` query parameter:

`DELETE /student/:id?force=1`

If your model has previously been soft deleted, you would need to combine this with the `trashed` query parameter, otherwise it will not be found:

`DELETE /student/:id?force=1&trashed=1`


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
