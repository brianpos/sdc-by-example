# Data Extraction from a Questionnaire Response

- Observation Based
- Definition Based
- Template Based (coming soon)
- StructureMap Based

If your extracted content is heavily profiled, then the definition approach is likely best.

To help understand the different approaches I've created a couple of examples that leverage the same questionnaire but use different extraction methods.
* [comparison-simple-obs](comparison-simple-obs) - A simple observation to be extracted
  - defn1 is unable to extract all the values as it is unable to read the QR metadata
* [comparison-complex](comparison-complex) - A complex multi-resource extraction
  - As there are more than just observations, this approach is not included here
  - as with the simple approach the definition based approach is unable to extract all the values
  - defn2 leverages the new extensions to read the QR metadata using fhirpath
  - defn3 adds the proposed `allocateId` extension to allocate an ID for the patient resource to use in the cross resource referencing
  - template leverages the new `template` extension to extract the data (as individual contained resources) from the QR
  - template2 leverages the new `template` extension to extract the data (as a single contained bundle resource) from the QR

