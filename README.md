# agentic-ai-federation
AI agent federated authentication
Last Update: July 2, 2025

| Document Tracking |  |  |
| ----- | ----- | ----- |
| **Date** | **Description** | **Version** |
| 2025/07/02 | Created | 1.0.0 |
|  |  |  |
|  |  |  |

| Document Contacts |  |  |  |  |
| ----- | ----- | ----- | ----- | ----- |
| **Last** | **First Name** | **Company** | **Email** | **Slack Handle** |
| Herardian | Ron | Aethercloud | - | - |
|  |  |  |  |  |
|  |  |  |  |  |

# Project Description

The goal of this project is to demonstrate federated authentication with two AI agents, each using a different Idp. Essentially, this is a standard, server-to-server scenario. Additionally, access to data must be controlled using OIDC JWTs.

**This is an open source project under the Apache 2.0 license: [https://www.apache.org/licenses/LICENSE-2.0.html](https://www.apache.org/licenses/LICENSE-2.0.html)**

## Community Covenant

*All participants stipulate that contributions are voluntary and can be included in open source software or documentation without any obligation at all whatsoever on the part of any other participant or sponsoring entity. All participants agree under penalty of perjury that contributions shall not contain proprietary intellectual property, including but not limited to, trade secrets,patents,  copyrights, or trademarks.*

# Scenario Detail

* The User first authenticates with Agent1 using its associated Identity Provider 1 (IdP1) via a standard OIDC/OAuth2 flow, gaining an ID Token and Access Token.  
* When Agent1 needs to interact with Agent2 on behalf of the user, and Agent2 uses a different Identity Provider 2 (IdP2), Agent1 cannot simply reuse its token from IdP1.  
* Instead, Agent1 (server-side) acts as a client to IdP2. It submits an assertion (e.g., a signed JWT containing the user's identity details obtained from IdP1, or an assertion of Agent1's own identity as the "acting on behalf of" party) to IdP2's token endpoint.  
* IdP2 validates this assertion. If trust is established (e.g., IdP2 trusts assertions signed by Agent1, or IdP1), IdP2 issues a new Access Token to Agent1, scoped for Agent2 and representing the original user.  
* Agent1 then uses this new Access Token (from IdP2) to securely call Agent2's APIs, completing the federated loop where Agent2 trusts an identity that originated from IdP1, mediated by Agent1 and IdP2.

# Web Sequence

![Agentic AI Federated Authentication]([https://github.com/aethercloud-ai/agentic-ai-federation/blob/main/README.png])

## Diagram Steps:

1. User Accesses Agent1: User initiates interaction with Application 1\.  
2. Agent1 Redirects to IdP1: Agent1 initiates an OAuth2 Authorization Code flow, redirecting the user's browser to IdP1 for authentication.  
3. User Authenticates with IdP1: The user logs in and grants consent at IdP1.  
4. IdP1 Redirects back to Agent1: IdP1 sends an authorization code back to Agent1 via the user's browser.  
5. Agent1 Exchanges Code for Tokens: Agent1's backend exchanges the code with IdP1's token endpoint for an Access Token (for Agent1's interaction with IdP1's resource servers) and an ID Token (containing user identity information).  
6. Agent1 Prepares Assertion: When Agent1 needs to call Agent2 on behalf of this user, it constructs an assertion. This assertion could be an OpenID Connect ID Token from IdP1 (if IdP2 trusts IdP1), or a specially signed JWT created by Agent1 that asserts the user's identity and Agent1's right to act on their behalf.  
7. Agent1 Requests Token from IdP2 (Token Exchange): Agent1 (server-side) sends this assertion to IdP2's token endpoint using a grant type like urn:ietf:params:oauth:grant-type:token-exchange (RFC 8693).  
8. IdP2 Validates and Issues Token: IdP2 validates the assertion. If valid and trusted, IdP2 issues a new Access Token to Agent1, scoped for Agent2, and representing the original user.  
9. Agent1 Calls Agent2: Agent1 uses this new Access Token (from IdP2) to make authenticated API calls to Agent2.  
10. Agent2 Validates Token with IdP2: Agent2 validates the received token with its own IdP (IdP2) to confirm its authenticity and the user's permissions.  
11. Agent2 Responds: Agent2 processes the request and sends a response back to Agent1.  
12. Agent1 Updates UI: Agent1 updates its user interface, potentially including data retrieved from Agent2.

# Appendix A: Websequence Diagram

sequenceDiagram  
    title AI Agent Federated Authentication  
    actor User  
    participant Browser  
    participant Agent1 as Application 1  
    participant IdP1 as IdP 1 (Auth Server)  
    participant Agent2 as Application 2  
    participant IdP2 as IdP 2 (Auth Server)

    User-\>Browser: 1\. Access Agent1  
    activate Agent1  
    Browser-\>Agent1: 2\. GET /  
    Agent1-\>Browser: 3\. Redirect to IdP1 /authorize (OAuth2 Auth Code Flow)

    Browser-\>IdP1: 4\. GET /authorize?client\_id=Agent1&...  
    activate IdP1  
    IdP1-\>User: 5\. Display Login/Consent  
    User-\>IdP1: 6\. Authenticate & Consent  
    IdP1-\>Browser: 7\. Redirect to Agent1 /callback?code=...

    Browser-\>Agent1: 8\. GET /callback?code=...  
    Agent1-\>IdP1: 9\. POST /token (code, client\_secret, grant\_type=authorization\_code)  
    deactivate IdP1  
    Agent1-\>Agent1: 10\. Stores Access Token (Agent1-specific) & ID Token (User Info) for IdP1  
    Agent1-\>Browser: 11\. Render Agent1 UI  
    deactivate Agent1

    Note over Agent1, Agent2: Server-to-Server Federated Authentication Begins  
    activate Agent1  
    Agent1-\>Agent1: 12\. Prepares assertion for user based on IdP1 ID Token or internal user ID  
    Agent1-\>IdP2: 13\. POST /token (grant\_type=urn:ietf:params:oauth:grant-type:token-exchange, subject\_token=\<assertion from Agent1\>)  
    activate IdP2  
    IdP2-\>IdP2: 14\. Validates assertion (trusts Agent1 or IdP1)  
    IdP2-\>Agent1: 15\. Access Token (scoped for Agent2, representing User via federation)  
    deactivate IdP2

    Agent1-\>Agent2: 16\. API Call (Authorization: Bearer \<Access Token from IdP2\>)  
    activate Agent2  
    Agent2-\>IdP2: 17\. Introspect/Validate Token (if not self-contained JWT)  
    activate IdP2  
    IdP2-\>Agent2: 18\. Token Valid, User Info (claims, scopes)  
    deactivate IdP2  
    Agent2-\>Agent1: 19\. Response (processed request for user)  
    deactivate Agent2

    Agent1-\>Browser: 20\. Update Agent1 UI with Agent2 data  
    deactivate Agent1  
