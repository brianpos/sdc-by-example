## SDC Extract comparison - Simple Observation
Comparing the template extract approach with the other existing approaches using the same questionnaire definition.

The [test questionnaire](extract-none.json) is a simple one with a single boolean question, and the extraction is to create an Observation resource with a single boolean value.

```json
{
    "resourceType": "Questionnaire",
    "id": "extract-demo-empty",
    "url": "http://fhir.forms-lab.com/Questionnaire/extract-demo-empty",
    "status": "draft",
    "title": "Sigmoidoscopy Complication",
    "publisher": "Brian Postlethwaite",
    "item": [
        {
            "linkId": "complication",
            "text": "Have you had a Sigmoidoscopy Complication (concern with invasive procedure, for example)",
            "type": "boolean",
            "required": true
        }
    ]
}
```
> **Note:** Throughout these examples I'm going to attempt to extract the same data with minimal changes to the questionnaire definition, and only the extraction definition will change.
> Since each are going to be compared to the observation approach I'm going to get all the approaches to generate all the data that the Observation approach would do, or note what is missing.

### Observation approach
[extract-obs.json](extract-obs.json)

The differences here are:
```json
    // Inclusion of the extract observation extension (to indicate to use the observation extraction approach)
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-observationExtract",
            "valueBoolean": "true"
        }
    ],

    // and for any items that are to be extracted, a code that is to be used to identify the observation
    "code": [
        {
            "system": "http://example.org/sdh/demo/CodeSystem/cc-screening-codes",
            "code": "sigmoidoscopy-complication"
        }
    ],
```

For this specific test case, the observation approach is by far the best and simplest one.
(including the observation category is also supported using this approach too, however anything beyond that you'll need to adopt one of the other approaches here)

### Definition approach 1 (hidden fields)
[extract-defn1.json](extract-defn1.json)

The differences here are:
```json
    // inclusion of the extract context extension (to indicate the type of resource to extract)
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-itemExtractionContext",
            "valueCode": "Observation"
        }
    ],

    // and definition property(s) to indicate where to put the value(s) in that context
    "definition": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.value[x]",

    // and hidden item(s) for other fields that need to be set in the extracted resource
    {
        "extension": [
            {
                "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-hidden",
                "valueBoolean": true
            }
        ],
        "linkId": "code",
        "definition": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.code.coding",
        "type": "choice",
        "required": true,
        "answerOption": [
            {
                "initialSelected": true,
                "valueCoding": {
                    "system": "http://example.org/sdh/demo/CodeSystem/cc-screening-codes",
                    "code": "sigmoidoscopy-complication"
                }
            }
        ]
    },
    {
        "extension": [
            {
                "url": "http://hl7.org/fhir/StructureDefinition/questionnaire-hidden",
                "valueBoolean": true
            }
        ],
        "linkId": "status",
        "definition": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.status",
        "type": "choice",
        "required": true,
        "answerOption": [
            {
                "initialSelected": true,
                "valueCoding": {
                    "system": "http://hl7.org/fhir/observation-status",
                    "code": "final"
                }
            }
        ]
    },
```

> **Note:** This approach requires the QR to be polluted with actual data specifically for the extract (which is otherwise of no value, and would prevent being able to refine the extract definition after QR data is complete).

> **Note:** this current definition approach doesn't have an ability to set the author, subject, performed, issued, effectiveDateTime, derivedFrom or effectiveDateTime properties.
>
> These values are from the QR metadata, and not from the answers to the questions (that could be calculated into other answers for extraction - unless that data was entered into the resource before it was completed - of particular issue is the `qr.authored` date property).
>
> *(would that be special knowledge to do here, are there other resources that we would need to define that for?)*

### Definition approach 2 (new extractValue extension)
[extract-defn1.json](extract-defn1.json)

The differences here are:
```json
    // inclusion of the extract context extension (to indicate the type of resource to extract)
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-itemExtractionContext",
            "valueCode": "Observation"
        }
    ],

    // and definition property(s) to indicate where to put the value(s) in that context
    "definition": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.value[x]",

    // And instead of hidden fields as in the previous example, we have a new extension to indicate where to get the value from
    // for each of the fields that need to be set in the extracted resource.
    // (I added all the ones that would be populated from the observation approach exactly)
    "extension": [
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.status"
                },
                {
                    "url": "fixed-value",
                    "valueCode": "final"
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        },
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.code.coding"
                },
                {
                    "url": "fixed-value",
                    "valueCoding": {
                        "system": "http://example.org/sdh/demo/CodeSystem/cc-screening-codes",
                        "code": "sigmoidoscopy-complication"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        },
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.subject"
                },
                {
                    "url": "expression",
                    "valueExpression": {
                        "language": "text/fhirpath",
                        "expression": "%resource.subject"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        },
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.effective"
                },
                {
                    "url": "expression",
                    "valueExpression": {
                        "language": "text/fhirpath",
                        "expression": "(%resource.authored | %resource.meta.lastUpdated | now()).first()"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        },
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.issued"
                },
                {
                    "url": "expression",
                    "valueExpression": {
                        "language": "text/fhirpath",
                        "expression": "(%resource.authored | %resource.meta.lastUpdated | now()).first()"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        },
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.performer"
                },
                {
                    "url": "expression",
                    "valueExpression": {
                        "language": "text/fhirpath",
                        "expression": "%resource.author"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        },
        {
            "extension": [
                {
                    "url": "definition",
                    "valueUri": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.derivedFrom.reference"
                },
                {
                    "url": "expression",
                    "valueExpression": {
                        "language": "text/fhirpath",
                        "expression": "'QuestionnaireResponse/' + %resource.id"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-definitionExtractValue"
        }
    ],
```
The more properties that need to be entered into the resource that are not from answers in the question make this more verbose, but does simplify the data captured in the questionnaire (lower complexity in the capture side).

### Template approach
[extract-template.json](extract-template.json)

The core difference between the approaches
Definition: Walks the QR item tree, then extracts the resource based on definition (tree structure must loosely be similar - particularly for repetitions)
Template: Walks the QR item tree to locate template links, then walks over all the elements in the template resource then locates points in the QR item structure to base and extract from.
Observation: Looks for specific codes/extensions then generates an entire resource based on a single item (mostly - components are the exception to this)

The differences here are:
```json
    // The extension to indicate the contained resource to use as a template
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtract",
            "extension": [
                {
                    "url": "template",
                    "valueReference": {
                        "reference": "#obsTemplate"
                    }
                }
            ]
        }
    ],

    // The contained resource that is the template
    "contained": [
        {
            "resourceType": "Observation",
            "id": "obsTemplate",
            "status": "final",
            "code": {
                "coding": [
                    {
                        "system": "http://example.org/sdh/demo/CodeSystem/cc-screening-codes",
                        "code": "sigmoidoscopy-complication"
                    }
                ]
            },
            "subject": {
                "extension": [
                    {
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtractValue",
                        "valueString": "%resource.subject"
                    }
                ]
            },
            "_effectiveDateTime": {
                "extension": [
                    {
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtractValue",
                        "valueString": "%resource.authored"
                    }
                ]
            },
            "_issued": {
                "extension": [
                    {
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtractValue",
                        "valueString": "%resource.authored"
                    }
                ]
            },
            "performer": [
                {
                    "extension": [
                        {
                            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtractValue",
                            "valueString": "%resource.author"
                        }
                    ]
                }
            ],
            "_valueBoolean": {
                "extension": [
                    {
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtractValue",
                        "valueString": "%resource.item.where(linkId='complication').answer.value"
                    }
                ]
            },
            "derivedFrom": [
                {
                    "_reference": {
                    "extension": [
                        {
                            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-templateExtractValue",
                            "valueString": "'QuestionnaireResponse/' + %resource.id"
                        }
                    ]
                    }
                }
            ]
        }
    ],
```
The approach is best when there are very few fields being populated into the template, and complexity is low in terms of iterations (though can't see that from this sample)

### StructureMap approach
[extract-template.json](extract-template.json)

The core difference between the approaches
Definition: Walks the QR item tree, then extracts the resource based on definition (tree structure must loosely be similar - particularly for repetitions)
Template: Walks the QR item tree to locate template links, then walks over all the elements in the template resource then locates points in the QR item structure to base and extract from.
Observation: Looks for specific codes/extensions then generates an entire resource based on a single item (mostly - components are the exception to this)

The differences here are:
```json
    // inclusion of the extension to reference the StructureMap to use
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-targetStructureMap",
            "valueCanonical": "http://example.org/fhir/StructureMap/QRSigmoidoscopyComplicationToObservation"
        }
    ],
```

And of course the actual [StructureMap definition](extract-smap.fml) too (external to the questionnaire)
```fml
map "http://fhir.forms-lab.com/StructureMap/extract-demo-smap" = "extract-demo-smap"

uses "http://hl7.org/fhir/StructureDefinition/QuestionnaireResponse" as source
uses "http://hl7.org/fhir/StructureDefinition/Bundle" as target
uses "http://hl7.org/fhir/StructureDefinition/Observation" as target

group ExtractBundle(source src : QuestionnaireResponse, target tgt : Bundle) {
  // Prepare the bundle entry
  src -> tgt.entry as entry, uuid() as fullUrl then {
    src -> entry.fullUrl = ('urn:uuid:' & %fullUrl) "SetFullUrl";
    src -> entry.request as req then {
      src -> req.method = 'POST' "setMethod";
    } "SetRequest";
    src -> entry.resource = create('Observation') as obs then PopulateObservation(src, obs) "popObs";
  } "CreateObsEntry";
}

group PopulateObservation(source src : QuestionnaireResponse, target tgt : Observation) {
  src -> tgt.status = 'final' "SetStatus";
  src -> tgt.code = cc('http://example.org/sdh/demo/CodeSystem/cc-screening-codes', 'sigmoidoscopy-complication') "SetObservationCode";
  src.subject -> tgt.subject;
  src.authored as s -> tgt.effective = s, tgt.issued = s  "SetAuthored";
  src.author -> tgt.performer;
  src.id as qrId -> tgt.derivedFrom as df then {
    qrId -> df.reference = ('QuestionnaireResponse/' & %qrId) "SetReference";
  } "SetDerivedFrom";

  src.item as item where (linkId = 'complication') -> tgt.value = create('CodeableConcept') as obsValue then {
        item -> tgt.value = (%item.answer.value.first()) "SetObservationValue";
      };
}
```
This IS the most complex approach from an editor/author perspective, but also the most powerful.

