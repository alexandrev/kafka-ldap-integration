# kafka-ldap-integration 

Enhancing kafka 2.x with
- customized SimpleLDAPAuthentication using LDAPS simple bind for authentication
- customized SimpleACLAuthorizer using LDAPS compare-matched for group membership verification

Thus, moving authentication from user and passwords in JAAS context file on kafka brokers to LDAP server

By defining Read/Write allowance with LDAP groups, authorization is moved from 
Zookeeper Access Control Lists to group membership verification.

Binding and group membership information is cached (limited lifetime after write),
giving minor performance penalty and reduced LDAPS traffic.

## Tools
- Kotlin
- Gradle build tool
- Spek test framework

## Components

1. Unboundid LDAP SDK for LDAPS interaction
2. Caffeine Cache
3. YAML Configuration for LDAP baseDN for users, groups and more. See `src/test/resources/ldapconfig.yaml` for details

**N.B.** that the directory hosting yaml configuration file must be in CLASSPATH.

## Kafka configuration examples

Example of Kafka server.properties for using the customized classes for authentication and authorization. The example
focus on minimum configuration only (sasl plaintext). A production environment should utilize plain with TLS.

```properties
# Configure inter-broker communication to use plaintext (Use SSL/TLS in Prod!)
listeners=SASL_PLAINTEXT://localhost:9092
security.inter.broker.protocol=SASL_PLAINTEXT

# Configure brokers to exchange plain text username/password.
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN

# Configure the JAAS context for plain.
# It is also possible to use an external JAAS file instead of this property
listener.name.sasl_plaintext.plain.sasl.jaas.config=\
  org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="ldap-user" \
  password="ldap-password";

# Configure the authentication to use LDAP (verify that client is actually who they say they are)
listener.name.sasl_plaintext.plain.sasl.server.callback.handler.class=\
  com.instaclustr.kafka.ldap.authentication.SimpleLDAPAuthentication

# Configure the authorization to use LDAP (verify that client is allowed to perform a specific action)
authorizer.class.name=com.instaclustr.kafka.ldap.authorization.SimpleLDAPAuthorizer
```

## Testing

Use of Unboundid in-memory LDAP server for all test cases.

Tested on Kafka version 2.x

See [Apache Kafka](https://kafka.apache.org/) in order to test locally.

## Build 

```
./gradlew clean build shadowJar
```

The result is `build/libs/kafka-ldap-integration-<version>.jar`, which contains the authentication and authorization classes, along with all of their dependencies.

**N.B.** This jar must be added to the classpath for the Kafka broker. The easist way to do that is to copy the jar into the directory `$KAFKA_HOME/libs`.

### Contact us
#### Code/project related questions can be sent to 
* Joe Schmetzer, `joe.schmetzer@instaclustr.com `
* Zeke Dean, `zeke.dean@instaclustr.com`

#### Credit to original authors

Maintainers of the GitHub project [kafka-plain-saslserver-2-ad](https://github.com/navikt/kafka-plain-saslserver-2-ad):
* Torstein Nesby, `torstein.nesby@nav.no`
* Trong Huu Nguyen, `trong.huu.nguyen@nav.no`
