## Template Extract comparison - Simple Observation
Comparing the template extract approach with the other existing approaches using the same questionnaire definition.

The [test questionnaire](extract-none.json) is a simple one with a single boolean question, and the extraction is to create an Observation resource with a single boolean value.

### Observation approach
[extract-obs.json](extract-obs.json)

The differences here are:
```json=
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
> **Note:** This approach will also populate the effectiveDateTime, derivedFrom and status properties.

For this specific test case, the observation approach is by far the best and simplest one.
(including the observation category is also supported using this approach too, however anything beyond that you'll need to adopt one of the other approaches here)

### Definition approach 1 (hidden fields)
[extract-defn1.json](extract-defn1.json)

The differences here are:
```json=
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
```

> **Note:** This approach requires the QR to be polluted with actual data specifically for the extract (which is otherwise of no value, and would prevent being able to refine the extract definition after QR data is complete).
> Also, this actual definition doesn't include anything to set the author, subject, issued or performed properties
> *(would that be special knowledge to do here, are there other resources that we would need to define that for?)*

### Definition approach 2 (new extractValue extension)
[extract-defn1.json](extract-defn1.json)

The differences here are:
```json=
    // inclusion of the extract context extension (to indicate the type of resource to extract)
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-itemExtractionContext",
            "valueCode": "Observation"
        }
    ],

    // and definition property(s) to indicate where to put the value(s) in that context
    "definition": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.value[x]",

    // And instead of hidden fields as in the previous example, we have a new extension to indicate where to get the value from (this is also able to retrieve the subject, author, issued and performer metadata properties)
    "extension": [
        {
            "url": "http://hl7.org/fhir/StructureDefinition/sdc-questionnaire-itemExtractionValue",
            "extension": [
                {
                    "url": "definition",
                    "valueCanonical": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.code.coding"
                },
                {
                    "url": "fixed-value",
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
                    "url": "definition",
                    "valueCanonical": "http://hl7.org/fhir/StructureDefinition/Observation#Observation.subject"
                },
                {
                    "url": "expression",
                    "valueExpression": {
                        "language": "text/fhirpath",
                        "expression": "%resource.subject"
                    }
                }
            ],
            "url": "http://hl7.org/fhir/StructureDefinition/sdc-questionnaire-itemExtractionValue"
        },
        ...
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
```json=
    // The extension to indicate the contained resource to use as a template
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-extractToTemplate",
            "valueReference": {
                "reference": "#obsTemplate"
            }
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
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-extractTemplateValue",
                        "valueString": "%resource.subject"
                    }
                ]
            },
            "_issued": {
                "extension": [
                    {
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-extractTemplateValue",
                        "valueString": "%resource.authored"
                    }
                ]
            },
            "performer": [
                {
                    "extension": [
                        {
                            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-extractTemplateValue",
                            "valueString": "%resource.author | %resource.subject"
                        }
                    ]
                }
            ],
            "_valueBoolean": {
                "extension": [
                    {
                        "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-extractTemplateValue",
                        "valueString": "%resource.item.where(linkId='complication').answer.value"
                    }
                ]
            }
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
```json=
    // inclusion of the extension to reference the StructureMap to use
    "extension": [
        {
            "url": "http://hl7.org/fhir/uv/sdc/StructureDefinition/sdc-questionnaire-targetStructureMap",
            "valueCanonical": "http://example.org/fhir/StructureMap/QRSigmoidoscopyComplicationToObservation"
        }
    ],
```

And of course the actual [StructureMap definition](extract-smap.fml) too (external to the questionnaire)
```fml=
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
  src -> tgt.code = cc('http://example.org/sdh/demo/CodeSystem/cc-screening-codes', 'sigmoidoscopy-complication') "SetObservationCode";
  src.subject -> tgt.subject;
  src.authored as s -> tgt.issued = s "SetAuthored";
  src.author -> tgt.performer;

  src.item as item where (linkId = 'complication') -> tgt.value = create('CodeableConcept') as obsValue then {
        item -> tgt.value = (%item.answer.value.first()) "SetObservationValue";
      };
}
```
This IS the most complex approach from an editor/author perspective, but also the most powerful.

