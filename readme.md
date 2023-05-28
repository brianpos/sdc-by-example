# FHIR Structured Data Capture (SDC) by example
This repository contains guidance and examples of using the FHIR SDC forms and questionnaires. The examples are based on the [FHIR SDC Implementation Guide](https://hl7.org/fhir/uv/sdc) along with the [core R4B specification](https://hl7.org/fhir/R4B/questionnaire.html).

This guide is not intended to re-define anything from the core specifications, only to provide additional guidance and annotated examples for using the specifications.


## Contents
1. The Basics - simple questionnaire structure and basics
2. Validation - configuration of validation rules inside a questionnaire
3. Pre-population - pre-population of a questionnaire based on existing data (e.g. from a patient record)
4. Extraction - data extraction from a questionnaire response into other clinical resources
5. Dynamic Behaviors - dynamic behaviors of a questionnaire (e.g. branching logic, scoring, etc.)
6. Advanced Rendering - advanced rendering including tables, images, sliders etc

## Editing and rendering tools
If you're looking for tooling to edit or render questionnaires, check out the [tooling page](https://confluence.hl7.org/display/FHIRI/SDC+Implementations) maintained by HL7 International.

From my own personal experience I'd point out that building forms is very rarely a one pass event, and you'll likely need to iterate on the form design and content. Each stage will be adding additional capabilities and features, so you'll need to be able to edit and render the form at each stage.

Some typical form editing stages you might encounter are:
* Initial question creation - Just getting the basic shape of the form/survey together, initial questions, wording, groups, item types.
    *(at this stage often the item type or choice options isn't critical and might be left as a placeholder)*
* Solidifying content - Ensuring the question structure is right, the item types are correct, the choice options are correct, the wording is correct, etc.
    *(At this stage I would recommend renaming the linkIds to their final names, as once you start adding the smarts into place, changing them is more difficult, and you'll be rewarded down the line if the names are meaningful and not just guids/uuids/random ids)*
* Validation - Adding validation rules to the form, ensuring they are correct and working as expected
* Dynamic Behaviors - Adding the "smarts" to the form, enable whens, calculated fields
* Pre-population - Adding the pre-population rules coming from various sources
* Extraction - Adding the extraction rules to extract data from the form into other clinical resources
* Language Translations

These stages don't necessarily happen in a linear fashion, and you may need to iterate back and forth between them, and likely to occur by different people at different times.

I've also found that no one tool caters for all requirements, so you may need to use a combination of tools to get the job done.

Notable ones that I've used in the past are:
* US NLM's [Questionnaire Builder](https://lhcformbuilder.nlm.nih.gov) - a web-based editor and renderer
* https://lhcforms.nlm.nih.gov/lhcforms


## Contributing
If you'd like to contribute to this guide, please fork the repository and submit a pull request. Or just get on touch and log issues to the issue tracker.
