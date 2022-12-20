# Approov QuickStart - Azure API Management

[Approov](https://approov.io) is an API security solution used to verify that requests received by your API services originate from trusted versions of your mobile apps.

This repo implements the Approov API request verification for the [Azure API Management Platform](https://azure.microsoft.com/en-gb/services/api-management), which performs the verification check on the Approov Token before allowing valid traffic to reach the API endpoint.

If you are looking for another Approov integration you can check our list of [quickstarts](https://approov.io/docs/latest/approov-integration-examples/backend-api/), and if you don't find what you are looking for, then please let us know [here](https://approov.io/contact).


## Approov Integration Quickstart

The quickstart assumes that you already have an Azure API management platform running, and that your are familiar with managing it.

The quickstart was tested with the following Operating Systems:

* Ubuntu 20.04
* MacOS Big Sur
* Windows 10 WSL2 - Ubuntu 20.04

If you find yourself lost or blocked in some part of the quickstart, then you can check the [detailed quickstart](docs/APPROOV_TOKEN_QUICKSTART.md).

First, setup the [Approov CLI](https://approov.io/docs/latest/approov-installation/index.html#initializing-the-approov-cli).

Now, register the API domain for which Approov will issues tokens:

```bash
approov api -add api.example.com
```

> **NOTE:** By default a symmetric key (HS256) is used to sign the Approov token on a valid attestation of the mobile app for each API domain it's added with the Approov CLI, so that all APIs will share the same secret and the backend needs to take care to keep this secret secure.
>
> A more secure alternative is to use asymmetric keys (RS256 or others) that allows for a different keyset to be used on each API domain and for the Approov token to be verified with a public key that can only verify, but not sign, Approov tokens.
>
> To implement the asymmetric key you need to change from using the symmetric HS256 algorithm to an asymmetric algorithm, for example RS256, that requires you to first [add a new key](https://approov.io/docs/latest/approov-usage-documentation/#adding-a-new-key), and then specify it when [adding each API domain](https://approov.io/docs/latest/approov-usage-documentation/#keyset-key-api-addition). Please visit [Managing Key Sets](https://approov.io/docs/latest/approov-usage-documentation/#managing-key-sets) on the Approov documentation for more details.

Next, enable your Approov `admin` role with:

```bash
eval `approov role admin`
````

For the Windows powershell:

```bash
set APPROOV_ROLE=admin:___YOUR_APPROOV_ACCOUNT_NAME_HERE___
```

Now, retrieve the Approov secret:

```bash
approov secret -get base64
```

Next, add the Approov secret, and it **MUST** be as a *named value* by following [their instructions](https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-properties?tabs=azure-portal), and we recommend to use `approov-base64-secret` as its name. **Never** use it directly in the policy statement.

The named value for the Approov secret **MUST** be created using the type `secret` to guarantee that is stored encrypted. You can also create it using the type `key vault`, that will retrieve it from the [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/quick-create-portal), therefore you must have already created it there.

> **IMPORTANT:** Never add the Approov Secret with the type `plain text` for the named value.

Finally, add an inbound processing policy to the API you want to protect with Approov, by using the `validate-jwt` [policy](https://docs.microsoft.com/en-us/azure/api-management/api-management-access-restriction-policies#ValidateJWT) with this policy statement:

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

Not enough details in the bare bones quickstart? No worries, check the [detailed quickstart](docs/APPROOV_TOKEN_QUICKSTART.md) that contain a more comprehensive set of instructions, including how to test the Approov integration.


## More Information

* [Approov Overview](OVERVIEW.md)
* [Detailed Quickstart](docs/APPROOV_TOKEN_QUICKSTART.md)
* [Testing](docs/APPROOV_TOKEN_QUICKSTART.md#test-your-approov-integration)


## Issues

If you find any issue while following our instructions then just report it [here](https://github.com/approov/quickstart-azure-api-management/issues), with the steps to reproduce it, and we will sort it out and/or guide you to the correct path.


## Useful Links

If you wish to explore the Approov solution in more depth, then why not try one of the following links as a jumping off point:

* [Approov Free Trial](https://approov.io/signup)(no credit card needed)
* [Approov Get Started](https://approov.io/product/demo)
* [Approov QuickStarts](https://approov.io/docs/latest/approov-integration-examples/)
* [Approov Docs](https://approov.io/docs)
* [Approov Blog](https://approov.io/blog/)
* [Approov Resources](https://approov.io/resource/)
* [Approov Customer Stories](https://approov.io/customer)
* [Approov Support](https://approov.io/contact)
* [About Us](https://approov.io/company)
* [Contact Us](https://approov.io/contact)
