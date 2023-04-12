# User Files API

The user files API is designed so kits can store small files for use as e.g. icons.

User files are referred to by GUID. GUIDs are not necessarily unique across a system; this allows users to have e.g. a dashboard referring to a particular file by GUID, but with each user installing their own preferred file. If multiple files exist with the same GUID, they are prioritized in the following order:

* Owned by the user
* Shared with a group the user is a member of
* Global

 Each file *does* have a unique UUID used by DatalaiQ for storage; the admin API, documented below, allows administrators to manage user files by referring to their "ThingUUID".

## Create a user file

User files can be created by POST request to `/api/files`. The request should be a multipart request in one of two formats.

### Simple file creation request format

Submit a POST request to `/api/files` with the following fields:

* `file`: the body of the file
* `name`: the name of the file
* `desc`: the description of the file
* `guid`: (optional) the desired GUID for this file. If not set, one will be generated.

### Detailed file creation request format

Submit a POST request to `/api/files` with the following fields:

* `file`: the body of the file
* `meta`: a JSON-encoded structure describing desired ownership / sharing details as shown below.

The "meta" field should be a JSON-encoded string describing sharing and ownership:

```
{
	"UID": 1,
	"GIDs": [2],
	"Global": false,
	"Name": "blah",
	"Desc": "bar",
	"Labels": ["foo"],
}
```

```{note}
Only administrators can set the UID field to another user's UID, or set the Global field to true.
```

## Listing files

User files may be listed by a GET on `/api/files`. The result is an array of structures containing file information:

```
[
	{
		"GUID": "1945c39e-0bd1-40cf-b069-6da64d3f8afe",
		"ThingUUID": "3525901f-5723-11e9-be65-54e1ad7c66cf",
		"UID": 1,
		"GIDs": null,
		"Global": false,
		"Size": 99,
		"Type": "text/plain; charset=utf-8",
		"Name": "blah",
		"Desc": "bar",
		"Labels": ["foo"],
		"Updated": "2019-04-04T15:50:19.290186504-06:00"
	}
]
```

## Reading a file's contents

The contents of a file may be read by a GET request on `/api/files/<uuid>`, e.g. to read the file in the listing above `/api/files/1945c39e-0bd1-40cf-b069-6da64d3f8afe`.

## Updating a file

A file's contents can be changed via a POST request to `/api/files/<uuid>`. The request should be identical to one used to create a new file, but only the `file` field will be honored. To change other fields, see below.

## Updating file metadata

The various fields of a file (Name, Desc, Global, GIDs, Labels) can be updated with a PATCH request to `/api/files/<uuid>`. The body of the request should contain the same structure as was returned in a list (GET `/api/files`), e.g.:

```
{
	"UID": 1,
	"GIDs": null,
	"Global": false,
	"Name": "blah",
	"Desc": "bar",
	"Labels": ["foo"],
}
```

Note that any fields beyond those show above may be present but will be ignored.

```{attention}
The UID of a file may be changed, but only by an administrator and only when the `?admin=true` parameter has been set.
```

## Deleting a file

User files may be removed via a DELETE on `/api/files/<uuid>`

## Admin actions

Admin users may occasionally need to view all user files on the system, modify them, or delete them. Because GUIDs are not necessarily unique, the admin API must refer instead to the unique UUID DatalaiQ uses internally to store the items. Note that the example file listings above include a field named "ThingUUID". This is the internal, unique identifier for that user file.

An administrator user may obtain a global listing of all user files in the system with a GET request on `/api/files?admin=true`.

The administrator may then delete a particular file with a DELETE message to `/api/files/<ThingUUID>?admin=true`, substituting in the ThingUUID value for the desired file. The same pattern applies to updating.
