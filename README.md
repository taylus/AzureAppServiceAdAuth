# Azure App Service + Azure Active Directory

A reference, since I always seem to mess the configuration of this up at some point...

This demo uses the [client credentials](https://oauth.net/2/grant-types/client-credentials/) grant type which requires keeping a secret and is suited for machine-to-machine authentication to the service. [More info on different grant types](https://oauth.net/2/grant-types/).

## Deploy and configure

Azure Portal UI specifics likely to change the older this gets, but the general idea should apply. (TODO: automation?)

1. [Publish this web app as an App Service](https://docs.microsoft.com/en-us/visualstudio/deployment/quickstart-deploy-to-azure)

1. Once published, `GET https://{appservicename}.azurewebsites.net` should succeed w/ HTTP 200 OK ✔️

1. Enable App Service Authentication in the Azure Portal:
   1. Azure Portal -> App Services -> this one -> Settings -> Authentication / Authorization
   1. App Service Authentication -> On
   1. Action to take when request is not authenticated -> Log in with Azure Active Directory
   1. Authentication Providers -> Azure Active Directory -> Management mode -> Express
      1. Management mode -> Create New AD App
      1. All other settings leave default
      1. Click OK
   1. Click Save to update the App Service

1. At this point, `GET https://{appservicename}.azurewebsites.net` should fail w/ HTTP 401 Unauthorized ❌

## Authenticate via client_credentials token

1. Get a token:
   1. Get the directory ID for your Azure AD:
      1. Azure Portal -> Azure Active Directory -> Properties -> Directory ID
   1. Get the client ID + client secret for the app service you just deployed
      1. Azure Portal -> App Services -> this one -> Settings -> Authentication / Authorization -> Authentication Providers -> Azure Active Directory -> Advanced
   1. Send a `POST` request to `https://login.microsoftonline.com/{directoryId}/oauth2/v2.0/token` w/ a body of content type `x-www-form-urlencoded` consisting of the following:
      1. client_id={client ID determined above}
      1. client_secret={client secret determined above}
      1. grant_type=client_credentials
      1. scope={client ID determined above}/.default
   1. POST should return HTTP 200 OK w/ a JSON body containing an `access_token` field, copy it

1. Supply the token in an `Authorization` header of the `GET` request to the service:
   1. E.g. in Postman: Request -> Authorization tab -> Type -> Bearer Token -> paste
   1. For later: Postman can [get the token for you](https://learning.postman.com/docs/postman/sending-api-requests/authorization/#oauth-20) as a logical next step

1. At this point, `GET https://{appservicename}.azurewebsites.net` should succeed w/ HTTP 200 OK as long as a valid, unexpired token is provided in the `Authorization: Bearer {token}` header. ✔️

## References

* <https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-aad>
* <https://blogsofshri.wordpress.com/2018/05/22/securing-azure-api-apps-with-azure-ad/>
* <https://stackoverflow.com/questions/34842895/difference-between-grant-type-client-credentials-and-grant-type-password-in-auth>
* <https://learning.postman.com/docs/postman/sending-api-requests/authorization/>