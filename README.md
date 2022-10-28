# Anubis Access Control Policy Vocabulary

This repository manage the vocabulary used by [Anubis](https://anubis-pep.readthedocs.io/en/latest/)
to support policy portability.

The vocabulary is published at [http://voc.orchestracities.io/oc-acl](http://voc.orchestracities.io/oc-acl)

The vocabulary has been developed using [Protege](https://protege.stanford.edu/).

## Description

The oc-acl vocabulary extends
[Web Access control](https://solid.github.io/web-access-control-spec/) with
specific actors, actions, and with support for constraints.

In the current version of the vocabulary, a policy is an
[authorization rule](https://solid.github.io/web-access-control-spec/#authorization-rule)
(i.e. instance of `acl:Authorization`) with the following properties:

- an [*access object*](https://solid.github.io/web-access-control-spec/#access-objects)
  (i.e. an `acl:accessTo` predicate) that defines the resources to which the
  authorization rule provide access to.
- an [*access mode*](https://solid.github.io/web-access-control-spec/#access-modes)
  (i.e. an `acl:mode` predicate) that defines the actions an authorized agent can
  perform on a resource.
- an [*access subject*](https://solid.github.io/web-access-control-spec/#access-subjects)
  (i.e. an `acl:agent` or `acl:agentClass` or `acl:agentGroup` predicate)
  that defines the authorized agents.
- an [*access constraint*](https://www.w3.org/TR/odrl-model/#constraint)
  (i.e. an `oc-acl:constraint` predicate) that [extends authorization](https://solid.github.io/web-access-control-spec/#authorization-extensions)
  supported by WAC vocabulary with ODRL constraints that need to be verified for
  an authorized agent to perform an operation on a resource.

As result, policies in oc-acl looks like:

```ttl
@base <http://example.com/> .
@prefix ex: <http://example.com/> .
@prefix acl: <http://www.w3.org/ns/auth/acl#> . 
@prefix oc-acl: <http://voc.orchestracities.io/oc-acl#> . 
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

ex:constraint1 a odrl:Constraint ;
      odrl:leftOperand odrl:dateTime ;
      odrl:operator odrl:lt ;
      odrl:rightOperand "2023-01-01"^^xsd:date .

ex:policy1 a acl:Authorization ;
        acl:accessTo <urn:entity:1> ;
        acl:agentClass <acl:agentClass:Admin> ;
        oc-acl:constraint ex:constraint1 ;
        acl:mode oc-acl:Decrypt, oc-acl:Delete .
```

This example policy states that users with role `Admin`
can access the resource `urn:entity:1` to perform `read`
and `delete` operations only before `2023-01-01`.

The oc-acl vocabulary, beyond introducing the `oc-acl:constraint` predicate, extends
Web Access Control vocabulary with the following classes:

- `oc-acl:Delete` [extends access modes](https://solid.github.io/web-access-control-spec/#access-mode-extensions)
  by providing a specific [write access mode](https://solid.github.io/web-access-control-spec/#acl-mode-write)
  which only allows to delete information.

- `oc-acl:ResourceTenantAgent`, an [authenticated agent](https://solid.github.io/web-access-control-spec/#acl-agentclass-authenticated-agent)
  that belongs to the same tenant as the resource authorized.

- `oc-acl:Decrypt` [extends access modes](https://solid.github.io/web-access-control-spec/#access-mode-extensions)
  by providing a specific [read access mode](https://solid.github.io/web-access-control-spec/#acl-mode-read)
  which only allows to decrypt encrypted data. This would allow to specify that
  a given actor can read decrypted values of a specific attribute. This access
  mode is meant to be combined with [data protection policies]().

### Attribute Based Access Control

Constraints in ODRL are either boolean expression (cf. [Constraint](https://www.w3.org/TR/odrl-vocab/#constraints))
or logical expressions (cf. [Logical Constraint](https://www.w3.org/TR/odrl-vocab/#logicalConstraints)
chaining multiple boolean expressions.

In the context of oc-acl there is not need to extend Logical Constraint with
additional operators, given that existing ones covers already all possible
logical expressions (and, or, ...).

Boolean expressions are composed of different items:

- a [`leftOperand`](https://www.w3.org/TR/odrl-vocab/#term-LeftOperand),
  the left operand for a constraint expression.

- an [`operator`](https://www.w3.org/TR/odrl-vocab/#constraintRelationalOperators)

- a [`rightOperand`](https://www.w3.org/TR/odrl-vocab/#term-rightOperand),
  the value of the right operand in a constraint expression, or a [`rightOperandReference`](https://www.w3.org/TR/odrl-vocab/#term-rightOperandReference),
  i.e. a reference to a web resource providing the value for the right operand
  of a Constraint.

- a [`dataType`](https://www.w3.org/TR/odrl-vocab/#term-dataType),
  the datatype of the value of the rightOperand or rightOperandReference
  of a Constraint.

- a [`unit`](https://www.w3.org/TR/odrl-vocab/#term-unit),
  the unit of measurement of the value of the rightOperand or
  rightOperandReference of a Constraint.

While several of the `LeftOperand` instances in place make sense only for
digital media access management (e.g. [Absolute Spatial Asset Position](https://www.w3.org/TR/odrl-vocab/#term-absoluteSpatialPosition)),
in the context of oc-acl, to support attribute based access control (ABAC),
extensions a required. In particular, to allow the definition
of attribute based expressions on `actors` and `resources`, we introduced
a sub class of the `LeftOperand` concept, called [`GenericLeftOperand`]( http://voc.orchestracities.io/oc-acl#GenericLeftOperand)
that has the following properties:

- [`scope`](http://voc.orchestracities.io/oc-acl#scope) that defines whether the
  scope of the operand is [`subject`](http://voc.orchestracities.io/oc-acl#subject)
  or [`object`](http://voc.orchestracities.io/oc-acl#object) (for the time being
  `environment`, often used in ABAC, is left out);

- [`attributeName`](http://voc.orchestracities.io/oc-acl#attributeName) that
  defines the name of the attribute to which the operator
  will be applied.

Leveraging this information, a policy that would allow read access to a given
resource only for actors born before 1st January 1978, would look something
like:

```ttl
@base <http://example.com/> .
@prefix ex: <http://example.com/> .
@prefix acl: <http://www.w3.org/ns/auth/acl#> . 
@prefix oc-acl: <http://voc.orchestracities.io/oc-acl#> . 
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .


ex:leftOperand1 a oc-acl:LeftOperandAttribute ;
    oc-acl:scope oc-acl:subject ;
    oc-acl:attributeName "dateOfBirth" .

ex:constraint1 a odrl:Constraint ;
    odrl:leftOperand ex:leftOperand1 ;
    odrl:operator odrl:lt ;
    odrl:rightOperand "1978-01-01"^^xsd:date .

ex:policy1 a acl:Authorization ;
        acl:accessTo <urn:entity:1> ;
        acl:agentClass acl:AuthenticatedAgent ;
        oc-acl:constraint ex:constraint1 ;
        acl:mode acl:Read .
```

Of course specific `leftOperand` may be additionally defined based on
the different scenarios of application of oc-acl.

### Data Protection Policies

Beyond access control policies, we introduced a new set of policies (agent
independent) that defines which data protection policy is applied to
a specific attribute within a data or data type. While such policies are not
a simple extension to Web Access Control, but rather a new set of policies,
we include them in the same vocabulary since they are strictly related.
The policies leverage the [Data Privacy Vocabulary (DPV)](https://w3c.github.io/dpv/dpv-skos/).

In the current version of the vocabulary, a data protection policy is an
data protection rule (i.e. instance of `oc-acl:DataProtection`) with the
following properties:

- an *protected resource*
  (i.e. an `oc-acl:protectedResource` predicate) that defines the resources to which the
  data protection rule applies to. It can by a resource, or a class
  of resources.
- an *protected attribute*
  (i.e. an `oc-acl:protectedAttribute` predicate) that defines the attribute of
  the resource that is protected by the rule (thus allowing multiple attributes
  having different protection techniques).
- an [*protection mode*] (i.e. an `oc-acl:protectionMode` predicate) that defines wether
  the protection occurs in transit (i.e. [`dpv:EncryptionInTransfer`](https://w3id.org/dpv/dpv-skos#EncryptionInTransfer))
  or at rest (i.e. [`dpv:EncryptionAtRest`](https://w3id.org/dpv/dpv-skos#EncryptionAtRest)).
- a [*protection technique*] (i.e. an `oc-acl:protectionTechnique` predicate) that defines which
  algorithm is used to protect the resource, i.e. instances of [`dpv:DataAnonymisationTechnique`](https://w3id.org/dpv/dpv-skos#DataAnonymisationTechnique)
  or [`dpv:CryptographicMethods`](https://w3id.org/dpv/dpv-skos#CryptographicMethods)
  techniques.

  Leveraging this information, a policy that would allow read access to a given
  resource only for actors born before 1st January 1978, would look something
  like:

```ttl
@base <http://example.com/> .
@prefix ex: <http://example.com/> .
@prefix acl: <http://www.w3.org/ns/auth/acl#> . 
@prefix oc-acl: <http://voc.orchestracities.io/oc-acl#> . 
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

ex:policyProtection1 a oc-acl:DataProtection ;
        oc-acl:protectedResource <urn:entity:1> ;
        oc-acl:protectedAttribute <urn:attribute:1> ;
        oc-acl:protectionMode dpv:EncryptionInTransfer ;
        oc-acl:protectionTechnique dpv:HomomorphicEncryption .
```
