## Description

(Description of the MDR)

### References

(Relevant references, if available)

### Data

_Required_
- **TIDeMEC** : [CDMXXXX](link_to_cdm_wiki_page)
- **TLP** : (default: GREEN)

_Optional_
- **Index** :  (index_to_use)
- **Sourcetype** :  (source_type_to_use)
- **Scheduling** : (desired_schedule)
- **Lookback** :  (desired_lookback)

### Design guidelines and instructions

#### Identify and reference detection logic prototypes

For each of following vetted source, search in their detection database if any detection rule is already drafted and documented.
If so, 
 - [ ] please add proper reference link into MDR file.
 - [ ] in MDR description, provide a summary of the detection logic used in the referenced link

#### Log collection
Please check log collections for intended detction logic(s) are:
- [ ] available; list them in MDR
- [ ] of good quality e.g. sourcetype in Splunk has been validated and key fields for search criteria properly extracted. 

##### Vetted sources
- [ ] SOC Prime TDM
    * [SOC Prime Platform](https://tdm.socprime.com/login/)
- [ ] [Splunk SURGe](https://www.splunk.com/en_us/surge.html) research team
    * [Splunk detection analytics and rules research site](https://research.splunk.com/)
    * [Splunk GITHUB repository](https://github.com/splunk/security_content)
    * [ES-SPLUNK PROD SSE app](https://es-splunk.tech.ec.europa.eu/en-GB/app/Splunk_Security_Essentials/contents)
    * [ES-SPLUNK ACC SSE app](https://es-splunk.acceptance.tech.ec.europa.eu/en-GB/app/Splunk_Security_Essentials/overview)
- [ ] Microsoft Sentinel library
    * [Azure-Sentinel detections](https://github.com/Azure/Azure-Sentinel/tree/master/Detections)

### Notes

(Optional freeform notes)

/label ~🚨 Managed Detection Rule ~"✨ New Object"
