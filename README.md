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
        acl:mode acl:Read, oc-acl:Delete .
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
