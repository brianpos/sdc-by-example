# Basic anatomy of a Questionnaire

## Item Types

- [Questionnaire.item.type](http://hl7.org/fhir/questionnaire-definitions.html#Questionnaire.item.type) -
  The type of questionnaire item. This is a required field.

---

### Display Text

One extension that REALLY helps when adding display text is the
[markdown extension](http://hl7.org/fhir/extension-questionnaire-itemControl.html).
This allows you to use markdown in the display text for better formatting, such
as bold text, or newlines on content that really needs it.

This is often used in instructional display text added into some forms for
general guidance on the questions or a section in a form.

```json
{
    "linkId": "explanation.header",
    "text": "Each of the questions in this section leverage different techniques for selecting possible values for the options",
    "_text": {
        "extension": [
            {
                "url": "http://hl7.org/fhir/StructureDefinition/rendering-markdown",
                "valueMarkdown": "**Each of the questions in this section leverage different techniques for selecting possible values for the options**"
            }
        ]
    },
    "type": "display"
},
```

---

### Choice / Open-Choice

The choice type requires the use of either answerValueSet, answerOption or the
answerExpression extension.

The answerValueSet is the most common way to define the options. The
answerOption is used when you want to define the options inline. The
answerExpression is used when you want to define the options using an
expression.

Refer to the [terminology](terminology.md) section for examples of the various
ways you can link in terminology.

Often will want to include which UI control is to be used with the
[itemControl](http://hl7.org/fhir/R4/extension-questionnaire-itemcontrol.html)
extension. The
[valueset-questionnaire-item-control](http://hl7.org/fhir/R4/valueset-questionnaire-item-control.html#expansion)
value set defines the various options. The most common ones used in choice types
are: radio-button, drop-down and autocomplete.

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


