# Python Decorators
A decorator is a design pattern in Python that allows a user to add new functionality to an existing object or function without modifying its structure.

## Passing Functions as Parameters

Functions are objects. In Python, functions can be passed around and used as arguments.



```python
def multiply(n1, n2):
    return n1*n2


def add(n1, n2):
    return n1+n2


def operation(operator, num1, num2):
    return operator(num1, num2)

```

Here's the output generated by calling `operation` with each of the 2 other functions.


```python
print('operation(multiply,100,30): ', operation(multiply, 100, 30))
print('operation(add,100,30): ', operation(add, 100, 30))

```

    operation(multiply,100,30):  3000
    operation(add,100,30):  130


## Inner Functions and Higher Order Functions
It’s possible to define functions inside other functions. Such functions are called Inner Functions.
Functions which take other functions as parameters or perform operations on other functions are called Higher order functions.


```python
def outer():
    print("Inside outer()")

    def inner():
        print("Inside inner()")

    print("Calling inner()")
    inner()


outer()
try:
    inner()
except Exception as e:
    print(str(e))

```

    Inside outer()
    Calling inner()
    Inside inner()
    name 'inner' is not defined


## Back to Decorators

Python decorators are functions or classes in python that takes another function as a parameter or return a function.


```python
import uuid
from functools import wraps
usernames = ["Charles", "Gift"]
counter = 0


def authorize(users):
    def authorize_decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            global counter
            if users[counter] == "Gift":
                print(f"User {users[counter]} is Allowed!")
                func(*args, **kwargs)
            else:
                print(f"User {users[counter]} is not Allowed!")
            counter = (counter + 1) % 2
        return wrapper
    return authorize_decorator


@authorize(users=usernames)
def secret():
    print(f"Here you go: {uuid.uuid1()}")


for user in usernames:
    secret()

```

    User Charles is not Allowed!
    User Gift is Allowed!
    Here you go: 0d0a1ed4-f553-11ec-8f36-00155d1251c3


# Web Security
## Playing With Fire
- Dealing with the web means that your data is exposed to virtually billions of people
- We can't "assume" that none of them are "bad actors" meaning to harm our data
- Assume that your environment is already compromised and apply a healthy level of mistrust to any user or device attempting to access data or services.

## Born from necessity
- To deal with that threat, two philosophies were born:
    - Authentication
        - Answers the question of [Who are you?]
        - Verifies the identity of the user
    - Authorization
        - Answers the question of [What are you allowed to do?]
        - Determines their access rights

## Authentication Methods
- Username/Password
- Single Sign-On (SSO)
    - Users only have to log in to one application and in doing so, gain access to many other applications. 
    - Often more convenient for users.
- Multi-Factor Authentication
    - Users more than one authentication factor to verify a user’s identity.
- Password-less
    - Verifying a user’s identity without using a password. Examples are: Biometrics, Magic Links, Smart Cards, NFC Devices, Ubikeys
- Biometric Authentication
    - Relies on the unique biological characteristics of individuals.
    - Examples include: fingerprints,  retina scans, Facial recognition, Iris recognition ... 




# JSON Web Tokens

## What is a JWT?

- JSON Web Token or JWT, is a standard for safely passing security information (specifically claims) between applications in a simple, optionally validated and/or encrypted, format.
- The standard is supported by all major web frameworks. (flask, Django, Express ...)

## What is a Claim?

- A Claim is a definition or assertion made about a certain party or object [typically a user]. Examples: Role, Permission …
- Some of these claims and their meaning are defined as part of the JWT spec.
- Standards claims allow frameworks to be able to check standard fields such as expiry (or validity) automatically without requiring additional code from the developer.

## Examples of Standard Claims

| Claim  |  Name | Format   |Usage    | 
|---|---|---|---|
| ‘exp’  |  Expiration | int  |The time after which the token is invalid.   |
| ‘nbf’  | Not Before  | int  | The time before which the token is invalid.  |
| ‘iss’  |  Issuer | str  |  The principal that issued the JWT. |
| ‘aud’  | Audience  |str or list(str)   | The recipient that the JWT is intended for.  |
| ‘iat’  | Issued At  | int  | The time at which the JWT was issued.  |

## Why we use JWTs
- They are simple, compact and usable.
- For Authentication: to identify the user.
- For Authorization: to evaluate the user's permissions.
- For maintaining Stateless sessions.

## Snippet: Encoding a Token



```python
#import sys
#!{sys.executable} -m pip install python-jose

from jose import jwt
import uuid
from datetime import datetime, timezone, timedelta

issue_time = datetime.now(tz=timezone.utc)
expiry_time = datetime.now(tz=timezone.utc) + timedelta(minutes=5)

payload = {
    'sub': 'ALX-T UDACITY',
    'iat': issue_time,
    'exp': expiry_time,
    'permissions': ['get:secret','get:top_secret']
}

secret = "&&12:forever:REPEATED:brother:95&&"
token = jwt.encode(payload, secret, algorithm='HS256')
print(token)

```

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJBTFgtVCBVREFDSVRZIiwiaWF0IjoxNjU2MjUzMDY2LCJleHAiOjE2NTYyNTMzNjYsInBlcm1pc3Npb25zIjpbImdldDpzZWNyZXQiLCJnZXQ6dG9wX3NlY3JldCJdfQ.OHPnRnKAmcZOFUmn9G5ckVDbnaJaEt2S4rXxwDsUDQk


## Snippet: Dissecting a Token

A JWT is made of the following parts:

- Header: Holds metadata such as encryption type
- Payload: Holds user-identifying data
- Signature: Holds the signature of the token

Here's a demonstration:


```python
from jose import jwt

header = jwt.get_unverified_header(token)
print(f'HEADER: {header}')

payload = jwt.get_unverified_claims(token)
print(f'PAYLOAD: {payload}')

```

    HEADER: {'alg': 'HS256', 'typ': 'JWT'}
    PAYLOAD: {'sub': 'ALX-T UDACITY', 'iat': 1656251840, 'exp': 1656252140, 'permissions': ['get:secret']}


## Snippet: Validate a JWT
<small>

```python
from jose import jwt

secret = "&&12:forever:REPEATED:brother:95&&"
token=("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9."
"eyJzdWIiOiIwMDY5MDY5OCIsImlhdCI6MTYzNjM2ODEw"
"OCwiZXhwIjoxNjM2Mzc1MzA4LCJwZXJtaXNzaW9ucyI6"
"WyJnZXQ6c3R1ZGVudHMiXX0.fYf45h6njtBfWQdBbtju"
"pxLUiDw3r1yqGX2Hoj97r4E")

def extract_payload(token, secret):
    try:
        payload = jwt.decode(token,secret,algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        abort(401, description='Token Expired!')
    except jwt.JWTClaimsError:
        abort(403, description='Token lacks required permissions!')
    except Exception:
        abort(401, description='Authentication Failure!')

def check_permissions(permission):
    payload = extract_payload(token,secret)
    if 'permissions' not in payload:
        abort(403, 'Token lacks required permissions!')
    if permission not in payload['permissions']:
        abort(403, 'Not permitted!')
    return True

```

</small>



```python
from functools import wraps
import secrets


def extract_payload(token, secret):
    try:
        payload = jwt.decode(token, secret, algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        print('401: Token Expired!')
    except jwt.JWTClaimsError:
        print('403: Token lacks required permissions!')
    except Exception:
        print('401: Authentication Failure!')


def check_permissions(permission):
    payload = extract_payload(token, secret)
    if payload is None:
        return False
    else:
        if 'permissions' not in payload:
            print('403:Token lacks required permissions!')
            return False
        if permission not in payload['permissions']:
            print('403: Not permitted!')
            return False
        return True


def authorize(permission):
    def authorize_decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if check_permissions(permission):
                print('Allowed!')
                return func(*args, **kwargs)
            else:
                print('Not Allowed!')
        return wrapper
    return authorize_decorator


@authorize('get:secret')
def confidential():
    return f'The secret is: {uuid.uuid1()}'


@authorize('get:top_secret')
def top_secret():
    return f'The biggest secret is: {secrets.token_urlsafe()} {secrets.token_hex()}'


print(f'confidential() --> {confidential()}')
print('\n')
print(f'top_secret() --> {top_secret()}')

```

    Allowed!
    confidential() --> The secret is: c770f19c-f55a-11ec-8f36-00155d1251c3
    
    
    Allowed!
    top_secret() --> The biggest secret is: m1MAuDeVufpsIPveyEw6Xc8jsPGOR7fol4-nnQzwhnY f1067b8c14b33a746314e3c393216e42deb6d476287f424366be525c5621b53e


## Snippet: Custom Error Handling in Flask
<small>

``` python
@app.errorhandler(401)
def error_401(error):
    return jsonify({
        'success': False,
        'error': error.code,
        'message': error.description
    }), error.code

@app.errorhandler(403)
def error_403(error):
    return jsonify({
        'success': False,
        'error': error.code,
        'message': error.description
    }), error.code
```

</small>

# Resources

https://realpython.com/primer-on-python-decorators/

https://stackoverflow.com/questions/308999/what-does-functools-wraps-do

https://datatracker.ietf.org/doc/html/draft-ietf-oauth-json-web-token

https://datatracker.ietf.org/doc/html/rfc7519

https://auth0.com/learn/json-web-tokens/

https://jwt.io/introduction

https://pypi.org/project/python-jose/

https://github.com/mpdavis/python-jose

https://jose.readthedocs.io/en/latest/

https://gist.github.com/Zearin/2f40b7b9cfc51132851a
