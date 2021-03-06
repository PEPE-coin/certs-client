<img src="docs/images/tls-certificate-64.png" alt="Cert" width=25 height=25> Certificate Web Service Client 
----------
[![Maven Central][maven-svg]][maven-url] [![changelog][cl-svg]][cl-url] [![javadoc][javadoc-svg]][javadoc-url]  

A java API for Certificate Web Service.


Download
--------

Download [the latest JAR][1] or grab via Maven:
```xml
<dependency>
  <groupId>com.oneops</groupId>
  <artifactId>certs-client</artifactId>
  <version>1.1.4</version>
</dependency>
```

## Examples

#### Initializing CWS Client

```java
CwsClient client = CwsClient.builder()
            .endPoint("Api Endpoint") 
            .appId("App ID")               
            .teamDL("Base Team DL")             
            .keystore("Keystore Path")
            .keystorePassword("Keystore password")
            .build();
```

- Keystore should be of type [PKCS#12][2] format. 
- For loading the keystore from classpath use, `classpath:/<your/cws/keystore/path>.p12`
- If the keystore contains multiple cert entries, use [.keyAlias("cws-client-key")][3] to select the proper client private key.
- To enable http debugging for troubleshooting, set [.debug(true)][4] to the [CwsClient.builder()][5]
- In order to create a `PKCS#12(.p12)` keystore from PEM/DER encoded certificate, use the following `openssl` command.

```ruby
$ openssl pkcs12 -export -chain -out cws-keystore.p12 -inkey private.key -password pass:test123 \
                  -in client.crt -certfile client.crt -CAfile cacert.crt -name cws-client-key \
                  -caname root-ca
              
# Add trust-store entry (cacert.crt) to the keystore.
$ keytool -importcert -trustcacerts -alias root-ca -storetype PKCS12 \
                       -keystore cws-keystore.p12 -storepass test123 -file cacert.crt
                   
# View pkcs12 keystore details                   
$ openssl pkcs12 -info -password pass:test123 -in cws-keystore.p12 
# keytool -list  -storepass test123 -keystore cws-keystore.p12 -v                
```

#### Create new certificate

```java
String cn = "test1.domain.com" ;
String teamDL = "test-teamDL"; // Relative to Base TeamDL.
List<String> sans = Arrays.asList("app1.domain.com","app2.domain.com");
    
String certName = client.createCert(cn,sans, teamDL);
```

#### Check certificate exists

```java
boolean exists = client.certExists(cn, teamDL);
```

#### Download certificate

  * Download the private key, certificate and it's trust chain as [PKCS#12][2] format. 
  
    ```java
    // Generate Keystore/key password (Optional)
    String keystorePasswd = PasswordGen.builder().build().generate(20); 
    String base64Content = client.downloadCert(cn, teamDL, keystorePasswd, CertFormat.PKCS12);
    ```
  
  * Download [CertBundle][6], which contains encrypted [PKCS#8][8] private key, client cert and cacerts.
  
    ```java
    // Private key password should be at-least 4 chars.
     CertBundle certBundle = client.downloadCert(cn, teamDL, Optional.of("test123"));
    // certBundle.key() 
    // certBundle.keyPassword() 
    // certBundle.cert()
    // certBundle.cacert()
    ```
  * Download [CertBundle][6] which contains encrypted `PKCS#1` private key, client cert and cacerts.
  
    ```java
     CertBundle certBundle = client.downloadCert(cn, teamDL, Optional.empty());
    // certBundle.key() 
    // certBundle.cert()
    // certBundle.cacert()
    ```


#### Get certificate expiration date

```java
LocalDateTime date = client.getCertExpirationDate(cn, teamDL);
```

#### View certificate details

```java
ViewRes viewRes = client.viewCert(cn, teamDL);
```


#### Revoke and disable the certificate

```java
RevokeRes revokeRes = client.revokeCert(cn, teamDL, RevokeReason.NONE, true);
```

#### Renew certificate

```java
boolean success = client.renewCert(cn, teamDL);
```

#### Delete certificate

```java
client.obsoleteCert(cn, teamDL);
```

## Testing

Set the following env variables and run `./mvnw clean test` to execute the unit tests.

```bash
 export cws_host=...     
 export cws_app_id=...
 export cws_team_dl=....
 export cws_domain=...
 export cws_keystore=.....p12
 export cws_keystore_pass=....
```

## Dependencies

   - [Retrofit](https://github.com/square/retrofit/)
   - [OkHttp](https://github.com/square/okhttp)
   - [Moshi](https://github.com/square/Moshi/)
   - [Bouncy Castle](https://github.com/bcgit/bc-java)

License
-------

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.



<!-- Badges -->

[1]: https://search.maven.org/remote_content?g=com.oneops&a=certs-client&v=LATEST
[2]: https://en.wikipedia.org/wiki/PKCS_12
[3]: http://oneops.com/certs-client/javadocs/com/oneops/certs/CwsClient.html#keyAlias--
[4]: http://oneops.com/certs-client/javadocs/com/oneops/certs/CwsClient.html#debug--
[5]: http://oneops.com/certs-client/javadocs/com/oneops/certs/CwsClient.Builder.html
[6]: http://oneops.com/certs-client/javadocs/com/oneops/certs/model/CertBundle.html
[7]: https://en.wikipedia.org/wiki/Privacy-enhanced_Electronic_Mail
[8]: https://en.wikipedia.org/wiki/PKCS_8

[maven-url]: http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.oneops%22%20AND%20a%3A%22certs-client%22
[maven-svg]: https://img.shields.io/maven-central/v/com.oneops/certs-client.svg?label=Maven%20Central&style=flat-square

[cl-url]: https://github.com/oneops/certs-client/blob/master/CHANGELOG.md
[cl-svg]: https://img.shields.io/badge/change--log-latest-green.svg?style=flat-square

[javadoc-url]: https://oneops.github.io/certs-client/javadocs/
[javadoc-svg]: https://img.shields.io/badge/api--doc-latest-cyan.svg?style=flat-square

