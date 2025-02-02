# FastAPI Third Party Auth

<p align="left">
    <a href="https://github.com/aiwizo/fastapi-third-party-auth/actions?query=workflow%3ATest"
       target="_blank">
       <img src="https://github.com/aiwizo/fastapi-third-party-auth/workflows/Test/badge.svg"  
            alt="Test">
    </a>
    <a href='https://fastapi-third-party-auth.readthedocs.io/en/latest/?badge=latest'>
        <img src='https://readthedocs.org/projects/fastapi-third-party-auth/badge/?version=latest' alt='Documentation Status' />
    </a>
    <a href="https://pypi.org/project/fastapi-third-party-auth" 
       target="_blank">
       <img src="https://img.shields.io/pypi/v/fastapi-third-party-auth?color=%2334D058&label=pypi%20package" 
            alt="Package version">
    </a>
</p>

---

**Documentation**: <a href="https://fastapi-third-party-auth.readthedocs.io/" target="_blank">https://fastapi-third-party-auth.readthedocs.io/</a>

**Source Code**: <a href="https://github.com/aiwizo/fastapi-third-party-auth" target="_blank">https://github.com/aiwizo/fastapi-third-party-auth</a>

---

Simple library for using a third party authentication service with 
[FastAPI](https://github.com/tiangolo/fastapi). Verifies and decrypts 3rd party
OpenID Connect tokens to protect your endpoints.

Easily used with authentication services such as:
- [Keycloak](https://www.keycloak.org/) (open source)
- [SuperTokens](https://supertokens.io/) (open source)
- [Auth0](https://auth0.com/)
- [Okta](https://www.okta.com/products/authentication/)

FastAPI's generated interactive documentation supports the grant flows:
```python3
GrantType.AUTHORIZATION_CODE
GrantType.IMPLICIT
GrantType.PASSWORD
GrantType.CLIENT_CREDENTIALS
```

![example documentation](example-docs.png)

## Installation

```
poetry add fastapi-third-party-auth
```

Or, for the old-timers:

```
pip install fastapi-third-party-auth
```

## Usage

See [this example](tree/master/example) for how to use
`docker-compose` to set up authentication with `fastapi-third-party-auth` +
[Keycloak](https://www.keycloak.org/).

### Standard usage

```python3
from fastapi import Depends
from fastapi import FastAPI
from fastapi import Security
from fastapi import status

from fastapi_third_party_auth import Auth
from fastapi_third_party_auth import GrantType
from fastapi_third_party_auth import KeycloakIDToken

auth = Auth(
    openid_connect_url="http://localhost:8080/auth/realms/my-realm/.well-known/openid-configuration",
    issuer="http://localhost:8080/auth/realms/my-realm",  # optional, verification only
    client_id="my-client",  # optional, verification only
    scopes=["email"],  # optional, verification only
    grant_types=[GrantType.IMPLICIT],  # optional, docs only
    idtoken_model=KeycloakIDToken,  # optional, verification only
)

app = FastAPI(
    title="Example",
    version="dev",
    dependencies=[Depends(auth)],
)

@app.get("/protected")
def protected(id_token: KeycloakIDToken = Security(auth.required)):
    return dict(message=f"You are {id_token.email}")
```

### Optional: Custom token validation

The IDToken class will accept any number of extra fields but you can also
validate fields in the token like this:

```python3
class MyAuthenticatedUser(IDToken):
    custom_field: str
    custom_default: float = 3.14

auth = Auth(
    ...,
    idtoken_model=MyAuthenticatedUser,
)
```
