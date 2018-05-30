# kafka-plain-saslserver-2-ad [![Build Status](https://travis-ci.org/navikt/kafka-plain-saslserver-2-ad.svg?branch=master)](https://travis-ci.org/navikt/kafka-plain-saslserver-2-ad)
Enhancing kafka with
- customized PlainSaslServer using LDAPS simple bind for authentication
- customized SimpleACLAuthorizer using LDAPS compare-matched for group membership verification

Thus, avoiding user and passwords in JAAS context file on kafka brokers

By defining Read/Write allowance with Active Directory groups, authorization is moved from 
Zookeeper Access Control Lists to group membership.

Binding and group membership information is cached (limited lifetime after write),
giving minor performance penalty and reduced LDAPS traffic.

## Tools
- Kotlin
- Gradle build tool
- Spek test framework

## Components

1. Unboundid LDAP SDK for LDAPS interaction
2. Caffeine Cache
3. YAML Configuration for LDAP baseDN for users, groups and more. See src/test/resources/ldapconfig.yaml for details

**Observe** that the directory hosting yaml configuration file must be in CLASSPATH.

## Kafka configuration examples

Example of JAAS context file on Kafka broker using the customized class for
authentication.

```
KafkaServer{
no.nav.common.security.plain.PlainLoginModule required
username="x"
password="y";
};
```

Example of Kafka server.properties for using the customized class for authorization.

```
authorizer.class.name=no.nav.common.security.authorization.SimpleLDAPAuthorizer
```


## Testing

Use of Unboundid in-memory LDAP server for all test cases.

Tested on confluent.io version 4.0.0.

See [Confluent Open Source distribution](https://www.confluent.io/product/confluent-open-source/) in order to test locally.

The related [Wiki](https://github.com/navikt/KafkaPlainSaslServer2AD/wiki) has a detailed guide for local testing.

## Build 

```
./gradlew clean build
./gradlew shadowJar

The result is KafkaPlainSaslServer2AD-<version>-all.jar both authentication and authorization logic.
```
**Observe** that the directory hosting the given JAR file must be in CLASSPATH.

### Contact us
#### Code/project related questions can be sent to 
* Torstein Nesby, `torstein.nesby@nav.no`
* Trong Huu Nguyen, `trong.huu.nguyen@nav.no`

For internal resources, send requests/questions to slack#kafka
