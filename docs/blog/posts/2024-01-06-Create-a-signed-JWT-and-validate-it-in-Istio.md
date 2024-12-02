---
title: Create a Signed JWT and validate it in Istio using a JWK
description: This guide shows how to create a public/private key pair and how to use these to create a JWK and a signed JWT and then validate a request to a service running in Kubernetes where Istio is running.
date: 2024-01-06
categories:
  - Istio
  - JWT
  - Tutorials
---
# Create a Signed JWT and validate it in Istio using a JWK

This guide shows how to create a public/private key pair and how to use these to create a JWK and a signed JWT and then validate a request to a service running in Kubernetes where Istio is running.

## Generate a Public/Private Key Pair

For this you’ll need openssl installed. If you have Chocolatey installed, `choco install openssl -y` will do the job.

Run the following commands:

```powershell
ssh-keygen -t rsa -b 4096 -m PEM -f jwtRS256.key
# Don't add passphrase
openssl rsa -in jwtRS256.key -pubout -outform PEM -out jwtRS256.key.pub
```

This produces a `.key` file containing the private key and a `.key.pub` file containing the public key.

## Create JWK from Public Key

Go to the [JWK Creator site](https://russelldavies.github.io/jwk-creator/) and paste in the contents of your public key. For purpose, choose `Signing` and for the algorithm, choose `RS256`.

## Manifests

The following will secure all workloads where they have a label of type and a value of api so they must have a JWT and it must be valid.

```yaml
apiVersion: security.istio.io/v1
kind: RequestAuthentication
metadata:
  name: "validate-jwt"
  namespace: istio-system
spec:
  selector:
    matchLabels:
      type: api
  jwtRules:
  - issuer: "testing@secure.istio.io"
    jwks: |
      {
        "keys": [
          <YOUR_JWK_GOES_HERE>
        ]
      }

---

apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "deny-requests-without-jwt"
  namespace: istio-system
spec:
  selector:
    matchLabels:
      type: api
  action: DENY
  rules:
  - from:
    - source:
        notRequestPrincipals: ["*"]
```

Replace `<YOUR_JWK_GOES_HERE>` with your JWK created in the previous step. Make sure your indentation doesn’t go to less than the existing `{` below the `jwks:` | line.

## Create a Test JWT

The following PowerShell script will create a JWT that will last 3 hours and print it to the screen.

```powershell
Clear-Host

# Install-Module jwtPS -force

Import-Module -Name jwtPS

$encryption = [jwtTypes+encryption]::SHA256
$algorithm = [jwtTypes+algorithm]::HMAC
$alg = [jwtTypes+cryptographyType]::new($algorithm, $encryption)

$key = "jwtRS256.key"
# The content must be joined otherwise you would have a string array.
$keyContent = (Get-Content -Path $key) -join ""
$payload = @{
    aud = "jwtPS"        
    iss = "testing@secure.istio.io"
    sub = "testing@secure.istio.io"
    nbf = 0
    groups = @(
      "group1",
      "group2"    
    )
    exp = ([System.DateTimeOffset]::Now.AddHours(3)).ToUnixTimeSeconds()
    iat = ([System.DateTimeOffset]::Now).ToUnixTimeSeconds()
    jti = [guid]::NewGuid()
}
$encryption = [jwtTypes+encryption]::SHA256
$algorithm = [jwtTypes+algorithm]::RSA
$alg = [jwtTypes+cryptographyType]::new($algorithm, $encryption)
$jwt = New-JWT -Payload $payload -Algorithm $alg -Secret $keyContent

Write-Host $jwt
```

Note that you will likely need to run the line `Install-Module jwtPS -force` in an elevated prompt.

## Testing

To test this, you will need a service you can point at in your cluster with a label of `type` set to `api`. Don’t forget to update URLs as needed.

```powershell
$TOKEN = jwt_from_previous_script
curl https://example.com/api/v1/test
curl --header "Authorization: Bearer $TOKEN" https://example.com/api/v1/test
```

In the above examples, the first request should return `401` and the second request should return `200`.