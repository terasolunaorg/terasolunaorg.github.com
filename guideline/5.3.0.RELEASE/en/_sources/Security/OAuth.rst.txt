.. _OAuth:

OAuth
================================================================================

.. only:: html

 .. contents:: Index
    :local:

.. _OAuthOverview:

Overview
--------------------------------------------------------------------------------
This section explains the overview of OAuth 2.0 and the method to implement authorization control function as per the specifications of OAuth 2.0,
by using Spring Security OAuth which is one of the Spring project.

.. tip:: ** Reference of Spring Security OAuth **

    Spring Security OAuth also provides the functions which are not introduced in this guideline.
    Refer to \   `OAuth 2 Developers Guide <https://projects.spring.io/spring-security-oauth/docs/oauth2.html>`_\  , for the details of Spring Security OAuth.

|

.. _OAuthAboutOAuth2.0:

About OAuth 2.0
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

OAuth 2.0 is the authorization framework where access range can be specified for the resources protected on server,
when HTTP service is used in third-party application.

OAuth 2.0 is specified as RFC, and it consists of various related technical specifications.

Important specifications of OAuth 2.0 are as follows.


.. tabularcolumns:: |p{0.15\linewidth}|p{0.30\linewidth}|p{0.55\linewidth}|
.. list-table:: ** OAuth 2.0 important specifications **
    :header-rows: 1
    :widths: 15 30 55

    * - RFC
      - Overview
      - Description
    * - | RFC 6749
      - | \ `The OAuth 2.0 Authorization Framework <http://tools.ietf.org/html/rfc6749>`_\
      - | Technical specifications describing the most fundamental contents of OAuth 2.0, such as terminology and authorization methods.
    * - | RFC 6750
      - | \ `Bearer Token Usage <https://tools.ietf.org/html/rfc6750>`_\
      - | Technical specifications related to method of transferring the "Access token without signature" (Hereafter, referred as access token) within servers, 
          which is used for authorization control described in RFC 6749.
        | Information about access token is described later.
    * - | RFC 6819
      - | \ `Threat Model and Security Considerations <https://tools.ietf.org/html/rfc6819>`_\
      - | Technical specifications about the security requirements that need to be considered while using OAuth 2.0
        | In these guidelines, the concrete explanation about the study items is omitted.
    * - | RFC 7519
      - | \ `JSON Web Token (JWT) <https://tools.ietf.org/html/rfc7519>`_\
      - | Technical specifications related to "JSON Web Token (JWT)" which is a token including JSON that can be signed.
    * - | RFC 7523
      - | \ `JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants <https://tools.ietf.org/html/rfc7523>`_\
      - | Technical specifications related to method of using JWT prescribed in RFC 6819 as an access token to be used for authorization control described in RFC 6749.
    * - | RFC 7009
      - | \ `OAuth 2.0 Token Revocation <https://tools.ietf.org/html/rfc7009>`_\
      - | Technical specifications related to additional end point invalidating the token.

|

In conventional client-server type authentication model, the third-party application performs authentication by using user authentication information (user name and password etc.)
in order to access the resources protected on server.

In other words, user needs to share the authentication information with third party in order to provide the rights to third party applications for accessing the resources; however it has the risks such as unintended access and information leakage by the user, if there are defects and malicious operations in third party application.




.. figure:: ./images/OAuth_TraditionalAuthenticationModel.png
    :width: 100%


On the other hand, in OAuth 2.0, authentication can be directly performed by the user, and third party applications can access resources without sharing authentication information to third parties by issuing information for authenticated requests called as "access token".



Moreover, more flexible access control is achieved as compared with conventional client-server type authentication model, by enabling the specification of access range (scope) of resources when access token is issued.



.. figure:: ./images/OAuth_OAuthAuthenticationModel.png
    :width: 100%

|

.. _OAuthArchitecture:

Architecture of OAuth 2.0
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section explains the roles, scope, authorization grant, and protocol flow defined in OAuth 2.0.
In OAuth 2.0, concepts such as scope and authorization grant are defined, and the specifications of authorization are prescribed by using these concepts.

|

.. _OAuthRole:

Roles
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Following 4 roles are defined in OAuth 2.0.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: ** Roles in OAuth 2.0 **
    :header-rows: 1
    :widths: 25 75
    :class: longtable

    * - Role name
      - Description
    * - | Resource owner
      - | Role permitting the access to protected resources. User (End user) etc.
    * - | Resource server
      - | Role providing the protected resources. Web server.
    * - | Authorization server
      - | Role authenticating resource owner and issuing the access token (Information necessary for  client to access the resource server). Web server.
    * - | Client
      - | Role getting the authorization of resource owner and making the request for protected resources on behalf of resource owner. Web application etc. Client information is registered in the authorization server in-advance and managed by client ID which is unique in the authorization server.
        | OAuth 2.0 defines following 2 client types based on the ability to maintain confidentiality of client credentials (client authentication information).
        | (1) Confidential
        |     Client that can maintain the confidentiality of client credential.
        | (2) Public
        |     Client that cannot maintain the confidentiality of client credentials and cannot perform secure client authentication by using other means like the client executed on the device of resource owner.
        |
        | Moreover, OAuth 2.0 is being designed by considering the following examples as a client.
        | (1) Web application (web application)
        |     Client executed on Web server (Confidential).
        | (2) User agent-based application (user-agent-based application)
        |     Client executed in user agents of resource owner, by downloading the client code from Web server (public) . Such as Javascript application.
        | (3) Native application (native application)
        |     Client installed and executed on the device of resource owner (Public).

|

.. _OAuthScope:

Scope
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
The concept "Scope" is being used as a method of controlling access for resources protected in OAuth 2.0.

In response to the request from client, the authorization server can specify the access rights (read, write permissions etc.) for the protected resources, including the scope in access token based on the policy of authorization server or the instructions of resource owner.



|

.. _OAuthProtocolFlow:

Protocol flow
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In OAuth 2.0, resources are accessed in the following flow.

.. figure:: ./images/OAuth_ProtocolFlow.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: ** Protocol flow of OAuth 2.0 **
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Client requests authorization for the resource owner. As shown in the above figure, request is directly made to the resource owner, but it is recommended to request through the authorization server.
        | 
        | Request is made through authorization server for the authorization code grant and implicit grant among the grant types described later.
        | 
    * - | (2)
      - | Client receives an authorization grant (described later) as the credentials representing authorization from resource owner.
    * - | (3)
      - | Client requests for access token by presenting own authentication information and authorization grant given by resource owner to the authorization server.
        | 
    * - | (4)
      - | Authorization server authenticates the client and confirms the validity of authorization grant. It issues the access token if authorization grant is valid.
        | 
    * - | (5)
      - | Client requests to the resource protected on the resource server and authenticates with the issued access token.
        | 
    * - | (6)
      - | Resource server checks the validity of access token and accepts the request if it is valid.

.. note::

    In order to simplify the complicated mechanism of signature and token exchange which was not popular in OAuth 1.0, the request handling access token needs to be made by HTTPS communication in OAuth 2.0.
    (Using HTTPS communication prevents eavesdropping of access token)

|

.. _AuthorizationGrant:

Authorization grant
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Authorization grant represents authorization from resource owner and it is used when client fetches the access token.
In OAuth 2.0, following 4 grant types are defined, however these can be individually extended such as adding the credential items etc.

Client requests the access token to authorization server from any of grant types and accesses the resource server with the fetched access token.
Authorization server must define 1 or more supporting grant types, and determine the grant type to be used among these as per the authorization request from client.

.. tabularcolumns:: |p{0.25\linewidth}|p{0.75\linewidth}|
.. list-table:: **Authorization grant in OAuth 2.0**
    :header-rows: 1
    :widths: 25 75

    * - Grant type
      - Description
    * - | Authorization code grant
      - | In the flow of authorization code grant, the authorization server issues authorization code to client as an intermediary between client and resource owner, and the client issues the access token by passing the authorization code to authorization server.
        | It is not necessary to share the credentials of resource owner to the client in order to issue the access token by using the authorization code issued by authorization server.
        | Authorization code grant is used, when confidential client uses OAuth 2.0 similar to a Web application.
    * - | Implicit grant
      - | In the flow of implicit grant, authorization server acts as an intermediary similar to authorization code grant, however it issues the access token directly instead of an authorization code.
        | Since access token is encoded in the URL, it may get leaked to a resource owner or other applications on the same device. Further, since the client is not authenticated, a risk of spoofing attack due to unauthorized usage of access token issued to other clients is also likely.
          
        | Implicit grant should be used only when client type is public, such as client implemented in Javascript.
    * - | Resource owner password credential grant
      - | In the flow of resource owner password credential grant, client uses the authentication information of resource owner as authorization grant and issues the access token directly.
        | Since the credentials of resource owner must be shared with client, a risk of unauthorized usage or leakage of credentials is likely if client's reliability is low.
        | Resource owner password credential grant should be used only when there is high reliability between resource owner and client and when other grant types cannot be used.
    * - | Client credential grant
      - | In the flow of client credential grant, authentication information of client is used as an authorization grant, and access token is issued directly.
        | It is used when client is a resource owner.

|

Flow of authorization code grant is as follows.

.. figure:: ./images/OAuth_AuthorizationCodeGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Authorization code grant flow**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Resource owner accesses the pages required for resources which are protected on resource server provided by the client through user agent.
        | Client gives access to authorization end point on authorization server for the user agent of resource owner, in order to fetch authorization from resource owner.
        | In this case, client includes the "Client ID for own identification", "Scope requested to the resource as an option", "Redirect URL for returning the user agent after authorization process performed by authorization server" and "State" in the request parameters.
        | state is a random value associated with user agent and is used to ensure that series of flows is executed by the same user agent (CSRF countermeasures)
    * - | (2)
      - | User agent accesses the authorization end point of authorization server indicated to the client.
        | Authorization server authenticates the resource owner through user agent and checks the validity of parameters by comparing it with the registered client information, based on the client ID, scope and redirect URL of request parameter.
        | After confirmation, resource owner is asked to approve/ deny the access request.
    * - | (3)
      - | Resource owner sends approval / denial of access request to the authorization server.
        | When resource owner grants access, the authorization end point of authorization server issues authorization code and gives the instructions to redirect the user agent to client by using the redirect URL of request parameter.
        | In such case, authorization code is assigned to redirect URL, as the request parameter of redirect URL.
    * - | (4)
      - | User agent accesses the redirect URL with assigned authorization code.
        | When processing of client is completed, the response is returned to resource owner.
    * - | (5)
      - | Client sends the authorization code to the token end point on authorization server, in order to request the access token.
        | Token end point of authorization server authenticates the client and verifies the validity of authorization code, and issues the access token and optional refresh token if it is valid.
        | Refresh token is used to issue new access token when the access token is disabled or expired.
    * - | (6)
      - | Client accesses the resource server with the fetched access token.
        | Resource server checks the validity of access token, and if it is valid, it processes the request and returns the response to client.

|

Flow of implicit grant is as follows.

.. figure:: ./images/OAuth_ImplicitGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Flow of implicit grant**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Resource owner accesses the pages required for resources which are protected on resource server provided by the client through user agent.
        | Client gives access to authorization end point on authorization server for the user agent of resource owner, in order to fetch authorization from resource owner and issue the access token.
        | In this case, client includes the "Client ID for own identification", "Scope requested to the resource as an option", "Redirect URL for returning the user agent after performing authorization process by authorization server" and "State", in the request parameters.
        | state is a random value associated with user agent and is used to ensure that series of flows is executed by the same user agent (CSRF countermeasures)
    * - | (2)
      - | User agent accesses the authorization end point of authorization server indicated to the client.
        | Authorization server authenticates the resource owner through user agent and checks the validity of parameters by comparing it with the registered client information, based on the client ID, scope and redirect URL of request parameter.
        | After confirmation, resource owner is asked to approve/ deny the access request.
    * - | (3)
      - | Resource owner sends approval / denial of access request to the authorization server.
        | When resource owner grants access, the authorization end point of authorization server gives the instructions to redirect the user agent to client resource by using the redirect URL of request parameter. In such case, access token is assigned to URL fragment of redirect URL.
    * - | (4)
      - | User agent sends the request to client resource as per the redirect instructions. In such case, information of URL fragment is saved locally, and URL fragment is not sent in case of redirect.
        | When client resource is accessed, the web page (usually, HTML document including the embedded script) is returned.
        | User agent executes the script included in web page and extracts the access token from the locally saved URL fragment.
    * - | (5)
      - | User agent passes the access token to client.
        | When processing of client is completed, the response is returned to resource owner.
    * - | (6)
      - | Client accesses the resource server with the fetched access token.
        | Resource server checks the validity of access token, and if it is valid, it processes the request and returns the response to client

|

Flow of resource owner password credential grant is as follows.

.. figure:: ./images/OAuth_ResourceOwnerPasswordCredentialsGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Flow of resource owner password credential grant**
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Resource owner provides the credentials (user name, password) to the client.
    * - | (2)
      - | Client accesses the token end point of authorization server, in order to request the access token.
        | In this case, client includes the credentials specified from resource owner and the scope requested to resource, in the request parameter.
    * - | (3)
      - | Token end point of authorization server authenticates the client and verifies the credentials of resource owner. If it is valid, it issues the access token.

|

Flow of client credential grant is as follows.

.. figure:: ./images/OAuth_ClientCredentialsGrant.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Flow of client credential grant**
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Client accesses the token end point of authorization server, in order to request the access token.
        | In this case, client requests for access token including the client's own credentials.
    * - | (2)
      - | Token end point of authorization server authenticates the client and issues the access token when authentication is successful.

|

.. _AccessTokenLifeCycle:

Life cycle of access token
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Access token is issued by authorization server, by confirming the validity of authorization grant presented by the client.
For an issued access token, the scope is assigned based on the policy of authority server or the instructions of resource owner and access is obtained for the protected resource.
Expiry period is set when the access token is issued, and if it expires, the access rights of protected resource are invalidated.

Flow from issue of access token till its invalidation is as follows.

.. figure:: ./images/OAuth_LifeCycleOfAccessToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Flow from issue of access token till its invalidation**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Client presents the authorization grant and requests for access token.
    * - | (2)
      - | Authorization server confirms the authorization grant presented by client and issues the access token.
    * - | (3)
      - | Client presents the access token and requests for resource protected on resource server.
    * - | (4)
      - | Resource server verifies the validity of access token presented by client and if it is valid, performs processing for the resource protected on resource server.
    * - | (5)
      - | Client presents the access token (expired), and requests for the resource protected on resource server.
    * - | (6)
      - | Resource server verifies the validity of access token presented by client, and returns error if the access token has expired.

| When access token expires, the access rights of protected resources are invalidated, but access rights of protected resources can also be invalidated by disabling the access token before expiry of access token.
| When access token is to be disabled before its expiry, the client requests authorization server to cancel the token. Access rights of protected resources are invalidated for the disabled access token.

| 

| When access token expires, authorization grant should be presented to authorization server again and validity should be re-confirmed by authorization server, in order to re-acquire the access token by client.
  Therefore, when short validity term of access token is set, the usability decreases. On the other hand, when long validity term of access token is set, a high risk of disclosure or access token, and misuse is likely if it is disclosed.
  
| Refresh token can be used to reduce the risk of disclosure without decreasing the usability of the token.
  Refresh token can be used to get the new access token without re-submitting the authorization grant when access token is invalidated or expired.
  When validity term is set at the time of issuing the refresh token and when refresh token expires, the access token cannot be re-issued.
| Risk of disclosure of access token and misuse at the time of disclosure can also be controlled while maintaining the usability with the re-issue of access token in short cycle, by setting the short validity term of access token and setting the long validity term of refresh token.

| Issuing the refresh token is optional and is to be determined by the authorization server.

Flow of re-issuing the access token as per the refresh token is as follows.

.. figure:: ./images/OAuth_LifeCycleOfAccessTokenWithRefreshToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Flow from issuing the access token till its re-issue**
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Client presents the authorization grant and requests for access token.
    * - | (2)
      - | Authorization server confirms the authorization grant presented by client and issues the access token and refresh token.
    * - | (3)
      - | Client presents the access token and requests for resource protected on resource server.
    * - | (4)
      - | Resource server verifies the validity of access token presented by client, and if it is valid, it processes the resource protected on resource server.
    * - | (5)
      - | Client presents the access token (expired), and requests for the resource protected on resource server.
    * - | (6)
      - | Resource server verifies the validity of access token presented by client, and returns error if the access token has expired.
    * - | (7)
      - | When access token expiration error is returned from resource server, the client requests a new access token by presenting the refresh token.
    * - | (8)
      - | Authorization server verifies the validity of refresh token presented by client and if it is valid, issues the access token and optional refresh token.

.. _SpringSecurityOAuthArchitecture:

Architecture of Spring Security OAuth
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Spring Security OAuth is a library that provides functions necessary while building 3 roles such as Authorization Server, Resource Server, and Client as Spring applications among the roles defined in OAuth 2.0.
Spring Security OAuth is the technique that works by linking with the functions provided by Spring Framework (Spring MVC) and Spring Security, and it can build the authorization server, resource server and client by appropriate configuration (Bean definition) of default package provided by Spring Security OAuth.
Further, many extension points are provided similar to Spring Framework and Spring Security in order to incorporate the requirements that cannot be implemented in default package provided by Spring Security OAuth.

Further, refer to \ :doc:`../../Security/Authentication`\  and \ :doc:`../../Security/Authorization`\  for the details, when the functions provided by Spring Security are to be used for authentication/ authorization of requests within each roles.

When authorization server, resource server, client are built by using Spring Security OAuth, process is performed with the flow given below.

.. figure:: ./images/OAuth_OAuth2Architecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Flow of Spring Security OAuth**
    :header-rows: 1
    :widths: 10 90


    * - Sr. No.
      - Description
    * - | (1)
      - | Resource owner accesses the client through user agent.
        | Client calls \ ``OAuth2RestTemplate``\  from service.
        | In case of authorization grant accessing the authorization end point, instructions are given to user agent to redirect to authorization end point of authorization server.
    * - | (2)
      - | User agent accesses the \ ``AuthorizationEndpoint``\  which is an authorization end point of authorization server.
        | \  ``AuthorizationEndpoint``\  displays the screen that demands the authorization to the resource owner.
        | Resource owner accesses \  ``AuthorizationEndpoint``\  by authorizing the scope for client.
        | \  ``AuthorizationEndpoint``\  issues the authorization code if authorization grant is authorization code grant and it issues the access token if it is the implicit grant.
        | Issued authorization code or access token is passed to the client through user agent by using the redirect.
    * - | (3)
      - | Client accesses \  ``TokenEndpoint``\  which is the token end point of authorization server from \  ``OAuth2RestTemplate``\.
        | \  ``TokenEndpoint``\  calls \  ``AuthorizationServerTokenService``\  and issues the access token. Issued access token is passed to client as a response.
        | If authorization grant is authorization code grant, the authorization code is presented to authorization server. Validity of authorization code is verified by \  ``TokenEndpoint``\  before issuing the access token.
    * - | (4)
      - | Client specifies the access token fetched in (2) or (3) and accesses the resource server from \  ``OAuth2RestTemplate``\.
        | Resource server calls the \  ``OAuth2AuthenticationManager``\  and fetches the authentication information associated with access token through \  ``ResourceServerTokenServices``\. Further, it verifies the access token when authentication information is fetched.
        | When access token verification is successful, the resource corresponding to request from the client is returned.


.. note::

    As mentioned earlier, usage of HTTPS communication at each end point is assumed in OAuth 2.0, however there may be the cases when HTTPS communication is used in
    SSL accelerator or web server, or when distributed in multiple AP servers by using load balancer.
    When authorization code or redirect URL is embedded in client for linking the access token after authorization by resource owner,
    the redirect URL indicating the SSL accelerator, Web server and load balancer should be embedded.
    
    Spring (Spring Security OAuth) provides the technique for assembling the redirect URL by using any of the following headers.
    
    * Forwarded header
    * X-Forwarded-Host header, X-Forwarded-Port header, X-Forwarded-Proto header

    Setting should be done in order to assign the above mentioned headers at SSL accelerator and Web server, load balancer side, so that an appropriate redirect URL can be created in Spring(Spring Security OAuth).
    If this is not performed, verification of request parameter (redirect URL) performed in the authorization code grant or implicit grant is likely to fail.

.. tip::

    End point provided by Spring Security OAuth is implemented by extending the Spring MVC function. \ ``@FrameworkEndpoint``\  annotation is set in class at end point provided by Spring Security OAuth.
    This is because \ ``@Controller``\  annotation does not conflict with the class registered by developer as a component.
    Further, end point registered in \ ``@FrameworkEndpoint``\  annotation as component handles the \ ``@FrameworkEndpoint``\  method conflicting with URL as Handler class, after reading \ ``@RequestMapping``\  annotation of end point by \ ``FrameworkEndpointHandlerMapping``\  which is an extension class of \ ``RequestMappingHandlerMapping``\.


|

.. _AuthorizationServer:

Authorization server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Authorization server authenticates the resource owner, fetches the authorization from the resource owner after authentication, and then issues the access token.

OAuth 2.0 supports the "Scope" concept as an expression for specifying the access scope.
Client specifies the scope while sending the authorization request, and when the resource owner authorizes the specified scope or when it matches with the scope registered previously in authorization server, that scope is authorized for the client by authorization server.
Authorization can be used together in accordance with the scope of client and roles of Spring Security explained in :ref:`SpringSecurityAuthorization`\  section.

Spring Security OAuth provides the function to get an authorization from resource owner in \ ``AuthorizationEndpoint``\, and 
provides the function to issue access token of client in \ ``AuthorizationEndpoint``\  or \ ``TokenEndpoint``\.
\ ``AuthorizationEndpoint``\  and \ ``TokenEndpoint``\  issues the access token by \ ``DefaultTokenServices``\  which is a default package of \ ``AuthorizationServerTokenService``\.

When access token is issued, the client information registered in authorization server through \ ``ClientDetailsService``\  is fetched,
and the same is used for verifying the possibility of access token issue.
In OAuth 2.0 specifications, the registration procedure of clients that uses authorization server is not prescribed, and the client registration procedure is not supported in Spring Security OAuth as well.

Hence, when client registration interface is to be provided in the application, it must be implemented independently.

Spring Security authentication function is used for resource owner authentication.
Refer to :ref:`SpringSecurityAuthentication`\  for the details.

Configuration of authorization server is shown below.

.. figure:: ./images/OAuth_AutohrizationServerAuthArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Working of authorization server (In case of authorization end point access)**
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | \  ``AuthorizationEndpoint``\  process is executed by accessing the authorization end point (/oauth/authorize) of authorization server by the user agent.
    * - | (2)
      - | Call \  ``ClientDetailsService``\  method, and verify request parameter after fetching the client information registered in advance.
    * - | (3)
      - | Call \ ``UserApprovalHandler``\  method and check whether authorization of scope is registered for client
        | When authorization is not registered, the screen (/oauth/confirm_access) asking the resource owner for authorization through user agent is displayed.
        | In such case, the scope to be inquired is linked by getting the product of request parameter and client information registered in advance, and by using \ ``@SessionAttributes``\  of Spring MVC.
    * - | (4)
      - | Manage authorization status by \ ``ApprovalStore``\  in \ ``ApprovalStoreUserApprovalHandler``\  which is package of \ ``UserApprovalHandler``\.
        | When authorization is performed by resource owner, call \ ``ApprovalStore``\  method and register specified information.

|

.. note::

    As described earlier, the scope to be inquired is the product of scope registered previously in authorization server and scope specified by request parameter at the time of authorization request by client.
    For example, when the scope of READ and CREATE is specified by request parameters for the client assigning the scopes such as READ, CREATE, DELETE on authorization server, then the scope such as READ, CREATE which is product of (READ,CREATE,DELETE) and (READ,CREATE) can be assigned. 
    When the scope that is not assigned to client on authorization server is specified by request parameter, an error occurs and access is denied.

|

.. figure:: ./images/OAuth_AutohrizationServerTokenArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Working of authorization server (In case of token end point access)**
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | \ ``TokenEndpoint``\  process is executed by accessing the token end point (/oauth/token) of authorization server by the client
    * - | (2)
      - | Call \ ``ClientDetailsService``\  method and check whether the scope of request parameter is registered in client after fetching the previously registered client information.
    * - | (3)
      - | When scope is registered, call \ ``TokenGranter``\  method and issue access token.
    * - | (4)
      - | Call \ ``AuthorizationServerTokenServices``\  method and issue an access token in \ ``AbstractTokenGranter``\  which is an implementation of \ ``TokenGranter``\.
        | \ ``AbstractTokenGranter``\  which is the base class of \ ``TokenGranter``\  implemented as per grant type, and actual process is assigned to each class.
    * - | (5)
      - | Call \ ``TokenStore``\  method and manage status of access token in \ ``DefaultTokenServices``\  which is an implementation of \ ``AuthorizationServerTokenServices``\.


|

.. _ResourceServer:

Resource server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Resource server processes the access request for protected resource from the client and returns the response.
Resource server verifies whether the access token added to the request from client is within the validity period and gets the authentication information associated with the access token.
After fetching the authentication information, requested resource verifies whether the concerned access token is within scope.
Process after access token verification can be implemented similar to the normal applications.

Spring Security OAuth implements the verification function of access token, by using authentication and authorization framework of Spring Security.
\ ``OAuth2AuthenticationProcessingFilter``\  provided by Spring Security OAuth is specifically used in \ ``ServletFilter``\.
\ ``OAuth2AuthenticationEntryPoint``\  is used as an \ ``AuthenticationEntryPoint``\  interface and \ ``OAuth2AuthenticationManager``\  is used as \ ``AuthenticationManager``\  respectively.
Refer to ":ref:`SpringSecurityAuthentication`\ " for the details of Spring Security.

Configuration of resource server is as follows.

.. figure:: ./images/OAuth_ResourceServerArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Working of resource server**
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When client accesses the resource server at the beginning, \ ``OAuth2AuthenticationProcessingFilter``\  is called.
        | 
        | Access token is extracted in \ ``OAuth2AuthenticationProcessingFilter``\.
    * - | (2)
      - | After extracting the access token, fetch the authentication information associated with access token through \ ``ResourceServerTokenServices``\  in \ ``OAuth2AuthenticationManager``\  which is an implementation of \ ``AuthenticationManager``\.
        | Further, verify the access token when authentication information is fetched.
        | As a method of fetching authentication information associated with access token, a method of fetching the same by using  \ ``TokenStore``\  commonly with the authorization server is listed besides the method of inquiring by HTTP to authorization server.
        | 
        | How to fetch the authentication information depends on the implementation of \ ``ResourceServerTokenServices``\.
    * - | (3)
      - | When verification of access token is successful, return the resource for the requests from client.
    * - | (4)
      - | Handle the exception occurred at the time of authentication error by using \ ``OAuth2AuthenticationEntryPoint``\ and receive error response.
    * - | (5)
      - | Handle the exception occurred at the time of authorization error by using \ ``OAuth2AccessDeniedHandler``\  and receive error response.


|

.. _Client:

Client
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Client authorizes the resource owner and fetches the access token, accesses the resource protected on resource server on behalf of resource owner.
In such a case, the access token issued from authorization server is added for the request to the resource.

Spring Security OAuth provides \ ``OAuth2RestTemplate``\  which is OAuth 2.0 implementation of \ ``RestOperations``\ , as the implementation method of client's basic functions.

In \ ``OAuth2RestTemplate``\, authorization function required in authorization code grant is implemented by using \ ``OAuth2ClientContextFilter``\  as servlet filter, in addition to functions such as issuing access token, re-issuing access token using refresh token and accessing the resource server using access token.


Further, in \  ``OAuth2RestTemplate``\ , access token acquired from authorization server along with grant type specified \ ``OAuth2ProtectedResourceDetails``\  is saved in \ ``DefaultOAuth2ClientContext`` \  which is an implementation of \ ``OAuth2ClientContext``\.
\ ``DefaultOAuth2ClientContext``\  is defined as Bean of session scope by default, and access token can be shared among multiple requests.

When function to access resource server is to be developed, it is implemented similarly as development of usual REST client, except using \ ``OAuth2RestTemplate``\  provided by Spring Security OAuth instead of \ ``RestTemplate``\  provided by Spring Web, as an implementation of package of \  ``RestOperations``\.

Configuration while using \  ``OAuth2RestTemplate``\  as client function is given below.

.. figure:: ./images/OAuth_ClientArchitecture.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table:: **Working of client**
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No,
      - Description
    * - | (1)
      - | User agent accesses the \ ``Controller``\  so that \ ``Service``\  of client can be called.
        | Servlet function (\  ``OAuth2ClientContextFilter``\  ) for capturing \ ``UserRedirectRequiredException``\  that may occur in (5) is applied for access along with access to resource server.
        | User agent can access the authorization end point on authorization server, by applying this servlet function when \ `` UserRedirectRequiredException`` \ occurs.
    * - | (2)
      - | Call \ ``OAuth2RestTemplate``\  from \ ``Service``\.
    * - | (3)
      - | Before accessing resource server, confirm that access token is retained by \ ``DefaultOAuth2ClientContext``\  which is retained as a member.
        | When access token is saved and is within the validity period, specify access token fetched from \ ``DefaultOAuth2ClientContext``\  and access the resource server.
    * - | (4)
      - | When access token is not saved at first access, or when validity period is lapsed, call \ ``AccessTokenProvider``\  and fetch the access token.
    * - | (5)
      - | Fetch \ ``AccessTokenProvider``\ the access token for grant type defined in \ ``OAuth2ProtectedResourceDetails``\  as detailed information of resource.
        | \ ``UserRedirectRequiredException``\  is thrown when fetching of authorization code is not completed in \ ``AuthorizationCodeAccessTokenProvider``\  which is an implementation of authorization code grant.
    * - | (6)
      - | Specify access token fetched in (3) or (5) and access resource server.
        | When an error occurs such as "access token expired" (\ ``AccessTokenRequiredException``\ ) while accessing, perform subsequent processes from (4) onwards again after initializing retained access token.

|


.. _HowToUse:

How to use
--------------------------------------------------------------------------------

Bean definition example and implementation method required for using Spring Security OAuth is described.

|

.. _OAuthSetup:

Set-up of Spring Security OAuth
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Security OAuth is added as a dependent library in order to use the class provided by Spring Security OAuth.

.. code-block:: xml

    <!-- (1) -->
    <dependency>
        <groupId>org.springframework.security.oauth</groupId>
        <artifactId>spring-security-oauth2</artifactId>
    </dependency>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add Spring Security OAuth as a dependent library in :file:`pom.xml` of project using Spring Security OAuth.
        | When resource server, authorization server, client are added as different project, respective description should be added.

.. note::

    As given in above setting example, since it is assumed that dependency library version is managed by the parent project "terasoluna- gfw- parent", it is not necessary to specify the version in pom.xml
    The dependent library mentioned above is defined in \ `Spring IO Platform <http://platform.spring.io/platform/>`_\  used by terasoluna-gfw-parent.

|

.. _OAuthHowToUseApplicationSettings:

Application settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Settings of the application using Spring Security OAuth are described.

As shown in "\ :ref:`AuthorizationGrant`\ ",flow differs between authorization server and client differs in OAuth 2.0.
Hence, settings should be done in the application using Spring Security OAuth in accordance with the grant type supported by application. 
Refer to implementation of each role for the setting details for each grant type.

|

.. _ImplementationOAuthAuthorizationServer:

Implementation of authorization server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implementation method of authorization server is described.

Authorization server provides end points for "fetching authorization from resource owners" and "issuing access tokens", by using the function of Spring Security OAuth.
Further, while accessing the above-mentioned end point, it is necessary to authenticate the resource owner or client, and these guidelines are implemented by using the technique for authentication/ authorization of Spring Security.

.. _OAuthAuthorizationServerCreateSettingFile:

Creation of setting file (Authorization server)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``oauth2-auth.xml``\  is created as configuration file for defining in authorization server.
| \ ``oauth2-auth.xml``\  performs Bean definition of end points for providing the functions of authentication server, and makes security setting for the end points and sets the grant type supporting the authorization server.

* ``oauth2-auth.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:sec="http://www.springframework.org/schema/security"
           xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
           xsi:schemaLocation="http://www.springframework.org/schema/security
               http://www.springframework.org/schema/security/spring-security.xsd
               http://www.springframework.org/schema/security/oauth2
               http://www.springframework.org/schema/security/spring-security-oauth2.xsd
               http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">


    </beans>

Next, settings are described in \ ``web.xml``\  in order to read the created \ ``oauth2-auth.xml``\.

* ``web.xml``

.. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-auth.xml  <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a Bean definition file of \  OAuth 2.0. It should be described before \ ``spring-security.xml``\ , by considering the case when URL targeted for access control set in \ ``oauth2-auth.xml``\  is included in the URL targeted for access control set in \ ``spring-security.xml``\.

|

.. _OAuthAuthorizationServerDefinition:

Defining authorization server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Definitions of authorization server are added as below.

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize" >  <!-- (1) -->
            <oauth2:authorization-code />  <!-- (2) -->
            <oauth2:implicit />  <!-- (3) -->
            <oauth2:refresh-token />  <!-- (4) -->
            <oauth2:client-credentials />  <!-- (5) -->
            <oauth2:password />  <!-- (6) -->
        </oauth2:authorization-server>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \  ``<oauth2:authorization-server>``\  tag and define settings of authorization server.
        | Authorization end points for authorization and token end points for issuing the access tokens are registered as components, by using the \ ``<oauth2:authorization-server>``\  tags.
        | Specify URL of token end point in \ ``token-endpoint-url``\  attribute. When it is not specified, default value "/oauth/token" is specified.
        | Specify URL of authorization end point in \ ``authorization-endpoint-url``\  attribute. When it is not specified, default value "/oauth/authorize" is specified.
    * - | (2)
      - | Use \ ``<oauth2:authorization-code>``\  tag and support authorization code grant.
    * - | (3)
      - | Use \ ``<oauth2:implicit>``\  tag and support implicit grant.
    * - | (4)
      - | Use \ ``<oauth2:refresh-token>``\  tag and support refresh token.
    * - | (5)
      - | Use \  ``<oauth2:client-credentials>``\  tag and support client credential grant.
    * - | (6)
      - | Use \  ``<oauth2:password>``\  tag and support resource owner password credential grant.

.. note::

    When multiple supporting grant types are to be specified, they should be specified in the above mentioned sequence.

.. note::

    Authorization code is used for only the short period from issuing the authorization code till issuing the access token, hence it is managed in in-memory by default.
    In case of configuration of multiple authorization servers, it should be managed in DB, in order to share authorization codes among multiple servers.
    When authorization code is to be managed in DB, the following table, consisting of the column having authorization code as primary key and the column having authentication information, is created. Following example explains the DB definitions when PostgreSQL is used.
    
    .. figure:: ./images/OAuth_ERDiagramCode.png
        :width: 30%
    
    Bean ID of \ ``org.springframework.security.oauth2.provider.code.JdbcAuthorizationCodeServices``\  managing authorization code in DB is specified in \ ``authorization-code-services-ref``\  of \ ``<oauth2:authorization-code>``\  tag, in the configuration file of authorization server.
    Data source to be connected to table for authorization code storage is specified in the constructor of \ ``JdbcAuthorizationCodeServices``\.
    **Always** refer to \ :ref:`OAuthAuthorizationServerHowToControllTarnsaction`\  for the precautions while managing the authorization code permanently in DB.
    
    * ``oauth2-auth.xml``
    
    .. code-block:: xml
    
            <oauth2:authorization-server
                 token-endpoint-url="/oth2/token"
                 authorization-endpoint-url="/oth2/authorize" >
                <oauth2:authorization-code authorization-code-services-ref="authorizationCodeServices"/>
                <!-- omitted -->
            </oauth2:authorization-server>
            
            <bean id="authorizationCodeServices"
                  class="org.springframework.security.oauth2.provider.code.JdbcAuthorizationCodeServices">
                <constructor-arg ref="codeDataSource"/>
            </bean>
            
            <!-- omitted -->

|

.. _OAuthAuthorizationServerClientAuthentication:

Client authentication
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Clients that have accessed end points should be authenticated, in order to confirm whether they are registered clients.
Client is authenticated by verifying the client ID and password passed by client as parameters based on the client information retained by authorization server. Basic authentication is used for authentication.

Spring Security OAuth provides the implementation class of \ ``oorg.springframework.security.oauth2.provider.ClientDetailsService`` \ which is an interface for fetching the client information.
Further, \ ``org.springframework.security.oauth2.provider.client.BaseClientDetails``\  class which is implementation class of \ ``ClientDetails``\  interface is provided as a class retaining the client information.
\ ``BaseClientDetails``\  provides the basic parameters such as client ID and supporting grant type etc. by using OAuth 2.0, and the parameters can be added by extending \ ``BaseClientDetails``\.
In this case, extension of \ ``BaseClientDetails``\  and creation of \ ``ClientDetailsService``\  implementation class is performed, and client information wherein client name is added as individual parameter is managed by using DB and the implementation method for authentication is explained.

At first, DB is created as below.

.. figure:: ./images/OAuth_ERDiagram.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | A table to retain client information. client_id is considered as primary key.
        | Roles of each column are as below.
        |  ・client_id: A column to retain the client ID which is an ID to identify a client.
        |  ・client_secret: A column to retain the password which is used to authenticate the client.
        |  ・client_name: An individual column to retain the client name. Not required since it is an individual column.
        |  ・access_token_validity: A column to retain the validity period [seconds] of access token.
        |  ・refresh_token_validity: A column to retain the validity period [seconds] of refresh token.
    * - | (2)
      - | A table to retain the scope information. It is mapped with the client information by considering client_id as the external key.
        | Scope used for client authorization is retained in scope column. Records are registered only for the scope of client.
    * - | (3)
      - | A table to retain the resource information. It is mapped with the client information by considering client_id as the external key.
        | Resource ID used in resource server for identifying whether it is a resource that can be accessed by client is retained in resource_id column.
        | Access to resource is permitted, only when resource ID defined for the resource retained by resource server is included in resource ID registered here.
        | Records are registered only for the resource IDs accessible by client.
        | When not even single resource ID is registered, all resources can be accessed, hence precaution must be taken when it is not registered.
    * - | (4)
      - | A table to retain the grant information. It is mapped with the client information by considering client_id as the external key.
        | Grant to be used by client is retained in authorized_grant_type column.
        | Records are registered only for the grant count to be used by client
    * - | (5)
      - | A table to retain the redirect URL information. It is mapped with the client information by considering client_id as the external key.
        | web_server_redirect_uri column retains the URL that redirects user agent after authorization by resource owner.
        | Redirect URLs are used only for authorization code grants, implicit grants.
        | Table itself is not required, when grant types other than authorization code grant and implicit grant are used.
        | An error occurs, when there is no URL declared by client at the time of authorization request and when there is no redirect URL matching with host and route path.
        | Records are registered only for the URLs that can be declared by client.


A model retaining the client information is created.

* ``Client.java``

.. code-block:: java

        public class Client implements Serializable{
            private String clientId; // (1)
            private String clientSecret; // (2)
            private String clientName; // (3)
            private Integer accessTokenValidity; // (4)
            private Integer refreshTokenValidity; // (5)
            // Getters and Setters are omitted
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | A field to retain the client ID identifying the client.
    * - | (2)
      - | A field to retain the password used for client authentication.
    * - | (3)
      - | An extended field to retain the client name which is not provided in Spring Security OAuth.
        | Not required, since it is an extended field.
    * - | (4)
      - | A field to retain the validity period [seconds] of access token.
    * - | (5)
      - | A field to retain the validity period [seconds] of refresh token.


Detailed information of client can be extended easily by creating the class inheriting the \ ``BaseClientDetails``\  class.

* ``OAuthClientDetails.java``

.. code-block:: java

        public class OAuthClientDetails extends BaseClientDetails{
            private Client client;
            // Getter and Setter are omitted
        }


\ ``org.springframework.security.oauth2.provider.ClientDetailsService``\  is an interface for fetching the detailed client information required in authorization process from the data store.
Creation of \ ``ClientDetailsService``\  implementation class is described below.

* ``OAuthClientDetailsService.java``

.. code-block:: java

        @Service("clientDetailsService") // (1)
        @Transactional
        public class OAuthClientDetailsService implements ClientDetailsService {
        
            @Inject
            ClientRepository clientRepository;
        
            @Override
            public ClientDetails loadClientByClientId(String clientId)
                    throws ClientRegistrationException {
                
                Client client = clientRepository.findClientByClientId(clientId); // (2)
                
                if (client == null) { // (3)
                      throw new NoSuchClientException("No client with requested id: " + clientId);
                }
                
                // (4)
                Set<String> clientScopes = clientRepository.findClientScopesByClientId(clientId);
                Set<String> clientResources = clientRepository.findClientResourcesByClientId(clientId);
                Set<String> clientGrants = clientRepository.findClientGrantsByClientId(clientId);
                Set<String> clientRedirectUris = clientRepository.findClientRedirectUrisByClientId(clientId);
                
                
                 // (5)
                OAuthClientDetails clientDetails = new OAuthClientDetails();
                clientDetails.setClientId(client.getClientId());
                clientDetails.setClientSecret(client.getClientSecret());
                clientDetails.setAccessTokenValiditySeconds(client.getAccessTokenValidity());
                clientDetails.setRefreshTokenValiditySeconds(client.getRefreshTokenValidity());
                clientDetails.setResourceIds(clientResources);
                clientDetails.setScope(clientScopes);
                clientDetails.setAuthorizedGrantTypes(clientGrants);
                clientDetails.setRegisteredRedirectUri(clientRedirectUris);
                clientDetails.setClient(client);
                
                return clientDetails;
            }
            
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add \ ``@Service``\  annotation to the class level as service in order to use it as target for component-scan.
        | Specify Bean name as \ ``clientDetailsService``\.
    * - | (2)
      - | Client information fetched from database is stored in the Client model.
    * - | (3)
      - | When the client information is not found, an exception of Spring Security OAuth - \ ``NoSuchClientException``\  is generated.
    * - | (4)
      - | Fetch information associated with the client.
        | If the process efficiency is reduced by calling Repository by dividing it for multiple times, fetch collectively in (2).
    * - | (5)
      - | Set various types of fetched information in \ ``OAuthClientDetails``\  field.


Add settings necessary for client authentication to \ ``oauth2-auth.xml``\.

* ``oauth2-auth.xml``

.. code-block:: xml

        <sec:http pattern="/oth2/*token*/**" 
            authentication-manager-ref="clientAuthenticationManager" realm="Realm">  <!-- (1) -->
            <sec:http-basic />           <!-- (2) -->
            <sec:csrf disabled="true"/>  <!-- (3) -->
            <sec:intercept-url pattern="/**" access="isAuthenticated()"/>  <!-- (4) -->
        </sec:http>

        <oauth2:authorization-server 
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             client-details-service-ref="clientDetailsService">  <!-- (5) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <sec:authentication-manager id="clientAuthenticationManager">  <!-- (6) -->
            <sec:authentication-provider user-service-ref="clientDetailsUserService" >  <!-- (7) -->
                <sec:password-encoder ref="passwordEncoder"/>  <!-- (8) -->
            </sec:authentication-provider>
        </sec:authentication-manager>

        <bean id="clientDetailsUserService"
            class="org.springframework.security.oauth2.provider.client.ClientDetailsUserDetailsService">  <!-- (9) -->
            <constructor-arg ref="clientDetailsService" />  <!-- (10) -->
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a location below \ ``/oth2/*token*/``\  as a target for access control
          as an endpoint URL in order to perform  security settings for endpoint related to access token operation.
        | Here, the endpoint URL acting as a access control target is set to a value starting from \ ``/oth2/``\ ,
          endpoint URL defined by Spring Security OAuth and its default value are as below.
        |  ・\ ``/oauth/token``\  - an endpoint URL of endpoint used in issuing a token 
        |  ・\ ``/oauth/check_token``\ - an endpoint URL of endpoint used to verify a token
        |  ・\ ``/oauth/token_key``\ - an endpoint URL of endpoint used for fetching a public key, when JWT signature is created by public key encryption method
        | Specify a Bean of \ ``AuthenticationManager``\  for client authentication defined in (5), in \ ``authentication-manager-ref``\  attribute.
        | Also, when Basic authentication is enabled by XML Namespace as shown in (2), Realm name of Basic authentication becomes \ ``"Spring Security Application"``\.
        | Specify an appropriate value in \ ``realm``\  attribute since internal information of the application gets revealed.
    * - | (2)
      - | Apply Basic authentication to client authentication.
        | For details, refer http://docs.spring.io/spring-security/site/docs/4.1.4.RELEASE/reference/html/basic.html .
    * - | (3)
      - | Disable CSRF countermeasures function for accessing \ ``/oth2/*token*/**``\.
        | Spring Security OAuth adopts validity check of request wherein state parameter is used, recommended as a CSRF countermeasure of OAuth 2.0.
    * - | (4)
      - | Settings which assign rights of access only to authenticated users, for locations under endpoint URL.
        | How to specify access policy for Web resource, refer \ :doc:`../../Security/Authorization`\.
    * - | (5)
      - | Specify a Bean of \ ``OAuthClientDetailsService``\  in \ ``client-details-service-ref``\  attribute.
        | BeanID to be specified must match with Bean ID specified by implementation class of \ ``ClientDetailsService``\.
    * - | (6)
      - | Define a Bean for \ ``AuthenticationManager``\  for authentication of client.
        | \ ``AuthenticationManager``\  used in the authentication of resource owner and Bean ID of alias must be specified.
        | For authentication of resource owner, refer \ :ref:`OAuthAuthorizationServerResourceOwnerAuthentication`\.
    * - | (7)
      - | Specify a Bean of \ ``ClientDetailsUserDetailsService``\  defined in (9), in \ ``user-service-ref``\  attribute of \ ``sec:authentication-provider``\.
    * - | (8)
      - | Specify a Bean of \ ``PasswordEncoder``\  used in password hashing, which is used in client authentication.
        | For details of password hashing, refer \ :ref:`SpringSecurityAuthenticationPasswordHashing`\.
    * - | (9)
      - | Define a Bean for \ ``ClientDetailsUserDetailsService``\  which is an implementation class of \ ``UserDetailsService``\  interface.
        | \ ``UserDetailsService``\  used in the resource owner authentication and Bean ID of alias must be specified.
    * - | (10)
      - | Specify a Bean of \ ``OAuthClientDetailsService``\  which fetches client information from  database, in constructor argument.
        | Bean ID to be specified must match with Bean ID specified by the implementation class of \ ``ClientDetailsService``\.


|

.. _OAuthAuthorizationServerResourceOwnerAuthentication:

Resource owner authentication
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When an authorization code grant is used for fetching an access token, a resource owner must be authenticated by a method like providing a login screen among others.

| This guideline assumes the use of Spring Security for authentication of resource owner.
| A URL with an authorization end point URL must be defined in authorization settings as an access policy to enable access to authorized endpoint URL only to authenticated users.
  Further, a controller URL which displays the authorization screen and a controller URL which handles exceptions in authorization endpoint must also be similarly defined as access policies.
| For the controller which displays authorization screen, refer \ :ref:`OAuthAuthorizationServerHowToCustomizeAuthorizeView`\  and for the controller which handles exception in authorization endpoint, refer \ :ref:`OAuthAuthorizationServerHowToHandleError`\.

For details of Spring Security, refer \ :doc:`../../Security/Authentication`\  and \ :doc:`../../Security/Authorization`\.

Definition examples of access policies which include authorization endpoint URL, a URL of controller which displays authorization and a URL of controller which handles errors of authorization endpoint are shown below.

* ``spring-security.xml``

.. code-block:: xml

        <sec:http authentication-manager-ref="userLoginManager"> <!-- (1) -->
            <sec:form-login login-page="/login"
                authentication-failure-url="/login?error=true"
                login-processing-url="/login" />
            <sec:logout logout-url="/logout" 
                logout-success-url="/" 
                delete-cookies="JSESSIONID" />
            <sec:access-denied-handler ref="accessDeniedHandler"/>
            <sec:custom-filter ref="userIdMDCPutFilter" after="ANONYMOUS_FILTER"/>
            <sec:session-management />
            <sec:intercept-url pattern="/login/**" access="permitAll" />
            <sec:intercept-url pattern="/oth2/**" access="isAuthenticated()" /> <!-- (2) -->
            <!-- omitted -->
        </sec:http>
        
         <sec:authentication-manager id="userLoginManager"> <!-- (3) -->
            <sec:authentication-provider
                user-service-ref="userDetailsService">
                <sec:password-encoder ref="passwordEncoder" />
            </sec:authentication-provider>
        </sec:authentication-manager>
        
        <bean id="userDetailsService"
            class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
            <property name="dataSource" ref="dataSource" />
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify location under root ("\ ``/``\ ") as an access control target, which includes authorization endpoint URL - \ ``/oth2/authorize``\ , a URL of controller which displays authorization screen - \ ``/oauth/confirm_access``\  and a URL which handles errors of authorization endpoint - \ ``/oauth/error``\.
    * - | (2)
      - | Specify location under root ("\ ``/``\ ") to enable the access only by authenticated users, which includes authorization endpoint URL - \ ``/oth2/authorize``\ , a URL of controller which displays authorization screen - \ ``/oauth/confirm_access``\  and a URL of controller which handles errors of authorization endpoint - \ ``/oauth/error``\.
    * - | (3)
      - | Define a Bean for \ ``AuthenticationManager``\  for authenticating resource owner.
        | \ ``AuthenticationManager``\  used in client authentication and Bean ID of alias must be specified.


.. _OAuthAuthorizationServerHowToAuthorizeByScope: 

Authorization for each scope
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
A setup method wherein each scope is authorized individually instead of authorizing the requested scope together while requesting the authorization for resource owner, is explained.

Authorization information must be controlled in a DB for performing perpetual management to prevent loss of authorization information at the time of restarting authorization server or to share authorization information across multiple authorization servers.
Following DB is created for storing authorization information for each scope. The example below explains DB definition when PostgreSQL is used.

.. figure:: ./images/OAuth_ERDiagramApprovals.png
    :width: 50%


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | A table to retain authorization information. userId, clientId and scope are used as primary keys.
        | Role of each column is as below.
        |  ・userId: A column to retain user name of resource owner who performs the authorization.
        |  ・clientId: A column to retain client ID of the client authorized by the resource owner.
        |  ・scope: A column to retain scope authorized by resource owner.
        |  ・status: A column to check whether the authorization is done by the resource owner. \ ``APPROVED``\  is set when authorized and \ ``DENIED``\  is set when authorization is denied.
        |  ・expiresAt: A column to retain validity period of authorization information.
        |  ・lastModifiedAt: A column to retain last modified date for authorization information.

Fetch authorization for each scope of resource owner and apply settings to store the same in DB and control.

Implementation example is as below.

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"> <!-- (1) -->
             
             <!-- omitted -->
             
        </oauth2:authorization-server>

        <bean id="userApprovalHandler"
              class="org.springframework.security.oauth2.provider.approval.ApprovalStoreUserApprovalHandler">  <!-- (2) -->
            <property name="clientDetailsService" ref="clientDetailsService"/>
            <property name="approvalStore" ref="approvalStore"/>
            <property name="requestFactory" ref="requestFactory"/>
            <property name="approvalExpiryInSeconds" value="3200" />
        </bean>

        <bean id="approvalStore"
              class="org.springframework.security.oauth2.provider.approval.JdbcApprovalStore">  <!-- (3) -->
            <constructor-arg ref="approvalDataSource"/>
        </bean>

        <bean id="requestFactory"
              class="org.springframework.security.oauth2.provider.request.DefaultOAuth2RequestFactory">
            <constructor-arg ref="clientDetailsService"/>
        </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a Bean of \ ``ApprovalStoreUserApprovalHandler``\  defined in (2), in \ ``user-approval-handler-ref``\  as \ ``UserApprovalHandler``\  which performs authorization process of scope.
    * - | (2)
      - | Define a Bean for \ ``ApprovalStoreUserApprovalHandler``\  which performs authorization process of scope.
        | Specify a Bean of \ ``JdbcApprovalStore``\  defined in (3), in \ ``approvalStore``\  property which manages authorization results of resource owner.
        | Specify a Bean of \ ``OAuthClientDetailsService``\  in \ ``clientDetailsService``\  property which fetches client information to be used in authorization process of scope.
        | Specify a Bean of \ ``DefaultOAuth2RequestFactory``\  in \ ``requestFactory``\  property.
        | Bean set in \ ``requestFactory``\  property is not used by ``ApprovalStoreUserApprovalHandler``\, however, since an error occurs at the time of Bean generation of \ ``ApprovalStoreUserApprovalHandler``\  when the setting is not applied, settings in \ ``requestFactory``\  property is necessary.
        | When validity period [Seconds] of authorization information is to be specified, validity period [Seconds] is set in \ ``approvalExpiryInSeconds``\  property. If any setting is not applied, the authorization information will remain valid a month from authorization.
    * - | (3)
      - | Define a Bean for \ ``JdbcApprovalStore``\  which manages authorization information in DB.
        | Specify a database in the constructor to connect to the table for storing authorization information.
        | For precautions while performing perpetual management of authorization information in DB, **always** refer \ :ref:`OAuthAuthorizationServerHowToControllTarnsaction`\.

.. note::

    When perpetual management is not required to be performed for authorization information and is to be managed by in-memory instead of a DB, a Bean is to be defined for \ ``org.springframework.security.oauth2.provider.approval.InMemoryApprovalStore``\  as \ ``approvalStore``\.


.. _OAuthAuthorizationServerHowToCustomizeAuthorizeView: 

Customising scope authorization screen
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When the scope authorization screen is to be customised, it can be done by creating controller and JSP. An example is explained below wherein the scope authorization screen is customised.

When an endpoint which requests authorization to resource owner is to be called, it is forwarded to (context path)/oauth/confirm_access.
A controller which handles (context path)/oauth/confirm_access is created.

* ``OAuth2ApprovalController.java``

.. code-block:: java

        @Controller
        public class OAuth2ApprovalController {
                
            @RequestMapping("/oauth/confirm_access") // (1)
            public String confirmAccess() {
                // omitted
                return "approval/oauthConfirm";
            }
        
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``@RequestMapping``\  annotation and perform mapping as a method for accessing \ ``"/oauth/confirm_access"``\.


Next, a JSP of scope authorization screen is created.
Since scope for authorization is registered in the Model by \ ``scopes``\  key, scope authorization screen is displayed by using the same.

* ``oauthConfirm.jsp``

.. code-block:: jsp

    <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
    
    <body>
        <div id="wrapper">
            <h1>OAuth Approval</h1>
            <p>Do you authorize '${f:h(authorizationRequest.clientId)}' to access your protected resources?</p>
            <form id='confirmationForm' name='confirmationForm' action='${pageContext.request.contextPath}/oth2/authorize' method='post'>
                <c:forEach var="scope" items="${scopes}" varStatus="status">  <!-- (1) -->
                    <li>
                        ${f:h(scope.key)}  <!-- (2) -->
                        <input type='radio' name="${f:h(scope.key)}" value='true'/>Approve
                        <input type='radio' name="${f:h(scope.key)}" value='false'/>Deny
                    </li>
                </c:forEach>
                <input name='user_oauth_approval' value='true' type='hidden'/>  <!-- (3) -->
                <sec:csrfInput />  <!-- (4) -->
                <label>
                    <input name='authorize' value='Authorize' type='submit'/>
                </label>
            </form>
        </div>
    </body>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1) 
      - | Output a radio button for specifying authorization for each scope. Since target scope is included in \ ``scopes``\  list, specify \ ``scopes``\  in \ ``items``\.
    * - | (2)
      - | Since key names of elements which retain \ ``scopes``\  list are used as respective scope names, key names are displayed on the screen.
        | Set output of Approve and Deny radio buttons for selecting "Authorise" and "Deny".
    * - | (3)
      - | Spring Security OAuth assigns \ ``user_oauth_approval``\  to the request parameter by embedding \ ``user_oauth_approval``\  as a hidden item.
        | \ ``user_oauth_approval``\  assigned to the request parameter can be used for executing a method which performs scope authorization of authorization endpoint.
    * - | (4)
      - | Specify \ ``<sec:csrfInput>``\  element in \ ``<form>``\  element of HTML in order to deliver CSRF.

.. _OAuthAuthorizationServerHowToHandleError:

Error handling at the time of authorization request
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When an authorization error (error related to security like "client does not exist" or redirect URL check error) occurs in authorization end point, \ ``OAuth2Exception``\  offered by Spring Security OAuth is thrown and the request (context path) /oauth/error is forwarded.
Hence, when exception in authorization end point is to be handled, a controller to handle (context path)/oauth/error must be created.

* ``OAuth2ErrorController.java``

.. code-block:: java

        @Controller
        public class OAuth2ErrorController {
                
            @RequestMapping("/oauth/error") // (1)
            public String handleError() {
                // omitted
                return "common/error/oauthError";
            }
        
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use \ ``@RequestMapping``\  annotation and perform mapping as a method for accessing \ ``"/oauth/error"``\.


Next, a JSP of error screen thus displayed is created.
Since details of error occurred in authorization end point are registered in the Model by \ ``error``\  key, error details are displayed on the screen using the same.

* ``oauthError.jsp``

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="utf-8">
    <title>OAuth Error!</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div id="wrapper">
            <h1>OAuth Error!</h1>
            <p>${f:h(error.OAuth2ErrorCode)}</p> <!-- (1) -->
            <p>${f:h(error.message)}</p> <!-- (2) -->
        <br>
        </div>
    </body>
    </html>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Output error response included in \ ``error``\.
    * - | (2) 
      - | Output error message included in \ ``error``\.

.. note::

    When an error other than authorization error (error related to security error like "client does not exist", redirect URL check error etc) occur in authorization endpoint,
    errors are notified on the client side after redirecting.



.. _OAuthAuthorizationServerHowToConfigureAccessToken:

How to share access token with resource server
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Authorization server links with the access token via \ ``TokenServices``\  so that the resource server can determine the authorization for accessing a resource based on access token.
Various methods exist for methods for linking.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 70
    :class: longtable

    * - Sr. No.
      - Method of linking
      - Description
    * - | (1)
      - | Linking via DB
      - | A method wherein a common DB is used and access token is linked.
        | It can be used when resource server and authorization server share a DB.
        | Authorization server specifies \ ``DefaultTokenServices``\  as an implementation of TokenService, and \ ``JdbcTokenStore``\  as an implementation of TokenStore.
    * - | (2)
      - | Linking via HTTP access
      - | A method wherein access token is linked by accessing HTTP.
        | This method is used when resource server and authorization server cannot use a common DB.
        | Since the resource server requests the fetching and verification of access token to authorization server, authorization server is overloaded.
        | Authorization server specifies \ ``DefaultTokenServices``\  as an implementation of TokenService.
        | Specify \ ``JdbcTokenStore``\  when the access token is to be managed by DB and \ ``InMemoryTokenStore``\  when it is to be managed by memory, as an implementation of TokenStore.
        | Implementation wherein the access token is managed by memory is exclusively for testing purpose since access tokens are lost due to operations like restarting a server etc.
    * - | (3)
      - | Linking via JWT
      - | A method wherein JWT is used and access token is linked.
        | This method is used when resource server and authorization server cannot use a common DB.
        | Since request is not sent to authorization server for fetching an access token, when compared with linking via HTTP access, authorization server is not overloaded.
        | Authorization server specifies \ ``DefaultTokenServices``\  as an implementation of TokenService and \ ``JwtTokenStore``\  as an implementation of TokenStore.
        | Access token is signed, encoded and decoded by using \ ``org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter``\.
        | There are two methods for signing and verification of access token - a method which uses public key and a method which uses common key.
    * - | (4)
      - | Linking via memory
      - | A method wherein access token is linked by sharing the memory.
        | It can be used when an application is designed wherein resource server and authorization server become a single process.
        | Authorization server specifies \ ``DefaultTokenServices``\  as an implementation of TokenService and \ ``InMemoryTokenStore``\  as an implementation of TokenStore.
        | Since the access token is linked via memory, linking of access token using common DB and HTTP access is not necessary.
        | Implementation wherein the access token is shared via memory is exclusively for testing purpose since access tokens are lost due to operations like restarting the server etc.

| 

.. todo:: **TBD**
    
    How to implement linking by using JWT will be explained in detail in next version.



Here, a method wherein access token is linked via common DB is explained as a representative method.
A method to link via HTTP access is explained in How To Extend of this section.

 * \ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\
 
When access token is linked via a common DB, \ ``JdbcTokenStore``\  offered by Spring Security OAuth is used.

Implementation example is as below.


* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices">  <!-- (1) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">  <!-- (2) -->
            <property name="tokenStore" ref="tokenStore" />
            <property name="clientDetailsService" ref="clientDetailsService" />
            <property name="supportRefreshToken" value="true" />  <!-- (3) -->
        </bean>

        <bean id="tokenStore"
          class="org.springframework.security.oauth2.provider.token.store.JdbcTokenStore"> <!-- (4) -->
          <constructor-arg ref="tokenDataSource" />
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``tokenServices``\  defined in (2) in \ ``token-services-ref``\  attribute as TokenService used by authorization server.
    * - | (2)
      - | Specify \ ``DefaultTokenServices``\  in \ ``tokenServices``\  class.
        | Specify \ ``TokenStore``\  defined in (4), in \ ``tokenStore``\  property as the token store which manages access tokens.
        | This setting is applied to resource server as well when the access token is linked with resource server via common DB.
    * - | (3)
      - | Specify \ ``true``\  in \ ``supportRefreshToken``\  attribute when refresh token is enabled.
    * - | (4)
      - | Define a Bean for \ ``JdbcTokenStore``\  as the token store.
        | Specify a data source in the constructor for connecting to a table wherein token information is stored.


\ ``JdbcTokenStore``\  creates following DB wherein a schema is defined for Spring Security OAuth, for linking the access tokens.
The example below explains the DB definition while using PostgreSQL as a common DB.

.. figure:: ./images/OAuth_ERDiagramToken.png
    :width: 100%

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | A table which manages the access token. It is used to share information of access token issued by authorization server with the resource server.
        | Roles of each column are as below.
        |  ・authentication_id: A column to store authentication key which uniquely identifies authentication information. It is used as a primary key.
        |  ・token: A column which retains token information as a binary after serializing. Validity period of access token, scope, token ID of access token, token ID of refresh token, token type which shows types of token to be used are stored as information of the token to be retained.
        |  ・token_id: A column to retain token ID which uniquely identifies access token.
        |  ・user_name: A column to retain user name of authenticated resource owner.
        |  ・client_id: A column to retain client ID of authenticated client.
        |  ・authentication: A column which retains authentication information of resource owner and client as a binary after serializing.
        |  ・refresh_token: A column which retains token ID of refresh token associated with access token.
    * - | (2)
      - | A table which manages refresh token associated with access token.
        | Roles of each column are as given below.
        |  ・token_id: A column to retain token ID which uniquely identifies refresh token. It is used as a primary key.
        |  ・token: A column which retains token information as binary after serializing. It retains validity period of refresh token.
        |  ・authentication: A column that retains authentication information of resource owner and client as binary after serializing. Information same as authentication information retained by table which manages access tokens, is retained.

| 

.. note::

    When the token is managed by common DB, the expired token is deleted within the timing of use of access token by client.
    Therefore, even if token has expired, access token is deleted only when it is accessed by the client.
    Expired tokens must be purged separately by batch processing in order to delete the same from DB.


.. _OAuthAuthorizationServerHowToCancelToken:

Canceling a token (authorization server)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
How to implement cancellation of issued access token is explained.

Access token can be cancelled by calling \ ``revokeToken``\  method of a class
which implements \ ``ConsumerTokenService``\  interface.
\ ``DefaultTokenService``\  class implements \ ``ConsumerTokenService``\  interface.

Authorization information can be deleted as well at the time of cancellation of access token.
When authorization request is sent without deleting authorization information after cancellation of access token while using authorization code grant and implicit grant, authorization information at the time of previous authorization request is reused.
Authorization information at the time of previous authorization request can be reused when validity period of authorization information is valid and entire authorization request scope is authorized.

Implementation example is given below.

An interface of service class which cancels a token and implementation class are created.

* ``RevokeTokenService.java``

.. code-block:: java

    public interface RevokeTokenService {
        
        String revokeToken(String tokenValue, String clientId);
        
    }

* ``RevokeTokenServiceImpl.java``

.. code-block:: java

    @Service
    @Transactional
    public class RevokeTokenServiceImpl implements RevokeTokenService {
        
        @Inject
        ConsumerTokenServices consumerService; // (1)
        
        @Inject
        TokenStore tokenStore; // (2)
        
        @Inject
        ApprovalStore approvalStore; // (3)
        
        public String revokeToken(String tokenValue, String clientId){ // (4)
            // (5)
            OAuth2Authentication authentication = tokenStore.readAuthentication(tokenValue);
            if (authentication != null) {                
                if (clientId.equals(authentication.getOAuth2Request().getClientId())) { // (6)
                    // (7)
                    Authentication user = authentication.getUserAuthentication();
                    if (user != null) {
                        Collection<Approval> approvals = new ArrayList<Approval>();
                        for (String scope : authentication.getOAuth2Request().getScope()) {
                            approvals.add(
                                    new Approval(user.getName(), clientId, scope, new Date(), ApprovalStatus.APPROVED));
                        }
                        approvalStore.revokeApprovals(approvals);
                    }
                    consumerService.revokeToken(tokenValue); // (8)
                    return "success";
                    
                } else {
                    return "invalid client";
                }
            } else {
                return "invalid token";
            }
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Inject implementation class of \ ``ConsumerTokenService``\  interface which cancels access tokens.
    * - | (2)
      - | Inject implementation class of \ ``TokenStore``\  used to fetch authentication information while issuing an access token.
    * - | (3)
      - | Inject implementation class of \ ``ApprovalStore``\  used to fetch authorization information while issuing an access token.
        | It is not required when authorization information is not to be deleted while cancelling an access token.
    * - | (4)
      - | Receive value of access token to be cancelled and client ID used for checking the client as parameters.
    * - | (5)
      - | Call \ ``readAuthentication``\  method of implementation class of \ ``TokenStore``\  and fetch authentication information while issuing an access token.
        | Token is deleted only when authentication information can be successfully fetched.
    * - | (6)
      - | Fetch client ID used at the time of issuing access token, from authentication information and check whether it matches with client ID of request parameter.
        | Delete access token only when it matches with client ID at the time of issuing an access token.
    * - | (7)
      - | Fetch authentication information of resource owner at the time of issuing an access token.
        | When authentication information of resource owner could be fetched, call \ ``revokeApprovals``\  method of implementation class of \ ``TokenStore``\  and delete authorization information.
        | Since authentication information for resource owner does not exist while using client credential grant, the parameters to be passed to \ ``revokeApprovals``\  method cannot be generated.
        | Therefore, authorization information cannot be deleted when it is not possible to fetch authentication information of resource owner.
        | This process is not required when authorization information is not deleted while cancelling the access token.
    * - | (8)
      - | Call \ ``revokeToken``\  method of implementation class \ ``ConsumerTokenService``\  and delete access token and refresh token associated with access token.


A controller which receives a cancellation request of token is created.

* ``TokenRevocationRestController.java``

.. code-block:: java

    @RestController
    @RequestMapping("oth2")
    public class TokenRevocationRestController {
        
        @Inject
        RevokeTokenService revokeTokenService;
        
        @RequestMapping(value = "tokens/revoke", method = RequestMethod.POST) // (1)
        @ResponseStatus(HttpStatus.OK)
        public String revoke(@RequestParam("token") String token,
            @AuthenticationPrincipal UserDetails user){
            
            // (2)
            String clientId = user.getUsername();
            String result = revokeTokenService.revokeToken(token, clientId); // (3)
            return result;
        }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. no.
      - Description
    * - | (1)
      - | Use \ ``@RequestMapping``\  annotation and map as a method for accessing \ ``"/oth2/tokens/revoke"``\.
        | The path specified here must apply Basic authentication and disable CSRF countermeasures, similar to settings performed in \ :ref:`OAuthAuthorizationServerClientAuthentication`\.
    * - | (2)
      - | Fetch client ID used while checking the cancellation of token, from authentication information generated in Basic authentication.
    * - | (3)
      - | Call \ ``RevokeTokenService``\  and delete a token.
        | Pass value of access token received as the request parameter and client ID received from authentication information as parameters.

| 

.. tip::

    RFC 7009 states that \ ``token_type_hint``\  can be arbitrarily assigned as a request parameter.
    \ ``token_type_hint``\  is a hint to determine whether to delete access token or refresh token.
    \ ``ConsumerTokenService``\  offered by Spring Security OAuth is not used in the implementation example above in order to delete both access token and refresh token by passing the access token.

.. note::

    The client which requests deletion of token to the authorization server should also delete a token retained by the client after deletion of authorization server.
    For deletion of token of client server, refer \ :ref:`OAuthClientServerHowToCancelToken`\.


|

.. _OAuthAuthorizationServerHowToControllTarnsaction:

Transaction control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
It explains precautions of transaction control in authorization server.

When the information handled by Spring Security Auth (authorization code, authorization information, token) is to be managed in DB, for authorization server, transaction control must be considered.

Default implementation of \ ``AuthorizationServerTokenServices``\  - \ ``DefaultTokenServices``\  contains a method to issue access token and refresh token.
These methods can be called under transaction control by assigning \ ``@Transactional``\  to \ ``createAccessToken``\  and \ ``refreshAccessToken``\  respectively, however, the operation otherwise becomes non-transactional control.

Hence, when \ ``Connection``\  fetched from  \ ``DataSource``\  is set to \ ``autoCommit=false``\, transaction control is imperative since information to be managed is not registered in DB.
Further, although it is not mandatory when \ ``autoCommit=true``\, adequate care must be taken since it is necessary to consider transaction control to ensure data consistency.

An example of transaction control when authorization code and authorization information are managed in DB is shown below.

* ``oauth2-auth.xml``

.. code-block:: xml
    
    <!-- omitted -->
    
    <tx:advice id="oauthTransactionAdvice">
        <tx:attributes>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    
    <aop:config>
        <aop:pointcut id="authorizationOperation"
                      expression="execution(* org.springframework.security.oauth2.provider.code.AuthorizationCodeServices.*(..))"/> <!-- (1) -->
        <aop:pointcut id="approvalOperation"
                      expression="execution(* org.springframework.security.oauth2.provider.approval.UserApprovalHandler.*(..))"/> <!-- (2) -->
        <aop:advisor pointcut-ref="authorizationOperation" advice-ref="oauthTransactionAdvice"/>
        <aop:advisor pointcut-ref="approvalOperation" advice-ref="oauthTransactionAdvice"/>
    </aop:config>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Use AOP and set transaction boundary in each method which operates authorization code.
    * - | (2)
      - | Use AOP and set transaction boundary in each method which operates authorization information.

|

Implementation of resource server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Here, how to implement a resource server is explained by using implementation example wherein authorization is set for REST API of TODO resource.

Creating a setup file (resource server)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A new Bean definition file for OAuth 2.0 is created while implementing a resource server.

Here, it is \ ``oauth2-resource.xml``\.

Following settings are added to \ ``oauth2-resource.xml``\.

* ``oauth2-resource.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
           xmlns:sec="http://www.springframework.org/schema/security"
           xsi:schemaLocation="http://www.springframework.org/schema/security
               http://www.springframework.org/schema/security/spring-security.xsd
               http://www.springframework.org/schema/security/oauth2
               http://www.springframework.org/schema/security/spring-security-oauth2.xsd
               http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">

        <sec:http pattern="/api/v1/todos/**" create-session="stateless"
                       entry-point-ref="oauth2AuthenticationEntryPoint"> <!-- (1) -->
            <sec:access-denied-handler ref="oauth2AccessDeniedHandler"/> <!-- (2) -->
            <sec:csrf disabled="true"/> <!-- (3) -->
            <sec:custom-filter ref="oauth2AuthenticationFilter"
                                    before="PRE_AUTH_FILTER" /> <!-- (4) -->
        </sec:http>

        <bean id="oauth2AccessDeniedHandler"
                  class="org.springframework.security.oauth2.provider.error.OAuth2AccessDeniedHandler" /> <!-- (5) -->
        
        <bean id="oauth2AuthenticationEntryPoint"
                  class="org.springframework.security.oauth2.provider.error.OAuth2AuthenticationEntryPoint" /> <!-- (6) -->

        <oauth2:resource-server id="oauth2AuthenticationFilter" resource-id="todoResource"
                  token-services-ref="tokenServices" entry-point-ref="oauth2AuthenticationEntryPoint" /> <!-- (7) -->

    </beans>

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the pattern on path that is the target of authorization setting of OAuth 2.0 in \  ``pattern``\  attribute.
        | Specify the Bean of \ ``OAuth2AuthenticationEntryPoint``\  in \  ``entry-point-ref``\. Settings are required for definition, however the specified Bean is not used.
          Actually, the Bean of \ ``OAuth2AuthenticationEntryPoint`` \ specified in \ ``OAuth2AuthenticationProcessingFilter``\  described later is used.
    * - | (2)
      - | Set the Bean of \ ``OAuth2AccessDeniedHandler``\  in \ ``access-denied-handler``\. Here, \ ``oauth2AccessDeniedHandler``\  described later is specified.
    * - | (3)
      - | Disable the CSRF since error occurs if the POST, PUT, DELETE requests are received by CSRF token check.
    * - | (4)
      - | Specify the authentication filter for resource server in \ ``custom-filter``\. Here \ ``oauth2AuthenticationFilter``\  described later is specified.
        | \ ``OAuth2AuthenticationProcessingFilter``\  is set as per this specification.
        | Since \ ``OAuth2AuthenticationProcessingFilter``\  is a filter for performing Pre-Authentication by using access token included in request,
        | Set in such a way that \ ``OAuth2AuthenticationProcessingFilter``\  process is executed before \ ``PRE_AUTH_FILTER``\  by specifying \ ``PRE_AUTH_FILTER``\  in  \ ``before``\.
        | For Pre-Authentication, refer \  `<http://docs.spring.io/spring-security/site/docs/3.0.x/reference/preauth.html>`_\.
    * - | (5)
      - | Define \ ``AccessDeniedHandler``\  for resource server provided by Spring Security OAuth.
        | \  ``OAuth2AccessDeniedHandler``\  sends the error response by handling the exceptions that occur at the time of authorization error.
    * - | (6)
      - | Define the Bean of \ ``OAuth2AuthenticationEntryPoint``\  for sending the error response for OAuth.
    * - | (7)
      - | Define the ServletFilter for the resource server provided by Spring Security OAuth.
        | The string specified in \ ``id``\  attribute is the Bean ID. Here,``oauth2AuthenticationFilter``\  is specified.
        | Specify the resource ID provided by server in \ ``resource-id``\  attribute. Here, \ ``todoResource``\  is specified.
        | It is verified whether the resource ID specified in \ ``resource-id``\  attribute is included for the resource ID of client information linked to access token.
        | As a result of verification, it is allowed to access the resource only when resource ID is included.
        | Note that, definition of \ ``resource-id`` \  is optional and when it is not defined, resource ID is not verified.
        | Specify the ID of \ ``TokenServices``\  in \ ``token-services-ref``\  attribute. \ ``TokenServices``\  is described later.
        | Specify the Bean of \ ``OAuth2AuthenticationEntryPoint``\  in \  ``entry-point-reff``\  attribute. Here, \ ``oauth2AuthenticationEntryPoint``\  is specified.

|

Add the settings in \  ``web.xml``\  so as to read a created \  ``oauth2-resource.xml``\.


* ``web.xml``

.. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-resource.xml <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | First read oauth2-resource.xml considering that the path including the path pattern set in oauth2-resource.xml is set as the access control target in ``spring-security.xml``\.

|

Setting of accessible scope for the resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In order to define the accessible scope for each resource, add the scope definition and Bean
definition for supporting SpEL expression in Bean definition file for OAuth 2.0.


Implementation example is as given below.
Note that, Basic authentication is not performed for the Client in the settings of access control since it is assumed that the security between authorization server and resource is secured.

* ``oauth2-resource.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
           xmlns:sec="http://www.springframework.org/schema/security"
           xsi:schemaLocation="http://www.springframework.org/schema/security
               http://www.springframework.org/schema/security/spring-security.xsd
               http://www.springframework.org/schema/security/oauth2
               http://www.springframework.org/schema/security/spring-security-oauth2.xsd
               http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">

        <sec:http pattern="/api/v1/todos/**" create-session="stateless"
                       entry-point-ref="oauth2AuthenticationEntryPoint">
            <sec:access-denied-handler ref="oauth2AccessDeniedHandler"/>
            <sec:csrf disabled="true"/>
            <sec:expression-handler ref="oauth2ExpressionHandler"/> <!-- (1) -->
            <sec:intercept-url pattern="/**" method="GET"
                                    access="#oauth2.hasScope('READ')" /> <!-- (2) -->
            <sec:intercept-url pattern="/**" method="POST"
                                    access="#oauth2.hasScope('CREATE')" /> <!-- (2) -->
            <sec:custom-filter ref="oauth2ProviderFilter"
                                    before="PRE_AUTH_FILTER" />
        </sec:http>

        <!-- omitted -->
        
        <oauth2:web-expression-handler id="oauth2ExpressionHandler" /> <!-- (3) -->

    </beans>

|

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify a Bean of \ ``OAuth2WebSecurityExpressionHandler``\  in  \ ``expression-handler``\.
    * - | (2)
      - | Define access policy as per the scope for the resource by using \ ``intercept-url``\.
        | Specify the path pattern of resource to be retained in \ ``pattern``\  attribute. In this implementation example, resource is retained under /api/v1/todos/.
        | Specify the HTTP method of resource in  \ ``method``\  attribute.
        | Specify the scope that authorizes the access to resource in \ ``access``\  attribute. Differentiate the setting value by upper-case and lower case characters.
        | Here, it is specified by using SpEL expression.
    * - | (3)
      - | Define the Bean of \  ``OAuth2WebSecurityExpressionHandler``\.
        | SpeL for performing OAuth 2.0 authorization setting provided by Spring Security OAuth is supported by defining this Bean.
        | Note that, the value specified in ``id``\  attribute is the id for this bean.

|

The main Expression provided by Spring Security OAuth is introduced.

For details, refer \ `JavaDoc <http://docs.spring.io/spring-security/oauth/apidocs/org/springframework/security/oauth2/provider/expression/OAuth2SecurityExpressionMethods.html>`_\  of \ ``OAuth2SecurityExpressionMethods``\.

.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table:: **Expression provided by Spring Security OAuth**
    :header-rows: 1
    :widths: 35 65

    * - Expression
      - Description
    * - | \ ``clientHasRole(String role)``\
      - | Return \ ``true``\  when the client performs the role specified in the argument.
    * - | \ ``clientHasAnyRole(String... roles)``\
      - | Return \ ``true``\  when the client performs one of the roles specified in the argument.
    * - | \ ``hasScope(String scope)``\
      - | Return \ ``true``\  when the scope authorized by the resource owner and the scope of the argument matches in the client.
    * - | \ ``hasAnyScope(String... scopes)``\
      - | Return \ ``true``\  when the scope authorized by the resource owner and one of the scopes of the argument matches in the client.
    * - | \ ``hasScopeMatching(String scopeRegex)``\
      - | Return \ ``true``\  when the scope authorized by the resource owner and the regular expression specified in the argument matches in the client.
    * - | \ ``hasAnyScopeMatching(String... scopesRegex)``\
      - | Return \ ``true``\  when the scope authorized by the resource owner and one of the regular expressions specified in the argument matches in the client.
    * - | \ ``denyOAuthClient``\
      - | Deny request in OAuth 2.0. It is used so that only the resource owner can access the resources.
    * - | \ ``isOAuth``\
      - | Allow request in OAuth 2.0. It is used so that the client can access the resources.

.. note::

   SpEL expression can be used with SpEL provided by Spring Security.
    
   Refer to \:ref:`built-incommon-expressions`\ for SpEL provided by Spring Security.

|

Configuration of access token (Resource Server)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Authorization Server is linked with Resource Server via access token \ ``TokenServices``\.

Refer to \ :ref:`OAuthAuthorizationServerHowToConfigureAccessToken`\  for the methods of linking.

Apply settings by the method of sharing database.

Refer to \ :ref:`OAuthAuthorizationServerHowToConfigureAccessToken`\  for the description of settings as the settings are same as \ ``TokenServices``\  settings in authorization server.


* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.DefaultTokenServices">
            <property name="tokenStore" ref="tokenStore" />
        </bean>

        <bean id="tokenStore"
          class="org.springframework.security.oauth2.provider.token.store.JdbcTokenStore">
          <constructor-arg ref="tokenDataSource" />
        </bean>

.. note:: 

    A method which use \ ``RemoteTokenServices``\  provided by Spring Security OAuth and a method which use
    JSON Web Token are offered as the methods for linking the authorization server and resource server.
    Refer to \ :ref:`OAuthAuthorizationServerHowToCooperateWithHttp`\  for how to use \ ``RemoteTokenServices``\  provided by Spring Security OAuth.

|

Fetching user information
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the Resource Server, the authenticated resource owner information can be received by specifying \ ``UserDetails``\  in method argument of Controller class and assigning \ ``@AuthenticationPrincipal``\  annotation similar to fetching method of the authentication information explained in \ :ref:`SpringSecurityAuthenticationIntegrationWithSpringMVC`\.
The implementation example is shown below.


.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {
        
        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(@AuthenticationPrincipal UserDetails user) { // (1)
        
            // omitted
        
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Authentication information of resource owner is stored in the argument \``user``\

In case of implementation that specifies \ ``UserDetails``\, it should be noted that the intended information cannot be fetched in the client credential grant wherein authentication process is not carried out.
When client credential grant is used, client ID can be fetched by specifying \ ``String``\  as the method argument of Controller and assigning \ ``@AuthenticationPrincipal``\  annotation.

Further, in case of implementation that specifies \ ``UserDetails``\, authentication information of client such as Client ID cannot be fetched.
When client authentication information is to be fetched, \ "OAuth2Authentication`` \  can be specified as the method argument of the Controller class.
An example wherein \ 'OAuth2Authentication`` \  is specified in the method argument of Controller class and authentication information of the client and the resource owner is fetched, is given below.

.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {
        
        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(OAuth2Authentication authentication) { // (1)
        
            String username = authentication.getUserAuthentication().getName(); // (2)
            String clientId = authentication.getOAuth2Request().getClientId();  // (3)
        
            // omitted
        
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | \ Authentication information of the resource owner and the client is stored in the \ argument \ ``authentication``\.
    * - | (2)
      - | \ Fetch resource owner name from \ ``authentication``\.
    * - | (3)
      - | \ Fetch client ID from  \ ``authentication``\.

.. note::

    The implementation example is given above wherein \ ``OAuth2Authentication``\  is used in the application layer.
    When the implementation is to be done without depending on \ ``OAuth2Authentication``\, the same function is feasible by implementing \ ``HandlerMethodArgumentResolver``\.
    Refer to \ :ref:`methodargumentresolver`\  for specific implementation method.

| 

Implementation of client
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

How to implement client is broadly classified into the following 2 methods based on the grant type to be used.


.. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 35 65

    * - How to implement
      - Grant type
    * - | OAuth2RestTemplate
      - | Authorization code grant
        | Resource owner password credential grant
        | Client credential grant
    * - | Javascript
      - | Implicit grant

In this guideline, \ :ref:`OAuthClientUsingOAuth2RestTemplate`\  and \ :ref:`OAuthClientUsingJavaScript`\ are described respectively in classification described above.

|

.. _OAuthClientUsingOAuth2RestTemplate:

Accessing a resource using OAuth2RestTemplate
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

How to access resource using \ ``OAuth2RestTemplate``\  is described as the implementation of the client for authorization code grant, resource owner password credential grant
and client credential grant.

In Spring Security OAuth, \ ``OAuth2RestTemplate``\  is provided as implementation for OAuth 2.0 of \ ``RestOperations``\.

In \ ``OAuth2RestTemplate``\, the functions for fetching access token, sharing access token between multiple requests by \ ``OAuth2ClientContext``\  and error handling when accessing the Resource Server
are implemented as per the grant type by \ ``AccessTokenProvider``\  as independent functions of OAuth 2.0.


In the client, the resource can be accessed using OAuth 2.0 functions by defining the parameters as per the
application requirements such as grant type, scope using \ ``OAuth2RestTemplate``\.

|

Creation of configuration file (Client)
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

First create \ ``oauth2-client.xml``\  as the configuration file for defining OAuth 2.0.

* ``oauth2-client.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security/oauth2 http://www.springframework.org/schema/security/spring-security-oauth2.xsd
        ">


    </beans>

|

Add the settings to \ ``web.xml``\ so as to read the created \ ``oauth2-client.xml``\.

* ``web.xml``

.. code-block:: xml

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath*:META-INF/spring/applicationContext.xml
            classpath*:META-INF/spring/oauth2-client.xml  <!-- (1) -->
            classpath*:META-INF/spring/spring-security.xml
        </param-value>
    </context-param>

    <!-- omitted -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Set so as to read \ ``oauth2-client.xml``\.

|

Applying OAuth2ClientContextFilter
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Register \ ``OAuth2ClientContextFilter``\  as a servlet filter.

By registering \ ``OAuth2ClientContextFilter``\, when an attempt is made to access the Resource Server when authorization cannot be fetched by the resource owner,
a function to capture \ ``UserRedirectRequiredException``\  and redirect to the page for fetching authorization of the resource owner provided by the Authorization Server
can be incorporated.

Add the following settings to \ ``oauth2-client.xml``\.

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:client id="oauth2ClientContextFilter" /> <!-- (1) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Explanation
    * - | (1)
      - |  Bean is defined for \ ``OAuth2ClientContextFilter``\ by using \ ``<oauth2:client>``\ tag. The string specified in \ ``id``\ attribute is used as Bean ID.

|

Add the settings of \ ``OAuth2ClientContextFilter``\ to \ ``web.xml``\.

* ``web.xml``

.. code-block:: xml

    <filter> <!-- (1) -->
        <filter-name>oauth2ClientContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>oauth2ClientContextFilter</filter-name>
        <url-pattern>/*</url-pattern> <!-- (2) -->
    </filter-mapping>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Register the Bean wherein Bean ID matches with the filter name (value specified in the \ ``<filter-name>``\  element) as a servlet filter by using  \ ``DelegatingFilterProxy``\.
        | Set the value same as the Bean ID specified in \ ``id``\  attribute of \ ``<oauth2:client>``\  in the filter name.
        | It is recommended to mention \ ``OAuth2ClientContextFilter``\  at the end of servlet filter definition so as not to perform  unintended exception handling of the exception generated in Spring Security OAuth.
    * - | (2)
      - | \``OAuth2ClientContextFilter`` \  is applied to the path wherein \``UserRedirectRequiredException`` \  may occur.
        | In the above example, \ "OAuth2ClientContextFilter`` \  is applied to all the requests.

.. note::
    Since \ ``OAuth2ClientContextFilter``\  generates redirect URL for returning the user agent after the authorization process by the Authorization Server in the pre-process of the filter,
    a wide definition such as \ ``/*``\  will result in futile processing in the path where \ ``UserRedirectRequiredException``\  does not occur.

.. note::
    \``OAuth2ClientContextFilter`` \  stores the URL to be redirected after the Authorization Server gets the authorization
    from the resource owner with the attribute name \ ``currentUri``\  in the request scope. Therefore, the attribute name \ ``currentUri``\  cannot be used in the client.

|

As mentioned above, \``Oauth2ClientContextFilter`` \  is a servlet filter that captures \``UserRedirectRequiredException`` \ and redirects it to the Authorization Server.

However, if the \``SystemExceptionResolver`` \  set in the blank project in advance, handles \``UserRedirectRequiredException`` \  first,
\``Oauth2ClientContextFilter``\  will not work as intended.

Therefore, it is necessary to change the setting of \ `spring-mvc.xml`` \  so that the \``SystemExceptionResolver`` \  does not handle \``UserRedirectRequiredException`` \.
Refer to \ :doc:`../ArchitectureInDetail/WebApplicationDetail/ExceptionHandling`\  for the description of \ ``SystemExceptionResolver``\ in detail.

Add \ ``UserRedirectRequiredException``\  to \ ``spring-mvc.xml``\  as the exclusion target of \ ``SystemExceptionResolver``\.

* ``spring-mvc.xml``

.. code-block:: xml

    <bean id="systemExceptionResolver"
        class="org.terasoluna.gfw.web.exception.SystemExceptionResolver">
        
        <!-- omitted -->
    
        <property name="excludedExceptions">
            <array>
                <!-- (1) -->
                <value>org.springframework.security.oauth2.client.resource.UserRedirectRequiredException
                </value>
            </array>
        </property>
    </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Exclude \ ``UserRedirectRequiredException``\  from handling target of \ ``SystemExceptionResolver``\.

|

.. _OAuth2RestTemplateSettings:

Settings of OAuth2RestTemplate
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

How to set \ ``OAuth2RestTemplate``\  for each grant type is explained.

Setting a resource while using authorization code grant
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The setting example of \ ``OAuth2RestTemplate``\  is shown while accessing with authorization code grant.

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:resource id="todoAuthCodeGrantResource" client-id="firstSec"
                     client-secret="firstSecSecret"
                     type="authorization_code"
                     scope="READ,WRITE"
                     access-token-uri="${auth.serverUrl}/oth2/token"
                     user-authorization-uri="${auth.serverUrl}/oth2/authorize"/> <!-- (1) -->

    <oauth2:rest-template id="todoAuthCodeGrantResourceRestTemplate" resource="todoAuthCodeGrantResource" /> <!-- (2) -->


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Define detailed information of the resource to be accessed , to be referred by \ ``OAuth2RestTemplate``\.
        | Refer to the following table for the setting value of each item.
    * - | (2)
      - | Define \ ``OAuth2RestTemplate``\.
        | Specify Bean ID of \ ``OAuth2RestTemplate``\  in \ ``id``\.
        | Specify \ ``id``\  of Bean defined in (1) for \ ``resource``\.

|

     .. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
     .. list-table:: **Resource detailed information**
         :header-rows: 1
         :widths: 35 65
     
         * - Items
           - Description
         * - | \ ``id``\
           - | Bean ID of resource.
         * - | \ ``client-id``\
           - | ID for identifying the client in the Authorization Server.
         * - | \ ``client-secret``\
           - | Password used for authentication of client in the Authorization Server.
         * - | \ ``type``\
           - | Grant type. Specify \ ``authorization_code``\  in case of authorization code grant.
         * - | \ ``scope``\
           - | Enumerate the scope that requires authorization by separating with commas. Setting value is case-sensitive.
             | Request all scopes set for the client in Authorization Server at the time of omitting.
         * - | \ ``access-token-uri``\
           - | URL of endpoint of the Authorization Server for requesting the issue of access token.
         * - | \ ``user-authorization-uri``\
           - | URL of endpoint of the Authorization Server for getting the authorization of resource owner.

.. note::

   In  \ ``<oauth2:resource>``\ tag,  \ ``client-authentication-scheme``\  parameter is provided as the method to specify
   client authentication method at the time of getting access token.
   The values that can be specified in \ ``client-authentication-scheme``\  parameter are as follows.
    
        * \ ``header``\ : Basic authentication using Authorization header. Default value.
        * \ ``query``\  : Authentication using URL query parameter at the time of requesting.
        * \ ``form``\   : Authentication using body parameter at the time of requesting.
    
    In this guideline, since Basic authentication is used for authenticating the client, parameters are not specified in the above-mentioned setting example,
    however, parameters should be specified as per the application requirements.
    
     .. code-block:: xml

        <oauth2:resource id="todoAuthCodeGrantResource" client-id="firstSec"
                         client-secret="firstSecSecret"
                         type="authorization_code"
                         scope="READ,WRITE"
                         access-token-uri="${auth.serverUrl}/oth2/token"
                         user-authorization-uri="${auth.serverUrl}/oth2/authorize"
                         client-authentication-scheme="form" />


.. _OAuth2ResourceOwnerPasswordCredentialGrantResourceSettings:

Settings of resource at the time of using resource owner password credential grant
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| In the resource owner password credential grant, the client requests the issue of access token by using the user name and password of the resource owner.
| There is a need to set the username and password of resource owner as separate parameters in \ ``OAuth2RestTemplate``\,
  however, when multiple resource owners use the same client, there is a need to consider switching the setting contents for each resource owner.
| The method to implement switching of setting contents for each resource owner by setting resource of \ ``OAuth2RestTemplate``\  in the Bean of Session scope and storing the information of resource owner in that Bean
  is explained here.

The setting example of \ ``OAuth2RestTemplate``\  when accessing with resource owner password credential grant, is shown.

Define \ ``ResourceOwnerPasswordResourceDetails``\  in the Session scope and set to \ ``OAuth2RestTemplate``\  to enable switching of setting contents of each resource owner.

* ``oauth2-client.xml``

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:sec="http://www.springframework.org/schema/security"
        xmlns:oauth2="http://www.springframework.org/schema/security/oauth2"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xsi:schemaLocation="
            http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security/oauth2 http://www.springframework.org/schema/security/spring-security-oauth2.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
        ">

        <!-- omitted -->

        <bean id="todoPasswordGrantResource" class="org.springframework.security.oauth2.client.token.grant.password.ResourceOwnerPasswordResourceDetails"
            scope="session"> 
            <aop:scoped-proxy>
            <property name="clientId" value="firstSec" />
            <property name="clientSecret" value="firstSecSecret" />
            <property name="accessTokenUri" value="${auth.serverUrl}/oth2/token" />
            <property name="scope" value="READ,WRITE" />
        </bean> <!-- (1) -->

        <oauth2:rest-template id="todoPasswordGrantResourceRestTemplate" resource="todoPasswordGrantResource" /> <!-- (2) -->



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define detailed information of the resource to be accessed, to be referred by \``OAuth2RestTemplate``\.
        | Refer to the following table for the setting value of each item.
    * - | (2)
      - | Define \``OAuth2RestTemplate``\.
        | Specify Bean ID \  of \ ``OAuth2RestTemplate``\  in ``id``\.
        | Specify \``id``\  of Bean defined in (1) in \ ``resource``\.


|

     .. tabularcolumns:: |p{0.35\linewidth}|p{0.65\linewidth}|
     .. list-table::
         :header-rows: 1
         :widths: 35 65

         * - Item
           - Description
         * - | \ ``class``\
           - | Specify the Bean that acts as a resource of \``OAuth2RestTemplate`` \. Specify \``ResourceOwnerPasswordResourceDetails`` \  here.
         * - | \ ``scope``\
           - | Specify session and consider the scope range as HTTPSession.
         * - | \ ``<aop:scoped-proxy>``\
           - | Set Bean of Session scope for injecting it in \ ``OAuth2RestTemplate``\  that is Bean of Singleton.
             | This is a setting that is necessary because the lifecycle of bean of Singleton is longer than the Bean of Session scope.
             | In order to use this tag, namespace and schema of \ "` aop`` \  is added.
         * - | \ ``clientId``\  property
           - |  Set ID for identifying the client in the Authorization Server for \ ``clientId``\  of Bean.
         * - | \ ``clientSecret``\  property
           - | Set password used for client authentication in the Authorization Server for \ ``clientSecrett``\  of Bean.
         * - | \ ``accessTokenUri``\  property
           - | Specify URL of endpoint of the Authorization Server for requesting the issue of access token.
         * - | \ ``scope``\  property
           - | Set scope list requesting authorization for \ ``scope``\  of Bean.

|

.. note::

    It is assumed that the username and password of resource owner is fetched after it is entered by the resource owner on the client screen when access is required, and is stored in Bean.
    In this guideline, the explanation of how to fetch username and password is omitted.

.. warning::

    In the Authorization Server described in this guideline, the password set in \`` ResourceOwnerPasswordResourceDetails``\  must be in plain text in order to perform comparative verification after hashing the password used for authentication.
    Since there is high risk in handling the password of resource owner in plain text by the client, the resource owner password credential grant
    should be used in very limited cases such as when there is highly trustworthy relationship between client and resource owner and the client is kept in secure environment.
    Refer to \ :ref:`OAuthAuthorizationServerClientAuthentication`\  for the settings of hashing in Authorization Server.

|

Settings of resource at the time of using client credential grant
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The setting example of \ ``OAuth2RestTemplate``\  is shown while accessing with client credential grant.

The explanation of settings common to authorization code grant is omitted.

* ``oauth2-client.xml``

.. code-block:: xml

    <oauth2:resource id="todoClientGrantResource" client-id="firstSecClient"
                    client-secret="firstSecSecret"
                    type="client_credentials"
                    access-token-uri="${auth.serverUrl}/oth2/token" /> <!-- (1) -->

    <oauth2:rest-template resource=id="todoClientGrantResourceRestTemplate" resource="todoClientGrantResource" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Specify the grant type in \ ``type``\  attribute. Specify \ ``client_credentials``\  in case of client credential grant.


Accessing a resource Server
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

How to access the Resource Server using \ ``OAuth2RestTemplate``\  is explained.

Since access to Authorization Server is hidden by \ ``OAuth2RestTemplate``\  and \ ``OAuth2ClientContextFilter``\,
the process to be performed for REST API provided by the Resource Server is described similar to normal access to REST API without being aware of the Authorization Server at the time of development.

The implementation example of Service class is shown below.

.. code-block:: java

    import org.springframework.web.client.RestOperations;

    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        @Named("todoAuthCodeGrantResourceRestTemplate")
        RestOperations restOperations; // (1)

        @Value("${resource.serverUrl}/api/v1/todos")
        String uri;

        @Override
        public List<Todo> getTodoList() {
            Todo[] todoArray = restOperations.
                getForObject(url, Todo[].class); // (2)
            return Arrays.asList(todoArray);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Inject \ ``RestOperations``\.
        | In the above-mentioned example, the Bean whose Bean ID is \ ``todoAuthCodeGrantResourceRestTemplate``\  is Injected. \ ``@Named``\  should be specified when multiple \ ``OAuth2RestTemplate``\  are defined.
    * - | (2)
      - | Access the specified URL with GET method in REST and receive the result in a list.

.. note::

    When access token is not issued if \ ``OAuth2RestTemplate``\  is used to access the Resource Server,
    it is redirected to the Authorization Server at once.
    Thereafter, when the issue of access token is complete, it is redirected to the client and access process to the resource server is called again.
    At this time, since redirect to client is performed by GET, the Controller that may be redirected from Authorization Server, must allow GET.
    
    If access to the Resource Server before the redirect is POST, the parameter will be lost in GET after redirect.
    In that case, a measure like retaining POST parameter in session, should be taken.
    In this guideline, the explanation of specific handling method is omitted.

|

.. _OAuthClientUsingJavaScript:

Accessing a resource using JavaScript
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the implicit grant, user agent such as Web browser requests authorization for the Authorization Server.

Therefore, implicit grant is usually used in JavaScript that runs on Web browser, however
since JavaScript library is not provided in Spring Security OAuth, client should be implemented independently
when using implicit grant.

In this guideline, how to fetch data in JSON format from the Resource Server using JavaScript and display it on the screen is explained
as the implementation of client for implicit grant.

.. note::

    Use jQuery as JavaScript library in the implementation example to be explained later.
    It is assumed to be stored under \ ``src/main/webapp``\  with js file.


API example wherein OAuth 2.0 function is implemented independently, is shown.

* ``oauth2.js``

.. code-block:: js

    var oauth2Func = (function(exp, $) {
        "use strict";
    
        var
            config = {},
            DEFAULT_LIFETIME = 3600;
    
        var uuid = function() {
            return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function(c) {
                var r = Math.random()*16|0, v = c == "x" ? r : (r&0x3|0x8);
                return v.toString(16);
            });
        };
    
        var encodeURL= function(url, params) {
            var res = url;
            var k, i = 0;
            for(k in params) {
                res += (i++ === 0 ? "?" : "&") + encodeURIComponent(k) + "=" + encodeURIComponent(params[k]);
            }
            return res;
        };
    
        var epoch = function() {
            return Math.round(new Date().getTime()/1000.0);
        };
    
        var parseQueryString = function (qs) {
            var e,
                a = /\+/g,
                r = /([^&;=]+)=?([^&;]*)/g,
                d = function (s) { return decodeURIComponent(s.replace(a, " ")); },
                q = qs,
                urlParams = {};
    
            while (e = r.exec(q)) {
               urlParams[d(e[1])] = d(e[2]);
            }
    
            return urlParams;
        };
    
        var saveState = function(state, obj) {
            localStorage.setItem("state-" + state, JSON.stringify(obj));
        };
    
        var getState = function(state) {
            var obj = JSON.parse(localStorage.getItem("state-" + state));
            localStorage.removeItem("state-" + state);
            return obj;
        };
    
        var hasScope = function(token, scope) {
            if (!token.scopes) return false;
            var i;
            for(i = 0; i < token.scopes.length; i++) {
                if (token.scopes[i] === scope) return true;
            }
            return false;
        };
    
        var filterTokens = function(tokens, scopes) { // (1)
            if (!scopes) scopes = [];
    
            var i, j,
            result = [],
            now = epoch(),
            usethis;
            for(i = 0; i < tokens.length; i++) {
                usethis = true;
    
                if (tokens[i].expires && tokens[i].expires < (now+1)) usethis = false;
    
                for(j = 0; j < scopes.length; j++) {
                    if (!hasScope(tokens[i], scopes[j])) usethis = false;
                }
    
                if (usethis) result.push(tokens[i]);
            }
            return result;
        };
    
        var saveTokens = function(provider, tokens) {
            localStorage.setItem("tokens-" + provider, JSON.stringify(tokens));
        };
    
        var getTokens = function(provider) {
            var tokens = JSON.parse(localStorage.getItem("tokens-" + provider));
            if (!tokens) tokens = [];
    
            return tokens;
        };
    
        var wipeTokens = function(provider) {
            localStorage.removeItem("tokens-" + provider);
        };
    
        var saveToken = function(provider, token) {
            var tokens = getTokens(provider);
            tokens = filterTokens(tokens);
            tokens.push(token);
            saveTokens(provider, tokens);
        };
    
        var getToken = function(provider, scopes) {
            var tokens = getTokens(provider);
            tokens = filterTokens(tokens, scopes);
            if (tokens.length < 1) return null;
            return tokens[0];
        };
    
        var sendAuthRequest = function(providerId, scopes) { // (2)
            if (!config[providerId]) throw "Could not find configuration for provider " + providerId;
            var co = config[providerId];
    
            var state = uuid();
            var request = {
                "response_type": "token"
            };
            request.state = state;
    
            if (co["redirectUrl"]) {
                request["redirect_uri"] = co["redirectUrl"];
            }
            if (co["clientId"]) {
                request["client_id"] = co["clientId"];
            }
            if (scopes) {
                request["scope"] = scopes.join(" ");
            }
    
            var authurl = encodeURL(co.authorization, request);
    
            if (window.location.hash) {
                request["restoreHash"] = window.location.hash;
            }
            request["providerId"] = providerId;
            if (scopes) {
                request["scopes"] = scopes;
            }
    
            saveState(state, request);
            redirect(authurl);
    
        };
    
        var checkForToken = function(providerId) { // (3)
            var h = window.location.hash;
    
            if (h.length < 2) return true;
    
            if (h.indexOf("error") > 0) { // (4)
                h = h.substring(1);
                var errorinfo = parseQueryString(h);
                handleError(providerId, errorinfo);
                return false;
            }
    
            if (h.indexOf("access_token") === -1) {
                return true;
            }
            h = h.substring(1);
            var atoken = parseQueryString(h);
    
            if (!atoken.state) {
                return true;
            }
    
            var state = getState(atoken.state);
            if (!state) throw "Could not retrieve state";
            if (!state.providerId) throw "Could not get providerId from state";
            if (!config[state.providerId]) throw "Could not retrieve config for this provider.";
    
            var now = epoch();
            if (atoken["expires_in"]) {
                atoken["expires"] = now + parseInt(atoken["expires_in"]);
            } else {
                atoken["expires"] = now + DEFAULT_LIFETIME;
            }
    
            if (atoken["scope"]) {
                atoken["scopes"] = atoken["scope"].split(" ");
            } else if (state["scopes"]) {
                atoken["scopes"] = state["scopes"];
            }
    
            saveToken(state.providerId, atoken);
    
            if (state.restoreHash) {
                window.location.hash = state.restoreHash;
            } else {
                window.location.hash = "";
            }
            return true;
        };
    
        var handleError = function(providerId, cause) { // (5)
            if (!config[providerId]) throw "Could not retrieve config for this provider.";
    
            var co = config[providerId];
            var errorDetail = cause["error"];
    
            // redirect error page
            if(co["errRedirectUrl"]) {
                redirect(co["errRedirectUrl"] + "/" + errorDetail);
            } else {
                alert("Access Error. cause: " + errorDetail);
            }
        };
    
    
        var redirect = function(url) {
            window.location = url;
        };
    
        var initialize = function(c) {
            config = c;
            try {
                var key, providerId;
                for(key in c) {
                        providerId = key;
                }
                return checkForToken(providerId);
            } catch(e) {
                console.log("Error when retrieving token from hash: " + e);
                window.location.hash = "";
                return false;
            }
        };
    
        var clearTokens = function() {
            var key;
            for(key in config) {
                wipeTokens(key);
            }
        };
    
        var oajax = function(settings) { // (6)
            var providerId = settings.providerId;
            var scopes = settings.scopes;
            var token = getToken(providerId, scopes);
    
            if (!token) {
                sendAuthRequest(providerId, scopes);
                return;
            }
    
            if (!settings.headers) settings.headers = {};
            settings.headers["Authorization"] = "Bearer " + token["access_token"];
    
            $.ajax(settings);
        };
    
        return {
            initialize: function(config) {
                return initialize(config);
            },
            clearTokens: function() {
                return clearTokens();
            },
            oajax: function(settings) {
                return oajax(settings);
            }
        };
    
    })(window, jQuery);


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Function to check the expiry and scope availability of access token.
        | The client returns the available access token.
    * - | (2)
      - | Function to request authorization of the Authorization Server.
        | Fetch the required parameters from the configuration information and create a request.
    * - | (3)
      - | Function to get access token from authorization response.
        | When access token can be fetched, store the information in the local storage.
    * - | (4)
      - | As the result of authorization, when an error is returned, call \ ``handleError``\  as error handling.
    * - | (5)
      - | Function to handle authorization error.
        | In this guideline, redirect to redirect destination URL is carried out at the time of error specified in configuration information
          as the implementation example of error handling.
    * - | (6)
      - | Function to request access resource to the Resource Server.
        | Fetch access token from local storage and send request to the Resource Server using ajax function of jQuery.

|

The implementation example of client is shown below.

* ``todoList.jsp``

.. code-block:: jsp

    <script type="text/javascript" src="${pageContext.request.contextPath}/resources/vendor/jquery/jquery.js"></script> <!-- (1) -->
    <script type="text/javascript" src="${pageContext.request.contextPath}/resources/app/js/oth2-implicit.js"></script> <!-- (1) -->
    <script type="text/javascript">
    "use strict";
    
    $(document).ready(function() {
        var result = oauth2Func.initialize({ // (2)
            "todo" : { // (3)
                clientId : "client", // (4)
                redirectUrl : "${client.serverUrl}/oth2/api", // (5)
                errRedirectUrl : "${client.serverUrl}/oth2/error", // (6)
                authorization : "${auth.serverUrl}/oth2/authorize" // (7)
            }
        });
    
        if (result) {
            oauth2Func.oajax({ // (8)
                url : "${resource.serverUrl}/api/v1/todos", // (9)
                providerId : "todo",  // (10)
                scopes : [ "READ" ],  // (11)
                dataType : "json",    // (12)
                type : "GET",  // (13)
                success : function(data) { // (14)
                    $("#message").text(JSON.stringify(data));
                },
                error : function() {
                    oauth2Func.clearTokens();
                }
            });
        } else {
            oauth2Func.clearTokens(); // (15)
        }
    
    
    };
    
    </script>
    <div id="wrapper">
        <p id="message"></p>
    </div>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90
    :class: longtable

    * - Sr. No.
      - Description
    * - | (1)
      - | As described later, specify the path storing js file, jQuery for which OAuth 2.0 function is implemented individually.
    * - | (2)
      - | Define configuration information to be used for authorization request and initialize the same.
    * - | (3)
      - | Specify a unique value as an identifier for distinguishing the configuration information of each client.
        | In the process of accessing resources described later, manage and fetch configuration information by using this item as a key.
    * - | (4)
      - | Specify ID identifying the client.
    * - | (5)
      - | Specify URL to redirect the client after authenticating the resource owner of authorization server.
        | In this implementation example, it is assumed that parameters of URL are received from the controller.
    * - | (6)
      - | Specify URL for redirecting when an error is received from authorization server as the authorization response.
        | In this implementation example, it is assumed that parameters of URL are received from the controller.
        | These guidelines show the method of redirecting to client screen and displaying the error screen, as an example of implementation when error is received.
          Description about client Controller is omitted.
    * - | (7)
      - | Specify authorization end point of authorization server.
        | In this implementation example, it is assumed that parameters of URL are received from the controller.
    * - | (8)
      - | Access the resource.
    * - | (9)
      - | Specify access destination of resource server.
    * - | (10)
      - | Specify identifier of configuration information to be referred.
    * - | (11)
      - | Specify scope to be requested to the resource. Setting value is classified by upper case character and lower chase character.
    * - | (12)
      - | Specify response type.
    * - | (13)
      - | Access resource server GET method.
    * - | (14)
      - | Specify process when processing is successful. Response after successful processing is stored in \ ``message``\.
    * - | (15)
      - | Clear the access token.

|

 .. todo:: **TBD**

    When authorization servers and resource server that are not in the same domain are accessed from the clients created in JavaScript, support of \ ``Cross-Origin Resource Sharing``\  is required on authorization server and resource server.

    Details will be described in the next versions.

.. _OAuthClientServerHowToCancelToken:

Cancellation of token (Client server)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Implementation method of cancelling the issued token is described.

| For cancelling the access token, request is made to authorization server and access token is deleted from  \ ``TokenStore``\. Request header of Basic authentication is set in order to perform Basic authentication of client when request is made to authorization server.
| Refer to \ :ref:`OAuthAuthorizationServerHowToCancelToken`\  , for the cancellation of tokens in authorization server.
| Access token retained in \ ``OAuth2RestTemplate``\  should be deleted after requesting the cancellation of access token to authorization server by the client server.


Implementation example is given below.

At first, settings of \ ``RestTemplate``\  are added in configuration file in order to request cancelation of token to authorization server.

* ``oauth2-client.xml``

.. code-block:: xml
    
    <!-- (1) -->
    <bean id="revokeRestTemplate" class="org.springframework.web.client.RestTemplate">
        <property name="interceptors">
            <list>
                <ref bean="basicAuthInterceptor" />
            </list>
        </property>
    </bean>
    
    <bean id="basicAuthInterceptor" class="com.example.oauth2.client.restclient.BasicAuthInterceptor" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``RestTemplate``\  in order to request the cancellation of token to authorization server.
        | Implementation class \ ``ClientHttpRequestInterceptor``\  is specified in \ ``interceptors``\  property, in order to set request header for basic authentication.
        | Refer to \ :ref:`RestClientHowToExtendClientHttpRequestInterceptorBasicAuthentication`\ , for the implementation details of \ ``ClientHttpRequestInterceptor``\  implementation class that sets the request header of Basic authentication.


Interface of service class and implementation class are created for cancelling the token.

* ``RevokeTokenClientService.java``

.. code-block:: java

    public interface RevokeTokenClientService {
        
        String revokeToken();
        
    }

* ``RevokeTokenClientServiceImpl.java``

.. code-block:: java

    @Service
    public class RevokeTokenClientServiceImpl implements RevokeTokenClientService {
    
        @Value("${auth.serverUrl}/api/v1/oth2/tokens/revoke")
        String revokeTokenUrl; // (1)
        
        @Inject
        @Named("todoAuthCodeGrantResourceRestTemplate")
        OAuth2RestOperations oauth2RestOperations; // (2)
        
        @Inject
        @Named("revokeRestTemplate")
        RestOperations revokeRestOperations; // (3)
        
        @Override
        public String revokeToken() {
            
            String token = getTokenValue(oauth2RestOperations);
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
            MultiValueMap<String, String> variables = new LinkedMultiValueMap<String, String>();
            variables.add("token", token);
            
            String result = revokeRestOperations.postForObject(revokeTokenUrl,
                new HttpEntity<MultiValueMap<String, String>>(variables, headers),
                String.class); // (4)
            // (5)
            if ("success".equals(result)) {
                initContextToken(oauth2RestOperations);
            }
            return result;
        }
    
        // (6)
        private String getTokenValue(OAuth2RestOperations oauth2RestOperations) {
            String tokenValue = "";
            OAuth2AccessToken token = oauth2RestOperations.getAccessToken();
            if (token != null) {
                tokenValue = token.getValue();
            }
            return tokenValue;
        }
    
        // (7)
        private void initContextToken(OAuth2RestOperations oauth2RestOperations) {
            oauth2RestOperations.getOAuth2ClientContext().setAccessToken(null);
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | URL to be used while requesting the cancellation of access token to the authorization server.
    * - | (2)
      - | Inject \ ``OAuth2RestTemplate``\  which retains the access token to be cancelled.
    * - | (3)
      - | Inject \ ``RestTemplate``\  for cancelling the access token.
    * - | (4)
      - | In order to cancel the access token in authorization server, it is accessed with method POST in REST.
        | Set value of access token to be cancelled in the request parameter for passing it to the authorization server.
        | Access token is fetched by passing \ ``OAuth2RestOperations``\  to \ ``getTokenValue``\  method defined in (5).
    * - | (5)
      - | Process result of authorization server is determined and only when it is normal, the access token retained by \``OAuth2RestOperations``\  is deleted.
        | Access token is deleted by passing \ ``OAuth2RestOperations``\  retaining the access token in \ ``initContextToken``\  method defined in (6).
    * - | (6)
      - | Method of getting the access token retained by \ ``OAuth2RestOperations``\  .
        | Access token is fetched and returned by calling the \ ``getAccessToken``\  method of \ ``OAuth2RestOperations``\  passed as a parameter.
    * - | (7)
      - | Method of deleting the access token retained by \ ``OAuth2RestOperations``\  .
        | Delete the access token by passing the null in \ ``setAccessToken``\  method of \ ``OAuth2RestOperations``\  passed as a parameter.



| Client cancels the token by calling the service created above when the access token is not required.
| Since the default implementation of Spring Security OAuth retains the access token in scope of session, the token seems to be cancelled when client user logs-out or when session is revoked due to session timeout.


.. _OAuthHowToExtend:

How to extend
--------------------------------------------------------------------------------

Linking Authorization Server and Resource Server through endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Resource Server and Authorization Server can be linked by performing HTTP access from the Resource Server to the check token endpoint of the Authorization Server.

Check token endpoint is the endpoint that receives the value of access token from the Resource Server and verifies the token instead of the Resource Server.
Check token endpoint fetches and verifies the access token using \``TokenServices``\, and passes the information associated with the access token to the Resource Server when there is no problem in the verification.


.. _OAuthAuthorizationServerHowToCooperateWithHttp:

Linkage through HTTP access
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Use \``org.springframework.security.oauth2.provider.token.RemoteTokenServices`` \  that is the implementation class of \ `` TokenServices`` \  for the Resource Server to access the check token endpoint of the Authorization Server.
| \``RemoteTokenServices``\  performs HTTP access of check token endpoint using \ ``org.springframework.web.client.RestTemplate``\  and fetches the information associated with the access token.
| Since the access token is verified by the check token endpoint, it is not verified by \ ``RemoteTokenServices``\.


The implementation example is shown below.

First, perform the settings for registering \ ``org.springframework.security.oauth2.provider.endpoint.CheckTokenEndpoint``\ class as component for verifying token in the Authorization Server.

* ``oauth2-auth.xml``

.. code-block:: xml

        <sec:http pattern="/oth2/check-token" security="none" />  <!-- (1) -->

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices"
             check-token-enabled="true"
             check-token-endpoint-url="/oth2/check-token">  <!-- (2) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | In this guideline, it is assumed that access to the check token endpoint is by network configuration that allows only the Resource Server and is excluded from the security function (Security Filter) of Spring Security.
        | It should be mentioned at the beginning of the configuration file so that it will be evaluated before any other security settings.
    * - | (2)
      - | \ ``CheckTokenEndpoint``\  is registered as the component by specifying \ ``true``\  in \ ``check-token-enabled``\  attribute of \ ``<oauth2:authorization-server>tag``\.
        | Specify URL of \ ``CheckTokenEndpoint``\  in \ ``check-token-endpoint-url``\  attribute of \ ``<oauth2:authorization-server>tag``\. When it is not specified, the default value "/oauth/check_token" is specified.


.. warning:: **Security measures of check token endpoint**

  - Check token endpoint is accessible only between Authorization Server and Resource Server so it cannot be accessed from open network.
    When disclosing the check token endpoint, it is necessary to take appropriate security measures.


.. note::

    Note that when Spring Security OAuth 2.0.12 or earlier version is used and \ ``authorization-endpoint-url``\  or \ ``token-endpoint-url``\ is not specified in \ ``check-token-endpoint-url``\, it will not be reflected.
    It is addressed in the following issue and is expected to be fixed in Version 2.0.13.
    
    https://github.com/spring-projects/spring-security-oauth/issues/897


.. note::

    Use \ ``DefaultTokenServices``\  same when linking  \ ``TokenServices``\  through shared DB.
    \ ``TokenStore``\  referred by \ ``TokenServices``\  uses the implementation class of the interface as per the application requirements.

| 

Perform the settings to use \ ``org.springframework.security.oauth2.provider.token.RemoteTokenServices``\  as \ ``TokenServices``\  in the configuration file of the Resource Server. 

* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.RemoteTokenServices">
            <property name="checkTokenEndpointUrl" value="${auth.serverUrl}/oth2/check-token" />  <!-- (1) -->
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define Bean for \ ``RemoteTokenServices``\  to enable fetching of information associated with the access token by accessing the check token endpoint of the Authorization Server.
        | Set the URL in \ ``checkTokenEndpointUrl``\  property to access check token endpoint.


.. note::

    When verification error of access token occurs in the check token endpoint, HTTP status code 400(Bad Request) is returned in \ ``RemoteTokenServices``\.
    When HTTP status code 400(Bad Request) is returned in the default implementation of \ ``RestTemplate``\, error handling is done and client error exception is generated.
    \ ``RestTemplate``\ to be used by \ ``RemoteTokenServices``\  by default, is extended so that error handling is not performed when HTTP status code of response is 400 in order to link verification error of the access token to the client server.
    When \ ``RestTemplate``\  is injected in \ ``RemoteTokenServices``\, note that this extension will not be applied.

| 

When \ ``RemoteTokenServices``\  is used in the Resource Server, the username of the resource owner can be fetched by specifying \ ``@AuthenticationPrincipal``\  annotation in the handler method as the argument annotation in \ ``String``\.

The implementation example is as follows.

.. code-block:: java

    @RestController
    @RequestMapping("api")
    public class TodoRestController {
        
        // omitted

        @RequestMapping(value = "todos", method = RequestMethod.GET)
        @ResponseStatus(HttpStatus.OK)
        public Collection<Todo> list(@AuthenticationPrincipal String userName) { // (1)
        
            // omitted
        
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | \ Username of resource owner is stored in argument \ ``userName``\.


| 

.. _OAuthAuthorizationServerHowToGetOtherThanUserName:

Extension of DefaultAccessTokenConverter
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Even in case of the structure where DB is not shared between Authorization Server and Resource Server, there may be a case to link the item corresponding to the user information from the Authorization Server to the Resource Server, however
when \ ``RemoteTokenServices``\  is to be used as \ ``TokenServices``\  by the Resource Server, the information other than the user information cannot be fetched by the annotation \ ``@AuthenticationPrincipal``\ of handler method argument of the Resource Server.

An example of extending \ ``org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter``\ that is a class to be used for linking access token by using \ ``RemoteTokenServices``\
and linking the information other than the username to the Resource Server is shown here.

DefaultAccessTokenConverter means
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Request authentication information of resource owner and client of the access token value to the Authorization Server from the Resource Server by using \ ``RestTemplate``\ and fetch the result as Map for linking access token wherein \ ``RemoteTokenServices``\  is used.
\ ``DefaultAccessTokenConverter``\  is used as a convertor that converts the authentication information in Authorization Server to Map and Map in Resource Server to the authentication information.

By using this, the parameters linked between the Authorization Server and the Resource Server can be customized by extending \``DefaultAccessTokenConverter``\  so as to add the return value from the Authorization Server to the Map.

In the following explanation, an example of linking independent items related to user information and independent items other than this to each other by customizing \``DefaultAccessTokenConverter`` \ and its property \``DefaultUserAuthenticationConverter`` \  in the Authorization Server, is shown.


Implementation of Authorization Server
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

How to implement Authorization Server is explained.

First, extend \ ``DefaultUserAuthenticationConverter``\  for adding independent item of the user information.

* ``AuthCustomUserTokenConverter.java``

.. code-block:: java

    public class CustomUserTokenConverter extends DefaultUserAuthenticationConverter {
        @Override
        public Map<String, ?> convertUserAuthentication(
                Authentication authentication) {
            Map<String, Object> response = new LinkedHashMap<String, Object>();
            response.put(USERNAME, authentication.getName());
            
            if (authentication.getAuthorities() != null &&
                    !authentication.getAuthorities().isEmpty()) {
                response.put(AUTHORITIES, AuthorityUtils.authorityListToSet(
                        authentication.getAuthorities()));
            }
            response.put("user_additional_key", "user_additional_value"); // (1)
            return response;
        }
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Define the information to be passed to the Resource Server as an independent item \ ``user_additional_key``\  and set to \ ``response``\.
        | The information set in \ ``response``\ , is returned to the Resource Server in JSON format as response BODY at the time of verifying the token of check token endpoint.


Then, extend \ ``DefaultAccessTokenConverter``\  for adding independent item of other than the user information.

* ``CustomAccessTokenConverter.java``

.. code-block:: java

        public class CustomAccessTokenConverter extends DefaultAccessTokenConverter {
        
            @Override
            public Map<String, ?> convertAccessToken(OAuth2AccessToken token, OAuth2Authentication authentication) {
                
                @SuppressWarnings("unchecked")
                Map<String, Object> response = (Map<String, Object>) super.convertAccessToken(token, authentication);
                response.put("client_additional_key","client_additional_value"); // (1)
                // omitted
                
                return response;
            }
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define the information to be passed to the Resource Server as an independent item \ ``client_additional_key``\  and set to \ ``response``\.
        | The information set in \ ``response``\ , is returned to the Resource Server in JSON format as response BODY at the time of verifying the token of check token endpoint.

Perform the settings of \ ``CustomUserTokenConverter``\  and \ ``CustomAccessTokenConverter``\  created in the configuration file of the Authorization Server.

* ``oauth2-auth.xml``

.. code-block:: xml

        <oauth2:authorization-server
             client-details-service-ref="clientDetailsService"
             token-endpoint-url="/oth2/token"
             authorization-endpoint-url="/oth2/authorize"
             user-approval-handler-ref="userApprovalHandler"
             token-services-ref="tokenServices"
             check-token-endpoint-url="/oth2/check-token">  <!-- (1) -->
            <oauth2:authorization-code />
            <oauth2:implicit />
            <oauth2:refresh-token />
            <oauth2:client-credentials />
            <oauth2:password />
        </oauth2:authorization-server>

        <bean id="checkTokenEndpoint"
            class="org.springframework.security.oauth2.provider.endpoint.CheckTokenEndpoint">  <!-- (2) -->
            <constructor-arg ref="tokenServices" />
            <property name="accessTokenConverter" ref="accessTokenConverter" />
        </bean>

        <bean id="accessTokenConverter"
            class="com.example.oauth2.auth.converter.CustomAccessTokenConverter"/>  <!-- (3) -->
            <property name="userTokenConverter">
                <bean
                    class="com.example.oauth2.auth.converter.CustomUserTokenConverter" />
            </property>
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the URL of \``CheckTokenEndpoint``\  defined in (2) in the \ ``check-token-endpoint-url``\  attribute of \``<oauth2: authorization-server>``\ tag. When not specified, the default value "/ oauth / check_token" is specified.
        | Since Bean definition for \``CheckTokenEndpoint``\  is done independently in (2), do not specify \``check-token-enabled``\ attribute of \``<oauth2:authorization-server>``\ tag.          
    * - | (2)
      - | Define Bean for \ ``CheckTokenEndpoint``\.
        | By specifying Bean of \ ``CustomAccessTokenConverter``\  defined in (2) in \ ``accessTokenConverter``\ property, independent items added in \ ``CustomAccessTokenConverter``\  and \ ``CustomUserTokenConverter``\  can be linked to the Resource Server.
    * - | (3)
      - | Define Bean for \ ``CustomAccessTokenConverter``\  wherein \ ``DefaultAccessTokenConverter``\  is extended.
        | Specify Bean for \ ``CustomUserTokenConverter``\  wherein \ ``DefaultUserAuthenticationConverter``\  is extended in \ ``userTokenConverter``\  property.


Implementation of Resource Server
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Add the function to fetch the information linked from Authorization Server in the Resource Server by annotation \ ``@AuthenticationPrincipal``\  of the handler method argument.
First, create \ ``OauthUser``\  class holding the information to be fetched by annotation \ ``@AuthenticationPrincipal``\.

* ``OauthUser.java``

.. code-block:: java

        public class OauthUser implements Serializable{
        
            private static final long serialVersionUID = 1L;
            
            private String username;
            
            private String userAdditionalValue;
            
            private String clientAdditionalValue;
            
            // omitted
            
            public User(String username, String additionalValue){
                this.username = username;
                this.additionalValue = additionalValue;
            }
            
            // Getters and Setters are omitted
            
        }

\ ``org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter``\  which performs setup for enabling fetching of user information by \ ``@AuthenticationPrincipal``\  annotationverter``\  is expanded and a function is added to enable fetching of information other than user name.

* ``CustomUserTokenConverter.java``

.. code-block:: java

        public class CustomUserTokenConverter extends DefaultUserAuthenticationConverter{
            
            private Collection<? extends GrantedAuthority> defaultAuthorities; // (1)
            
            public void setDefaultAuthorities(String[] defaultAuthorities) {
                this.defaultAuthorities = AuthorityUtils.commaSeparatedStringToAuthorityList(StringUtils
                        .arrayToCommaDelimitedString(defaultAuthorities));
            }
            
             // (2)
            @Override
            public Authentication extractAuthentication(Map<String, ?> map) {
                if (map.containsKey(USERNAME)) {
                    Collection<? extends GrantedAuthority> authorities = getAuthorities(map);
                    OauthUser user = new OauthUser(
                            (String) map.get(USERNAME),
                            (String) map.get("user_additional_key"),
                            (String) map.get("client_additional_key"),
                            (String) map.get("client_id")); // (3)
                    
                    // omitted
                    
                    return new UsernamePasswordAuthenticationToken(user, "N/A", authorities); // (4)
                }
                return null;
            }
            
            private Collection<? extends GrantedAuthority> getAuthorities(Map<String, ?> map) {
                if (!map.containsKey(AUTHORITIES)) {
                    return defaultAuthorities;
                }
                Object authorities = map.get(AUTHORITIES);
                if (authorities instanceof String) {
                    return AuthorityUtils.commaSeparatedStringToAuthorityList((String) authorities);
                }
                if (authorities instanceof Collection) {
                    return AuthorityUtils.commaSeparatedStringToAuthorityList(StringUtils
                            .collectionToCommaDelimitedString((Collection<?>) authorities));
                }
                throw new IllegalArgumentException("Authorities must be either a String or a Collection");
            }
        }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Implement \ ``defaultAuthorities``\  used by \ ``defaultAuthorities``\  and \ ``getAuthorities``\  method since \ ``getAuthorities``\  method implemented by \ ``DefaultUserAuthenticationConverter``\  is defined by private.
    * - | (2)
      - | A method that extracts authentication information from the information linked from authorization server.
    * - | (3)
      - | Set information linked from authorization server in \ ``OauthUser``\  class.
    * - | (4)
      - | Information linked from authorization server can be fetched \ ``@AuthenticationPrincipal``\  annotation by setting \ ``OauthUser``\  in the first argument of \ ``UsernamePasswordAuthenticationToken``\.



Configure \ ``CustomUserTokenConverter``\  in setup file of resource server.

* ``oauth2-resource.xml``

.. code-block:: xml

        <bean id="tokenServices"
            class="org.springframework.security.oauth2.provider.token.RemoteTokenServices">
            <property name="checkTokenEndpointUrl" value="${auth.serverUrl}/oth2/check-token" />
            <property name="accessTokenConverter" ref="accessTokenConverter" />
        </bean>
        
        <bean id="accessTokenConverter"
            class="org.springframework.security.oauth2.provider.token.DefaultAccessTokenConverter">
            <property name="userTokenConverter">
                <bean class="com.example.oauth2.resource.converter.CustomUserTokenConverter"/>  <!-- (1) -->
            </property>
        </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Information other than user name can be fetched  by \ ``@AuthenticationPrincipal``\  annotation, by specifying \ ``CustomUserTokenConverter``\  class in \ ``userTokenConverter``\  property of \ ``accessTokenConverter``\.



