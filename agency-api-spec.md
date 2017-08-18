# FOIA.gov Draft RESTful HTTPS API Spec

This is [a draft spec](https://github.com/18F/foia/issues/32) for integrating
the FOIA.gov portal with existing FOIA case management systems (_e.g._,
[FOIAonline](https://foiaonline.regulations.gov/foia/action/public/home)) in the
federal government. This work stems from the interviews and research that led to
our [FOIA Portal Discovery
Recommendations](https://docs.google.com/document/d/1sRWq2vDAdoz97zdxgiOzU-9N7nBNEUiEeHzsa7J3j_A/edit).

Once a case management system supports this specification, it can receive FOIA
requests directly from the FOIA.gov portal, rather than having the request data sent via e-mail.

To minimize agency effort, we've designed this spec so that some of
the tedious bits of implementing an API can be handled by a service like
[api.data.gov](https://api.data.gov/about/), which provides a free API
management service for federal agencies.


## Make FOIA Request

### Notes

This draft does not address:
* versioning
* size or rate limits
* error message/status code related to exceeded the rate limit
* any subsequent calls to the internal FOIA.gov API (to capture info needed to subsequently retrieve status, for example)


### URL

You may choose any pathname you wish. If your system handles requests for multiple agency components (like a decentralized agency), we recommend using
a URL structure that explicitly identifies the agency component receiving the FOIA request. Your URL should not
contain any query parameters.

    /components/:id/requests/

For example:
* `/components/88/requests/`

But not:
* `/requests?component=88`

In addition, we recommend hosting the API on a dedicated sub-domain like `foia-api.agency.gov`. Using this kind of pathname hierarchy allows us to add additional API
endpoints for future development and features.


### Method:

  `POST`


###  URL Params

**Required:**

`id=[integer]`, where `id` is the unique identifier of the agency component that should receive the request.


### Data Params

JSON payload that contains the form fields.

    Content-Type: application/json


#### Request fields

**Field:** | `agency_name`
:--- |:---
**Type:** | text
**Description:** | Name of the tier 1 agency.
**Required:** | yes
**Example:** | `"Department of Justice"`

**Field:** | `agency_component_name`
:--- |:---
**Type:** | text
**Description:** | Name of the department, bureau, or office.
**Required:** | yes
**Example:** | `"Office of Information Policy"`

**Field:** | `requester_name`
:--- |:---
**Type:** | object
**Description:** | Full name of the requester.
**Required:** | yes
**Example:** | `{"first": "George", "last": "Washington"}`

**Field:** | `requester_address`
:--- |:---
**Type:** | object
**Description:** | Mailing address of the requester.
**Required:** | yes
**Example:** | `{"address1": "1800 F Street", "address2": "Suite 400", "city": "Mount Vernon", "state": "Virginia", "zip": "98273"}`

**Field:** | `description`
:--- |:---
**Type:** | text
**Description:** | Description of the records the requester is seeking.
**Required:** | yes
**Example:** | `"I am seeking records pertaining to ..."`

**Field:** | `max_fee`
:--- |:---
**Type:** | money
**Description:** | The amount in USD that a requester is willing to pay in order to cover costs related to this request.
**Required:** | no
**Example:** | `25.00`

**Field:** | `fee_waiver`
:--- |:---
**Type:** | boolean
**Description:** | The requester would like to request that fees associated with the request be waived.
**Required:** | no, defaults to `false`
**Example:** | `false`

**Field:** | `expedited`
:--- |:---
**Type:** | boolean
**Description:** | The requester would like this request to be processed on an expedited basis.
**Required:** | no, defaults to `false`
**Example:** | `false`

**Field:** | `organization`
:--- |:---
**Type:** | text
**Description:** | Name of the organization or company on which the requester is making a request on behalf of.
**Required:** | no
**Example:** | `"Newspaper Inc"`

**Field:** | `email`
:--- |:---
**Type:** | text
**Description:** | Email address of the requester.
**Required:** | yes
**Example:** | `"george.washington@example.com"`

**Field:** | `phone`
:--- |:---
**Type:** | text
**Description:** | Phone number of the requester.
**Required:** | no
**Example:** | `"+15551234567"`

**Field:** | `fax`
:--- |:---
**Type:** | text
**Description:** | Fax number of the requester.
**Required:** | no
**Example:** | `"+15551234589"`

**Field:** | `attachments`
:--- |:---
**Type:** | [object]
**Description:** | Documents or attachments supporting the request provided by the requester.
**Required:** | no
**Example:** | `[{"filename": "letter.pdf", "content_type": "application/pdf", "filesize": 27556, "filedata": "YSBiYXNlNjQgZW5jb2RlZCBmaWxlCg=="}]`

**Field:** | `agency_component_specific_fields`
:--- |:---
**Type:** | object
**Description:** | Agency component specific request form fields as specified in your [agency's metadata file][agency-metadata-file-schema].
**Required:** | if applicable
**Example:** | `{"form_460A_case_number": "3347", "requester_type": "Journalist"}`


#### Agency component specific form fields

Your agency component might have additional fields specified in your [agency
metadata file][agency-metadata-file-schema].  These additional fields are unique
to your agency and are captured separately in the
`agency_component_specific_fields` object.

The fields in `agency_component_specific_fields` will be defined by your agency metadata
file which includes both required and optional form fields. Any fields in
`required_form_fields` will be considered required. Any fields in
`additional_form_fields` will be considered optional. The FOIA.gov portal will
ensure that required fields are present before POSTing a request to your
endpoint.

**Field:** | `*`
:--- |:---
**Type:** | determined by the [agency metadata file][agency-metadata-file-schema]
**Description:** | Agency component specific request form field as specified in your [agency's metadata file][agency-metadata-file-schema].
**Required:** | if applicable
**Example:** | See [below](#agency-form-fields-example)


<a id="agency-form-fields-example"></a>
##### Example

Consider this sample [agency metadata
file](https://github.com/18F/foia/blob/master/GSA.json). A truncated version is
provided below.

```
{
    "abbreviation": "GSA",
    "departments": [
        {
            // ...
            "additional_form_fields": [
                {
                    "help_text": "If your request relates to a GSA contract, please provide the contract number (which starts with \"GS-\")",
                    "label": "GS- Contract number",
                    "name": "contract_number"

                },
                {
                    "help_text": "(i.e. New England Region (1A) - States Served: CT, MA, ME, NH, RI, VT",
                    "label": "GSA Region",
                    "name": "region"
                }
            ],
            "required_form_fields": [
                {
                    "enum": [
                        "Company",
                        "Individual/Self",
                        "Organization"
                    ],
                    "help_text": "Company",
                    "label": "Request Origin",
                    "name": "request_origin",
                    "regs_url": null
                }
            ]
        }
    ]
}
```

`agency_component_specific_fields` therefore has the following fields.

- (required) `request_origin`
- `contract_number`
- `region`

So in the request payload, `agency_component_specific_fields` would look like:

```
{
    // ...
    "agency_component_specific_fields": {
        "contract_number": "5547",
        "region": "9",
        "request_origin": "Individual/Self"
    }
}
```


### Success Response

**Code:** | 200 OK
:--- |:---
**Content:** | `{ "id" : 33, "status_tracking_number": "doj-1234" }`
**Meaning:** | Confirm that the request was created and return an `id` that can uniquely identify the request in the case management system. The (optional) status tracking number can be used by a requester to track a request.


### Error Response

**Code:** | 404 NOT FOUND
:--- |:---
**Content:** | `{ "code" : "A234", "message" : "agency component not found", "description": "description of the error that is specific to the case management system"}`
**Meaning:** | The target agency component specified in URI was not found (error payload includes a place for a system-specific message, to make it easier to track down problems)

**Code:** | 500 INTERNAL SERVER ERROR
:--- |:---
**Content:** | `{ "code" : "500", "message" : "internal error", "description": "description of the error that is specific to the case management system"}`
**Meaning:** | The case management system encountered an internal error when trying to create the FOIA request (error payload includes a place for a system-specific message, to make it easier to track down problems)


### Authentication

To ensure that your API and case management system aren't publicly exposed, we recommend restricting your API access to the FOIA.gov portal. This is done via a secret HTTP header token. You will provide this secret token to the portal though configuration. Every request from the portal will include this token, and your API should validate that it is the correct token.

Services like [api.data.gov](https://api.data.gov/about/) provide this authentication for you.


### Sample Call

    $ curl -X POST -H "Content-Type: application/json" -d @- https://foia-api.agency.gov/components/234/requests <<EOF
    {
	"agency": "Department of Justice",
	"agency_component_name": "Office of Information Policy",
	"attachments": [
	    {
		"content_type": "application/pdf",
		"filedata": "YSBiYXNlNjQgZW5jb2RlZCBmaWxlCg==",
		"filename": "letter.pdf",
		"filesize": 27556
	    }
	],
	"agency_component_specific_fields": {
	    "contract_number": "5547",
	    "region": "9",
	    "request_origin": "Individual/Self"
	},
	"description": "I am seeking records pertaining to ...",
	"email": "george.washington@example.com",
	"expedited": false,
	"fax": "+15551234589",
	"fee_waiver": false,
	"max_fee": 25.0,
	"organization": "Newspaper Inc",
	"phone": "+15551234567",
	"requester_address": {
	    "address1": "1800 F Street",
	    "address2": "Suite 400",
	    "city": "Mount Vernon",
	    "state": "Virginia",
	    "zip": "98273"
	},
	"requester_name": {
	    "first": "George",
	    "last": "Washington"
	}
    }
    EOF


[agency-metadata-file-schema]: https://github.com/18F/foia-recommendations/blob/master/schemas.md#agency-metadata-file