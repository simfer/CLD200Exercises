# Exercise 1

No code



# Exercise 2

### Task 1 - Chapter 1 - Point (b)
```
mkdir ~/projects/risk-management
cd ~/projects/risk-management/
git clone https://github.com/SAP-samples/extension-suite-learning-journey ./
```

### Task 1 - Chapter 1 - Point (g)
```
npm ci
```

### Task 1 - Chapter 1 - Point (h)
```
cds watch
```

### Task 2 - Chapter 1 - Point (d)
```
namespace riskmanagement;

using {managed} from '@sap/cds/common';

entity Risks : managed {
    key ID     : UUID @(Core.Computed : true);
        title  : String(100);
        owner  : String;
        prio   : String(5);
        descr  : String;
        miti   : Association to Mitigations;
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

### Task 3 - Chapter 1 - Point (d)
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


# Exercise 3

### Task 2 - Chapter 1 - Point (d)
```
using riskmanagement as rm from '../db/schema'; 

// Annotate Risk elements
annotate rm.Risks with { 
    ID @title : 'Risk'; 
    title @title : 'Title'; 
    owner @title : 'Owner'; 
    prio @title : 'Priority'; 
    descr @title : 'Description'; 
    miti @title : 'Mitigation'; 
    impact @title : 'Impact';
    criticality @title : 'Criticality';
    } 
    
// Annotate Miti elements 
annotate rm.Mitigations with { 
    ID @( 
        UI.Hidden, 
        Common : {Text : descr} 
    ); 
    owner @title : 'Owner';
    descr @title : 'Description'; 
}

annotate rm.Risks with { 
    miti @(Common : { 
        //show text, not id for mitigation in the context of risks
        Text : miti.descr, 
        TextArrangement : #TextOnly, 
        ValueList : { 
            Label : 'Mitigations', 
            CollectionPath : 'Mitigations', 
            Parameters : [ 
                { 
                    $Type : 'Common.ValueListParameterInOut', 
                    LocalDataProperty : miti_ID, 
                    ValueListProperty : 'ID' 
                }, { 
                    $Type : 'Common.ValueListParameterDisplayOnly', 
                    ValueListProperty : 'descr' 
                } 
            ] 
        } 
    }); 
}
```

### Task 2 - Chapter 1 - Point (e)
```
using from './risks/annotations';
using from './common';
```

### Task 2 - Chapter 1 - Point (f)
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


# Exercise 4

### Task 1 - Chapter 1 - Point (b)
```
risk-service.js
```

### Task 1 - Chapter 1 - Point (d)
```
// Imports 
const cds = require("@sap/cds");

/*
 * The service implementation with all service handlers 
 */

module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartners entities from the risk-service.cds file 
    const { Risks, BusinessPartners } = this.entities;
    /*
     * Set criticality after a READ operation on /risks 
     */
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


# Exercise 5

### Task 1 - Chapter 1 - Point (b)
```
namespace riskmanagement;

using {managed} from '@sap/cds/common';

entity Risks : managed {
    key ID     : UUID @(Core.Computed : true);
        title  : String(100);
        owner  : String;
        prio   : String(5);
        descr  : String;
        miti   : Association to Mitigations;
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

//### BEGIN OF INSERT
// using an external service from S/4
using { API_BUSINESS_PARTNER as external } from '../srv/external/API_BUSINESS_PARTNER.csn';
entity BusinessPartners as projection on external.A_BusinessPartner { 
    key BusinessPartner, 
    LastName, 
    FirstName 
} 
//### END OF OF INSERT
```

### Task 1 - Chapter 1 - Point (d) - Open the *risk-service.js* file and uncomment the BusinessPartners line
```
using { riskmanagement as rm } from '../db/schema'; 

@path: 'service/risk' 
service RiskService { 
    entity Risks as projection on rm.Risks; 
    annotate Risks with @odata.draft.enabled; 
    
    entity Mitigations as projection on rm.Mitigations; 
    annotate Mitigations with @odata.draft.enabled; 
    @readonly entity BusinessPartners as projection on rm.BusinessPartners; 
}
```

### Task 2 - Chapter 1 - Point (f) - Add the APIKey to the *.env* file
```
apikey=<YOUR-API-KEY>
```

### Task 2 - Chapter 1 - Point (h) -  Change the *packahe.json* file to include the new credentials
```
{
    "name": "risk-management",
    "version": "1.0.0",
    "description": "Template for the the SAP Extension Suite Learning Journey",
    "author": "m.haug@sap.com",
    "license": "SAP SAMPLE CODE LICENSE",
    "repository": "https://github.com/SAP-samples/sap-learning-extension-suite",
    "engines": {
        "node": ">=14"
    },
    "private": true,
    "dependencies": {
        "@sap/cds": "5.1.5",
        "@sap/cds-dk": "4.1.5",
        "express": "^4"
    },
    "devDependencies": {
        "@sap/ux-specification": "^1.96.4",
        "sqlite3": "^5.0.2"
    },
    "scripts": {
        "start": "cds run"
    },
    "sapux": [
        "app/risks"
    ],
    "cds": {
        "requires": {
            "API_BUSINESS_PARTNER": {
                "kind": "odata",
                "model": "srv/external/API_BUSINESS_PARTNER",
                "credentials": {
                    "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER/"
                }
            }
        }
    }
}
```

### Task 2 - Chapter 1 - Point (i) - Replace *risk-service.js* content with this
```
// Imports 
const cds = require("@sap/cds");

/*
 * The service implementation with all service handlers 
 */

module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartners entities from the risk-service.cds file 
    const { Risks, BusinessPartners } = this.entities;
    /*
     * Set criticality after a READ operation on /risks 
     */
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

### Task 3 - Chapter 1 - Point (b) - Uncomment the *bp* property
```
namespace riskmanagement;

using {managed} from '@sap/cds/common';

entity Risks : managed {
    key ID     : UUID @(Core.Computed : true);
        title  : String(100);
        owner  : String;
        prio   : String(5);
        descr  : String;
        miti   : Association to Mitigations;
        impact : Integer; 
        bp : Association to BusinessPartners; 
        criticality : Integer; 
}

entity Mitigations : managed { 
    key ID : UUID @(Core.Computed : true); 
    descr : String; 
    owner : String; 
    timeline : String; 
    risks : Association to many Risks on risks.miti = $self; 
}

// using an external service from S/4
using { API_BUSINESS_PARTNER as external } from '../srv/external/API_BUSINESS_PARTNER.csn';
entity BusinessPartners as projection on external.A_BusinessPartner { 
    key BusinessPartner, 
    LastName, 
    FirstName 
}
```

### Task 3 - Chapter 1 - Point (d) - Replace the content of the file riskmanagement-Risks.csv with this
```csv
ID;createdAt;createdBy;title;owner;prio;descr;miti_id;impact;bp_BusinessPartner
20466922-7d57-4e76-b14c-e53fd97dcb11;2019-10-24;SYSTEM;CFR non-compliance;Fred Fish;3;Recent restructuring might violate CFR code 71;20466921-7d57-4e76-b14c-e53fd97dcb11;10000;9980000448
20466922-7d57-4e76-b14c-e53fd97dcb12;2019-10-24;SYSTEM;SLA violation with possible termination cause;George Gung;2;Repeated SAL violation on service delivery for two successive quarters;20466921-7d57-4e76-b14c-e53fd97dcb12;90000;9980002245
20466922-7d57-4e76-b14c-e53fd97dcb13;2019-10-24;SYSTEM;Shipment violating export control;Herbert Hunter;1;Violation of export and trade control with unauthorized downloads;20466921-7d57-4e76-b14c-e53fd97dcb13;200000;9980000230
```

### Task 4 - Chapter 1 - Point (b) - In the common.cds add the parts between BEGIN and END OF INSERT
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
    bp @title : 'Business Partner';
    //### END OF INSERT
    criticality @title : 'Criticality';
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

### Task 4 - Chapter 1 - Point (c) - In the annotations.cds add the parts between BEGIN and END OF INSERT
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

### Task 4 - Chapter 1 - Point (f) - In the risk-service.js add the parts between BEGIN and END OF INSERT
```
// Imports
const cds = require("@sap/cds");

/**
 * The service implementation with all service handlers
 */
module.exports = cds.service.impl(async function () {
    // Define constants for the Risk and BusinessPartners entities from the risk-service.cds file
    const { Risks, BusinessPartners } = this.entities;

    /**
     * Set criticality after a READ operation on /risks
     */
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
        /*
            Check whether the request wants an "expand" of the business partner
            As this is not possible, the risk entity and the business partner entity are in different systems (SAP BTP and S/4 HANA Cloud),
            if there is such an expand, remove it
            */

        if (!req.query.SELECT.columns) return next();

        const expandIndex = req.query.SELECT.columns.findIndex(
            ({ expand, ref }) => expand && ref[0] === "bp"
        );
        console.log(req.query.SELECT.columns);
        if (expandIndex < 0) return next();

        req.query.SELECT.columns.splice(expandIndex, 1);
        if (
            !req.query.SELECT.columns.find((column) =>
                column.ref.find((ref) => ref == "bp_BusinessPartner")
            )
        ) {
            req.query.SELECT.columns.push({ ref: ["bp_BusinessPartner"] });
        }

        /*
            Instead of carrying out the expand, issue a separate request for each business partner
            This code could be optimized, instead of having n requests for n business partners, just one bulk request could be created
            */
        try {
            res = await next();
            res = Array.isArray(res) ? res : [res];

            await Promise.all(
                res.map(async (risk) => {
                    const bp = await BPsrv.transaction(req).send({
                        query: SELECT.one(this.entities.BusinessPartners)
                            .where({ BusinessPartner: risk.bp_BusinessPartner })
                            .columns(["BusinessPartner", "LastName", "FirstName"]),
                        headers: {
                            apikey: process.env.apikey,
                        },
                    });
                    risk.bp = bp;
                })
            );
        } catch (error) { }
    });
    //### END OF INSERT
});
```


# Exercise 6 - Deploy manually

### Task 4 - Chapter 1 - Point (b) - Generate the xs-security.json file
```
cds add hana,xsuaa,cf-manifest
```

### Task 3 - Chapter 1 - Point (l) - Generate the xs-security.json file
```
cds compile srv/ --to xsuaa > xs-security.json
```

You should obtain something like this:

```
{
    "xsappname": "risk-management-dev",
    "tenant-mode": "dedicated",
    "scopes": [],
    "attributes": [],
    "role-templates": []
  }
```


### Task 4 - Chapter 1 - Point (d) - Add the destination service resource to the risk management service
```
  - risk-management-destination-service
```

It should look like this:

```
# Generated manifest.yml based on template version 0.1.0
# appName = risk-management
# language=nodejs
# multitenancy=false
---
applications:
# -----------------------------------------------------------------------------------
# Backend Service
# -----------------------------------------------------------------------------------
- name: risk-management-srv
  random-route: true  # for development only
  path: gen/srv
  memory: 256M
  buildpack: nodejs_buildpack
  services:
  - risk-management-db
  - risk-management-xsuaa
  - risk-management-destination-service

# -----------------------------------------------------------------------------------
# HANA Database Content Deployer App
# -----------------------------------------------------------------------------------
- name: risk-management-db-deployer
  path: gen/db
  no-route: true
  health-check-type: process
  memory: 256M
  instances: 1
  buildpack: nodejs_buildpack
  services:
  - risk-management-db
```

### Task 4 - Chapter 1 - Point (e) - Edit the *package.json* file
```
{
    "name": "risk-management",
    "version": "1.0.0",
    "description": "Template for the the SAP Extension Suite Learning Journey",
    "author": "m.haug@sap.com",
    "license": "SAP SAMPLE CODE LICENSE",
    "repository": "https://github.com/SAP-samples/sap-learning-extension-suite",
    "engines": {
        "node": ">=14"
    },
    "private": true,
    "dependencies": {
        "@sap/cds": "5.1.5",
        "@sap/cds-dk": "4.1.5",
        "express": "^4",
        "hdb": "^0.18.3"
    },
    "devDependencies": {
        "@sap/ux-specification": "^1.96.4",
        "sqlite3": "^5.0.2"
    },
    "scripts": {
        "start": "cds run"
    },
    "sapux": [
        "app/risks"
    ],
    "cds": {
        "requires": {
            "API_BUSINESS_PARTNER": {
                "kind": "odata",
                "model": "srv/external/API_BUSINESS_PARTNER",
                "[development]": {
                    "credentials": {
                        "url": "https://sandbox.api.sap.com/s4hanacloud/sap/opu/odata/sap/API_BUSINESS_PARTNER/"
                    }
                },
                "[production]": {
                    "credentials": {
                        "destination": "API_BUSINESS_PARTNER"
                    }
                }
            },
            "db": {
                "kind": "sql"
            },
            "xsuaa": {
                "kind": "xsuaa"
            }
        },
        "hana": {
            "deploy-format": "hdbtable"
        }
    }
}
```

### Task 4 - Chapter 1 - Point (m) - Set the API Key
```
cf set-env risk-management-srv apikey <your-api-key>
cf env risk-management-srv
cf restart risk-management-srv
```

### Task 6 - Chapter 1 - Point (b) - Generate the *mta.yaml*
```
cds add mta
```

### Task 6 - Chapter 1 - Point (f) - Full *mta.yaml*
```
---
_schema-version: '3.1'
ID: risk-management
version: 1.0.0
description: "Template for the the SAP Extension Suite Learning Journey"
parameters:
  enable-parallel-deployments: true
build-parameters:
  before-all:
    - builder: custom
      commands:
        - npm ci
        - npx -p @sap/cds-dk cds build --production

modules:
  - name: risk-management-srv
    type: nodejs
    path: gen/srv
    parameters:
      buildpack: nodejs_buildpack
    provides:
      - name: srv-api # required by consumers of CAP services (e.g. approuter)
        properties:
          srv-url: ${default-url}
    requires:
      - name: risk-management-db
      - name: risk-management-uaa
      - name: risk-management-destination-service
  - name: risk-management-db-deployer
    type: hdb
    path: gen/db
    parameters:
      buildpack: nodejs_buildpack
    requires:
      - name: risk-management-db

resources:
  - name: risk-management-db
    type: com.sap.xs.hdi-container
    parameters:
      service: hana # or 'hanatrial' on trial landscapes
      service-plan: hdi-shared
    properties:
      hdi-service-name: ${service-name}

  - name: risk-management-uaa
    type: org.cloudfoundry.managed-service
    parameters:
      service: xsuaa
      service-plan: application
      path: ./xs-security.json
      
  - name: risk-management-destination-service
    type: org.cloudfoundry.managed-service
    parameters:
      service: destination
      service-plan: lite
```
>NOTE: Remember to remove the `npm ci` build command on the service



# Exercise 7 - Define restrictions and roles

### Task 1 - Chapter 1 - Point (b) - Install *passport*
```
npm install --save passport
```

### Task 2 - Chapter 1 - Point (b) - Replace *srv/risk-service.cds* with this content
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

### Task 3 - Chapter 1 - Point (b) - Replace *.cdsrc.json* with this content
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
              "roles": ["RiskViewer"]
            },
            "risk.manager@tester.sap.com": {
              "password": "initial",
              "ID": "riskmanager",
              "userAttributes": {
                "email": "risk.manager@tester.sap.com"
              },
              "roles": ["RiskManager"]
            }
          }
        }
      }
    }
  }
```


# Exercise 8 - Setup SAP Authorization and Trust Management

### Task 1 - Chapter 1 - Point (b) - Enable authentication support

```
npm i --save @sap/xssec @sap/xsenv
```

### Task 3 - Chapter 1 - Create the new *xs-security.json* file
```
cds compile srv --to xsuaa >xs-security.json
```

It should look like the following (you need to manually add the **xsappname** and the **tenant-mode**:
```
{
    "xsappname": "risk-management",
    "tenant-mode": "dedicated",
    "scopes": [
    {
      "name": "$XSAPPNAME.RiskViewer",
      "description": "RiskViewer"
    },
    {
      "name": "$XSAPPNAME.RiskManager",
      "description": "RiskManager"
    }
  ],
  "attributes": [],
  "role-templates": [
    {
      "name": "RiskViewer",
      "description": "generated",
      "scope-references": [
        "$XSAPPNAME.RiskViewer"
      ],
      "attribute-references": []
    },
    {
      "name": "RiskManager",
      "description": "generated",
      "scope-references": [
        "$XSAPPNAME.RiskManager"
      ],
      "attribute-references": []
    }
  ]
}
```

### Task 4 - Chapter 1 - Point (a) - This is the new *mta.yaml* file

```
## Generated mta.yaml based on template version 0.4.0
## appName = risk-management
## language=nodejs; multitenant=false
## approuter=
_schema-version: '3.1'
ID: risk-management
version: 1.0.0
description: "Template for the the SAP Extension Suite Learning Journey"
parameters:
  enable-parallel-deployments: true
   
build-parameters:
  before-all:
   - builder: custom
     commands:
      - npm install --production
      - npx -p @sap/cds-dk cds build --production

modules:
 # --------------------- SERVER MODULE ------------------------
 - name: risk-management-srv
 # ------------------------------------------------------------
   type: nodejs
   path: gen/srv
   parameters:
     buildpack: nodejs_buildpack
   requires:
    # Resources extracted from CAP configuration
    - name: risk-management-xsuaa
    - name: risk-management-db
    - name: risk-management-destination-service
   provides:
    - name: srv-api      # required by consumers of CAP services (e.g. approuter)
      properties:
        srv-url: ${default-url}

 # -------------------- SIDECAR MODULE ------------------------
 - name: risk-management-db-deployer
 # ------------------------------------------------------------
   type: hdb
   path: gen/db  
   parameters:
     buildpack: nodejs_buildpack
   requires:
    # 'hana' and 'xsuaa' resources extracted from CAP configuration
    - name: risk-management-xsuaa
    - name: risk-management-db



resources:
 # services extracted from CAP configuration
 # 'service-plan' can be configured via 'cds.requires.<name>.vcap.plan'
# ------------------------------------------------------------
 - name: risk-management-xsuaa
# ------------------------------------------------------------
   type: org.cloudfoundry.managed-service
   parameters:
     service: xsuaa
     service-plan: application  
     path: ./xs-security.json
     config:
       xsappname: 'risk-management-${space}'
       role-collections:
        - name: 'RiskManager-${space}'
          description: Manage Risks
          role-template-references:
          - $XSAPPNAME.RiskManager
        - name: 'RiskViewer-${space}'
          description: View Risks
          role-template-references:
          - $XSAPPNAME.RiskViewer
          
# ------------------------------------------------------------
 - name: risk-management-db
# ------------------------------------------------------------
   type: com.sap.xs.hdi-container
   parameters:
     service: hana  # or 'hanatrial' on trial landscapes
     service-plan: hdi-shared
   properties:
     hdi-service-name: ${service-name}

# ------------------------------------------------------------
 - name: risk-management-destination-service
# ------------------------------------------------------------
   type: org.cloudfoundry.managed-service
   parameters:
     service: destination
     service-plan: lite
```


# Exercise 9 - Create an Application Router

### Task 1 - Chapter 1 - Point (a) - Create approuter folder
```
mkdir approuter
cd approuter
```

### Task 1 - Chapter 1 - Point (b) - Install approuter
```
npm init --yes
npm install @sap/approuter
```

### Task 1 - Chapter 1 - Point (c) - Check approuter version
```
cat node_modules/@sap/approuter/package.json | grep '"node"'
```

### Task 1 - Chapter 1 - Point (d) - Add required approuter version
```
{
  "name": "approuter",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node node_modules/@sap/approuter/approuter.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@sap/approuter": "^11.0.1"
  }
}
```

### Task 2 - Chapter 1 - Point (a) - Configure approuter through *xs-app.json* file
```
{
    "welcomeFile": "/app/risks/webapp/index.html",
    "authenticationMethod": "route",
    "sessionTimeout": 30,
    "logout": {
      "logoutEndpoint": "/do/logout",
      "logoutPage": "/"
    },
    "routes": [
      {
        "source": "^/app/(.*)$",
        "target": "$1",
        "localDir": "resources",
        "authenticationType": "xsuaa"
      },
      {
        "source": "^/service/(.*)$",
        "destination": "srv-binding",
        "authenticationType": "xsuaa"
      }
    ]
  }
```


# Exercise 10 - Add Approuter to MTA

### Task 1 - Chapter 1 - Point (a) - Add the UI and the Approuter to the *mta.yaml* file
```
## Generated mta.yaml based on template version 0.4.0
## appName = risk-management
## language=nodejs; multitenant=false
## approuter=
_schema-version: '3.1'
ID: risk-management
version: 1.0.0
description: "Template for the the SAP Extension Suite Learning Journey"
parameters:
  enable-parallel-deployments: true
   
build-parameters:
  before-all:
   - builder: custom
     commands:
      - npm install --production
      - npx -p @sap/cds-dk cds build --production

modules:
 # --------------------- SERVER MODULE ------------------------
 - name: risk-management-srv
 # ------------------------------------------------------------
   type: nodejs
   path: gen/srv
   parameters:
     buildpack: nodejs_buildpack
   requires:
    # Resources extracted from CAP configuration
    - name: risk-management-xsuaa
    - name: risk-management-db
    - name: risk-management-destination-service
   provides:
    - name: srv-api      # required by consumers of CAP services (e.g. approuter)
      properties:
        srv-url: ${default-url}

 # -------------------- SIDECAR MODULE ------------------------
 - name: risk-management-db-deployer
 # ------------------------------------------------------------
   type: hdb
   path: gen/db  
   parameters:
     buildpack: nodejs_buildpack
   requires:
    # 'hana' and 'xsuaa' resources extracted from CAP configuration
    - name: risk-management-xsuaa
    - name: risk-management-db


 # --------------------  APPROUTER -----------------------------
 - name: risk-management-approuter
 # ------------------------------------------------------------
   type: nodejs
   path: approuter
   requires:
     - name: risk-management-xsuaa
     - name: srv-api
       group: destinations
       properties:
         forwardAuthToken: true
         strictSSL: true
         name: srv-binding
         url: "~{srv-url}"
   build-parameters:
     requires:
       - name: risk-management-app
         artifacts:
           - ./*
         target-path: resources

# ------------------- UI ------------------------
 - name: risk-management-app
# ------------------------------------------------------------
   type: html5
   path: app
   build-parameters:
     supported-platforms: []

resources:
 # services extracted from CAP configuration
 # 'service-plan' can be configured via 'cds.requires.<name>.vcap.plan'
# ------------------------------------------------------------
 - name: risk-management-xsuaa
# ------------------------------------------------------------
   type: org.cloudfoundry.managed-service
   parameters:
     service: xsuaa
     service-plan: application  
     path: ./xs-security.json
     config:
       xsappname: 'risk-management-${space}'
       role-collections:
        - name: 'RiskManager-${space}'
          description: Manage Risks
          role-template-references:
          - $XSAPPNAME.RiskManager
        - name: 'RiskViewer-${space}'
          description: View Risks
          role-template-references:
          - $XSAPPNAME.RiskViewer
          
# ------------------------------------------------------------
 - name: risk-management-db
# ------------------------------------------------------------
   type: com.sap.xs.hdi-container
   parameters:
     service: hana  # or 'hanatrial' on trial landscapes
     service-plan: hdi-shared
   properties:
     hdi-service-name: ${service-name}

# ------------------------------------------------------------
 - name: risk-management-destination-service
# ------------------------------------------------------------
   type: org.cloudfoundry.managed-service
   parameters:
     service: destination
     service-plan: lite
```


# Exercise 11 - Assign role collections

No code


# Exercise 12 - Create github repository

### Task 3 - Chapter 1 - Point (c) -  Enter your email address and username for Github
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

### Task 4 - Chapter 1 - Initialize Git
```
git add .
git commit -m "Push project content to GitHub"
git remote set-url origin <copied Git repository url.git>
git push -u origin main
```


# Exercise 13 - Enable SAP Continuous Integration and Delivery

No code


# Exercise 14 - Configure a CI/CD Job

No code


# Exercise 15 - Configure the stages of CI/CD Pipeline

### Task 1 - Chapter 1 - Point (b) - Create a *.pipeline/config.yml* file with this content

```
###
# This file configures the project “Piper” pipeline of your project.
# For a reference of the configuration concept and available options, please have a look into its documentation.
#
# The documentation for the most recent pipeline version can always be found at:
#    https://sap.github.io/jenkins-library/
#
# This is a YAML-file. YAML is an indentation-sensitive file format. Please make sure to properly indent changes to it.
###

### General project setup
---
general:
  pipeline: "sap-cloud-sdk"
  buildTool: "mta"
stages:
  Build:
    mavenExecuteStaticCodeChecks: false
    npmExecuteLint: false
  Additional Unit Tests:
    npmExecuteScripts: false
    karmaExecuteTests: false
  Release:
    cloudFoundryDeploy: true
    tmsUpload: false
steps:
  cloudFoundryDeploy:
    cloudFoundry:
      apiEndpoint: "https://api.cf.eu10.hana.ondemand.com" # please verify if this is your API endpoint. If not, please change it
      org: "myOrg" # add your org here
      space: "mySpace" # add your space here
      credentialsId: "cfdeploy"
      appName: ""
    mtaDeployParameters: "-f --version-rule ALL"
  artifactPrepareVersion:
    versioningType: "cloud_noTag"
```

# Exercise 16 - Verify Build Success

No code
