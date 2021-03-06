# platsbanken-vacancy
Submit vacancies to the Arbetsförmedlingen Platsbanken (Swedish employment agency
job centre).

## Description
This module provides an interface for generating an appropriate XML document for
submission to the Arbetsförmedlingen Platsbanken API.

It does not perform the submission or handle responses. See `./example/example.js`.

## Install

`npm install --save @othermachines/platsbanken-vacancy`

## Scripts

npm test: `mocha`

npm run build: `babel src --out-dir build --source-maps`

npm run watch: `babel src --out-dir build --source-maps --watch`

npm prepare: `npm run build`

## Dependencies
This module was developed against node v8.1.2.

- joi
- source-map-support
- xml

## Usage

```
const vacancy = require('@othermachines/platsbanken-vacancy');

const request = vacancy();

try {
  request
    .sender({
      id: '12345678',
      email: 'foo@bar.com',
    })
    .transaction({
      id: 'TRANSACTION GUID',
    })
    .jobPositionPosting({
      id: '46-123456-1234-123-123',
      status: 'active',
    })
    .hiringOrg({
      name: 'Company Name',
      id: '46-123456-1234'
      url: 'http://example.org',
      description: 'Organizational unit description',
    })
    .hiringOrgContact({
      countryCode: 'SE',
      postalCode: '11356',
      municipality: '0180',
      addressLine: 'Birger Jarlsgatan 58, 11356, Stockholm',
      streetName: 'Birger Jarlsgatan 58',
    })
    .postDetail({
      startDate: '2018-09-01',
      endDate: '2018-12-01',
      recruiterName: 'Alex Smith',
      recruiterEmail: 'alexsmith@example.org',
    })
    .jobPositionTitle({
      title: 'Job Title',
    })
    .jobPositionPurpose({
      purpose: 'Job purpose',
    })
    .jobPositionLocation({
      countryCode: 'SE',
      postalCode: '11356',
      municipality: '0180',
      addressLine: 'Birger Jarlsgatan 58, 11356, Stockholm',
      streetName: 'Birger Jarlsgatan 58',
    })
    .classification({
      scheduleType: 'part',
      duration: 'temporary',
      scheduleSummaryText: 'Schedule Summary',
      durationSummaryText: 'Duration Summary',
      termLength: 2,
    })
    .compensationDescription({
      currency: 'SEK',
      salaryType: 1,
      benefits: 'Benefits',
      summary: 'summary text',
    })
    .qualificationsRequiredSummary({
      summary: 'Summary of qualifications',
    })
    .qualification({
      type: 'license',
      description: 'DriversLicense',
      category: 'B',
    })
    .qualification({
      type: 'experience',
      required: true,
    })
    .qualification({
      type: 'equipment',
      description: 'Car',
    })
    .qualificationsPreferredSummary({
      summary: 'Preferred qualifications',
    })
    // applicationMethods() not neccessary, will be called by byWeb()
    // or byEmail(), included for clarity
    .applicationMethods()
    .byWeb({
      url: 'http://example.org',
    })
    .byEmail({
      email: 'foo@example.org',
    })
    .numberToFill({
      number: 1,
    })
    .hiringOrgDescription({
      description: 'Hiring org description',
    })
    .occupationGroup({
      code: 7652,
    });
} catch (err) {
  console.log(err);
  if (err.isJoi) {
    console.log('-----');
    console.log(err.annotate());
  }
  process.exit(1);
}
const xmlString = request.toString();
```

## Sample scripts
There are three scripts in the examples directory:
- minimal.js: demonstrate creation of XML for adding a new vacancy. Minimal fluff, start here.
- example.js: full fat demo, including adding, updating, and deleting vacancies.
See comments at top of script for more detail.
- apiTest.js: executes tests required by Arbetsförmedling to use API.*

\* Note that currently the API is returning an error for test #7 (add vacancy
  outside of Sweden), apparently on the address.

## Running the example script (./example/example.js)
The script uses [config](https://www.npmjs.com/package/config) to set organisation-specfic
information (company name, customer number, organisation number). If you just
want to see what the final XML looks like, you don't need to fill those in (but you
do need to create the configuration file):

```
cd example/config/

cp default.sample.json default.json

node example.js --create --xml
```

You can output the JSON object that is used to create the XML:

`node example.js --json`

You can use example.js to submit a test request. The API will return an error
unless valid values have been set in your config file:

`node example.js --create --submit`

You can set NODE_ENV to "production" if you want to submit to the live API.

## Oddities
Errors may be returned with HTTP status 202 or 400.

202 is returned with errors that appear to have been generated at the application level, e.g., from invalid values such as non-existent postal codes. Just because you got a 2xx response back doesn't mean the post was successful, check your response error code! (See example.js).

400 seems to usually be errors when validating against XSDs, although it will also be returned if you have an invalid customer number.

The JPPExtension element requires that children occur in a specific order. If you receive an error similar to:
```
  The element 'JPPExtension' in namespace
  'http://arbetsformedlingen.se/LedigtArbete' has
  invalid child element 'ApplicationReferenceID' in
  namespace
  'http://arbetsformedlingen.se/LedigtArbete'. List
  of possible elements expected:
  'LastDateApplication, ApplicationReferenceId,
  RequiredQualification, PreferredQualification' in
  namespace
  'http://arbetsformedlingen.se/LedigtArbete'.
```
check the order in which you are adding elements. Method calls must be in this order, though only occupationGroup() is required:
```
vacancy
  .hiringOrgDescription({
    description: 'Hiring org description',
  })
  .contact({
    name: 'Contact Name',
    phone: '555.555.5555',
    email: 'contact@example.org',
  })
  .occupationGroup({
    code: 7652,
  })
  .applicationReferenceID({
    id: 'ABC123',
  });
```

(Note that this appears to apply to other tag sets as well - the XSDs make liberal use of sequence - but this was the first time it bit me.)

There are two places where a description of the organization can be added. These are:
```
<Envelope><Packet><Payload><JobPositionPosting><HiringOrg><OrganizationalUnit><Description>
```
added via the `hiringOrg({name, id, url, description})` method, and
```
<Envelope><Packet><Payload><JPPExtension><HiringOrgDescription>
```
added via the `hiringOrgDescription({ description })` method.

Neither is required, and the documentation is unclear on the difference. Best guess until I try this out is that they appear in a different order in the job ad.

## Thank you
Development of this module was made possible by
[Internationella Engelska Skolan](https://engelska.se).

[@buren's ruby gem](https://github.com/buren/arbetsformedlingen) for creating
submissions was an invaluable resource. Also see that github page for
additional resources, including links to a Postman collection for querying
the Arbetsförmedlingen taxonomy service and data from that service in CSV
format.

## Contributing
Pull requests are welcome!

Methods implement a tag, or set of tags if they are logically a group. Each tag
or set is implemented with three methods:
- `jsonTagName()`: creates json for the tag/set,
- `validateTagName()`: performs parameter validation, throws Joi error on failure,
- `tagName()`: calls `validateTagName()` and `jsonTagName()`, performs any neccessary
business logic and attaches the new JSON object. Methods are chainable, so must
return `this`.

## Author
Dwayne Holmberg <dholmberg@othermachines.com>
[https://github.com/dwayneholmberg](https://github.com/dwayneholmberg)

## License
This module is freely distributable under the terms of the
[MIT license](https://opensource.org/licenses/MIT).
