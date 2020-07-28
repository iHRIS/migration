# Migration from iHRIS Manage 4.x to iHRIS V5

iHRIS Version 5.0 is based on the popular and easy-to-use FHIR interoperability standard for health information systems. iHRIS V5 uses Hapi FHIR server, a free and open source
implemetation of the FHIR standard.
To move the data from iHRIS Manage V.4.x to the version 5, data needs to be extracted in FHIR format then pushed to the Hapi Fhir server.
iHRIS v4.x is a collection of PHP modules organized by the features that the provides. This module is the update of the iHRIS Manage 4.3 native module.
So to perform the migration to iHRIS V5, your iHRIS sites must be updated to the last version of iHRIS 4.3.


## Prerequisites
Before you perform the migration, you will need:

 1. An updated iHRIS Manage 4.3 instance.
 2. A [Hapi]((https://hapifhir.io/)) Fhir server
 3. The [data forms map](https://wiki.ihris.org/wiki/Create_a_Data_Form_Map_For_My_Custom_Site) of your 4.3 iHRIS custom site.

## Upgrating iHRIS to the version 4.3

If you are running a 4.1 or 4.2, you will need to follow the Upgrading instruction as described in the [iHRIS wiki](https://wiki.ihris.org/wiki/Upgrading_to_iHRIS_4.2).
Since this procedure is for the migration to 4.2, please replace in the instruction 4.2 by 4.3.
Since the upgrading is an incremetal process, you will need to migrate gradually. If you are running iHRIS 4.1, migrate to 4.2 then to 4.3.
For iHRIS Manage 4.3 installation, please follow this [link](https://wiki.ihris.org/wiki/IHRIS_Manage_Installation_-_4.3.3)
And for iHRIS Manage 4.2 installation, please follow this [link](https://wiki.ihris.org/wiki/Installing_iHRIS_4.2)

## Installing the module for FHIR extraction
Download the module mCSD Update Supplier for FHIR resources extraction and copy it in the /modules folder of your iHRIS site. 
Disable the native module if enabled: Go to iHRIS Manage > Configure system > Configure modules > In section Application=>iHRIS Manage]> Sub-modules > identify the module mCSD Update Supplier > Uncheck > click enable
Enable the new Module: Go to iHRIS Manage > Configure system > Configure modules > In the site section,Identify the module mCSD Update Supplier for FHIR resources extraction > Check it and click enable.

# Prepare for the migration
This module allows to extraction the following resources:
 1. [Practitioner](https://www.hl7.org/fhir/practitioner.html): All persons recorded as health worker in iHRIS. Information on contacts,address, educations, trainings,registration are respectivelly linked to telecom,
 qualifications. All other information such as administrative situation,attached documents and so on are added as extension.
 2. [PractitionerRole](https://www.hl7.org/fhir/practitionerrole.html): All positions hold by practitioner in specific locations or organizations. The employement history is all treated as PractitionerRoles. 
 Informations on salary or other benefits as well as additonnal custom fields are managed as extensions.
 3. [Location](https://www.hl7.org/fhir/location.html): Related to facilities and geographic entities.
 4. [Organization](https://www.hl7.org/fhir/organization.html): This includes name of education,training institutuon and associations.
 
 Since each iHRIS Manage site is customized based to the user needs,new fields could be added to the standard forms, new forms may be created as well as new list elements. 
 You need to retrieve all the fields to you want to export and link them to the existing FHIR resource attribute. For the fields that does not match any of the concerned resource
 attributes, they can be add as extension. This operation is called mapping.
 
 It is advices to create a Excel file when you will put the name of the form, the field,the data type,the nature of the link (one-one,one-multiple), 
 the concerned fhir resource, the corresponding attribute or extension with its name.
 Now go to the module's folder <yoursitepath>/modules/mCSDUpdateSupplier/mapping, each resource has a config file to perform the mapping of iHRIS form's fields 
 to the FHIR res apart for the resource Location.
 - practitoner.json for the mapping of the Practitioners.
 - practitionerrole.json for the mapping of the PractitionerRoles.
 - organization.json for the mapping of the Organizations.
 
### The structure of the mapping configuration file
The mapping config file is a json file format which has the following properties
```
{
    "resourceType"":"Practitioner",
    "fieldsMapping":[
        {
            "iHRISForm":"person",
            "useForLinked":true,
            "iHRISField":[...],
            "extension":[...],
            "iHRISJoinedForms":[...]
         }
    ],
    "fieldsLinked":[
		...
    ],
    "configs":{...}
}
            
```
- fieldsMapping.iHRISForm = person: defines the primary form.
- fieldsMapping.useForLinked = true: defines that the iHRISForm is used to link with forms in "fieldsLinked". Link are interpreted as LEFT JOIN SQL statement
- fieldsMapping.iHRISField: contains the list of field to import and their corresponding FHRI resource attributes.
- fieldsMapping.extension: contains field to generate as extension. An extension can be applied to the iHRIS field directly in this case the fields 
will be generated as a simple field  extension. if the fields are defined in the extension:[] properties,all the fields are generated as the collection of extension fields 
for the resource.

- fieldsMapping.iHRISJoinedForms: contains the forms definition and the fields for all one-one form link. such as the forms: demographics,personal_contact,work_contact.
- fieldsLinked: contains the list of forms that have one-multiple link. Such as positions, educations, work history,languages,trainings,registrations.
- configs: any additional configuration related to the processing of a particular field or a group of field. If you don't want to use one of the configuration just leave 
the value empty.

### The structure definition of the iHRISField  

The fieldsMapping.iHRISField collections contents looks like this:
```
{
	"fieldId":"csd_uuid",
	"iHRISFieldAlias":"uuid",
	"fhirField":"id",
	"active":true,
	"fhirBaseElement":"Practitioner"
},
{
	"fieldId":"id",
	"iHRISFieldAlias":"id",
	"fhirField":"value",
	"active":true,
	"fhirBaseElement":"identifier"
},
{
	"fieldId":"surname",
	"iHRISFieldAlias":"family",
	"fhirField":"family",
	"active":true,
	"fhirBaseElement":"name"
},
{
	"fieldId":"firstname",
	"iHRISFieldAlias":"given1",
	"fhirField":"given",
	"active":true,
	"fhirBaseElement":"name"
}
```
- fieldId: the name of the field in the iHRIS form. This is considered as the parent form.
- iHRISFieldAlias: the alias of the field. This prevent to generate 2 fields in the sql request that has the same name in the dataset.The iHRISFieldAlias must be unique. 
- fhirField: the name of the corresponding fhir attribute. This field is related to the Fhir attribute for the simple attribute (string,integer) 
the fhirBaseElement is the Resource and for the composite attribute (field is an complex type),the fhirBaseElement is the name if the field+".nameofthefieldattribute"
- active: Weither to extract the current field or not.
- fhirBaseElement: The name of resource where the fhirField is attached to. The name of the resource for the simple field type or the name of the resource.attribute for
the complex type.

### The structure definition of the iHRISJoinedForms  
The iHRISJoinedForms  content looks like this:
```
{
"iHRISForm":"demographic",
"iHRISFormAlias":"demographic",
"iHRISFormJoinFieldSource":"id",
"iHRISFormJoinFieldDestination":"parent",
"iHRISField":[
	
	{
		"fieldId":"dependents",
		"iHRISFieldAlias":"dependents",
		"fhirField":"dependents",
		"active":false,
		"fhirBaseElement":"Practitioner",
		"extension":{
			"extensionName":"iHRISPractitionerDependents",
			"fieldType":"integer"
		}
	},
	{
		"fieldId":"birth_date",
		"iHRISFieldAlias":"birthdate",
		"fhirField":"birthDate",
		"active":true,
		"fhirBaseElement":"Practitioner"
	}
],
"iHRISJoinedForms":[
	{
		"iHRISForm":"gender",
		"iHRISFormAlias":"gender",
		"iHRISFormJoinFieldSource":"gender",
		"iHRISFormJoinFieldDestination":"id",
		"iHRISField":[
			{
				"fieldId":"name",
				"iHRISFieldAlias":"gender",
				"fhirField":"gender",
				"active":true,
				"fhirBaseElement":"Practitioner"
			}
		]
	},
	{
		"iHRISForm":"marital_status",
		"iHRISFormAlias":"marital_status",
		"iHRISFormJoinFieldSource":"marital_status",
		"iHRISFormJoinFieldDestination":"id",
		"iHRISField":[
			{
				"fieldId":"name",
				"iHRISFieldAlias":"marital_status",
				"fhirField":"maritalStatus",
				"active":true,
				"fhirBaseElement":"Practitioner",
				"extension":{
					"extensionName":"iHRISPractitionerMaritalStatus",
					"fieldType":"code"
				}
			}
		]
	}
]

}
```
- iHRISForm: is the iHRIS form namejoin to the fieldsMapping.iHRISForm. This is considered as one of the child of the parent form.
- iHRISFormAlias: is the alias of the form,It must be unique
- iHRISFormJoinFieldSource: the field use on the parent form to join the child form.
- iHRISFormJoinFieldDestination: the field use on the child form to establish the link with the parent form.
- iHRISField:collection of the fields to extract in the form.

### The structure definition of the extension

The extension can be applied directly the fields. This field will be generated as an extension.
```
"iHRISField":[
	
	{
		"fieldId":"dependents",
		"iHRISFieldAlias":"dependents",
		"fhirField":"dependents",
		"active":false,
		"fhirBaseElement":"Practitioner",
		"extension":{
			"extensionName":"iHRISPractitionerDependents",
			"fieldType":"integer"
		}
	},
...
]
```
- fieldId: the name of the field in the iHRIS form. This is considered as the parent form.
- iHRISFieldAlias: the alias of the field. This prevent to generate 2 fields in the sql request that has the same name in the dataset.The iHRISFieldAlias must be unique. 
- fhirField: the name of the corresponding fhir attribute. This field is related to the Fhir attribute for the simple attribute (string,integer) 
the fhirBaseElement is the Resource and for the composite attribute (field is an complex type),the fhirBaseElement is the name if the field+".nameofthefieldattribute"
- active: Weither to extract the current field or not.
- extension.extensionName: The name of extension as used in the url attribute.
- extension.fieldType: The type of the field. The values are: string,integer,date,dateTime,code. Code is used for list values or list of options.

The group of the fields can constitute also an extension.

```
"extension":[{
	"extensionName":"iHRISPractitionerIdentityDetails",
	"fhirBaseElement":"Practitioner",
	"type":"inGroup",
	"appliedFields":"",
	"iHRISField":[
		{
			"fieldId":"name_father",
			"iHRISFieldAlias":"name_father",
			"fhirField":"fatherNames",
			"active":true,
			"fieldType":"string"
		},
		{
			"fieldId":"name_mother",
			"iHRISFieldAlias":"name_mother",
			"fhirField":"motherNames",
			"active":true,
			"fieldType":"string"
		}
	]
}]
```
- extensionName: The name of extension as used in the url attribute.
- fhirBaseElement: Practitioner. extension is applied to the pratitioner resource level. By default.It could also be applied to the complex type attribute of the FHIR resource.
- type:inGroup. The type of the extension with 2 values: inGroup|byFields.
- appliedFields: The seperated comma list of iHRISFieldAlias where this extension will be applied to.
- iHRISField: collection of the fields to extract in the form.

### the practitioner config sections

```
"configs":{
        "identifierSystemUrl":"http://localhost/ihris-guinee/",
        "extensionUrl":"http://ihris.org/fhir/StructureDefinition/",
        "nameUse":"official",
        "binaryServerUrl":"http://localhost/ihris_photo/",
        "docServerUrl":"http://localhost/ihris_docs/",
        "photoFileExtension":".jpg",
        "transformQualificationInstitutionNameToId":{
            "iHRISFieldAlias":"education_institution,dip_training_institution,registration_board"
        },
        "concatenateDocServerUrl":{
            "iHRISFieldAlias":"doc_url",
            "value":"http://localhost/ihris_docs/"
        },
        "appendCountryCodeToPhoneNumber":{
            "value":"+243"
        },
        "genderCode":{
            "Masculin":"male",
            "Féminin":"female",
            "autres":"other"
        }
    }
"
```
- identifierSystemUrl: the default url used in the identifier.system.
- extensionUrl: the default url used in the url attribute of the extension.
- nameUse: the default use of the name.
- binaryServerUrl: url location to reference to the photo. this is appended to the photo name. The photo need to be extracted from iHRIS table and store as jpg file on a server 
and accessed through a url. The script save_photo2file.php in tools folder will be used to extract photo from the database to a file.
- docServerUrl: cation to reference to the attached document. this is appended to the document name.
- photoFileExtension: The extension define for photo when they will be extracted from iHRIS database.
- transformQualificationInstitutionNameToId: Some fields related to the qualification issuer are entered on a simple textbox. they need to be used as Organization.id if necessary
- concatenateDocServerUrl: Defines the iHRISFieldAlias and value to append to the field to form an accessible url.
- appendCountryCodeToPhoneNumber: wetheir the contry code should be appended to the phone number. Necessary for mHero.
- genderCode: Valid correspondent for gender code valueSet.

### The practitionerRole config sections

```
"configs":{
        "identifierSystemUrl":"http://localhost/ihris-guinee/",
        "extensionUrl":"http://ihris.org/fhir/StructureDefinition/",
        "removeCurrencyFromAmount":{
            "iHRISFieldAlias":"salary_amount"
        },
        "transformFieldToResourceID":{
            "iHRISFieldAlias":"deployment_uuid"
        }
}
```
- identifierSystemUrl: the default url used in the identifier.system.
- extensionUrl: the default url used in the url attribute of the extension.
- removeCurrencyFromAmount: by default iHRIS append the currency code to the moneyAmount. true remove it,otherwise use false.
- transformFieldToResourceID: Some fields related to the work history contains only the position name instead of position form as used in the module position.

### The organization config sections

```
"configs":{
        "identifierSystemUrl":"http://localhost/ihris-guinee/",
        "extensionUrl":"http://ihris.org/fhir/StructureDefinition/",
        "transformFieldToResourceID":{
            "iHRISFieldAlias":"educ_institution,jobdip_institution,nodip_institution,council_id"
        },
        "setDistinctSelectionFields":{
            "iHRISFieldAlias":"educ_institution,jobdip_institution,nodip_institution"
        }
}
```
- identifierSystemUrl: the default url used in the identifier.system.
- extensionUrl: the default url used in the url attribute of the extension.
- transformFieldToResourceID: In case some   fields related to the qualification issuer are entered on a simple textbox. they need to be used as Organization.id.

## Copy the binary data from iHRIS to the file system (Optional)

If the photos and attached documents need to be copied out of iHRIS you will need to execute the script save_photo2file.php
- First copy the scripts in the tools folder of your sites
- Then run the script to extract the photo
```
/tools$ php save_photo2file.php
```
For other attachement, you can change the name of the variable $formName.
Then copy the file to the location (server) where it can be referenced in iHRIS v5.

## Extract data as FHIR resource

Restart the memcached
```
sudo service memcached restart
```
- To extract the list of Practitioner, open this link
<yourihrissiteurl>/FHIR/Practitioner/_history?_format=json

- To extract the list of Location, open this link
<yourihrissiteurl>/FHIR/Location/_history?_format=json

- To extract the list of PractitionerRole, open this link
<yourihrissiteurl>/FHIR/PractitionerRole/_history?_format=json

- To extract the list of Organization, open this link
<yourihrissiteurl>/FHIR/Organization/_history?_format=json

## Push the practitioner in the Hapi

In the FHIR, use the tool pullScratchpad. Copy it on the home directory (or any prefered location ) of the iHRISv5 server, install the dependencies and post the FHIR resources
in the Hapi FHIR

```
cd pullScratchpad
npm install
node index.js
```




