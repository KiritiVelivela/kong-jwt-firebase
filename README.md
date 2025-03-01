## JWT Firebase custom [Kong](https://www.getkong.org) plugin

#### This is forked from original repo to support kong version 3.0+ and adding ability to define an anonymous user and JWT service user.
#### JWT Firbase plugin
This plugin is basically to verify the JWT Firbase Token following the [firebase doc](https://firebase.google.com/docs/auth/admin/verify-id-tokens)
What we need to run this plugin is just the firebase project name.

#### Installtion

```bash
luarocks install kong-jwt-firebase
```

###### Load the plugin by kong.conf file
- By editting the kong.conf file 
```
plugins = bundled, jwt-firebase
```

###### Environment variable if you are using docker
```
KONG_PLUGINS = bundled, jwt-firebase
```

#### How it works
According to the [firebase doc](https://firebase.google.com/docs/auth/admin/verify-id-tokens) this plugin verifies the header, payload, and signature of the ID token.
- Verify that the alg is "RS256"
- Verify that the kid must correspond too one of the pubic key listed at https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com
- Verify that the exp must be in the future. (UNIX epoch)
- Verify that the aud must be your Firebase project ID
- Verify that the iss must be "https://securetoken.google.com/<projectId>", where <projectId> is the same project ID used for aud above.
- Verify that the sub must be non-empty string and must be the uid of the user or device.
and Finally, ensure that the ID token was signed by the private key corresponding to the token's kid claim. 
Grab the public key from https://www.googleapis.com/robot/v1/metadata/x509/securetoken@system.gserviceaccount.com 
and use a JWT library to verify the signature. 

#### Plugin pamameters
```
    anonomous: Anonymous user to fall back to if authentication fails
    project_id: Your firebase project id
    jwt_service_user: Username that'll be added to request header after authentication is successful    
```

#### Configuration
This is the example of using the JWT firebase plugin to verify JWT token in Firebase project id chatq-dev
- Create a service
```sh
$ curl -i -X POST localhost:8001/services \
    --data "name=test" \
    --data "url=http://httpbin.org"
```
- Create a route
```sh
$ curl -i -X POST localhost:8001/services/test/routes \
    --data "name=test" \
    --data "paths[]=/test"
```
- Add the JWT Firebase plugin to test route
```sh
$ curl -i -X POST localhost:8001/routes/test/plugins \
    --data "name=jwt-firebase" \
    --data "config.project_id=chatq-dev"
```

Now you send the requests throuhgh, only tokens signed by Firebase project "chatq-dev" will work:
```sh
$ curl -ik -X GET \
    --url https://localhost:8443/test \
    --header 'Authorization: Bearer <token-id> '
```
This plugin also supports legacy authenticaion without Bearer
```sh
$ curl -ik -X GET \
    --url https://localhost:8443/test \
    --header 'Authorization: <token-id> '
```
This plugin also supports legacy authenticaion without Bearer
```sh
$ curl -ik -X GET \
    --url https://localhost:8443/test \
    --header 'Authorization: <token-id> '
```

This plugin also support authentication with URL query parameter.
```sh
$ curl -ik -X GET \
    --url https://localhost:8443/test?jwt=<token>
``` 
#### How to release
Create the Lua rock in current directory:
```sh
$ luarock make
$ luarocks pack kong-jwt-firebase
```
