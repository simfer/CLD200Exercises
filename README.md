# CLD200_21 - Leonardo

## Solution 1 - Create a Dev Space for Business Applications
No Code

## Solution 2 - Create a CAP-Based Service


#### Task 1 - Point b - Clone the project
```bash
mkdir ~/projects/risk-management
cd ~/projects/risk-management/ 
git clone https://github.com/SAP-samples/btp-side-by-side-extension-learning-journey ./
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
using { managed } from '@sap/cds/common'; 

entity Risks : managed { 
	key ID : UUID @(Core.Computed : true); 
	title : String(100); 
	owner : String; 
	prio : String(5); 
	descr : String; 
	miti : Association to Mitigations; 
	impact : Integer; 
	//bp : Association to BusinessPartners; 
	// You will need this definition in a later step 
	criticality : Integer; 
}

entity Mitigations : managed { 
	key ID : UUID @(Core.Computed : true); 
	descr : String; 
	owner : String; 
	timeline : String; 
	risks : Association to many Risks on risks.miti = $self; 
}
```

#### Task 3 - Point d - srv/risk-service.cds
```
using { riskmanagement as rm } from '../db/schema'; 
@path: 'service/risk' 
service RiskService { 
	entity Risks as projection on rm.Risks; 
	annotate Risks with @odata.draft.enabled; 
	entity Mitigations as projection on rm.Mitigations; 
	annotate Mitigations with @odata.draft.enabled; 
	//@readonly entity BusinessPartners as projection on rm.BusinessPartners; 
}
```

NOTE: createdAt and modifiedBy are automatically addedd

#### Task 4 - Point b - db/data/riskmanagement-Risks.csv

> Just take a look at this file: you don't have to change anything

```csv
ID;createdAt;createdBy;title;owner;prio;descr;miti_id;impact;
20466922-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;CFR non-compliance;Fred Fish;3;Recent restructuring might violate CFR code 71;
20466921-7d57-4e76-b14c-e53fd97dcb11;10000;20466922-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;SLA violation with possible termination cause;George Gung;2;Repeated SAL violation on service delivery for two successive quarters;
20466921-7d57-4e76-b14c-e53fd97dcb12;90000; 20466922-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Shipment violating export control;Herbert Hunter;1;Violation of export and trade control with unauthorized downloads;20466921-7d57-4e76-b14c-e53fd97dcb13;200000;
```

#### Task 4 - Point d - db/data/riskmanagement-Mitigations.csv

> Just take a look at this file: you don't have to change anything

```csv
ID;createdAt;createdBy;descr;owner;timeline 
20466921-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;SLA violation: authorize account manager to offer service credits for recent delivery issues;suitable BuPa;Q2 2020
20466921-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;"SLA violation: review third party contractors to ease service delivery challenges; trigger budget review";suitable BuPa;Q3 2020
20466921-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Embargo violation: investigate source of shipment request, revoke authorization;SFSF Employee with link possible?;29.03.2020
20466921-7d57-4e76-b14c-e53fd97dcb14;2019-10-24;SYSTEM;Embargo violation: review shipment proceedure and stop delivery until further notice;SFSF Employee with link possible?;01.03.2020
```

## Solution 3 - Generate a User Interface Using SAP Fiori Elements

#### Task 2 - Point d - app/common.cds
```cds
using riskmanagement as rm from '../db/schema';

// Annotate Risk elements
annotate rm.Risks with {
    ID     @title : 'Risk';
    title  @title : 'Title';
    owner  @title : 'Owner';
    prio   @title : 'Priority';
    descr  @title : 'Description';
    miti   @title : 'Mitigation';
    impact @title : 'Impact';
}

// Annotate Miti elements
annotate rm.Mitigations with {
    ID    @(
        UI.Hidden,
        Common : {Text : descr}
    );
    owner @title : 'Owner';
    descr @title : 'Description';
}

annotate rm.Risks with {
    miti @(Common : {
        //show text, not id for mitigation in the context of risks
        Text            : miti.descr,
        TextArrangement : #TextOnly,
        ValueList       : {
            Label          : 'Mitigations',
            CollectionPath : 'Mitigations',
            Parameters     : [
                {
                    $Type             : 'Common.ValueListParameterInOut',
                    LocalDataProperty : miti_ID,
                    ValueListProperty : 'ID'
                },
                {
                    $Type             : 'Common.ValueListParameterDisplayOnly',
                    ValueListProperty : 'descr'
                }
            ]
        }
    });
}

```

#### Task 2 - Point e - app/service.cds
```
using from './risks/annotations';
using from './common';
```

#### Task 2 - Point f - app/risks/annotations.cds
```
using RiskService from '../../srv/risk-service';

// Risk List Report Page
annotate RiskService.Risks with @(UI : {
    HeaderInfo      : {
        TypeName       : 'Risk',
        TypeNamePlural : 'Risks',
        Title          : {
            $Type : 'UI.DataField',
            Value : title
        },
        Description    : {
            $Type : 'UI.DataField',
            Value : descr
        }
    },
    SelectionFields : [prio],
    Identification  : [{Value : title}],
    // Define the table columns
    LineItem        : [
        {Value : title},
        {Value : miti_ID},
        {Value : owner},
        {
            Value       : prio,
            Criticality : criticality
        },
        {
            Value       : impact,
            Criticality : criticality
        },
    ],
});

// Risk Object Page
annotate RiskService.Risks with @(UI : {
    Facets           : [{
        $Type  : 'UI.ReferenceFacet',
        Label  : 'Main',
        Target : '@UI.FieldGroup#Main',
    }],
    FieldGroup #Main : {Data : [
        {Value : miti_ID},
        {Value : owner},
        {
            Value       : prio,
            Criticality : criticality
        },
        {
            Value       : impact,
            Criticality : criticality
        }
    ]},
});
```

## Solution 4 - Add Custom Business Logic to Your Application

#### Task 1 - Point d - srv/risk-service.js
```js
// Imports
const cds = require("@sap/cds");

/** 
 * The service implementation with all service handlers 
 **/

module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartners entities from the risk-service.cds file
    const { Risks, BusinessPartners } = this.entities;

    /** 
     * Set criticality after a READ operation on /risks
     **/

    this.after("READ", Risks, (data) => {
        const risks = Array.isArray(data) ? data : [data];
        risks.forEach((risk) => {
            if (risk.impact >= 100000) {
                risk.criticality = 1;
            } else {
                risk.criticality = 2;
            }
        });
    });
});
```

## Solution 5 - Add an External Service

#### Task 1 - Point b - db/schema.cds
```cds
namespace riskmanagement;

using {managed} from '@sap/cds/common';

entity Risks : managed {
    key ID          : UUID @(Core.Computed : true);
        title       : String(100);
        owner       : String;
        prio        : String(5);
        descr       : String;
        miti        : Association to Mitigations;
        impact      : Integer;
        //bp : Association to BusinessPartners;
        // You will need this definition in a later step
        criticality : Integer;
}

entity Mitigations : managed {
    key ID       : UUID @(Core.Computed : true);
        descr    : String;
        owner    : String;
        timeline : String;
        risks    : Association to many Risks
                       on risks.miti = $self;
}

// using an external service from S/4
using {API_BUSINESS_PARTNER as external} from '../srv/external/API_BUSINESS_PARTNER.csn';

entity BusinessPartners as projection on external.A_BusinessPartner {
    key BusinessPartner,
        LastName,
        FirstName
}

```

#### Task 1 - Point d - srv/risk-service.cds
```cds
using {riskmanagement as rm} from '../db/schema';

@path : 'service/risk'
service RiskService {
    entity Risks            as projection on rm.Risks;
    annotate Risks with @odata.draft.enabled;
    entity Mitigations      as projection on rm.Mitigations;
    annotate Mitigations with @odata.draft.enabled;

    @readonly
    entity BusinessPartners as projection on rm.BusinessPartners; // <--- uncomment this line
}
```

#### Task 2 - Point f - .env
```
apikey=<YOUR-API-KEY>
```

#### Task 2 - Point g - .gitignore
```
........
.env
........
```

#### Task 2 - Point h - package.json
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

#### Task 2 - Point i - srv/risk-service.js
```js
// Imports
const cds = require("@sap/cds");

/** 
 * The service implementation with all service handlers 
 **/

module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartners entities from the risk-service.cds file
    const { Risks, BusinessPartners } = this.entities;

    /** 
     * Set criticality after a READ operation on /risks
     **/

    this.after("READ", Risks, (data) => {
        const risks = Array.isArray(data) ? data : [data];
        risks.forEach((risk) => {
            if (risk.impact >= 100000) {
                risk.criticality = 1;
            } else {
                risk.criticality = 2;
            }
        });
    });
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

#### Task 3 - Point b - db/schema.cds
```
namespace riskmanagement;

using {managed} from '@sap/cds/common';

entity Risks : managed {
    key ID          : UUID @(Core.Computed : true);
        title       : String(100);
        owner       : String;
        prio        : String(5);
        descr       : String;
        miti        : Association to Mitigations;
        impact      : Integer;
        bp          : Association to BusinessPartners; // We have uncommented this line
        criticality : Integer;
}

entity Mitigations : managed {
    key ID       : UUID @(Core.Computed : true);
        descr    : String;
        owner    : String;
        timeline : String;
        risks    : Association to many Risks
                       on risks.miti = $self;
}

// using an external service from S/4
using {API_BUSINESS_PARTNER as external} from '../srv/external/API_BUSINESS_PARTNER.csn';

entity BusinessPartners as projection on external.A_BusinessPartner {
    key BusinessPartner,
        LastName,
        FirstName
}
```

#### Task 2 - Point d - db/data/riskmanagement-Risks.csv
```csv
ID;createdAt;createdBy;title;owner;prio;descr;miti_id;impact;bp_BusinessPartner
20466922-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;CFR non-compliance;Fred Fish;3;Recent restructuring might violate CFR code 71;20466921-7d57-4e76-b14c-e53fd97dcb11;10000;9980000448
20466922-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;SLA violation with possible termination cause;George Gung;2;Repeated SAL violation on service delivery for two successive quarters;20466921-7d57-4e76-b14c-e53fd97dcb12;90000;9980002245
20466922-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Shipment violating export control;Herbert Hunter;1;Violation of export and trade control with unauthorized downloads;20466921-7d57-4e76-b14c-e53fd97dcb13;200000;9980000230
```

#### Task 4 - Point b - app/common.cds
```
using riskmanagement as rm from '../db/schema';

// Annotate Risk elements
annotate rm.Risks with {
    ID     @title : 'Risk';
    title  @title : 'Title';
    owner  @title : 'Owner';
    prio   @title : 'Priority';
    descr  @title : 'Description';
    miti   @title : 'Mitigation';
    impact @title : 'Impact';
    //### BEGIN OF INSERT
    bp     @title : 'Business Partner';
//### END OF INSERT
}

// Annotate Miti elements
annotate rm.Mitigations with {
    ID    @(
        UI.Hidden,
        Common : {Text : descr}
    );
    owner @title : 'Owner';
    descr @title : 'Description';
}

//### BEGIN OF INSERT
annotate rm.BusinessPartners with {
    BusinessPartner @(
        UI.Hidden,
        Common : {Text : LastName}
    );
    LastName        @title : 'Last Name';
    FirstName       @title : 'First Name';
}
//### END OF INSERT

annotate rm.Risks with {
    miti @(Common : {
        //show text, not id for mitigation in the context of risks
        Text            : miti.descr,
        TextArrangement : #TextOnly,
        ValueList       : {
            Label          : 'Mitigations',
            CollectionPath : 'Mitigations',
            Parameters     : [
                {
                    $Type             : 'Common.ValueListParameterInOut',
                    LocalDataProperty : miti_ID,
                    ValueListProperty : 'ID'
                },
                {
                    $Type             : 'Common.ValueListParameterDisplayOnly',
                    ValueListProperty : 'descr'
                }
            ]
        }
    });

    //### BEGIN OF INSERT
    bp   @(Common : {
        Text            : bp.LastName,
        TextArrangement : #TextOnly,
        ValueList       : {
            Label          : 'Business Partners',
            CollectionPath : 'BusinessPartners',
            Parameters     : [
                {
                    $Type             : 'Common.ValueListParameterInOut',
                    LocalDataProperty : bp_BusinessPartner,
                    ValueListProperty : 'BusinessPartner'
                },
                {
                    $Type             : 'Common.ValueListParameterDisplayOnly',
                    ValueListProperty : 'LastName'
                },
                {
                    $Type             : 'Common.ValueListParameterDisplayOnly',
                    ValueListProperty : 'FirstName'
                }
            ]
        }
    })
//### END OF INSERT
}
```

#### Task 4 - Point c - app/risk/annotations.cds
```
using RiskService from '../../srv/risk-service';

// Risk List Report Page
annotate RiskService.Risks with @(UI : {
    HeaderInfo      : {
        TypeName       : 'Risk',
        TypeNamePlural : 'Risks',
        Title          : {
            $Type : 'UI.DataField',
            Value : title
        },
        Description    : {
            $Type : 'UI.DataField',
            Value : descr
        }
    },
    SelectionFields : [prio],
    Identification  : [{Value : title}],
    // Define the table columns
    LineItem        : [
        {Value : title},
        {Value : miti_ID},
        {Value : owner},
        //### BEGIN OF INSERT
        {Value : bp_BusinessPartner},
        //### END OF INSERT
        {
            Value       : prio,
            Criticality : criticality
        },
        {
            Value       : impact,
            Criticality : criticality
        },
    ],
});

// Risk Object Page
annotate RiskService.Risks with @(UI : {
    Facets           : [{
        $Type  : 'UI.ReferenceFacet',
        Label  : 'Main',
        Target : '@UI.FieldGroup#Main',
    }],
    FieldGroup #Main : {Data : [
        {Value : miti_ID},
        {Value : owner},
        //### BEGIN OF INSERT
        {Value : bp_BusinessPartner},
        //### END OF INSERT
        {
            Value       : prio,
            Criticality : criticality
        },
        {
            Value       : impact,
            Criticality : criticality
        }
    ]},
});
```

#### Task 4 - Point f - srv/risk-service.js
```js
// Imports
const cds = require("@sap/cds");

/** 
 * The service implementation with all service handlers 
 **/

module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartners entities from the risk-service.cds file
    const { Risks, BusinessPartners } = this.entities;

    /** 
     * Set criticality after a READ operation on /risks
     **/

    this.after("READ", Risks, (data) => {
        const risks = Array.isArray(data) ? data : [data];
        risks.forEach((risk) => {
            if (risk.impact >= 100000) {
                risk.criticality = 1;
            } else {
                risk.criticality = 2;
            }
        });
    });
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

    /**
     * Event-handler on risks.
     * Retrieve BusinessPartner data from the external API 
     */
    this.on("READ", Risks, async (req, next) => {
        /* Check whether the request wants an "expand" of the business partner 
        As this is not possible, the risk entity and the business partner entity are in different systems (SAP BTP and S/4 HANA Cloud),
        if there is such an expand, remove it */
        if (!req.query.SELECT.columns) return next();
        const expandIndex = req.query.SELECT.columns.findIndex(({ expand, ref }) => expand && ref[0] === "bp");
        console.log(req.query.SELECT.columns);
        if (expandIndex < 0) return next();
        req.query.SELECT.columns.splice(expandIndex, 1);
        if (!req.query.SELECT.columns.find((column) => column.ref.find((ref) => ref == "bp_BusinessPartner"))) {
            req.query.SELECT.columns.push({ ref: ["bp_BusinessPartner"] });
        }
        /* Instead of carrying out the expand, issue a separate request for each business partner 
        This code could be optimized, instead of having n requests for n business partners, 
        just one bulk request could be created */
        try {
            res = await next();
            res = Array.isArray(res) ? res : [res];
            await Promise.all(
                res.map(async (risk) => {
                    const bp = await BPsrv.transaction(req)
                        .send({
                            query: SELECT.one(this.entities.BusinessPartners)
                                .where({ BusinessPartner: risk.bp_BusinessPartner })
                                .columns(["BusinessPartner", "LastName", "FirstName"]),
                            headers: { apikey: process.env.apikey, },
                        });
                    risk.bp = bp;
                }));
        }
        catch (error) { }
    });
    //### END OF INSERT
});
```

## Solution 6 - Deploy SAP BTP Cloud Foundry Applications Manually

#### Task 1 - Points a,b,c,d
```bash
cds add hana --for production
cds add xsuaa --for production
cds compile srv/ --to xsuaa > xs-security.json
cds add mta
```

#### Task 1 - Point e - mta.yaml
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

#### Task 1 - Point f - Add the managed approuter
```bash
cds add approuter --for production
```

#### Task 1 - Point g - app/xs-app.json
```json
{
  "welcomeFile": "/app/risks/webapp/index.html",
  "routes": [
    {
      "source": "^/app/(.*)$",
      "localDir": "./",
      "target": "$1"
    },
    {
      "source": "^/service/(.*)$",
      "destination": "srv-api"
    }
  ]
}
```

#### Task 1 - Point h - Freeze dependencies
```bash
npm update --package-lock-only
```

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



## Solution 7 - Define Restrictions and Roles in CDS

#### Task 1 - Point b - srv/risk-service.cds
```
using {riskmanagement as rm} from '../db/schema';

@path : 'service/risk'
service RiskService {
    entity Risks @(restrict : [
        {
            grant : ['READ'],
            to    : ['RiskViewer']
        },
        {
            grant : ['*'],
            to    : ['RiskManager']
        }
    ])                      as projection on rm.Risks;

    annotate Risks with @odata.draft.enabled;

    entity Mitigations @(restrict : [
        {
            grant : ['READ'],
            to    : ['RiskViewer']
        },
        {
            grant : ['*'],
            to    : ['RiskManager']
        }
    ])                      as projection on rm.Mitigations;

    annotate Mitigations with @odata.draft.enabled;

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

## Solution 8 - Configure XSUAA for Production

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
```

#### Task 1 - Point a - Build and deploy
```
mbt build -t gen --mtar mta.tar
```

#### Task 1 - Point b - Build and deploy
```
cf deploy gen/mta.tar
```

## Solution 9 - Assign Role Collections to an Application in BTP
No code


## Solution 10 - Create and Connect a GitHub Repository

#### Task 4 - Points a,b,c,d - Build and deploy
```
git add .
git commit -m "Initial Commit"
git remote set-url origin https://github.com/simfer/rm21_leonardo.git
git push -u origin main
```

## Solution 11 - Enable SAP Continuous Integration and Delivery
No code

## Solution 12 - Configure a CI/CD Job
No code

## Solution 13 - Verify Build Success
No code

