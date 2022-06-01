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
supported by WAC vocabulary with ODRL constraints that need to be verified for an
authorized agent to perform an operation on a resource.

As result, policies in oc-acl looks like:

```ttl
@base <http://example.com/> .
@prefix ex: <http://example.com/> .
@prefix acl: <http://www.w3.org/ns/auth/acl#> . 
@prefix oc-acl: <http://voc.orchestracities.io/oc-acl#> . 
@prefix odrl: <http://www.w3.org/ns/odrl/2/> .

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

- `oc-acl:ResourceTenantAgent`, an [authenticated agent](https://solid.github.io/web-access-control-spec/#acl-agentclass-authenticated-agent) that belongs to the same tenant as the resource authorized.

