# API Licensing and API Economy Pricing

## ON LICENSING

1. First consideration is how the licensor and licensed content is framed. We are showing, up-front. that an object is part of licensed content and requires an attestation the requestor has usage rights to the licensed content. This means there are effectively two types of delivered content not related to paid or free. It is "licensed" or "unlicensed".
2. Second consideration is that all content from the API is "paid". The cost might be $0 making it effectively free. The cost of something "effectively free" today may be $1 tomorrow. It is an important nuance as it means there is an economy placed on the entire platform. As such "paid" and "free" do not provide great guidance as an indicator and all cost is based on what it costs.
3. Third consideration involves a priori inheritance. (all bachelors are unmarried) All citations in an authority document have the same license an authority document has - as a citation is the content of an authority document. There would be no mixed options where some citations in an authority document have one license, while the authority document has another, and a different citation has another.

### Examples

* AD unlicensed - citations unlicensed
* AD licensed - citation licensed - with the same license
* Mandates I think are always licensed. These represent IP provided by some agency.  Even if the cost is 0, the license exists.
* Controls should always be licensed. (there may not be a cost)
* Licensed or unlicensed should be independent of cost.  Some licensed content may cost $0. Some unlicensed content might cost $1.

Even with these assumptions, we may need to fall back on ALL CONTENT is licensed with either a Commons-like license or a more restrictive license. This would make "availability" values different than "licensed" and "unlicensed", and we would need to determine what those "values" would be. They should not be "paid" and "free", however.

## Endpoints

There are 5 endpoints: GET AuthorityDocument List, GET AuthorityDocument, GET Citation, GET Mandate, GET Control - these nest in exactly that format.

> A. GET AuthortyDocument List - This provides the summary object list of all AuthorityDocuments. (we will be working on rather standard search filtering, but for the moment it is just the list of all of them) There is ONE argument in the header: "X-License-Attestation": "Boolean". The summary object provides the basic target to the full object. (please note, right now we do not have the license\_url with the draft license or the licensor\_id... we have a 1 in place for UCF licensor)

```
{
    "@type": "AuthorityDocument",
    "LicenseInfo": {
        "@type": "LicenseInfo",
        "availability": "licensed",
        "license_url": "https://unifiedcompliance.com/???",
        "licensor_id": 1
    },
    "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/authority-document/3288",
    "element_id": 3288,
    "property_name": "published_name",
    "property_value": "Trust Services Criteria"
},
{
    "@type": "AuthorityDocument",
    "LicenseInfo": {
        "@type": "LicenseInfo",
        "availability": "unlicensed",
        "license_url": "https://unifiedcompliance.com/???",
        "licensor_id": 1
    },
    "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/authority-document/3289",
    "element_id": 3289,
    "property_name": "published_name",
    "property_value": "United States Code - 15 U.S.C. 278g-3a to 278g-3e, IoT Cybersecurity Improvement Act of 2020"
},
```

> B. GET AuthorityDocument - This provides one authority document from the prior list. There is ONE argument in the header: "X-License-Attestation": "Boolean". There is one PATH agument for the id ([https://ucf-paid-content-prototype.p.rapidapi.com/paid/authority-document/3288](https://ucf-paid-content-prototype.p.rapidapi.com/paid/authority-document/3288)). Within the AuthorityDocument object, the LicenseInfo is present which matches the LicenseInfo from the prior list. Within the AuthorityDocument object, a CitationCount object presents the quantity of Citations and the quantity of mandates. (mandates nest off citations... as you can see 511 citations created 716 mandates in the example below) Within the AuthorityDocument object, a Citations list is provided with Summary citation objects representing the full Citations data. Each summary Citation object has LicenseIfo that matches the license info for the Authority document. (see Third Consideration above)

```
{
  "@context": "https://grcschema.org/",
  "@type": "AuthorityDocument",
  "availability": "For Purchase",
  "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/authority-document/3288",
  "official_name": "Trust Services Criteria, (includes March 2020 updates)",
  "published_name": "Trust Services Criteria",
  "type": "Self-Regulatory Body Requirement",
  "citation_format": "¶ (Numbered Paragraphs)",
  //...Other Data Properties
  "CitationCount": {
    "@type": "CitationCount",
    "citation_count": 511,
    "mandate_count": 716
  },
  "Citations": {
    "@set": [
    {
        "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/citation/211276",
        "@type": "Citation",
        "element_id": 211276,
        "reference": "CC6.8 ¶ 2 Bullet 4 Uses Antivirus and Anti-Malware Software",
        "authority_document_fk": 3288,
        "LicenseInfo": {
            "@type": "LicenseInfo",
            "availability": "unlicensed",
            "license_url": "https://unifiedcompliance.com/???",
            "licensor_id": 1
        }
    },
    {
        "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/citation/211622",
        "@type": "Citation",
        "element_id": 211622,
        "reference": "CC7.3 ¶ 2 Bullet 1 Responds to Security Incidents",
        "authority_document_fk": 3288,
        "LicenseInfo": {
            "@type": "LicenseInfo",
            "availability": "unlicensed",
            "license_url": "https://unifiedcompliance.com/???",
            "licensor_id": 1
        }
    },
      // ... More Citations
      ]
  },
  "LicenseInfo": {
      "@type": "LicenseInfo",
      "availability": "licensed",
      "license_url": "https://unifiedcompliance.com/???",
      "licensor_id": 1
  }
  // ... Other Data Objects
}
```

> C. GET Citation - this provides one Citation object. There is ONE argument in the header: "X-License-Attestation": "Boolean". There is one PATH agument for the id. ([https://ucf-paid-content-prototype.p.rapidapi.com/paid/citation/211276](https://ucf-paid-content-prototype.p.rapidapi.com/paid/citation/211276)) Within the Citation object is the LicenseInfo that was present in the Citation summary object (from AuthorityDocument Citations list). Within the Citation object is a mandate\_count property that supplies how many mandates are applied to this Citation. Within the Citation object are the summary Mandates representing the full Mandates for this Citation object. Each summary Mandate has a LicenseInfo that represents the Licensor supplying the Mandate object. (this can and often will be a different license than the AuthorityDocument/Citation Licensor).

```
{
    "availability": "For Purchase",
    "@context": "https://grcschema.org/",
    "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/citation/211276",
    "guidance": "Antivirus and anti-malware software is implemented and maintained to provide for the interception or detection and remediation of malware.",
    "element_id": 211276,
    "reference": "CC6.8 ¶ 2 Bullet 4 Uses Antivirus and Anti-Malware Software",
    "CoreMetaData": {
        "modified_audit_id": null,
        "live_status": true,
        "superseded_by": null,
        "created_audit_id": null,
        "checksum": 1,
        "notes": null,
        "date_modified": "2021-02-12",
        "date_created": "2021-02-02",
        "validated": null
    },
    "LicenseInfo": {
        "@type": "LicenseInfo",
        "availability": "licensed",
        "license_url": "https://unifiedcompliance.com/???",
        "licensor_id": 1
    },
    "language": "eng",
    "parent_id": 212182,
    "authority_document_id": 3288,
    "@type": "Citation",
    "Mandates": {
        "@set": [
            {
                "LicenseInfo": {
                    "@type": "LicenseInfo",
                    "availability": "licensed",
                    "license_url": "https://unifiedcompliance.com/???",
                    "licensor_id": 1
                },
                "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/mandate/211276",
                "@type": "Mandate",
                "element_id": 211276,
                "citation_fk": 211276
            }
        ]
    },
    "mandate_count": 1
}
```

> D. GET Mandate - this provides one Mandate object. (it is a valuable object) There is ONE argument in the header: "X-License-Attestation": "Boolean". There is one PATH agument for the id. ([https://ucf-paid-content-prototype.p.rapidapi.com/paid/mandate/211276](https://ucf-paid-content-prototype.p.rapidapi.com/paid/mandate/211276)) Within the Mandate object is the LicenseInfo that was present in the Mandate summary object (from Citation Mandates list). Within the Mandate object is one MatchedControl which contains the summary Control that was matched. The summary Control has LicenseInfo relative to the Control Authority's license.

```
{
    "sort_value": 5,
    "@context": "https://grcschema.org/",
    "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/mandate/211276",
    "citation_id": 211276,
    "guidance_as_tagged": "{antivirus software} Antivirus and anti-malware software is implemented and maintained to provide for the interception or detection and remediation of malware.",
    "guidance": "Antivirus and anti-malware software is implemented and maintained to provide for the interception or detection and remediation of malware.",
    "MatchedControl": {
        "@type": "MatchedControl",
        "attestation_url": "https://www.unifiedcompliance.com/???",
        "certainty": null,
        "method": null,
        "Control": {
            "@type": "Control",
            "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/control/575",
            "LicenseInfo": {
                "@type": "LicenseInfo",
                "availability": "licensed",
                "license_url": "https://unifiedcompliance.com/???",
                "licensor_id": 1
            },
            "element_id": 575,
            "control_authority_id": 1
        }
    },
    "element_id": 211276,
    "reference": "CC6.8 ¶ 2 Bullet 4 Uses Antivirus and Anti-Malware Software",
    "CoreMetaData": {
        "modified_audit_id": null,
        "live_status": true,
        "superseded_by": null,
        "created_audit_id": null,
        "checksum": 1,
        "notes": null,
        "date_modified": "2021-02-12",
        "date_created": "2021-02-02",
        "validated": null
    },
    "sort_id": "006 026 005",
    "LicenseInfo": {
        "@type": "LicenseInfo",
        "availability": "licensed",
        "license_url": "https://unifiedcompliance.com/???",
        "licensor_id": 1
    },
    "language": "eng",
    "authority_document_id": 3288,
    "@type": "Mandate",
    "TaggedSentence": {
        "@context": "https://grcschema.org/",
        "@id": null,
        "sentence": "{antivirus software} Antivirus and anti-malware software is implemented and maintained to provide for the interception or detection and remediation of malware.",
        "element_id": 89267,
        "@type": "TaggedSentence",
        "TaggedPhrases": [
            {
                "@context": "https://grcschema.org/",
                "tagged_sentence_fk": 89267,
                "@id": null,
                "start": 1,
                "type": "Primary Noun",
                "TaggedPhraseTerm": {
                    "@type": "TaggedPhraseTerm",
                    "element_id": 3459,
                    "nonstandard": false,
                    "preferred_term": null
                },
                "element_id": 497882,
                "nonstandard": true,
                "term_preferred_term": null,
                "end": 19,
                "@type": "TaggedPhrase",
                "TaggedPhraseDefinition": {
                    "@type": "TaggedPhraseDefinition",
                    "element_id": 30635,
                    "definition": "A program that monitors a computer or network to identify all viruses and prevent or contain virus incidents.",
                    "other_form": null,
                    "word_type": "Asset"
                },
                "term_id": 3459,
                "term_nonstandard": false
            },
            {
                "@context": "https://grcschema.org/",
                "tagged_sentence_fk": 89267,
                "@id": null,
                "start": 35,
                "type": "Primary Noun",
                "TaggedPhraseTerm": {
                    "@type": "TaggedPhraseTerm",
                    "element_id": 252184,
                    "nonstandard": false,
                    "preferred_term": null
                },
                "element_id": 497883,
                "nonstandard": true,
                "term_preferred_term": null,
                "end": 56,
                "@type": "TaggedPhrase",
                "TaggedPhraseDefinition": {
                    "@type": "TaggedPhraseDefinition",
                    "element_id": 198576,
                    "definition": "A program that monitors a computer or network to identify all major types of malware: virus, trojan horse, spyware, Adware, worms, rootkits, etc.",
                    "other_form": null,
                    "word_type": "Asset"
                },
                "term_id": 252184,
                "term_nonstandard": false
            },
            {
                "@context": "https://grcschema.org/",
                "tagged_sentence_fk": 89267,
                "@id": null,
                "start": 60,
                "type": "Primary Verb",
                "TaggedPhraseTerm": {
                    "@type": "TaggedPhraseTerm",
                    "element_id": 17589,
                    "nonstandard": false,
                    "preferred_term": 253298
                },
                "element_id": 497884,
                "nonstandard": true,
                "term_preferred_term": 253298,
                "end": 86,
                "@type": "TaggedPhrase",
                "TaggedPhraseDefinition": {
                    "@type": "TaggedPhraseDefinition",
                    "element_id": 1259,
                    "definition": "To lay the groundwork for something and uphold it or ensure continuation by requiring maintenance.",
                    "other_form": null,
                    "word_type": "Verb"
                },
                "term_id": 17589,
                "term_nonstandard": false
            }
        ],
        "correct": true
    }
}
```

> E. GET Control - This provides one Control object. There is ONE argument in the header: "X-License-Attestation": "Boolean". There is one PATH agument for the id. ([https://ucf-paid-content-prototype.p.rapidapi.com/paid/control/575](https://ucf-paid-content-prototype.p.rapidapi.com/paid/control/575)) Within the Control is a LicenseInfo object that was present in the Control summary object from the MatchedControl object in Mandate.

```
{
    "ControlAuthority": {
        "@type": "ControlAuthority",
        "control_authority_abbreviation": "UCF",
        "control_authority_id": 1,
        "control_authority_name": "Unified Compliance"
    },
    "sort_value": 2,
    "@context": "https://grcschema.org/",
    "@id": "https://ucf-paid-content-prototype.p.rapidapi.com/paid/control/575",
    "name": "Install security and protection software, as necessary.",
    "type": "Configuration",
    "classification": "Preventive",
    "impact_zone": "Technical security",
    "element_id": 575,
    "CoreMetaData": {
        "modified_audit_id": null,
        "live_status": true,
        "superseded_by": null,
        "created_audit_id": null,
        "checksum": 9,
        "notes": null,
        "date_modified": "2021-05-04",
        "date_created": "2005-12-28",
        "validated": null
    },
    "sort_id": "001 004 018 002",
    "LicenseInfo": {
        "@type": "LicenseInfo",
        "availability": "licensed",
        "license_url": "https://unifiedcompliance.com/???",
        "licensor_id": 1
    },
    "language": "eng",
    "parent_id": 574,
    "@type": "Control",
    "Metric": {
        "@type": "Metric",
        "metric_calculation": null,
        "metric_image_reference": null,
        "metric_information_source": null,
        "metric_name": null,
        "metric_presentation_format": null,
        "metric_target_result": null
    },
    "TaggedSentence": {
        "@context": "https://grcschema.org/",
        "@id": null,
        "sentence": "Install security and protection software, as necessary.",
        "element_id": 94450,
        "@type": "TaggedSentence",
        "TaggedPhrases": [
            {
                "@context": "https://grcschema.org/",
                "tagged_sentence_fk": 94450,
                "@id": null,
                "start": 0,
                "type": "Primary Verb",
                "TaggedPhraseTerm": {
                    "@type": "TaggedPhraseTerm",
                    "element_id": 4329,
                    "nonstandard": false,
                    "preferred_term": null
                },
                "element_id": 524288,
                "nonstandard": false,
                "term_preferred_term": null,
                "end": 7,
                "@type": "TaggedPhrase",
                "TaggedPhraseDefinition": {
                    "@type": "TaggedPhraseDefinition",
                    "element_id": 26557,
                    "definition": "In Computing: to set up or place software for use on a machine or network.",
                    "other_form": null,
                    "word_type": "Verb"
                },
                "term_id": 4329,
                "term_nonstandard": false
            },
            {
                "@context": "https://grcschema.org/",
                "tagged_sentence_fk": 94450,
                "@id": null,
                "start": 8,
                "type": "Primary Noun",
                "TaggedPhraseTerm": {
                    "@type": "TaggedPhraseTerm",
                    "element_id": 252187,
                    "nonstandard": false,
                    "preferred_term": null
                },
                "element_id": 524289,
                "nonstandard": false,
                "term_preferred_term": null,
                "end": 40,
                "@type": "TaggedPhrase",
                "TaggedPhraseDefinition": {
                    "@type": "TaggedPhraseDefinition",
                    "element_id": 198581,
                    "definition": "Software that is put in place to scan a computer or network in order to detect and mitigate threats such as malware and cracking.",
                    "other_form": null,
                    "word_type": "Asset"
                },
                "term_id": 252187,
                "term_nonstandard": false
            }
        ],
        "correct": true
    }
}
```

> F. On all GET data requests, not supplying the X-License-Attestation header with a boolean true value results in the following error. The object is not provided.

```
{
    "error": {
        "message": "Valid attestation header was not received."
    },
 ... currently has stack trace info which will be gone in production.
}
```

_Note: Other than filter and search parameters which will be added on the AuthorityDocument List endpoint, there are no other "Query Parameters" on these endpoints. There are other header parameters added for Authorization depending on the auth model, but these are not particularly novel.... and do not need to be included._

## Online Documentation

Here is some quick online documentation: [https://documenter.getpostman.com/view/7580355/TzeXjnA5](https://documenter.getpostman.com/view/7580355/TzeXjnA5) You can use this key: 827808040cmsh098351ace91b363p15bd43jsnb37d899c2453

## ON COST & CHARGING

There are two approaches being discussed for COST.

### Fixed/Published Pricing

This is a blanket cost for objects of a specific type. This is accomplished with a published pricing table that can be used to determine the costs for objects in the system. The costs shown here are illustrative only.

| Object            | Cost |
| ----------------- | ---- |
| AuthorityDocument | 1.00 |
| Citation          | 1.00 |
| Mandate           | 5.00 |
| Control           | 2.00 |

When querying an Authority Document object, the cost will be one dollar. By doing so, the total counts are given which allows for the developer application to determine total cost.

```
"CitationCount": {
  "@type": "CitationCount",
  "citation_count": 511,
  "mandate_count": 716
},
```

One dollar for the AD call.

* $511 to pull all the citation records.
* $3580 to pull all the mandate records.
* $1432 to pull all the controls.  (mandate:control is 1:1)
* 511+3580+1432+1 = $5524

Subsets of that, like a condition where it is known that one citation and its mandates are needed would be priced by

* $1 to query the Authority document
* \$$1 to query the Citation (assume 3 mandates)
* \$$15 to query the Mandates
* \$$6 to query the Controls

This puts the burden of determining cost on the application based on PUBLISHED cost tables. It is what we are going with first as an argument can be made that accounts can estimate their consumption based on their desired documents and fund the account for charge debit.

### Advanced Lookup Pricing

A future model we discussed briefly would include an endpoint for sending in an AuthorityDocument ID and the objects needed (e.g. Citations but not Mandates or Controls), and get back the Price of the calls calls to get those objects.

Query Authority Document ID 10, for Citations - if it has 10 citations, your cost would be the AD plus 10 citations... or $11 (assuming we still have to maintain some form of global price table)

This can expand into more granular pricing and price lookups as time goes on.

### Charging Prepaid Credit

No matter the COST lookup and understanding, there is the actual CHARGE. The charge is the amount you just spent getting the object you wanted. For the prior CitationCount example, when you pulled the AD, recursively pulled the citations, recursively pulled the mandates, and recursively pulled the controls... your account got "Charged" $5524.

We are specifying the DEBIT to your account, for each object, in a header response that will be read by the gateway. There are a lot of normal objects added into a header during normal API communications. The novel part here is X-Object-Cost. It is the value that represents the amount you just paid. (shown here as $40... again just as an example). The API responses from above stamp this into the header now... it isn't yet tied to a price table or published pricing - as we move through the prototyping.

![](.gitbook/assets/2021-06-21\_11-40.png)
