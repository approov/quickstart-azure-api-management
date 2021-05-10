# Approov Token Quickstart

This quickstart is for developers familiar with Azure who are looking for a quick intro into how they can add [Approov](https://approov.io) into an existing project. Therefore this will guide you through the necessary steps for adding Approov to an existing Azure API management platform.


## TOC - Table of Contents

* [Why?](#why)
* [How it Works?](#how-it-works)
* [Requirements](#requirements)
* [Approov Setup](#approov-setup)
* [Approov Token Check](#approov-token-check)
* [Test your Approov Integration](#test-your-approov-integration)


## Why?

To lock down your API server to your mobile app. Please read the brief summary in the [README](/README.md#why) at the root of this repo or visit our [website](https://approov.io/product.html) for more details.

[TOC](#toc---table-of-contents)


## How it works?

For more background, see the overview in the [README](/README.md#how-it-works) at the root of this repo.


[TOC](#toc---table-of-contents)


## Requirements

To complete this quickstart you will need to already have an Azure API management platform created, and the Approov CLI tool installed.

* [Azure API Management Platform](https://docs.microsoft.com/en-us/azure/api-management/import-and-publish)
* [Approov CLI](https://approov.io/docs/latest/approov-installation/#approov-tool) - Learn how to use it [here](https://approov.io/docs/latest/approov-cli-tool-reference/)

[TOC](#toc---table-of-contents)


## Approov Setup

To use Approov with the Azure API management platform we need a small amount of configuration. First, Approov needs to know the API domain that will be protected. Second, the Azure API management platform needs the Approov Base64 encoded secret that will be used to verify the tokens generated by the Approov cloud service.

### Configure API Domain

Approov needs to know the domain name of the API for which it will issue tokens.

Add it with:

```text
approov api -add your.azure-api.domain.com
```

Adding the API domain also configures the [dynamic certificate pinning](https://approov.io/docs/latest/approov-usage-documentation/#approov-dynamic-pinning) setup, out of the box.

> **NOTE:** By default the pin is extracted from the public key of the leaf certificate served by the domain, as visible to the box issuing the Approov CLI command and the Approov servers.

### Approov Secret

Approov tokens are signed with a symmetric secret. To verify tokens, we need to grab the secret using the [Approov secret command](https://approov.io/docs/latest/approov-cli-tool-reference/#secret-command) and plug it into the Azure API management platform environment to check the signatures of the [Approov Tokens](https://www.approov.io/docs/latest/approov-usage-documentation/#approov-tokens) that it processes.

Retrieve the Approov secret with:

```text
approov secret -get base64
```

> **NOTE:** The `approov secret` command requires an [administration role](https://approov.io/docs/latest/approov-usage-documentation/#account-access-roles) to execute successfully.

#### Set the Approov Secret

The Approov secret **MUST** be a [named value](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-properties?tabs=azure-portal), and we recommend to use `approov-base64-secret` as its name. **Never** use it directly in the policy statement.

The named value for the Approov secret **MUST** be created using the type `secret` to guarantee that is stored encrypted. You can also create it using the type `key vault`, that will retrieve it from the [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/quick-create-portal), therefore you must already have created it there.

> **IMPORTANT:** Never add the Approov Secret with the type `plain text` for the named value.


[TOC](#toc---table-of-contents)


## Approov Token Check

To check the Approov token in any existing Azure API management platform project we just need to add an inbound processing policy to the API we want to protect with Approov.

We will use the `validate-jwt` [policy](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT) with this policy statement:

```xml
<policies>
    <inbound>
        <base />
        <validate-jwt header-name="Approov-Token" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized" require-expiration-time="true" require-signed-tokens="true">
            <issuer-signing-keys>
                <!-- Replace approov-base64-secret with whatever you have used to add the Approov Secret as a named value. -->
                <key>{{approov-base64-secret}}</key>
            </issuer-signing-keys>
        </validate-jwt>
    </inbound>
</policies>
```

> **NOTE:** When the Approov token validation fails we return a `401` with no further details in the message, because we don't want to give the specifics to an attacker about the reason the request failed, and you can go even further by returning a `400`.


[TOC](#toc---table-of-contents)


## Test your Approov Integration

The following examples below use cURL, but you can also use the [Postman Collection](/README.md#testing-with-postman) to make the API requests. Just remember that you need to adjust the urls and tokens defined in the collection to match your deployment. Alternatively, the README for the Postman Collection also contains instructions for using the preset _dummy_ secret to test your Approov integration.

#### With Valid Approov Tokens

Generate a valid token example from the Approov Cloud service:

```text
approov token -genExample your.azure-api.domain.com
```

Then make the request with the generated token:

```text
curl -i --request GET 'https://your.azure-api.domain.com' \
  --header 'Ocp-Apim-Subscription-Key: ___AZURE_SUBSCRIPTION_KEY_HERE___' \
  --header 'Approov-Token: ___APPROOV_TOKEN_EXAMPLE_HERE___'
```

The request should be accepted. For example:

```text
HTTP/1.1 200 OK

...

{"key": "The response from your backend API"}
```

#### With Invalid Approov Tokens

Generate an invalid token example from the Approov Cloud service:

```text
approov token -type invalid -genExample your.azure-api.domain.com
```

Then make the request with the generated token:

```text
curl -i --request GET 'https://your.azure-api.domain.com' \
  --header 'Ocp-Apim-Subscription-Key: ___AZURE_SUBSCRIPTION_KEY_HERE___' \
  --header 'Approov-Token: ___APPROOV_INVALID_TOKEN_EXAMPLE_HERE___'
```

The above request should fail with an Unauthorized error. For example:

```text
HTTP/1.1 401 Unauthorized

...
{
    "statusCode": 401,
    "message": "Unauthorized"
}
```

[TOC](#toc---table-of-contents)