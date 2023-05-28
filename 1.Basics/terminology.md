# Basic anatomy of a Questionnaire

## Using Terminology in a Questionnaire Item

All aspects of this questionnaire can be found in the sample:
[coding-sampler](https://sqlonfhir-r4.azurewebsites.net/fhir/Questionnaire/coding-sampler)

There are many ways that terminology can be used in a questionnaire. This
questionnaire demonstrates all of them. The questionnaire is designed to
demonstrate the use of terminology, and is not intended to be a useful
questionnaire in any other way.

1. Locally defined codings - no terminology server _**(common)**_
2. Contained ValueSet with included expansion - no terminology server
3. Canonical ValueSet Reference (no version) - not terminology server specific
   _**(common)**_
4. Canonical ValueSet Reference (version specific) - not terminology server
   specific
5. Canonical ValueSet Reference (no version) - preferred terminology server
6. Contained ValueSet (no expansion) - preferred terminology server
   (https://r4.ontoserver.csiro.au/fhir/)
7. Options from a coding expanded using a fhir query, and a fhirpath expression.
   Useful when the expression is effected by other values on the form
8. Languages extracted from the patient in the launch context
9. Explicit ValueSet Reference (STU3/DSTU2 old style reference - not conformant
   with R4, but testing backward compatibility error handling)

> TODO: Cleanup: There are 3 typical user interface controls that are used to
> render terminology in a questionnaire:
>
> - choice - a drop down list of options
> - open-choice - a drop down list of options, with the ability to enter a
  > custom value
> - string - a text box

**Note:** Each of the examples included below document the details of an item in
the questionnaire (except where illustrates the contained resource required for
that type of item)

---

### 1. Locally defined codings - no terminology server

This is one of the more common form of terminology usage. The codings are
defined locally in the questionnaire, and are not expected to be available on
any terminology server. This is a good way to include a small value set that is
not expected to change often, and is not shared.

The codes _could_ come from a real code-system and ideally would as that would
help with interoperability, enabling mapping to other more common code systems.

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

### 2. Contained ValueSet with included expansion - no terminology server

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
        "status": "draft",
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

### 3. Canonical ValueSet Reference (no version) - not terminology server specific

This is a reference to a common shared ValueSet that is expected to be available
on any terminology server. The reference is to the canonical URL of the value
set, and the version is not specified. The terminology server is expected to
return the latest version of the value set.

```json
{
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-itemControl",
            "valueCodeableConcept": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/questionnaire-item-control",
                        "code": "drop-down"
                    }
                ]
            }
        }
    ],
    "linkId": "language_vsc.3",
    "text": "language (ValueSet canonical - no version)",
    "type": "choice",
    "answerValueSet": "http://hl7.org/fhir/ValueSet/languages"
},
```

---

### 4. Canonical ValueSet Reference (version specific) - not terminology server specific

This is similar to the previous example, but the version is specified. The
terminology server is expected to return the specified version of the value set.
If the version is not available, the terminology server is expected to return an
error.

> **TODO:** Check if this error report is the expected behavior, or if it should
> just use the current one and a warning.

```json
{
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-itemControl",
            "valueCodeableConcept": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/questionnaire-item-control",
                        "code": "drop-down"
                    }
                ]
            }
        }
    ],
    "linkId": "language_vsc.4",
    "text": "language (ValueSet canonical - with version - 3.0.2)",
    "type": "choice",
    "answerValueSet": "http://hl7.org/fhir/ValueSet/languages|3.0.2"
},
```

---

### 5. Canonical ValueSet Reference (no version) - preferred terminology server

Some lists come from complex systems where only very specific servers can be
used to provide the values. Examples of there are where they might link back to
some internal system like a product catalog, or some organization specific
server that has all the organizations approved content defined.

```json
{
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-itemControl",
            "valueCodeableConcept": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/questionnaire-item-control",
                        "code": "drop-down"
                    }
                ]
            }
        },
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

### 6. Contained ValueSet (no expansion) - preferred terminology server (https://r4.ontoserver.csiro.au/fhir/)

```json
// The questionnaire requires the ValueSet to be in the contained element with the id used in the `answerValueSet` property of the item (with the # prefix)
"contained": [
    {
        "resourceType": "ValueSet",
        "id": "filtered-languages",
        "url": "http://demo.forms-lab.com/ValueSet/filtered-languages",
        "version": "4.0.1",
        "name": "FilteredLanguages",
        "title": "Filtered Languages",
        "status": "draft",
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
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-itemControl",
            "valueCodeableConcept": {
                "coding": [
                    {
                        "system": "http://hl7.org/fhir/questionnaire-item-control",
                        "code": "drop-down"
                    }
                ]
            }
        },
        {
            "url": "http://hl7.org/fhir/StructureDefinition/terminology-server",
            "valueUrl": "https://r4.ontoserver.csiro.au/fhir/"
        }
    ],
    "linkId": "language_vscp.6",
    "text": "language (ValueSet canonical - preferred server - ontoserver)",
    "type": "choice",
    "answerValueSet": "#filtered-languages"
},
```

---

### 7. Options from a coding expanded using a fhir query, and a fhirpath expression. Useful when the expression is effected by other values on the form

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
                "name": "ListOfLanguages",
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

---

### 8. Languages extracted from the patient in the launch context

```json
{
    "resourceType": "Questionnaire",
    ...
    "extension": [
        {
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
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-launchContext"
        }
    ],
    "item": [
        {
            "extension": [
                {
                    "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-answerExpression",
                    "valueExpression": {
                        "description": "Select one of the patient's langauges",
                        "name": "PatientLanguages",
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

Do NOT use this format.

```json
{
  "extension": [
    {
      "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-itemControl",
      "valueCodeableConcept": {
        "coding": [
          {
            "system": "http://hl7.org/fhir/questionnaire-item-control",
            "code": "drop-down"
          }
        ]
      }
    }
  ],
  "linkId": "language_vsd.9",
  "text": "language (VS direct)",
  "type": "choice",
  "answerValueSet": "https://sqlonfhir-r4.azurewebsites.net/fhir/ValueSet/25b0ec18fd3511d28b3e0020182939f7"
}
```
