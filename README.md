# loopback-swagger

**This module is in Active LTS mode, new features are no longer accepted.**
<br/>(See [Module Long Term Support Policy](#module-long-term-support-policy)
below.)

LoopBack 3 users looking for new features are encouraged to upgrade
to LoopBack 4. Refer to
[loopback-next#1849](https://github.com/strongloop/loopback-next/issues/1849)
for more information on how to upgrade.

## Overview

Utilities to transform between Swagger API specs and LoopBack remoting metadata.

This is an internal module used by the following user-facing tools:

 - [slc loopback:swagger](https://github.com/strongloop/loopback-swagger)
 - [loopback-component-explorer](https://github.com/strongloop/loopback-component-explorer)
 
---
# Swagger-ui v3 with Security

This is a WIP fork of advanced swagger.json OAuth2 support for Loopback using latest swagger-ui (currently 3.9)

## Key Repositories

- [drmikecrowe: loopback-component-explorer](https://github.com/drmikecrowe/loopback-component-explorer)
- [drmikecrowe: loopback-swagger](https://github.com/drmikecrowe/loopback-swagger) 
 
## SwaggerUI v3 Support Status

- [x]  Integrate v3 of the SwaggerUI (using [swagger-ui-dist](https://www.npmjs.com/package/swagger-ui-dist))
- [x]  securityDefinitions Support
- [x]  Integrate with [loopback-component-explorer](https://github.com/strongloop/loopback-component-explorer).
  - [x] Update explorer to use new Swagger-UI
  - [x] Test and document using oauth2-redirect for OAuth2 local development
- [ ] Tested Grant Types
  - [x] implicit
  - [ ] jwt
  - [ ] clientCredentials 
  - [x] authorizationCode 
  - [x] refreshToken 
  - [ ] resourceOwnerPasswordCredentials
- [ ]  Review @raymongfeng realm discussions and determine insure this strategy meets Strongloop plans 
- [ ]  Update automated tests
- [ ]  Review loopback style guide

## Installation

- See [drmikecrowe: loopback-component-explorer README](https://github.com/drmikecrowe/loopback-component-explorer) for current installation instructions

### Development redirectURIs

For development only, you may add the following redirectURI to your applications object (assuming your API is running on localhost:3001, and your authentication server is running on localhost:3000):

    "redirectURIs" : [
        "http://localhost:3001/explorer/oauth2-redirect.html"
    ], 

### Add detailed API info to your `component-config`

In your component-config.local.js, OAuth2 configuration may be specified as follows:

```js
var path = require('path');

module.exports = {
  "loopback-component-explorer":        {
    "mountPath":           "/explorer",
    "uiDirs":              "explorer",
    "apiInfo":             {
      "title":          "Loopback API Explorer",
      "description":    "A description of my API",
      "version":        require('../package.json').version,
      "termsOfService": "http://my-super-site.com/terms",
      "contact":        {
        "name":  "Support Team",
        "email": "support@my-super-site.com"
      }
    },
    "host":                "my-super-site.com",
    "schemes":             [
      "http",
      "https"
    ],
    "consumes":            [
      "application/json"
    ],
    "produces":            [
      "application/json",
      "application/xml"
    ],
    "securityDefinitions": {
      "OauthPassword": {
        "type":             "oauth2",
        "flow":             "implicit",
        "description":      "OAuth2 Password authentication",
        "authorizationUrl": path.join("/oauth/authorize"),
        "scopes":           {
          "read": "Access API methods",
          "write": "Update API methods"
        }
      },
      "OauthAccessCode": {
        "type":             "oauth2",
        "description":      "OAuth2 Access Code authentication",
        "flow":             "accessCode",
        "authorizationUrl": path.join("/oauth/authorize"),
        "tokenUrl":         path.join("/oauth/token"),
        "scopes":           {
          "read": "Access API methods",
          "write": "Update API methods"
        }
      }
    },
    "security":            [
      {
        "OauthAccessCode": ["read", "write"],
        "OauthPassword": ["read", "write"]
      }
    ]
  }
};
```

I implemented via component-config.local.js in order to allow the _path_ module in case the authorization server were needed to be a separate URL.

This produces swagger.yaml such as the following:

```yaml
info:
  title: Loopback API Explorer
  description: A description of my API
  version: 0.1.26
  contact:
    email: support@my-super-site.com
    name: Support Team
  termsOfService: 'http://my-super-site.com/terms'
basePath: /api/v1
host: my-super-site.com
consumes:
  - application/json
produces:
  - application/json
  - application/xml
securityDefinitions:
  OauthPassword:
    type: oauth2
    flow: implicit
    description: OAuth2 Password authentication
    authorizationUrl: /oauth/authorize
    scopes:
      read: Access API methods
      write: Update API methods
  OauthAccessCode:
    type: oauth2
    description: OAuth2 Access Code authentication
    flow: accessCode
    authorizationUrl: /oauth/authorize
    tokenUrl: /oauth/token
    scopes:
      read: Access API methods
      write: Update API methods
security:
  - OauthAccessCode:
      - read
      - write
    OauthPassword:
      - read
      - write
paths: {}
definitions: {}
tags: {}
```

### Route Specific Scopes

*IF* you need to specify a specific OAuth2 Scope at a Loopback method level, this project supports that functionality in a limited fashion:

* The swagger.json will syntatically be correct per the JSON schema definition for Swagger v2.
* It does **NOT** implement a custom role-resolver for that specific method.  That will be left to each application.

To specify a specific scope on a single route, in the model definition file:

```json
{
  "acls": [
    {
      "accessType": "EXECUTE",
      "principalType": "OauthPassword",
      "principalId": "createSomething",
      "scope": "write",
      "permission": "ALLOW"
    }
  ]
}
```

will produce Swagger yaml such as:

```yaml
  '/MyModel/createSomething':
    post:
      tags:
        - MyModel
      operationId: MyModel.createSomething
      parameters:
        - name: data
          in: body
          description: >-
            Data object to be saved
          required: true
          schema:
            description: >-
              Data object to be saved
            $ref: '#/definitions/SomethingObject'
      responses:
        '200':
          description: Request was successful
          schema:
            $ref: '#/definitions/Something'
      deprecated: false
      security:
        - OauthPassword:
            - write
```

 - [slc loopback:swagger](https://docs.strongloop.com/display/LB/Swagger+generator)
 - [loopback-explorer](https://github.com/strongloop/loopback-explorer)

## Module Long Term Support Policy

This module adopts the [
Module Long Term Support (LTS)](http://github.com/CloudNativeJS/ModuleLTS) policy,
with the following End Of Life (EOL) dates:

| Version | Status          | Published | EOL      |
| ------- | --------------- | --------- | -------- |
| 5.x     | Active LTS      | Sep 2017  | Dec 2019 |
| 4.x     | Maintenance LTS | Feb 2017  | Apr 2019 |
| 3.x     | Maintenance LTS | Dec 2016  | Apr 2019 |

Learn more about our LTS plan in [docs](https://loopback.io/doc/en/contrib/Long-term-support.html).
