# Basic anatomy of a Questionnaire

## Using Terminology in a Questionnaire Item - Coding

All aspects of this questionnaire can be found in the sample:
[coding-sampler](https://sqlonfhir-r4.azurewebsites.net/fhir/Questionnaire/coding-sampler)

There are many ways that terminology can be used in a questionnaire. This
questionnaire demonstrates all of them. The questionnaire is designed to
demonstrate the use of terminology, and is not intended to be a useful
questionnaire in any other way.

1. Locally defined codings - no terminology server required _**(common)**_
2. Contained ValueSet with included expansion - no terminology server required
3. Canonical ValueSet Reference (no version) _**(common)**_
4. Canonical ValueSet Reference (version specific)
5. Canonical ValueSet Reference - preferred terminology server
6. Contained ValueSet  with no expansion
7. Options from a coding expanded using a fhir query, and a fhirpath expression.
8. Languages extracted from the patient in the launch context
9. Explicit ValueSet Reference (STU3/DSTU2 old style reference - not conformant
   with R4, but testing backward compatibility error handling)

A terminology is required with `item.type` of `choice` or `open-choice` using
the
[answerValueSet](https://hl7.org/fhir/R4B/questionnaire-definitions.html#Questionnaire.item.answerValueSet),
[answerOption](https://hl7.org/fhir/R4B/questionnaire-definitions.html#Questionnaire.item.answerOption)
or
[answerExpression](http://build.fhir.org/ig/HL7/sdc/StructureDefinition-sdc-questionnaire-answerExpression.html).
When including an answer in the QuestionnaireResponse, always uses the
valueCoding (or valueString for `item.type` = `open-choice` selections) in the
answer.

There are 3 typical user interface controls that are used to render terminology
in a questionnaire: | Type | Description | Preferred conditions | | - | - | - |
| Radio Buttons | a set of radio buttons, may be vertical or horizontal | small
set of values, typically < 10 | | Drop Down/Combobox | a list of options in a
dropdown style list | medium set of values < 100 | | Auto-Complete | a simple
textbox that you can type in which will perform a lookahead into the ValueSet to
find matches on the display value | large - very large sets of values |

**Note:** for the `open-choice` item.type the user interface can a little more
varied:

- some systems choose to always default this type to an autocomplete textbox
  where if the value is in the list, selects the code otherwise the user entered
  data is used (this is a great choice as user doesn't have to switch to another
  field if they don't find the data they are after - particularly when a large
  set is used)
- others include an additional virtual option in the selection (which isn't an
  actual coded selection) and when selected enable/show an additional textbox to
  enter the data into

**Note:** Each of the examples included below document the details of an item in
the questionnaire (except where illustrates the contained resource required for
that type of item)

For most approaches the use of a terminology server is required and utilizes the ValueSet [`$expand`](https://hl7.org/fhir/R4B/valueset-operation-expand.html) operation, and in many cases will use the filter parameter to restrict the results to user data entry

``` javascript
// Sample javascript to retrieve the expansion of a ValueSet
// the filterValue would likely come from the user interface.
let filterValue = 'austr';
const url = 'https://tx.example.org/ValueSet/$expand?url=http://hl7.org/fhir/ValueSet/languages&version=3.0.2&filter=' + filterValue;

fetch(url)
  .then(response => response.json())
  .then(data => console.log(data));
```
This can also use the POST form of the operation, which is required when the ValueSet needs to be sent to the server for evaluation (option 6)

---

### 1. Locally defined codings - no terminology server required

This is one of the more common form of terminology usage. The codings are
defined locally in the questionnaire, and are not expected to be available on
any terminology server. This is a good way to include a small value set that is
not expected to change often, and is not shared (internally or externally).

The codes _could_ come from a real code-system and ideally would as that would
help with interoperability, enabling mapping to other more common code systems.

This option can make working with the data entered in the form more difficult to
work with as other tooling will not be able to easily map the codes to other
code systems.

```json
{
    "linkId": "language_options.1",
    "text": "language (options)",
    "type": "choice",
    "answerOption": [
        {
            "valueCoding": {
                "system": "urn:ietf:bcp:47",
                "code": "en-us",
                "display": "English (USA)"
            }
        },
        {
            "valueCoding": {
                "system": "urn:ietf:bcp:47",
                "code": "en-au",
                "display": "Australian"
            }
        },
        {
            "valueCoding": {
                "system": "urn:ietf:bcp:47",
                "code": "en-bg",
                "display": "Australian (bogan)"
            }
        }
    ]
},
```

Although the display is optional, many renderers will desire (or require) this
property to be included for rendering.

_(I've also included a fictional localization of Australian (bogan) for fun)_

---

### 2. Contained ValueSet with included expansion - no terminology server required

This is a simple way to include a value set in a questionnaire. The value set is
included in the questionnaire, and the expansion is included in the value set.
This is a good way to include a small value set that is not expected to change
often, and is shared among several questions within the same questionnaire.

This format also helps with distribution where the renderer doesn't require
access to a terminology server to be functional.

This style should NOT be used where the set of values is long, or where the set
of values is expected to change often. In these cases, a canonical reference to
a value set is preferred.

```jsonc
// The questionnaire requires the ValueSet to be in the contained element with the id used in the `answerValueSet` property of the item (with the # prefix)
"contained": [
    {
        "resourceType": "ValueSet",
        "id": "pre-expanded",
        "title": "Filtered Languages - au*",
        "status": "active",
        "expansion": {
            "total": 5,
            "contains": [
                {
                    "system": "http://example.org/demo/fhir/terminology/COU",
                    "code": "1100",
                    "display": "Australia"
                },
                {
                    "system": "http://example.org/demo/fhir/terminology/COU",
                    "code": "1703",
                    "display": "Australian Antarctic Territory"
                },
                {
                    "system": "http://example.org/demo/fhir/terminology/COU",
                    "code": "1200",
                    "display": "Australian External Territories"
                },
                {
                    "system": "http://example.org/demo/fhir/terminology/COU",
                    "code": "2301",
                    "display": "Austria"
                },
                {
                    "system": "http://example.org/demo/fhir/terminology/COU",
                    "code": "1299",
                    "display": "Other Australian External Territories"
                }
            ]
        }
    }
],
...
{
    "linkId": "language_options.2",
    "text": "language (expansion- au*)",
    "type": "choice",
    "answerValueSet": "#pre-expanded"
},
```

---

### 3. Canonical ValueSet Reference (no version)

This is a reference to a common shared ValueSet that is expected to be available
on any terminology server, and is expected to be the most common option used
from this list. The reference is to the canonical URL of the value set with no
version specified.

It is a great choice for most situations.

```json
{
    "linkId": "language_vsc.3",
    "text": "language (ValueSet canonical - no version)",
    "type": "choice",
    "answerValueSet": "http://hl7.org/fhir/ValueSet/languages"
},
```

---

### 4. Canonical ValueSet Reference (version specific)

This is similar to the previous example, but with the version is specified. The
terminology server is expected to return the specified version of the value set.
If the version is not available, the terminology server is expected to return an
error.

> **TODO:** Check if this error report is the expected behavior, or if it should
> just use the current one and a warning.

This option is best used where a known specific version of a ValueSet is
required to be used, as is often found in pre-defined jurisdictional forms that
change periodically.

```json
{
    "linkId": "language_vsc.4",
    "text": "language (ValueSet canonical - with version - 3.0.2)",
    "type": "choice",
    "answerValueSet": "http://hl7.org/fhir/ValueSet/languages|3.0.2"
},
```

---

### 5. Canonical ValueSet Reference - preferred terminology server

_(More advanced functionality)_

Some lists come from complex systems where only very specific servers can be
used to provide the values. Examples of there are where they might link back to
some internal system like a product catalog, or some organization specific
server that has all the organizations approved content defined.

```json
{
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/terminology-server",
            "valueUrl": "https://r4.ontoserver.csiro.au/fhir/"
        }
    ],
    "linkId": "language_vscp.5",
    "text": "language (ValueSet canonical - preferred server - ontoserver)",
    "type": "choice",
    "answerValueSet": "http://hl7.org/fhir/ValueSet/languages"
},
```

---

### 6. Contained ValueSet with no expansion included

_(More advanced functionality)_

This style of referencing terminology is commonly used where the ValueSet doesn't have any reason to exist outside the questionnaire, or the author is unable to write the ValueSet to a terminology server. It requires the terminology server to be able to perform the expansion on an anonymous or inline ValueSet which passed to the $expand operation.

```jsonc
// The questionnaire requires the ValueSet to be in the contained element with the id used in the `answerValueSet` property of the item (with the # prefix)
"contained": [
    {
        "resourceType": "ValueSet",
        "id": "filtered-languages",
        "url": "http://demo.forms-lab.com/ValueSet/filtered-languages",
        "title": "Filtered Languages",
        "status": "active",
        "compose": {
            "include": [
                {
                    "ValueSet": [
                        "http://hl7.org/fhir/ValueSet/languages"
                    ]
                }
            ]
        }
    }
],
...
{
    "linkId": "language_vscp.6",
    "text": "language (Contained ValueSet - no expansion)",
    "type": "choice",
    "answerValueSet": "#filtered-languages"
},
```

---

### 7. Options from a coding expanded using a fhir query, and a fhirpath expression

_(More advanced functionality)_

The [answerExpression](https://hl7.org/fhir/uv/sdc/StructureDefinition-sdc-questionnaire-answerExpression.html) extension provides a way to use a FHIRPath expression to select arbitrary data for a choice type question, provided that it returns a list of codings.

A common way to use this is with an `x-fhir-query` style variable to retrieve data, then an expression to select the properties within. 
This query could use other values from the form to filter the results, or retrieve other related data from the clinical record.

This example shows performing a specific $expand operation against a specific
server and then read the values from its expansion.

```json
{
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/variable",
            "valueExpression": {
                "description": "Variable to hold the results of the expansion",
                "name": "vsListOfLanguages",
                "language": "application/x-fhir-query",
                "expression": "https://r4.ontoserver.csiro.au/fhir/ValueSet/$expand?url=http://hl7.org/fhir/ValueSet/languages"
            }
        },
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-answerExpression",
            "valueExpression": {
                "description": "select the coded values from the expansion results in the above expression",
                "language": "text/fhirpath",
                "expression": "%vsListOfLanguages.expansion.contains"
            }
        }
    ],
    "linkId": "lang_vsd.7",
    "text": "language (fhir query)",
    "type": "choice"
},
```
**Note:** This style could just as easily query anything at all provided that the content returned by the answerExpression is a list of codings.

This is often used in a similar way to option 8 leveraging launch context or other variables.

---

### 8. Languages extracted from the patient in the launch context

_(More advanced functionality)_

As with the previous option, uses a FHIRPath expression in the [answerExpression](https://hl7.org/fhir/uv/sdc/StructureDefinition-sdc-questionnaire-answerExpression.html) extension to select the codings,
however in this case it is reading the values from a launch context variable that would be provided to the renderer at runtime.

```json
{
    "resourceType": "Questionnaire",
    ...
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-launchContext",
            "extension": [
                {
                    "url": "name",
                    "valueId": "LaunchPatient"
                },
                {
                    "url": "type",
                    "valueCode": "Patient"
                },
                {
                    "url": "description",
                    "valueString": "The patient that is to be used to pre-populate the form, and provide context for variable based expressions"
                }
            ]
        }
    ],
    "item": [
        {
            "extension": [
                {
                    "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-answerExpression",
                    "valueExpression": {
                        "description": "Select one of the patient's langauges",
                        "language": "text/fhirpath",
                        "expression": "%LaunchPatient.communication.language"
                    }
                }
            ],
            "linkId": "language_from_patient.8",
            "text": "country (from patient)",
            "type": "choice"
        }
    ]
}
```

---

### 9. Explicit ValueSet Reference (not conformant with R4)

This is not a valid R4 reference, but is included to test/show how it was
represented in STU3/DSTU2

Often these were also defined as relative references which was great when
defining everything locally, however when forms started to be distributed, this
made things much more challenging, and often required the server to support
client allocated IDs, and the chance for collisions far greater, which is the
issue that using Canonical resources resolves - hence why things were changed to
this in the R4 release of FHIR.

Do **NOT** use this format.

```json
{
  "linkId": "language_vsd.9",
  "text": "language (VS direct)",
  "type": "choice",
  "answerValueSet": "https://sqlonfhir-r4.azurewebsites.net/fhir/ValueSet/25b0ec18fd3511d28b3e0020182939f7"
}
```
