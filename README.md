# AWS API Gateway Custom Authorizer for RS256 JWTs

An AWS API Gateway [Custom Authorizer](http://docs.aws.amazon.com/apigateway/latest/developerguide/use-custom-authorizer.html) that authorizes API requests by requiring
that the OAuth2 [bearer token](https://tools.ietf.org/html/rfc6750) is a JWT that can be validated using the RS256 (asymmetric) algorithm with a public key that is obtained from a [JWKS](https://tools.ietf.org/html/rfc7517) endpoint.

## About

### What is AWS API Gateway?

API Gateway is an AWS service that allows for the definition, configuration and deployment of REST API interfaces.
These interfaces can connect to a number of back-end systems.
One popular use case is to provide an interface to AWS Lambda functions to deliver a so-called 'serverless' architecture.

### What are "Custom Authorizers"?

In February 2016 Amazon
[announced](https://aws.amazon.com/blogs/compute/introducing-custom-authorizers-in-amazon-api-gateway/)
a new feature for API Gateway -
[Custom Authorizers](http://docs.aws.amazon.com/apigateway/latest/developerguide/use-custom-authorizer.html). This allows a Lambda function to be invoked prior to an API Gateway execution to perform custom authorization of the request, rather than using AWS's built-in authorization. This code can then be isolated to a single centralized Lambda function rather than replicated across every backend Lambda function.

### What does this Custom Authorizer do?

This package gives you the code for a custom authorizer that will, with a little configuration, perform authorization on AWS API Gateway requests via the following:

* It confirms that an OAuth2 bearer token has been passed via the `Authorization` header.
* It confirms that the token is a JWT that has been signed using the RS256 algorithm with a specific public key
* It obtains the public key by inspecting the configuration returned by a configured JWKS endpoint
* It also ensures that the JWT has the required Issuer (`iss` claim) and Audience (`aud` claim)

## Local setup and testing

Checkout the [original repo's instructions](https://github.com/auth0-samples/jwt-rsa-aws-custom-authorizer#setup)

## AWS Deployment

Now we're ready to deploy the custom authorizer to an AWS API Gateway.

### Using serverless

Export the envitonment variables:

* `TOKEN_ISSUER`: The issuer of the token. If you're using Auth0 as the token issuer, this would be: `https://your-tenant.auth0.com/`
* `JWKS_URI`: This is the URL of the associated JWKS endpoint. If you are using Auth0 as the token issuer, this would be: `https://your-tenant.auth0.com/.well-known/jwks.json`
* `AUDIENCE`: This is the required audience of the token. If you are using Auth0 as the Authorization Server, the audience value is the same thing as your API **Identifier** for the specific API in your [APIs section]((https://manage.auth0.com/#/apis)).

The execute:

``sls deploy --verbose``

The serverless deployment will give us the lambda that will be used as a custom authorizer on API Gateway endpoints. So after the deployment is done make note of the **AuthLambdaFunctionQualifiedArn** value on the output, which will by the authorizer on deployment of the function you want to secure like below:

~~~~
authorizer:
    arn: ${env:LAMBDA_AUTHORIZER_ARN}
~~~~

Using this configuration on the other lambda function/ API Gateway will automatically create the custom configurator on the API Gateway's side and also link it to the endpoint.

### Test the Custom Authorizer in the API Gateway

Get the ACCESS_TOKEN can be found using the cURL:

~~~~
curl --request POST \
  --url https://lariskovski.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"xxxxxxxxxxxxxx","client_secret":"yyyyyyyyyyyyy","audience":"api.mycompany.com/pokedex","grant_type":"client_credentials"}'
~~~~

> This formatted command can be found on yout application page on Auth0 management console.

Go to the authorizers tab on the management console so you can then test the new custom authorizer by providing the **Access Token** return by the above command and clicking **Test**.

```
Bearer ACCESS_TOKEN
```

A successful test will look something like:

```
Latency: 2270 ms
Principal Id: oauth|1234567890
Policy
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1459758003000",
            "Effect": "Allow",
            "Action": [
                "execute-api:Invoke"
            ],
            "Resource": [
                "arn:aws:execute-api:*"
            ]
        }
    ]
}
```

#### With Postman

You can use Postman to test the REST API

* Method: (matching the Method in API Gateway, eg. `GET`)
* URL: `INVOKE_URL/your/resource`
* Headers tab: Add an `Authorization` header with the value `Bearer ACCESS_TOKEN`

#### With curl from the command line

```
curl -i "INVOKE_URL/your/resource" \
  -X GET \
  -H 'Authorization: Bearer ACCESS_TOKEN'
```

The above command is performed using the `GET` method.

---

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amongst others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free account in Auth0

1. Go to [Auth0](https://auth0.com) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE.txt) file for more info.
