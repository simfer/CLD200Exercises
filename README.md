# CLD200_22


## Unit 1: Discovering the SAP Cloud Application Programming Model (CAP)

### Lesson 1: Identifying the need for Side-By-Side Extensibility 10 Minutes
No code

### Lesson 2: Exploring the SAP Cloud Application Programming Model 40 Minutes
No code


## Unit 2: Setting up the CAP-Project

### Lesson 1: Introducing the OData protocol 20 Minutes
No code

### Lesson 2: Explaining JSON/YAML
No code

### Lesson 3: Discovering the End-to-End Use Case 15 Minutes
No Code

### Lesson 4: Exercise 1: Creating a CAP-Based Service 15 Minutes

#### Task 1 - Point b - Clone the project
```bash
mkdir ~/projects/risk-management
cd ~/projects/risk-management/
git clone https://github.com/SAP-samples/extension-suite-learning-journey ./
```

#### Task 1 - Point g
```bash
npm ci
```

#### Task 1 - Point h
```bash
cds watch
```

#### Task 2 - Point d - db/schema.cds
```cds
namespace riskmanagement;

using {
    managed,
    cuid,
    User,
    sap.common.CodeList
} from '@sap/cds/common';

entity Risks : cuid, managed {
    title                   : String(100);
    owner                   : String;
    prio                    : Association to Priority;
    descr                   : String;
    miti                    : Association to Mitigations;
    impact                  : Integer;
    // bp : Association to BusinessPartners;
    virtual criticality     : Integer;
    virtual PrioCriticality : Integer;
}

entity Mitigations : cuid, managed {
    descr    : String;
    owner    : String;
    timeline : String;
    risks    : Association to many Risks
                   on risks.miti = $self;
}

entity Priority : CodeList {
    key code : String enum {
            high   = 'H';
            medium = 'M';
            low    = 'L';
        };
}
```

#### Task 3 - srv/risk-service.cds
```
using {riskmanagement as rm} from '../db/schema';

@path: 'service/risk'
service RiskService {
    entity Risks       as projection on rm.Risks;
    annotate Risks with @odata.draft.enabled;
    entity Mitigations as projection on rm.Mitigations;
    annotate Mitigations with @odata.draft.enabled;
    // BusinessPartner will be used later
    //@readonly entity BusinessPartners as projection on rm.BusinessPartners;
}
```

NOTE: createdAt and modifiedBy are automatically addedd

#### Task 4 - Point b - db/data/riskmanagement-Risks.csv

> Just take a look at this file: you don't have to change anything

```csv
ID;createdAt;createdBy;title;owner;prio_code;descr;miti_ID;impact;
20466922-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;CFR non-compliance;Fred Fish;H;Recent restructuring might violate CFR code 71;20466921-7d57-4e76-b14c-e53fd97dcb11;10000;
20466922-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;SLA violation with possible termination cause;George Gung;M;Repeated SAL violation on service delivery for two successive quarters;20466921-7d57-4e76-b14c-e53fd97dcb12;90000;
20466922-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Shipment violating export control;Herbert Hunter;L;Violation of export and trade control with unauthorized downloads;20466921-7d57-4e76-b14c-e53fd97dcb13;200000;
```

#### Task 4 - Point b - db/data/riskmanagement-Mitigations.csv

> Just take a look at this file: you don't have to change anything

```csv
ID;createdAt;createdBy;descr;owner;timeline
20466921-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;SLA violation: authorize account manager to offer service credits for recent delivery issues;suitable BuPa;Q2 2020
20466921-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;"SLA violation: review third party contractors to ease service delivery challenges; trigger budget review";suitable BuPa;Q3 2020
20466921-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Embargo violation: investigate source of shipment request, revoke authorization;SFSF Employee with link possible?;29.03.2020
20466921-7d57-4e76-b14c-e53fd97dcb14;2019-10-24;SYSTEM;Embargo violation: review shipment proceedure and stop delivery until further notice;SFSF Employee with link possible?;01.03.2020
```



## Unit 3: Serving User Interfaces in CAP

### Lesson 1: Serving User Interfaces in CAP 15 Minutes
No code

### Lesson 2: Exercise 2: Generating a User interface 25 Minutes
No code



## Unit 4: Adding Custom Business Logic

### Lesson 1: Explaining Event Handling in CAP 15 Minutes
No code

### Lesson 2: Explaining the Need for Custom Business Logic 20 Minutes
No code

### Lesson 3: Describing Error Handling 15 Minutes

```cds
const cds = require('@sap/cds');

module.exports = cds.service.impl(function (srv) {
    this.before('READ', `Risks`, async (req) => {
        console.log("Before read risks")
    });

})
```


### Lesson 4: Exercise 3: Adding Custom Business Logic 20 Minutes

#### Task 1 - Point d - srv/risk-service.js
```js
// Import the cds facade object (https://cap.cloud.sap/docs/node.js/cds-facade)
const cds = require('@sap/cds')

// The service implementation with all service handlers
module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartner entities from the risk - service.cds file
    const { Risks, BusinessPartners } = this.entities;
    // This handler will be executed directly AFTER a READ operation on RISKS
    // With this we can loop through the received data set and manipulate the single risk entries
    this.after("READ", Risks, (data) => {
        // Convert to array, if it's only a single risk, so that the code won't break here
        const risks = Array.isArray(data) ? data : [data];
        // Looping through the array of risks to set the virtual field 'criticality' that you defined in the schema
        risks.forEach((risk) => {
            if (risk.impact >= 100000) {
                risk.criticality = 1;
            } else {
                risk.criticality = 2;
            }
            // set criticality for priority
            switch (risk.prio_code) {
                case 'H':
                    risk.PrioCriticality = 1;
                    break;
                case 'M':
                    risk.PrioCriticality = 2;
                    break;
                case 'L':
                    risk.PrioCriticality = 3;
                    break;
                default:
                    break;
            }
        })
    })
});

```



## Unit 5: Consuming External Services

### Lesson 1: Explaining Extensibility and Connectivity in CAP 15 Minutes
No code

### Lesson 2: Exercise 4: Adding an External Service 15 Minutes

#### Task 1 - Point b - db/schema.cds
```cds
namespace riskmanagement;

using {
    managed,
    cuid,
    User,
    sap.common.CodeList
} from '@sap/cds/common';

entity Risks : cuid, managed {
    title                   : String(100);
    owner                   : String;
    prio                    : Association to Priority;
    descr                   : String;
    miti                    : Association to Mitigations;
    impact                  : Integer;
    // bp : Association to BusinessPartners;
    virtual criticality     : Integer;
    virtual PrioCriticality : Integer;
}

entity Mitigations : cuid, managed {
    descr    : String;
    owner    : String;
    timeline : String;
    risks    : Association to many Risks
                   on risks.miti = $self;
}

entity Priority : CodeList {
    key code : String enum {
            high   = 'H';
            medium = 'M';
            low    = 'L';
        };
}

// using an external service from SAP S/4HANA Cloud
using {API_BUSINESS_PARTNER as external} from '../srv/external/API_BUSINESS_PARTNER.csn';

entity BusinessPartners as
    projection on external.A_BusinessPartner {
        key BusinessPartner,
            BusinessPartnerFullName as FullName,
    }

```


#### Task 1 - Point c - db/schema.cds
```cds
namespace riskmanagement;

using {
    managed,
    cuid,
    User,
    sap.common.CodeList
} from '@sap/cds/common';

entity Risks : cuid, managed {
    title                   : String(100);
    owner                   : String;
    prio                    : Association to Priority;
    descr                   : String;
    miti                    : Association to Mitigations;
    impact                  : Integer;
    bp                      : Association to BusinessPartners;
    virtual criticality     : Integer;
    virtual PrioCriticality : Integer;
}

entity Mitigations : cuid, managed {
    descr    : String;
    owner    : String;
    timeline : String;
    risks    : Association to many Risks
                   on risks.miti = $self;
}

entity Priority : CodeList {
    key code : String enum {
            high   = 'H';
            medium = 'M';
            low    = 'L';
        };
}

// using an external service from SAP S/4HANA Cloud
using {API_BUSINESS_PARTNER as external} from '../srv/external/API_BUSINESS_PARTNER.csn';

entity BusinessPartners as
    projection on external.A_BusinessPartner {
        key BusinessPartner,
            BusinessPartnerFullName as FullName,
    }

```


#### Task 1 - Point d - srv/risk-service.cds
```cds
using {riskmanagement as rm} from '../db/schema';

@path: 'service/risk'
service RiskService {
    entity Risks            as projection on rm.Risks;
    annotate Risks with @odata.draft.enabled;
    entity Mitigations      as projection on rm.Mitigations;
    annotate Mitigations with @odata.draft.enabled;

    // BusinessPartner will be used later
    @readonly
    entity BusinessPartners as projection on rm.BusinessPartners;
}
```

#### Task 2 - Point f - .env
```
apikey=<YOUR-API-KEY>
```


#### Task 2 - Point g - package.json
```json
..........

  "cds": {
    "requires": {
      "API_BUSINESS_PARTNER": {
        "kind": "odata-v2",
        "model": "srv\\external\\API_BUSINESS_PARTNER",
        "credentials": {
			"url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER/"
		  }
      }
    }
  },

```

#### Task 2 - Point h - srv/risk-service.js
```js
// Import the cds facade object (https://cap.cloud.sap/docs/node.js/cds-facade)
const cds = require('@sap/cds')

// The service implementation with all service handlers
module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartner entities from the risk - service.cds file
    const { Risks, BusinessPartners } = this.entities;
    // This handler will be executed directly AFTER a READ operation on RISKS
    // With this we can loop through the received data set and manipulate the single risk entries
    this.after("READ", Risks, (data) => {
        // Convert to array, if it's only a single risk, so that the code won't break here
        const risks = Array.isArray(data) ? data : [data];
        // Looping through the array of risks to set the virtual field 'criticality' that you defined in the schema
        risks.forEach((risk) => {
            if (risk.impact >= 100000) {
                risk.criticality = 1;
            } else {
                risk.criticality = 2;
            }
            // set criticality for priority
            switch (risk.prio_code) {
                case 'H':
                    risk.PrioCriticality = 1;
                    break;
                case 'M':
                    risk.PrioCriticality = 2;
                    break;
                case 'L':
                    risk.PrioCriticality = 3;
                    break;
                default:
                    break;
            }
        })
    })

    //### BEGIN OF INSERT
    // connect to remote service
    const BPsrv = await cds.connect.to("API_BUSINESS_PARTNER");
    /** 
     * Event-handler for read-events on the BusinessPartners entity.
     * Each request to the API Business Hub requires the apikey in the header. 
     */
    this.on("READ", BusinessPartners, async (req) => {
        // The API Sandbox returns alot of business partners with empty names.
        // We don't want them in our application
        req.query.where("LastName <> '' and FirstName <> '' ");
        return await BPsrv.transaction(req).send({
            query: req.query,
            headers: {
                apikey: process.env.apikey,
            },
        });
    });
    //### END OF INSERT 
});

```


#### Task 3 - Point b - db/data/riskmanagement-Risks.csv
```csv
ID;createdAt;createdBy;title;owner;prio_code;descr;miti_ID;impact;bp_BusinessPartner
20466922-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;CFR non-compliance;Fred Fish;H;Recent restructuring might violate CFR code 71;20466921-7d57-4e76-b14c-e53fd97dcb11;10000;1000060
20466922-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;SLA violation with possible termination cause;George Gung;M;Repeated SAL violation on service delivery for two successive quarters;20466921-7d57-4e76-b14c-e53fd97dcb12;90000;9980002245
20466922-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Shipment violating export control;Herbert Hunter;L;Violation of export and trade control with unauthorized downloads;20466921-7d57-4e76-b14c-e53fd97dcb13;200000;9980000230
```

#### Task 5 - Point b - srv/risk-service.js
```
// Import the cds facade object (https://cap.cloud.sap/docs/node.js/cds-facade)
const cds = require('@sap/cds')

// The service implementation with all service handlers
module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartner entities from the risk - service.cds file
    const { Risks, BusinessPartners } = this.entities;
    // This handler will be executed directly AFTER a READ operation on RISKS
    // With this we can loop through the received data set and manipulate the single risk entries
    this.after("READ", Risks, (data) => {
        // Convert to array, if it's only a single risk, so that the code won't break here
        const risks = Array.isArray(data) ? data : [data];
        // Looping through the array of risks to set the virtual field 'criticality' that you defined in the schema
        risks.forEach((risk) => {
            if (risk.impact >= 100000) {
                risk.criticality = 1;
            } else {
                risk.criticality = 2;
            }
            // set criticality for priority
            switch (risk.prio_code) {
                case 'H':
                    risk.PrioCriticality = 1;
                    break;
                case 'M':
                    risk.PrioCriticality = 2;
                    break;
                case 'L':
                    risk.PrioCriticality = 3;
                    break;
                default:
                    break;
            }
        })
    })

    // connect to remote service
    const BPsrv = await cds.connect.to("API_BUSINESS_PARTNER");
    /** 
     * Event-handler for read-events on the BusinessPartners entity.
     * Each request to the API Business Hub requires the apikey in the header. 
     */
    this.on("READ", BusinessPartners, async (req) => {
        // The API Sandbox returns alot of business partners with empty names.
        // We don't want them in our application
        req.query.where("LastName <> '' and FirstName <> '' ");
        return await BPsrv.transaction(req).send({
            query: req.query,
            headers: {
                apikey: process.env.apikey,
            },
        });
    });

    //### BEGIN OF INSERT
    // Risks?$expand=bp (Expand on BusinessPartner)
    this.on("READ", Risks, async (req, next) => {
        /*
        Check whether the request wants an "expand" of the business partner
        As this is not possible, the risk entity and the business
        partner entity are in different systems (SAP BTP and S/4 HANA Cloud),
        if there is such an expand, remove it
        */
        if (!req.query.SELECT.columns) return next();
        const expandIndex = req.query.SELECT.columns.findIndex(
            ({ expand, ref }) => expand && ref[0] === "bp"
        );
        if (expandIndex < 0) return next();
        // Remove expand from query
        req.query.SELECT.columns.splice(expandIndex, 1);
        // Make sure bp_BusinessPartner (ID) will be returned
        if (!req.query.SELECT.columns.find((column) => column.ref.find((ref) => ref == "bp_BusinessPartner"))) {
            req.query.SELECT.columns.push({
                ref:
                    ["bp_BusinessPartner"]
            });
        }
        const risks = await next();
        const asArray = x => Array.isArray(x) ? x : [x];
        // Request all associated BusinessPartners
        const bpIDs = asArray(risks).map(risk => risk.bp_BusinessPartner);
        const busienssPartners = await BPsrv.transaction(req).send({
            query:
                SELECT.from(this.entities.BusinessPartners).where({
                    BusinessPartner:
                        bpIDs
                }),
            headers: {
                apikey: process.env.apikey,
            }
        });
        // Convert in a map for easier lookup
        const bpMap = {};
        for (const businessPartner of busienssPartners)
            bpMap[businessPartner.BusinessPartner] = businessPartner;
        // Add BusinessPartners to result
        for (const note of asArray(risks)) {
            note.bp = bpMap[note.bp_BusinessPartner];
        }
        return risks;
    });
    //### END OF INSERT

});
```



## Unit 6: Understanding Authorization and Trust Management

### Lesson 1: Describing Authorization and Trust Management (XSUAA) 20 Minutes
No code

### Lesson 2: Exercise 5: Defining CDS Restrictions and Roles 20 Minutes

#### Task 1 - Point b - srv/risk-service.cds
```
using {riskmanagement as rm} from '../db/schema';

@path: 'service/risk'
service RiskService @(requires: 'authenticated-user') {
    entity Risks @(restrict: [
        {
            grant: 'READ',
            to   : 'RiskViewer'
        },
        {
            grant: [
                'READ',
                'WRITE',
                'UPDATE',
                'UPSERT',
                'DELETE'
            ],
            to   : 'RiskManager'
        }
    ])                      as projection on rm.Risks;

    annotate Risks with @odata.draft.enabled;

    entity Mitigations @(restrict: [
        {
            grant: 'READ',
            to   : 'RiskViewer'
        },
        {
            grant: '*',
            to   : 'RiskManager'
        }
    ])                      as projection on rm.Mitigations;

    annotate Mitigations with @odata.draft.enabled;

    // BusinessPartner
    @readonly
    entity BusinessPartners as projection on rm.BusinessPartners;
}
```

#### Task 2 - Point b - .cdsrc.json
```json
{
    "[development]": {
        "auth": {
            "passport": {
                "strategy": "mock",
                "users": {
                    "risk.viewer@tester.sap.com": {
                        "password": "initial",
                        "ID": "riskviewer",
                        "userAttributes": {
                            "email": "risk.viewer@tester.sap.com"
                        },
                        "roles": [
                            "RiskViewer"
                        ]
                    },
                    "risk.manager@tester.sap.com": {
                        "password": "initial",
                        "ID": "riskmanager",
                        "userAttributes": {
                            "email": "risk.manager@tester.sap.com"
                        },
                        "roles": [
                            "RiskManager"
                        ]
                    }
                }
            }
        }
    }
}
```



## Unit 7: Deploying the Application

### Lesson 1: Identifying Deployment Options in CAP 10 Minutes
No code

### Lesson 2: Explaining the Deployment Process 17 Minutes
No code

### Lesson 3: Using the Cloud Foundry CLI 10 Minutes
No code

### Lesson 4: Exercise 6: Preparing the Application for Deployment 20 Minutes

#### Task 1 - Points a,b,c,d
```bash
cds add hana --for production
cds add xsuaa --for production
cds compile srv/ --to xsuaa > xs-security.json
npm i
cds add mta
```


#### Task 1 - Point e - Add the managed approuter
```bash
cds add approuter --for production
```

#### Task 1 - Point g - app/xs-app.json
```json
{
  "welcomeFile": "app/launchpad.html",
  "routes": [
    {
      "source": "^/app/(.*)$",
      "target": "$1",
      "localDir": ".",
      "cacheControl": "no-cache, no-store, must-revalidate"
    },
    {
      "source": "^/appconfig/",
      "localDir": ".",
      "cacheControl": "no-cache, no-store, must-revalidate"
    },
    {
      "source": "^/(.*)$",
      "target": "$1",
      "destination": "srv-api",
      "csrfProtection": true
    }
  ]
}

```

#### Task 1 - Point h - Freeze dependencies
```bash
npm update --package-lock-only
```

#### Task 1 - (If Required) - mta.yaml
```yaml
.........
- name: risk-management-auth
  type: org.cloudfoundry.managed-service
  parameters:
    service: xsuaa
    service-plan: application
    path: ./xs-security.json
    config:
      xsappname: risk-management-${org}-${space}
      tenant-mode: dedicated
      oauth2-configuration:
        redirect-uris:           
        - https://*.cfapps.eu10-004.hana.ondemand.com/login/callback
```


### Lesson 5: Exercise 7: Performing a Manual Deployment 20 Minutes


#### Task 1 - Point i - Build and deploy
```bash
mbt build -t gen --mtar mta.tar
```

#### Task 1 - Point j - Build and deploy
First you have to connect:
```bash
cf api https://api.cf.eu10-004.hana.ondemand.com
cf login --sso
```

Then
```bash
cf deploy gen/mta.tar
```

#### Task 1 - Point a - Recompile your service definition
```bash
cds compile srv/ --to xsuaa > xs-security.json
```

#### Task 1 - Point b - Define the role collections in the mta.yaml
```
...........
      config:
        xsappname: risk-management-${org}-${space}
        tenant-mode: dedicated
        role-collections:
        - name: 'RiskManager-${space}'
          description: Manage Risks
          role-template-references:
          - $XSAPPNAME.RiskManager
        - name: 'RiskViewer-${space}'
          description: View Risks
          role-template-references:
          - $XSAPPNAME.RiskViewer
        oauth2-configuration:
          redirect-uris:
            - https://*.cfapps.eu10-004.hana.ondemand.com/login/callback
```

#### Task 1 - Point a - Build and deploy
```
mbt build -t gen --mtar mta.tar
```

#### Task 1 - Point b - Build and deploy
```
cf deploy gen/mta.tar
```



## Unit 8: Performing Automated Deployment (SAP Continuous Integration and Delivery)

### Lesson 1: Describing Continuous Integration and Delivery
No code

### Lesson 2: Exercise 8: Creating and Connecting a Remote Git-Repository 20 Minutes

#### Task 4 - Points a,b,c,d - Build and deploy
```
git add .
git commit -m "Initial Commit"
git remote set-url origin https://github.com/simfer/rm21_leonardo.git
git push -u origin main
```

### Lesson 3: Exercise 9: Enabling SAP Continuous Integration and Delivery 10 Minutes
No code

### Lesson 4: Exercise 10: Configuring a SAP Continuous Integration and Delivery Job
No code

### Lesson 5: Exercise 11: Verifying the Build Success 10 Minutes
No code


## Appendix 1:

Add an environment variable directly in the mta.yaml:

```

```
 

## Appendix 2:

Full annotations.cds file

```cds
using RiskService as service from '../../srv/risk-service';

annotate service.Risks with @(
    UI.LineItem : [
        {
            $Type : 'UI.DataField',
            Label : '{i18n>Title}',
            Value : title,
        },
        {
            $Type : 'UI.DataField',
            Value : miti.descr,
            Label : '{i18n>Mitigation}',
        },
        {
            $Type : 'UI.DataField',
            Label : '{i18n>Owner}',
            Value : owner,
        },
        {
            $Type : 'UI.DataField',
            Label : '{i18n>Priority}',
            Value : prio_code,
            Criticality : PrioCriticality,
        },
        {
            $Type : 'UI.DataField',
            Label : '{i18n>Impact}',
            Value : impact,
            Criticality : criticality,
        },
        {
            $Type : 'UI.DataFieldForAnnotation',
            Target : 'bp/@Communication.Contact#contact',
            Label : '{i18n>BusinessPartner}',
        },
    ]
);
annotate service.Risks with {
    miti @Common.ValueList : {
        $Type : 'Common.ValueListType',
        CollectionPath : 'Mitigations',
        Parameters : [
            {
                $Type : 'Common.ValueListParameterInOut',
                LocalDataProperty : miti_ID,
                ValueListProperty : 'ID',
            },
            {
                $Type : 'Common.ValueListParameterDisplayOnly',
                ValueListProperty : 'descr',
            },
            {
                $Type : 'Common.ValueListParameterDisplayOnly',
                ValueListProperty : 'owner',
            },
            {
                $Type : 'Common.ValueListParameterDisplayOnly',
                ValueListProperty : 'timeline',
            },
        ],
        Label : '{i18n>Mitigation}',
    }
};
annotate service.Risks with @(
    UI.FieldGroup #GeneratedGroup1 : {
        $Type : 'UI.FieldGroupType',
        Data : [
            {
                $Type : 'UI.DataField',
                Label : 'title',
                Value : title,
            },
            {
                $Type : 'UI.DataField',
                Label : 'owner',
                Value : owner,
            },
            {
                $Type : 'UI.DataField',
                Label : 'prio_code',
                Value : prio_code,
            },
            {
                $Type : 'UI.DataField',
                Label : 'descr',
                Value : descr,
            },
            {
                $Type : 'UI.DataField',
                Label : 'impact',
                Value : impact,
            },
            {
                $Type : 'UI.DataField',
                Label : 'criticality',
                Value : criticality,
            },
            {
                $Type : 'UI.DataField',
                Label : 'PrioCriticality',
                Value : PrioCriticality,
            },
        ],
    },
    UI.Facets : [
        {
            $Type : 'UI.CollectionFacet',
            Label : 'Risk Overview',
            ID : 'RiskOverview',
            Facets : [
                {
                    $Type : 'UI.ReferenceFacet',
                    Label : 'Risk Details',
                    ID : 'RiskDetails',
                    Target : '@UI.FieldGroup#RiskDetails1',
                },],
        },
        {
            $Type : 'UI.CollectionFacet',
            Label : '{i18n>MitigationOverview}',
            ID : 'MitigationOverview',
            Facets : [
                {
                    $Type : 'UI.CollectionFacet',
                    Label : '{i18n>Mitigation}',
                    ID : 'i18nMitigation',
                    Facets : [
                        {
                            $Type : 'UI.ReferenceFacet',
                            Label : '{i18n>MitigationDetails}',
                            ID : 'i18nMitigationDetails',
                            Target : '@UI.FieldGroup#i18nMitigationDetails',
                        },],
                },],
        },]
);
annotate service.Risks with @(
    UI.SelectionFields : [
        prio_code,
    ]
);
annotate service.Risks with {
    prio @Common.Label : '{i18n>Priority}'
};
annotate service.Risks with @(
    UI.HeaderInfo : {
        Title : {
            $Type : 'UI.DataField',
            Value : title,
        },
        TypeName : '',
        TypeNamePlural : '',
        Description : {
            $Type : 'UI.DataField',
            Value : descr,
        },
        TypeImageUrl : 'sap-icon://alert',
    }
);
annotate service.Risks with @(
    UI.FieldGroup #RiskDetails : {
        $Type : 'UI.FieldGroupType',
        Data : [
        ],
    }
);
annotate service.Risks with @(
    UI.FieldGroup #RiskDetails1 : {
        $Type : 'UI.FieldGroupType',
        Data : [
            {
                $Type : 'UI.DataField',
                Value : title,
                Label : '{i18n>Title}',
            },{
                $Type : 'UI.DataField',
                Value : owner,
                Label : '{i18n>Owner}',
            },{
                $Type : 'UI.DataField',
                Value : descr,
                Label : '{i18n>Description}',
            },{
                $Type : 'UI.DataField',
                Value : prio_code,
                Criticality : PrioCriticality,
            },
            {
                $Type : 'UI.DataField',
                Value : impact,
                Label : '{i18n>Impact}',
                Criticality : criticality,
            },
            {
                $Type : 'UI.DataFieldForAnnotation',
                Target : 'bp/@Communication.Contact#contact1',
                Label : '{i18n>BusinessPartner}',
            },],
    }
);
annotate service.Risks with @(
    UI.FieldGroup #MitigationDetails : {
        $Type : 'UI.FieldGroupType',
        Data : [
            {
                $Type : 'UI.DataField',
                Value : miti.ID,
                Label : '{i18n>Mitigation}',
            },{
                $Type : 'UI.DataField',
                Value : miti.owner,
                Label : '{i18n>Owner}',
            },{
                $Type : 'UI.DataField',
                Value : miti.timeline,
                Label : '{i18n>Timeline}',
            },],
    }
);
annotate service.Mitigations with {
    ID @Common.Text : descr
};
annotate service.Mitigations with {
    ID @(Common.ValueList : {
            $Type : 'Common.ValueListType',
            CollectionPath : 'Mitigations',
            Parameters : [
                {
                    $Type : 'Common.ValueListParameterInOut',
                    LocalDataProperty : ID,
                    ValueListProperty : 'ID',
                },
            ],
            Label : '{i18n>Mitigation}',
        },
        Common.ValueListWithFixedValues : true
)};
annotate service.Mitigations with {
    owner @Common.FieldControl : #ReadOnly
};
annotate service.Mitigations with {
    timeline @Common.FieldControl : #ReadOnly
};
annotate service.Risks with @(
    UI.FieldGroup #i18nMitigationDetails : {
        $Type : 'UI.FieldGroupType',
        Data : [
            {
                $Type : 'UI.DataField',
                Value : miti_ID,
                Label : '{i18n>Mitigation}',
            },{
                $Type : 'UI.DataField',
                Value : miti.owner,
                Label : '{i18n>Owner}',
            },{
                $Type : 'UI.DataField',
                Value : miti.timeline,
                Label : '{i18n>Timeline}',
            },],
    }
);
annotate service.Risks with {
    miti @Common.Text : {
            $value : miti.descr,
            ![@UI.TextArrangement] : #TextOnly,
        }
};
annotate service.Risks with {
    miti @Common.ValueListWithFixedValues : true
};
annotate service.Risks with {
    prio @Common.Text : {
            $value : prio.descr,
            ![@UI.TextArrangement] : #TextOnly,
        }
};
annotate service.BusinessPartners with @(
    Communication.Contact #contact : {
        $Type : 'Communication.ContactType',
        fn : FullName,
    }
);
annotate service.BusinessPartners with @(
    Communication.Contact #contact1 : {
        $Type : 'Communication.ContactType',
        fn : FullName,
    }
);
```

## Appendix 3

Full i18n.properties file:

```

#XFLD,120: Label for a filter field
Priority=Priority

#XFLD,120: Label for a column title
Title=Title

#XFLD,120: Label for a column title
Owner=Owner

#XFLD,120: Label for a column title
Impact=Impact

#XFLD,120: Label for a column title
Mitigation=Mitigation

#XFLD,120: Label for a field
Timeline=Timeline

#XFLD,50: Label for a section
MitigationOverview=Mitigation Overview

#XFLD,50: Label for a section
MitigationDetails=Mitigation Details

#XFLD,120: Label for a column title
BusinessPartner=Business Partner

```
