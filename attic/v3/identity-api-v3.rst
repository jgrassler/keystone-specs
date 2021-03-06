OpenStack Identity API v3
=========================

The Identity API primarily fulfills authentication and authorization needs
within OpenStack, and is intended to provide a programmatic facade in front of
existing authentication and authorization system(s).

The Identity API also provides endpoint discovery through a service catalog,
identity management, project management, and a centralized repository for
policy engine rule sets.

What's New in Version 3.7
-------------------------

- Addition of the ``password_expires_at`` field to the user response object.
- Introduce a flag to bypass expiration and revocation checking.

What's New in Version 3.6
-------------------------

- Listing role assignments for a tree of projects.
- Setting the project ``is_domain`` attribute enables a project to behave as
  a domain.
- Addition of the ``is_domain`` field to project scoped token response that
  represents whether a project is acting as a domain.
- Enable or disable a subtree in the project hierarchy.
- Delete a subtree in the project hierarchy.
- Additional identifier for tokens scoped to the designated ``admin project``.
- Addition of ``domain_id`` filter to list user projects
- One Role can imply another via role_inference rules.
- Enhance list role assignment to optionally provide names of entities.
- The defaults for domain-specific configuration options can be retrieved.
- Assignments can be specified as inherited, causing the assignment to be
  placed on any sub-projects.
- Support for domain specific roles.
- Support ``enabled`` and ``id`` as optional attributes to filter identity
  providers when listing.

What's New in Version 3.5
-------------------------

- Addition of ``type`` optional attribute to list credentials.
- Addition of ``region_id`` optional attribute to list endpoints.
- Addition of ``is_domain`` optional attribute to projects. Setting this
  currently has no effect, it is reserved for future use.

What's New in Version 3.4
-------------------------

- For tokenless authorization, the scope information may be set in the
  request headers.
- Addition of ``parent_id`` optional attribute to projects. This enables the
  construction of a hierarchy of projects.
- Addition of domain specific configuration management for a domain entity.
- Removal of ``url`` optional attribute for ``regions``. This attribute was
  only used for the experimental phase of keystone-to-keystone federation and
  has been superseded by making service provider entries have its own entry in
  the service catalog.
- The JSON Home support now will indicate the status of resource if it is not
  stable and current.

What's New in Version 3.3
-------------------------

These features are considered stable as of September 4th, 2014.

- Addition of ``name`` optional variable to be included from service definition
  into the service catalog.
- Introduced a stand alone call to retrieve a service catalog.
- Introduced support for JSON Home.
- Introduced a standard call to retrieve possible project and domain scope
  targets for a token.
- Addition of ``url`` optional attribute for ``regions``.

What's New in Version 3.2
-------------------------

These features are considered stable as of January 23, 2014.

- Introduced a mechanism to opt-out from catalog information during
  token validation
- Introduced a region resource for constructing a hierarchical
  container of groups of service endpoints
- Inexact filtering is supported on string attributes
- Listing collections may indicate only a subset of the data has been
  provided if a particular deployment has limited the number of entries
  a query may return

What's New in Version 3.1
-------------------------

These features are considered stable as of July 18, 2013.

- A token without an explicit scope of authorization is issued if the user does
  not specify a project and does not have authorization on the project
  specified by their default project attribute
- Introduced a generalized call for getting role assignments, with filtering
  for user, group, project, domain and role
- Introduced a mechanism to opt-out from catalog information during token
  creation
- Added optional bind information to token structure

What's New in Version 3.0
-------------------------

These features are considered stable as of February 20, 2013.

- Former "Service" and "Admin" APIs (including CRUD operations previously
  defined in the v2 OS-KSADM extension) are consolidated into a single core API
- "Tenants" are now known as "projects"
- "Groups": a container representing a collection of users
- "Domains": a high-level container for projects, users and groups
- "Policies": a centralized repository for policy engine rule sets
- "Credentials": generic credential storage per user (e.g. EC2, PKI, SSH, etc.)
- Roles can be granted at either the domain or project level
- User, group and project names only have to be unique within their owning
  domain
- Retrieving your list of projects (previously ``GET /tenants``) is now
  explicitly based on your user ID: ``GET /users/{user_id}/projects``
- Tokens explicitly represent user+project or user+domain pairs
- Partial updates are performed using the HTTP ``PATCH`` method
- Token ID values no longer appear in URLs

Document Overview
-----------------

This document is always evolving as new features are added, use cases are
clarified, etc. API features added since the original publication of this
document are summarized above, grouped by the API version in which they were
first introduced. This document is treated as the single source of truth for
the entire 3.x series of the API.

A particular implementation of the API is never referenced by name. This is
documentation for the HTTP API itself. Details of the code-base servicing the
API, such as architecture, configuration, and deployment, are not relevant
here.

The "API Conventions" section defines architectural patterns applied to the
entire API until a portion of the API documents an exception to the overall
conventions. Details of the conventions are not repeated throughout the
document, except in examples, so the reader is expected to have understood the
conventions before reading any further. The goal is to reduce the cost of
documentation maintenance (DRY) and improve self-consistency across the API,
which makes the API more intuitive to readers and fosters simpler
implementations.

A high level overview of the resources presented by the API are documented in
the "API Resources" section, including required and optional attributes, use
cases and expected behaviors. Specific API calls are not enumerated, although
the feature set of the related calls should be described if it deviates from
the conventions used by the rest of the API (for example, a resource could be
constrained as "a read-only collection").

Finally, the specific calls supported by the API are enumerated with examples
at the end of the document. The examples are intended to be "realistic"
representations of actual requests and responses you could expect from an
implementation of the API. Specifically, the JSON should be syntactically valid
and use data that is self-consistent with related calls.

The features described by this document are intended to be applicable to all
implementations of the API. If a particular implementation or deployment should
not be expected to have a use case for a particular feature, that feature
should be documented as an extension to this API. Extensions may suffix
existing resources with their own namespace in order to add new resources, or
prefix new attributes on existing resource representations. To clearly
distinguish extensions from the core API (which is described by this document)
and avoid namespace collisions between extensions, suffixes and prefixes are
composed of an uppercased abbreviation of the organization supporting the
extension (such as "OS" for OpenStack), followed by a hyphen ("-"), followed by
an uppercased abbreviation of the extension name (such as "OAUTH1" for OAuth
1.0). Therefore, an extension could be identified as "OS-OAUTH1".

API Conventions
---------------

This section describes architectural patterns applied throughout the Identity
API, unless an exception to these conventions is specifically documented. In
general, the Identity API provides an HTTP interface using JSON as the primary
transport format.

Each resource contains a canonically unique identifier (ID) defined by the
Identity service implementation and is provided as the ``id`` attribute;
Resource ID's are strings of non-zero length.

The resource paths of all collections are plural and are represented at the
root of the API (e.g. ``/v3/policies``).

TCP port 35357 is designated by the Internet Assigned Numbers Authority
("IANA") for use by OpenStack Identity services. Example API requests and
responses in this document therefore assume that the Identity service
implementation is deployed at the root of ``http://identity:35357/``.

Headers
~~~~~~~

- ``X-Auth-Token``

  This header is used to convey the API user's authentication token when
  accessing Identity APIs.

- ``X-Subject-Token``

  This header is used to convey the subject of the request for token-related
  operations.

*New in version 3.4*

- ``X-Project-Id``
- ``X-Project-Name``
- ``X-Project-Domain-Id``
- ``X-Project-Domain-Name``

  For tokenless authorization, these headers are used to convey the project
  scope information. To scope to a project, caller must provide either
  ``X-Project-Id`` or combination of ``X-Project-Name`` and
  ``X-Project-Domain-Id`` or ``X-Project-Domain-Name``.

- ``X-Domain-Id``
- ``X-Domain-Name``

  For tokenless authorization, these headers are used to convey the domain
  scope information. To scope to a domain, caller must provide either
  ``X-Domain-Id`` or ``X-Domain-Name``.

Required Attributes
~~~~~~~~~~~~~~~~~~~

For collections:

- ``links`` (object)

  Specifies a list of relational links to the collection.

- ``self`` (url)

  A self-relational link provided as an absolute URL. This attribute is
  provided by the identity service implementation.

- ``previous`` (url)

  A relational link to the previous page of the list, provided as an absolute
  URL. This attribute is provided by the identity service implementation. May
  be null.

- ``next`` (url)

  A relational to the next page of the list, provided as an absolute URL. This
  attribute is provided by the identity service implementation. May be null.

For members:

- ``id`` (string)

  Globally unique resource identifier. This attribute is provided by the
  identity service implementation.

- ``links`` (object)

  Specifies a set of relational links relative to the collection member.

- ``self`` (url)

  A self-relational link provided as an absolute URL. This attribute is
  provided by the identity service implementation.

Optional Attributes
~~~~~~~~~~~~~~~~~~~

For collections:

- ``truncated`` (boolean)

  In the case where a particular implementation has restricted the number of
  entries that can be returned in a collection and not all entries could be
  included, the list call will return a status code of 200 (OK), with
  ``truncated`` set to ``true``. If this attribute is not present (or is set to
  ``false``) then the list represents the complete collection, unless either
  the ``next`` or ``previous`` links are not ``null``, in which case the list
  represents a page within the complete collection.

CRUD Operations
~~~~~~~~~~~~~~~

Unless otherwise documented (tokens being the notable exception), all resources
provided by the Identity API support basic CRUD operations (create, read,
update, delete).

The examples in this section utilize a resource collection of Entities on
``/v3/entities`` which is not actually a part of the Identity API, and is used
for illustrative purposes only.

Create an Entity
^^^^^^^^^^^^^^^^

When creating an entity, you must provide all required attributes (except those
provided by the Identity service implementation, such as the resource ID):

Request:

::

    POST /entities

    {
        "entity": {
            "name": string,
            "description": string,
            "enabled": boolean
        }
    }

The full entity is returned in a successful response (including the new
resource's ID and a self-relational link), keyed by the singular form of the
resource name:

::

    201 Created

    {
        "entity": {
            "id": string,
            "name": string,
            "description": string,
            "enabled": boolean,
            "links": {
                "self": url
            }
        }
    }

List Entities
^^^^^^^^^^^^^

Request the entire collection of entities:

::

    GET /entities

A successful response includes a list of anonymous dictionaries, keyed by the
plural form of the resource name (identical to that found in the resource URL):

::

    200 OK

    {
        "entities": [
            {
                "id": string,
                "name": string,
                "description": string,
                "enabled": boolean,
                "links": {
                    "self": url
                }
            },
            {
                "id": string,
                "name": string,
                "description": string,
                "enabled": boolean,
                "links": {
                    "self": url
                }
            }
        ],
        "links": {
            "self": url,
            "next": url,
            "previous": url
        }
    }

List Entities filtered by attribute
'''''''''''''''''''''''''''''''''''

Beyond each resource's canonically unique identifier (the ``id`` attribute),
not all attributes are guaranteed unique on their own. To filter a list of
resources based on a specific attribute, we can perform a filtered query using
one or more query parameters:

::

    GET /entities?name={entity_name}&enabled

If multiple filters are specified in a query, then all filters must match for
an entity to be included in the response. The values specified in a filter must
be of the same type as the attribute, and in the case of strings are limited to
the same maximum length as the attribute.

The response is a subset of the full collection:

::

    200 OK

    {
        "entities": [
            {
                "id": string,
                "name": string,
                "description": string,
                "enabled": boolean,
                "links": {
                    "self": url
                }
            }
        ],
        "links": {
            "self": url,
            "next": url,
            "previous": url
        }
    }

*New in version 3.2* String attributes may also be filtered using inexact
patterns, for example:

::

    GET /entities?name__startswith={initial_characters_of_entity_name}

The following inexact suffixes are supported:

- ``__startswith``

  Matches if the attribute starts with the characters specified, with the
  comparison being case-sensitive.

- ``__istartswith``

  Matches if the attribute starts with the characters specified, with the
  comparison being case-insensitive.

- ``__endswith``

  Matches if the attribute ends with the characters specified, with the
  comparison being case-sensitive.

- ``__iendswith``

  Matches if the attribute ends with the characters specified, with the
  comparison being case-insensitive.

- ``__contains``

  Matches if the attribute contains the characters specified, with the
  comparison being case-sensitive.

- ``__icontains``

  Matches if the attribute contains the characters specified, with the
  comparison being case-insensitive.

  Inexact filters specified for non-string attributes will be ignored.

Get an Entity
^^^^^^^^^^^^^

Request a specific entity by ID:

::

    GET /entities/{entity_id}

The full resource is returned in response:

::

    200 OK

    {
        "entity": {
            "id": string,
            "name": string,
            "description": string,
            "enabled": boolean,
            "links": {
                "self": url
            }
        }
    }

Nested collections
''''''''''''''''''

An entity may contain nested collections, in which case the required attributes
for collections still apply; however, to avoid conflicts with other required
attributes, the required attributes of the collection are prefixed with the
name of the collection. For example, if an ``entity`` contains a nested
collection of ``objects``, the ``links`` for the collection of ``objects`` is
called ``objects_links``:

::

    {
        "entity": {
            "id": string,
            "name": string,
            "description": string,
            "enabled": boolean,
            "links": {
                "self": url
            },
            "objects": [
                {
                    "id": string,
                    "name": string,
                    "description": string,
                    "enabled": boolean,
                    "links": {
                        "self": url
                    }
                }
            ],
            "objects_links": {
                "self": url,
                "next": url,
                "previous": url
            }
        }
    }

Update an Entity
^^^^^^^^^^^^^^^^

Partially update an entity (unlike a standard ``PUT`` operation, only the
specified attributes are replaced):

::

    PATCH /entities/{entity_id}

    {
        "entity": {
            "description": string
        }
    }

The full entity is returned in response:

::

    200 OK

    {
        "entity": {
            "id": string,
            "name": string,
            "description": string,
            "enabled": boolean,
            "links": {
                "self": url
            }
        }
    }

Delete an Entity
^^^^^^^^^^^^^^^^

Delete a specific entity by ID:

::

    DELETE /entities/{entity_id}

A successful response does not include a body:

::

    204 No Content

HTTP Status Codes
~~~~~~~~~~~~~~~~~

The Identity API uses a subset of the available HTTP status codes to
communicate specific success and failure conditions to the client.

``200 OK``
^^^^^^^^^^

This status code is returned in response to successful ``GET``, ``HEAD`` and
``PATCH`` operations.

``201 Created``
^^^^^^^^^^^^^^^

This status code is returned in response to successful ``POST`` operations.

``204 No Content``
^^^^^^^^^^^^^^^^^^

This status code is returned in response to successful ``HEAD``, ``PUT`` and
``DELETE`` operations.

``300 Multiple Choices``
^^^^^^^^^^^^^^^^^^^^^^^^

This status code is returned by the root identity endpoint, with references to
one or more Identity API versions (such as ``/v3/``).

``400 Bad Request``
^^^^^^^^^^^^^^^^^^^

This status code is returned when the Identity service fails to parse the
request as expected. This is most frequently returned when a required attribute
is missing, a disallowed attribute is specified (such as an ``id`` on ``POST``
in a basic CRUD operation), or an attribute is provided of an unexpected data
type.

The client is assumed to be in error.

``401 Unauthorized``
^^^^^^^^^^^^^^^^^^^^

This status code is returned when either authentication has not been performed,
the provided X-Auth-Token is invalid or authentication credentials are invalid
(including the user, project or domain having been disabled).

``403 Forbidden``
^^^^^^^^^^^^^^^^^

This status code is returned when the request is successfully authenticated but
not authorized to perform the requested action.

``404 Not Found``
^^^^^^^^^^^^^^^^^

This status code is returned in response to failed ``GET``, ``HEAD``, ``POST``,
``PUT``, ``PATCH`` and ``DELETE`` operations when a referenced entity cannot be
found by ID. In the case of a ``POST`` request, the referenced entity may be in
the request body as opposed to the resource path.

``409 Conflict``
^^^^^^^^^^^^^^^^

This status code is returned in response to failed ``POST`` and ``PATCH``
operations. For example, when a client attempts to update an entity's unique
attribute which conflicts with that of another entity in the same collection.

Alternatively, a client should expect this status code when attempting to
perform the same create operation twice in a row on a collection with a
user-defined and unique attribute. For example, a User's ``name`` attribute is
defined to be unique and user-defined, so making the same ``POST /users``
request twice in a row will result in this status code.

The client is assumed to be in error.

``500 Internal Server Error``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This status code is returned when an unexpected error has occurred in the
Identity service implementation.

``501 Not Implemented``
^^^^^^^^^^^^^^^^^^^^^^^

This status code is returned when the Identity service implementation is unable
to fulfill the request because it is incapable of implementing the entire API
as specified.

For example, an Identity service may be incapable of returning an exhaustive
collection of Projects with any reasonable expectation of performance, or lack
the necessary permission to create or modify the collection of users (which may
be managed by a remote system); the implementation may therefore choose to
return this status code to communicate this condition to the client.

``503 Service Unavailable``
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This status code is returned when the Identity service is unable to communicate
with a backend service, or by a proxy in front of the Identity service unable
to communicate with the Identity service itself.

API Resources
-------------

Users: ``/v3/users``
~~~~~~~~~~~~~~~~~~~~

User entities represent individual API consumers and are owned by a specific
domain.

Role grants explicitly associate users with projects or domains. Each
user-project or user-domain pair can have a unique set of roles granted on
them.

A user without any role grants is effectively useless from the perspective of
an OpenStack service and should never have access to any resources. It is
allowed, however, as a means of acquiring or loading users from external
sources prior to mapping them to projects.

Additional required attributes:

- ``name`` (string)

  Unique user name, within the owning domain.

Optional attributes:

- ``domain_id`` (string)

  References the domain which owns the user; if a domain is not specified by
  the client, the Identity service implementation will default it to the domain
  to which the client's token is scoped.

- ``default_project_id`` (string)

  References the user's default project against which to authorize, if the API
  user does not explicitly specify one when creating a token. Setting this
  attribute does not grant any actual authorization on the project, and is
  merely provided for the user's convenience. Therefore, the referenced project
  does not need to exist within the user's domain.

  *New in version 3.1* If the user does not have authorization to their default
  project, the default project will be ignored at token creation.

- ``description`` (string)

- ``enabled`` (boolean)

  Setting this value to ``false`` prevents the user from authenticating or
  receiving authorization. Additionally, all pre-existing tokens held by the
  user are immediately invalidated. Re-enabling a user does not re-enable
  pre-existing tokens.

- ``password`` (string)

  The default form of credential used during authentication.

Example entity:

::

    {
        "user": {
            "default_project_id": "263fd9",
            "domain_id": "1789d1",
            "enabled": true,
            "id": "0ca8f6",
            "links": {
                "self": "http://identity:35357/v3/users/0ca8f6"
            },
            "name": "Joe",
            "password_expires_at": "2016-11-06T15:32:17.000000"
        }
    }

Note: password_expires_at is new a response only attribute; is not a valid
request attribute. A null value means that the password never expires.

Groups: ``/v3/groups``
~~~~~~~~~~~~~~~~~~~~~~

Group entities represent a collection of Users and are owned by a specific
domain. As with individual users, role grants explicitly associate groups with
projects or domains. A group role grant onto a project/domain is the equivalent
of granting each individual member of the group the role on that
project/domain. Once a group role grant has been made, the addition or removal
of a user to such a group will result in the automatic granting/revoking of
that role to the user, which will also cause any token containing that user and
project/domain to be revoked.

As with users, a group entity without any role grants is effectively useless
from the perspective an OpenStack service and should never have access to any
resources. It is allowed, however, as a means of acquiring or loading
users/groups from external sources prior to mapping them to projects/domains.

Additional required attributes:

- ``name`` (string)

  Unique group name, within the owning domain.

Optional attributes:

- ``domain_id`` (string)

  References the domain which owns the group; if a domain is not specified by
  the client, the Identity service implementation will default it to the domain
  to which the client's token is scoped.

- ``description`` (string)

Example entity:

::

    {
        "group": {
            "description": "Developers cleared for work on all general projects"
            "domain_id": "1789d1",
            "id": "70febc",
            "links": {
                "self": "http://identity:35357/v3/groups/70febc"
            },
            "name": "Developers"
        }
    }

Credentials: ``/v3/credentials``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Credentials represent arbitrary authentication credentials associated with a
user. A user may have zero or more credentials, each optionally scoped to a
specific project.

Additional required attributes:

- ``user_id`` (string)

  References the user which owns the credential.

- ``type`` (string)

  Representing the credential type, such as ``ec2`` or ``cert``. A specific
  implementation may determine the list of supported types.

- ``blob`` (blob)

  Arbitrary blob of the credential data, to be parsed according to the
  ``type``.

Optional attributes:

- ``project_id`` (string)

  References a project which limits the scope the credential applies to.
  This attribute is **mandatory** if the credential type is ``ec2``.

Example entity:

::

    {
        "credential": {
            "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
            "id": "80239a",
            "links": {
                "self": "http://identity:35357/v3/credentials/80239a"
            },
            "project_id": "263fd9",
            "type": "ec2",
            "user_id": "0ca8f6"
        }
    }

Projects: ``/v3/projects``
~~~~~~~~~~~~~~~~~~~~~~~~~~

Projects represent the base unit of "ownership" in OpenStack, in that all
resources in OpenStack should be owned by a specific project ("projects" were
also formerly known as "tenants"). A project itself must be owned by a specific
domain.

*New in version 3.6*, projects may, in addition to acting as containers for
OpenStack resources, act as a domain (by setting the attribute ``is_domain`` to
``true``), in which case it provides a namespace in which users, groups and
other projects can be created. In fact, a domain created using the
``POST /domains`` API will actually be represented as a project with
``is_domain`` set to ``true`` with no parent (``parent_id`` is ``null``).
Projects that are acting as a domain created via the ``POST /projects`` API
must also be specified with no parent (i.e. if ``parent_id`` is included it
must be ``null``). Issuing a create project request for a project acting as a
domain with a ``parent_id`` that is not null will cause an HTTP
``400 Bad Request`` to be returned.

Given this, all projects are considered part of a project hierarchy. Projects
created in a domain prior to version 3.6 are represented as a two-level
hierarchy, with a project that has ``is_domain`` set to ``true`` as the root
and all other projects referencing the root as their parent.

A project acting as a domain can potentially also act as a container for
OpenStack resources, although this depends on whether the policy rule for the
relevant resource creation allows this.

Required attributes:

- ``name`` (string)

  Unique project name, within the owning domain. A project name for a project
  acting as a domain must be unique across all domains.

Optional attributes:

- ``is_domain`` (boolean) *New in version 3.6*

  Represents if the project is acting as a domain. If this flag is set to
  ``true``, the project also acts as a domain, providing a namespace in which
  users, groups and other projects can be created. If the flag is set to
  ``false``, then this is a regular project, which can only contain resources.
  If not provided on project creation, ``is_domain`` defaults to ``false``.
  This flag is immutable and can't be updated after the project is created.

- ``parent_id`` (string) *New in version 3.4*

  References the parent project. If specified on project creation, this places
  the project within a hierarchy and implicitly defines the owning domain,
  which will be the closest ancestor in the hierarchy acting as a domain.
  If a parent project is not specified and ``is_domain`` is ``false``, then the
  project will use its owning domain as its parent. If a parent project is not
  specified and ``is_domain`` is ``true``, then the project is acting as a
  top level domain with no parents and ``parent_id`` will be set to ``null``.
  ``parent_id`` is immutable, and can't be updated after the project is
  created - hence a project cannot be moved within the hierarchy.

- ``domain_id`` (string)

  References the domain which owns the project. For projects not acting as a
  domain, this will be the closest ancestor in the hierarchy to be acting as
  a domain. Projects acting as a domain have no ancestors, and the domain_id
  will be set to ``null``. On project creation, if neither ``domain_id`` or
  ``parent_id`` is specified by the client, the Identity service implementation
  will default to the domain to which the client's token is scoped. If one or
  other of ``domain_id`` and ``parent_id`` is specified, then this will define
  the domain of the project. If both ``domain_id`` and ``parent_id`` are
  specified, and they do not indicate the same domain, an HTTP
  ``400 Bad Request`` will be returned.

- ``description`` (string)

- ``enabled`` (boolean)

  Setting this attribute to ``false`` prevents users from authorizing against
  this project. Additionally, all pre-existing tokens authorized for the
  project are immediately invalidated. Re-enabling a project does not re-enable
  pre-existing tokens.

Example entity of a project acting as a domain:

::

    {
        "project": {
            "domain_id": null,
            "enabled": true,
            "id": "1789d1",
            "links": {
                "self": "http://identity:35357/v3/projects/1789d1"
            },
            "name": "development",
            "parent_id": null,
            "is_domain": true
        }
    }

A project that is not acting as a domain, but is a top level project, will have
both the ``parent_id`` and ``domain_id`` referencing the project acting as a
domain, for example:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "project-x",
            "parent_id": "1789d1",
            "is_domain": false
        }
    }

A project that is the child of the above project, will have the same
``domain_id`` as its parent, and a ``parent_id`` that references its parent,
for example:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "63abc1",
            "links": {
                "self": "http://identity:35357/v3/projects/63abc1"
            },
            "name": "project-y",
            "parent_id": "263fd9",
            "is_domain": false
        }
    }

Domains: ``/v3/domains``
~~~~~~~~~~~~~~~~~~~~~~~~

Domains represent collections of users, groups and projects. Each is owned by
exactly one domain. Users, however, can be associated with multiple projects by
granting roles to the user on a project (including projects owned by other
domains).

Starting with version 3.6, domains created using the ``POST /domains`` API will
actually be represented as a project with ``is_domain`` set to ``true`` with no
parent.

Each domain defines a namespace in which certain API-visible name attributes
exist, which affects whether those names need to be globally unique or simply
unique within that domain. Within the Identity API, there are five such name
attributes:

- *Domain Name*: This is always globally unique across all domains.

- *Role Name*: This is always globally unique across all domains.

- *User Name*: This is only unique within the owning domain.

- *Project Name*: This is only unique within the owning domain.

- *Group Name*: This is only unique within the owning domain.

Additional required attributes:

- ``name`` (string)

  Globally unique name.

Optional attributes:

- ``description`` (string)

- ``enabled`` (boolean)

  Setting this attribute to ``false`` prevents users from authorizing against
  this domain or any projects owned by this domain, and prevents users owned by
  this domain from authenticating or receiving any other authorization.
  Additionally, all pre-existing tokens applicable to the above entities are
  immediately invalidated. Re-enabling a domain does not re-enable pre-existing
  tokens.

Example entity:

::

    {
        "domain": {
            "enabled": true,
            "id": "1789d1",
            "links": {
                "self": "http://identity:35357/v3/domains/1789d1"
            },
            "name": "example.com"
        }
    }

Roles: ``/v3/roles/``
~~~~~~~~~~~~~~~~~~~~~

Roles entities are named identifiers used to map a collection of actions from a
user to either a specific project or across an entire domain.

*New in version 3.6* A role can be a global policy role (i.e. it will appear in
a policy file in one of the services) or a domain specific role. A domain
specific role can be used to build role inference rules that better model the
sets of policy roles that need to be assigned for users or groups of a
particular domain. Domain specific roles never actually appear in policy files,
they are expanded into their implied global policy roles at token
generation/validation time.

Additional required attributes:

- ``name`` (string)

  Globally (or domain-wide) unique name of the role.

Optional attributes:

- ``domain_id`` (string) *New in version 3.6*

  This attribute is immutable.

Example entity:

::

    {
        "role": {
            "domain_id": null,
            "id": "76e72a",
            "links": {
                "self": "http://identity:35357/v3/roles/76e72a"
            },
            "name": "admin"
        }
    }

Regions: ``/v3/regions``
~~~~~~~~~~~~~~~~~~~~~~~~

*New in version 3.2*

Region entities represent a general division of an OpenStack deployment. A
region may have zero or more sub-regions associated with it, making a tree-like
structured hierarchy possible for the OpenStack deployment.

It is important to note that the concept of a Region has no geographical
connotation to it. Deployers are free to use geographical names for their
regions, for example "us-east", but there is no requirement to do so.

Optional attributes:

- ``description`` (string)

  Freeform description field for the deployer to use as they choose to describe
  the region.

- ``parent_region_id`` (string)

  If the region is hierarchically a child of another region, this field shall
  be set to the id of the parent region.

Example entity:

::

    {
        "region": {
            "description": "2nd sub-region inside the US East region.",
            "id": "us-east-2",
            "links": {
              "self": "https://identity:35357/v3/regions/us-east-2"
            },
            "parent_region_id": "us-east"
        }
    }

Services: ``/v3/services``
~~~~~~~~~~~~~~~~~~~~~~~~~~

Service entities represent web services in the OpenStack deployment. A service
may have zero or more endpoints associated with it, although a service with
zero endpoints is essentially useless in an OpenStack configuration.

Additional required attributes:

- ``type`` (string)

  Describes the API implemented by the service. The following values are
  recognized within the OpenStack ecosystem: ``compute``, ``image``, ``ec2``,
  ``identity``, ``volume``, ``network``. To support non-core and future
  projects, the value should not be validated against this list.

Optional attributes:

- ``description`` (string)

  User-facing description of the service.

- ``enabled`` (boolean)

  Setting this value to ``false`` prevents the service and its endpoints from
  appearing in the service catalog.

- ``name`` (string)

  User-facing name of the service.

Example entity:

::

    {
        "service": {
            "enabled": true,
            "id": "ee057c",
            "links": {
                "self": "http://identity:35357/v3/services/ee057c"
            },
            "name": "Keystone",
            "type": "identity"
        }
    }

Endpoints: ``/v3/endpoints``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Endpoint entities represent URL endpoints for OpenStack web services.

Additional required attributes:

- ``service_id`` (string)

  References the service to which the endpoint belongs.

- ``interface`` (string)

  Describes the visibility of the endpoint according to one of the following
  values:

  - `public`: intended for consumption by end users, generally on a publicly
    available network interface

  - `internal`: intended for consumption by end users, generally on an
    unmetered internal network interface

  - `admin`: intended only for consumption by those needing administrative
    access to the service, generally on a secure network interface

- ``url`` (string)

  Fully qualified URL of the service endpoint.

Optional attributes:

- ``region`` (string)

  **Deprecated in v3.2**. Use ``region_id``

  Represents the geographic location of the service endpoint, if relevant to
  the deployment. The value of this attribute is intended to be implementation
  specific in meaning.

- ``region_id`` (string)

  Represents the containing region of the service endpoint. *New in v3.2*

- ``enabled`` (boolean)

  Setting this value to ``false`` prevents the endpoint from appearing in the
  service catalog.

Example entity:

::

    {
        "endpoint": {
            "enabled": true,
            "id": "6fedc0",
            "interface": "internal",
            "links": {
                "self": "http://identity:35357/v3/endpoints/6fedc0"
            },
            "region_id": "us-east-2",
            "service_id": "ee057c",
            "url": "http://identity:35357/"
        }
    }

Tokens
~~~~~~

Tokens represent an authenticated user's identity and, potentially, explicit
authorization on a specific project or domain.

Tokens are generated by the Identity service via authentication, and may be
subsequently validated and/or revoked.

Unlike all other resources in the Identity API, ``token`` objects returned by
the API do not have ``id`` attributes. While ``token`` objects do have
identifiers, they are not passed in resource URL's nor are they included in the
objects themselves. Instead, they are passed in the ``X-Auth-Token`` and
``X-Subject-Token`` headers, along with a ``Vary: X-Auth-Token,
X-Subject-Token`` header to inform caches of this pattern.

``token`` objects are only created by the identity service implementation;
clients are not expected to create them. Instead, clients provide the service
with ``auth`` objects in exchange for ``token`` objects.

Required attributes:

- ``expires_at`` (string, ISO 8601 extended format date time with microseconds)

  Specifies the expiration time of the token. Once established, a token's
  expiration may not be changed. A token may be revoked ahead of expiration. If
  the value represents a time in the past, the token is invalid.

- ``issued_at`` (string, ISO 8601 extended format date time with microseconds)

  Specifies the time at which the token was issued.

- ``user`` (object)

  References the user to which the token belongs.

  Includes the full resource description of a user.

- ``methods`` (list)

  The ``methods`` attribute indicates the accumulated set of authentication
  methods used to obtain the token. For example, if the token was obtained by
  ``password`` authentication, it will contain ``password``. Later, if the
  token is exchanged using the ``token`` authentication method one or more
  times, the subsequently created tokens will contain both ``password`` and
  ``token`` in their ``methods`` attribute.

  Notice the difference between ``methods`` and multifactor authentication. The
  ``methods`` attribute merely indicates the methods used to authenticate the
  user for the given token. It is up to the client to look for specific methods
  to determine the total number of factors.

- ``audit_ids`` (array)

  The ``audit_ids`` attribute is a list that contains no more than two
  elements. Each id in the ``audit_ids`` attribute is a randomly (unique)
  generated string that can be used to track the token.

  Each token will have its own unique audit identifier as the first element of
  the array. In the case of a token that was rescoped (exchanged for another
  token of the same or different scope), there will be a second audit
  identifier as the second element of the array. This conditional second
  identifier is the audit id string from the original token (i.e. the first
  token issued that was not a rescoped token).

  These audit identifiers can be used to track a specific use of token (or
  chain of tokens) across multiple requests and endpoints without exposing the
  token id to non-privileged users (e.g. via logs).

  Each audit identifier is a short urlsafe string.

Example token with ``audit_ids`` attribute (first element is the token's
``audit_id``, second is the ``audit_chain_id``):

::

    {
        "token": {
            "audit_ids": ["VcxU2JYqT8OzfUVvrjEITQ", "qNUTIJntTzO1-XUk5STybw"],
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "user": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                }
                "id": "0ca8f6",
                "name": "Joe",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        }
    }

Tokens issued prior to the inclusion of the audit id code will lack the
``audit_ids`` attribute. These tokens lacking ``audit_ids`` will
continue to function normally until revoked or expired. All newly issue
tokens will have the expected ``audit_ids`` attribute.

Optional attributes:

- ``project`` (object)

  Specifies the project authorization scope of the token. If this attribute is
  not provided, then the token is not authorized to access any project
  resources. The presence of this attribute conveys multi-tenancy to cloud
  services such that they can achieve resource isolation based on the
  authorized request context included in the token. This attribute must not be
  included if a ``domain`` attribute is included. A token with project-level
  authorization does not express any authorization on any domain-level
  resource.

  Includes the full resource description of a project.

- ``domain`` (object)

  Specifies the domain authorization scope of the token. This is to provide
  authorization appropriate to domain-level APIs, for example user and group
  management within a domain. If this attribute is not provided, then the token
  is not authorized to access any domain level resources. This attribute must
  not be included if a ``project`` attribute is included. A token with
  domain-level authorization does not express any authorization on any
  project-level resource.

  Includes the full resource description of a domain.

- ``is_admin_project`` (boolean) *New in version 3.6*

  A deployment can and should indicate that a specific project has elevated
  priviledges.  Tokens scoped to this project, called the `admin` project can
  then be identified as being associated with this project.  For deployments
  using this mechanism, policy rules can then check the value of this field
  to allow the bearer of such a token access to priviledged operations, for
  example those that are not project specific or are deployment-wide.

  Example project scoped token with ``is_admin_project`` attribute:

::

    {
        "token": {
            "audit_ids": ["VcxU2JYqT8OzfUVvrjEITQ", "qNUTIJntTzO1-XUk5STybw"],
            "expires_at": "2023-02-27T18:30:59.999999Z",
            "is_admin_project": true
            "is_domain": false,
            "issued_at": "2023-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "project": {
                "domain": {
                  "id": "b3652a",
                  "name": "domain_name1"
                },
                "id": "78eed8",
                "name": "project_name1"
            },
            "roles": [
                {
                    "id": "76e72a",
                    "name": "admin"
                }
            ],
            "user": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                }
                "id": "0ca8f6",
                "name": "Joe",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        }
    }

- ``catalog`` (list of object)

  Specifies all the services available to/for the token. It is represented as a
  list of service dictionaries with the following format:

::

        [
            {
                "id": "--service-id--",
                "type": "--service-type--",
                "name": "--service-name--",
                "endpoints": [
                    {
                        "id": "--endpoint-id--",
                        "interface": "--interface-name--",
                        "region": "--region-name--",
                        "url": "--endpoint-url--"
                    },
                    ...
                ]
            },
            ...
        ]

Required attributes for the service object are:

- ``id``: the service entity id.

- ``type``: Describes the API implemented by the service.

Optional attributes for the service object are:

- ``name``: User-facing name of the service. *New in version 3.3*

Required attributes for the endpoint object are:

- ``id``: The endpoint entity id.

- ``interface``: The visibility of the endpoint. Should be one of ``public``,
  ``internal`` or ``admin``.

- ``url``: Fully qualified URL of the service endpoint.

Optional attributes for the endpoint object are:

- ``region``: The geographic location of the service endpoint.

- ``bind`` (object) *New in version 3.1*

  Token binding refers to the practice of embedding information from external
  authentication providers (like a company's Kerberos server) inside the token
  such that a client may validate that the token is used in conjunction with
  that authentication mechanism. By coupling this authentication we can prevent
  re-use of a stolen token as an attacker would not have access to the external
  authentication.

  Specifies one or more external authorization mechanisms that can be used in
  conjunction with the token for it to be validated by a bind enforcing client.
  For example a token may only be used over a Kerberos authenticated connection
  or with a specific client certificate.

  Includes one or more mechanism identifiers with protocol specific data. The
  officially supported mechanisms are ``kerberos`` and ``x509`` where:

  - The ``kerberos`` bind payload is of the form::

        "kerberos": {
            "principal": "USER@REALM"
        }

    ... where the user's Kerberos principal is "USER@REALM".

  - The ``x509`` bind payload is of the form::

        "x509": {
            "fingerprint": "0123456789ABCDEF",
            "algorithm": "sha1"
        }

    The ``fingerprint`` is the hash of the client certificate to be validated
    in the specified algorithm. It should be the hex form without separating
    spaces or colons. The only supported ``algorithm`` is currently ``sha1``.

Example entity:

::

    {
        "token": {
            "audit_ids": [
                "VcxU2JYqT8OzfUVvrjEITQ",
                "qNUTIJntTzO1-XUk5STybw"
            ],
            "bind": {
                "kerberos": {
                    "principal": "USER@REALM"
                }
            },
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "user": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                },
                "id": "0ca8f6",
                "name": "Joe",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        }
    }

Policy
~~~~~~

Policies represent arbitrarily serialized policy engine rule sets to be
consumed by remote services.

Additional required attributes:

- ``blob`` (string)

  The policy rule set itself, as a serialized blob.

- ``type`` (string)

  The MIME Media Type of the serialized policy blob.

Example entity:

::

    {
        "policy": {
            "blob": "{\"default\": false}",
            "id": "c41a4c",
            "links": {
                "self": "http://identity:35357/v3/policies/c41a4c"
            },
            "type": "application/json"
        }
    }

JSON Home
---------

*New in version 3.3*

The Identity API supports JSON Home for resource and extension discovery. The
identity server will return a JSON Home document on a ``GET /v3`` request where
the ``Accept`` header indicates that the response should be
``application/json-home``. The JSON Home document contains a mapping of
"relationships" to the relative path or path template to the actual resource.

The JSON Home document includes not only the core APIs that are supported for
that version of the identity API, but also the resources for the extensions.

Each of the resources in the Core API below specify the "relationship" for the
resource. A client application can look up the resource path or path template
for a resource by looking for that resource in the JSON Home document.

*New in version 3.4*

By default all core resources defined by the v3 API should be considered as
stable and current. However, the JSON Home response document will indicate any
variance to this in the ``status`` property of the ``hints`` property of a
given resource.

Example resource response::

    {
          "resources": {
              "http://docs.openstack.org/api/openstack-identity/3/rel/domain_config" : {
                  "href-template": "/domains/{domain_id}/config",
                  "href-vars": {
                      "domain_id": "http://docs.openstack.org/api/openstack-identity/3/param/domain_id"
                  },
                  "hints": {
                      "status": "experimental"
                }
            }
        }
    }

Supported values of ``status`` are ``deprecated``, ``experimental`` and
``stable`` (which is the default). These values
have the following meanings:

- ``deprecated``: The resource has been marked as deprecated and will be
  removed in a future release. Clients using such a resource should
  plan to migrate to more current resources as soon as possible.

- ``experimental``: The resource is valid and can be used but is still
  maturing. While every attempt will be made to maintain the resource as is
  ahead of being marked as stable, it is possible that changes may need to
  be made.

- ``stable``: The resource is stable and current. This is the default, and
  the lack of a hints property, or a status property within that, can be taken
  as an indication that this resource is stable.

Core API
--------

Versions
~~~~~~~~

Describe API version
^^^^^^^^^^^^^^^^^^^^

::

    GET /v3/

The attributes in the ``version`` object are as follows:

- ``id``: A string with the current version, with major and minor components.
  For V3, the major version is "3". For an Identity server running Mitaka, the
  minor version is "7", so the ``id`` is "3.7".

- ``status``: A string with the current maturity level of the specification.
  This may be one of ``stable``, or ``deprecated``.

- ``updated``: A string with the time when the specification status last
  changed in ISO8601 format. For example, "2013-03-06T00:00:00Z".

Response:

::

    Status: 200 OK

    {
        "version": {
            "id": "v3.7",
            "links": [
                {
                    "href": "http://identity:35357/v3/",
                    "rel": "self"
                }
            ],
            "status": "stable",
            "updated": "2013-03-06T00:00:00Z"
        }
    }

*New in version 3.3*: ``GET /v3/`` will return a JSON Home response if the
``Accept`` header indicates that the client wants an ``application/json-home``
response. Note that the client must check the ``Content-Type`` in the response
because older servers will return a normal JSON response rather than the JSON
Home response. See the `JSON Home spec
<http://tools.ietf.org/html/draft-nottingham-json-home-03>`__ for a description
of the JSON Home document format.

The JSON Home document returned includes all the core components and also the
resources for the enabled extensions. Resources for disabled extensions aren't
included.

Request:

::

    GET /v3
    Accept: application/json-home

Response:

::

    {
        "resources": {
            "http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens": {
                "href": "/auth/tokens"
            }
        }
    }

Tokens
~~~~~~

Use cases:

- Given a user name and password, get a token to represent the user.

- Given a token, get a list of other domain/projects the user can access.

- Given a token, validate the token and return user, domain, project, roles
  and potential endpoints.

- Given a valid token, request another token with a different domain/project
  (change domain/project being represented with the user).

- Given a valid token, force it's immediate revocation.

Authenticate
^^^^^^^^^^^^

::

    POST /auth/tokens

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

Each request to create a token contains an attribute with ``identity``
information and, optionally, a ``scope`` describing the authorization scope
being requested. Example request structure:

::

    {
        "auth": {
            "identity": { ... },
            "scope": { ... }
        }
    }

Authentication: ``authentication``
''''''''''''''''''''''''''''''''''

Authentication is performed by specifying a list of authentication ``methods``,
each with a corresponding object, containing any attributes required by the
authentication method. Example request structure for three arbitrary
authentication methods:

::

    {
        "auth": {
            "identity": {
                "methods": ["x", "y", "z"],
                "x": { ... },
                "y": { ... },
                "z": { ... }
            }
        }
    }

The ``password`` authentication method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To authenticate by ``password``, the user must be uniquely identified in
addition to providing a ``password`` attribute.

The ``user`` may be identified by either ``id`` or ``name``. A user's ``id`` is
sufficient to uniquely identify the ``user``. Example request:

::

    {
        "auth": {
            "identity": {
                "methods": [
                    "password"
                ],
                "password": {
                    "user": {
                        "id": "0ca8f6",
                        "password": "secretsecret"
                    }
                }
            }
        }
    }

If the ``user`` is specified by ``name``, then the ``domain`` of the ``user``
must also be specified in order to uniquely identify the ``user``. Example
request:

::

    {
        "auth": {
            "identity": {
                "methods": [
                    "password"
                ],
                "password": {
                    "user": {
                        "domain": {
                            "id": "1789d1"
                        },
                        "name": "Joe",
                        "password": "secretsecret"
                    }
                }
            }
        }
    }

Alternatively, a ``domain`` ``name`` may be used to uniquely identify the
``user``. Example request:

::

    {
        "auth": {
            "identity": {
                "methods": [
                    "password"
                ],
                "password": {
                    "user": {
                        "domain": {
                            "name": "example.com"
                        },
                        "name": "Joe",
                        "password": "secretsecret"
                    }
                }
            }
        }
    }

The ``token`` authentication method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the authenticating user is already in possession of a valid token, then that
token is sufficient to identity the user. This method is typically used in
combination with request to change authorization scope.

::

    {
        "auth": {
            "identity": {
                "methods": [
                    "token"
                ],
                "token": {
                    "id": "e80b74"
                }
            }
        }
    }

Scope: ``scope``
^^^^^^^^^^^^^^^^

An authorization scope, including either a ``project``, ``domain``, or
``unscoped``, can be optionally specified as part of the request. If both a
``domain`` and a ``project`` are specified, an HTTP ``400 Bad Request`` will be
returned, as a token cannot be simultaneously scoped to both a ``project`` and
``domain``.

Project Scope
'''''''''''''

A ``project`` may be specified by either ``id`` or ``name``. An ``id`` is
sufficient to uniquely identify a ``project``. The contents of the ``identity``
section are orthogonal to the scope as it contains identity attributes for
authenticating the user, and nothing to do with authorization. Example request:

::

    {
        "auth": {
            "identity": {
                ...
            },
            "scope": {
                "project": {
                    "id": "263fd9"
                }
            }
        }
    }

If a ``project`` is specified by ``name``, then the ``domain`` of the
``project`` must also be specified in order to uniquely identify the
``project``.

Example request:

::

    {
        "auth": {
            "identity": {
                ...
            },
            "scope": {
                "project": {
                    "domain": {
                        "id": "1789d1"
                    },
                    "name": "project-x"
                }
            }
        }
    }

Alternatively, a ``domain`` ``name`` may be used to uniquely identify the
``project``. Example request:

::

    {
        "auth": {
            "identity": {
                ...
            },
            "scope": {
                "project": {
                    "domain": {
                        "name": "example.com"
                    },
                    "name": "project-x"
                }
            }
        }
    }

Domain Scope
''''''''''''

A ``domain`` scope may be specified by either the domain's ``id`` or ``name``
with equivalent results. Example request specifying a domain by ``id``:

::

    {
        "auth": {
            "identity": {
                ...
            },
            "scope": {
                "domain": {
                    "id": "1789d1"
                }
            }
        }
    }

Example request specifying a domain by ``name``:

::

    {
        "auth": {
            "identity": {
                ...
            },
            "scope": {
                "domain": {
                    "name": "example.com"
                }
            }
        }
    }

The catalog returned for a domain-scoped request can contain different services
and endpoints from a project ID depending on the deployment.

Unscoped
''''''''

A token request may, or may not, contain ``scope``. If an unscoped token
request contains ``scope`` and it is set to ``unscoped``, it is considered an
explicit unscoped token request. Which will return an unscoped response without
any authorization.

::

    {
        "auth": {
            "identity": {
                ...
            },
            "scope": "unscoped"
        }
    }

A request that does not explicitly set ``scope`` to ``unscoped`` may return a
project-scoped token if the user making the request has a role assigned to its
default project. Thus, it is recommended to set the authorization ``scope`` to
``unscoped`` if the intent is to receive an unscoped token. The following
request body would return a project-scoped response, if user ``0ca8f6`` had a
role assignment on their default project.

::

    {
        "auth": {
            "identity": {
                ...
            }
        }
    }


If there is no default project defined, or the user's default project has been
disabled or deleted, an unscoped token will be issued. Which is the same
behavior as asking for an explicit unscoped token.

*New in version 3.1* Additionally, if the user's default project is invalid, a
token will be issued without an explicit scope of authorization.

*New in version 3.4* A user may explicitly request an unscoped token by setting
the "scope" value of the token request to the string "unscoped."  This will
behave the same as a token request with no scope, where the user has no
default project defined.

Catalog Opt-Out
^^^^^^^^^^^^^^^

::

    POST /v3/auth/tokens?nocatalog

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

*New in version 3.1* If the caller specifies a ``nocatalog`` query parameter in
the authentication request, then the authentication response will not contain
the service catalog. The service catalog will otherwise be included in the
response by default.


Ignore Expiry and Revocation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /v3/auth/tokens?allow_expired=1

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

*New in version 3.7* If the caller specifies a true ``allow_expired`` query
parameter in the authentication request, then the authentication will return
expired token data within a server specified window.


Authentication responses
''''''''''''''''''''''''

A response without an explicit authorization scope does not contain a
``catalog``, ``project``, ``domain`` or ``roles`` but can be used to uniquely
identify the user. Example response:

::

    Headers:
        X-Subject-Token: e80b74

    {
        "token": {
            "audit_ids": ["VcxU2JYqT8OzfUVvrjEITQ", "qNUTIJntTzO1-XUk5STybw"],
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "user": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                },
                "id": "0ca8f6",
                "name": "Joe",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        }
    }

Notice that token ID is not part of the token data. Rather, it is conveyed in
the ``X-Subject-Token`` header.

A token scoped to a ``project`` will also have a service ``catalog``, along
with the user's roles applicable to the ``project``.

Provided there are enabled service providers, a token will be populated with
a list of such service providers, accessible in the token dictionary with the
key ``service_providers``.
Additional information about service providers can be found `here
<http://specs.openstack.org/openstack/keystone-specs/api/v3/identity-api-v3-os-federation-ext.html#service-providers>`__

*New in version 3.6* Project scoped tokens return an additional boolean field
called ``is_domain`` that represents whether a project acts as a domain.

Example response:

::

    Headers: X-Subject-Token

    X-Subject-Token: e80b74

    {
        "token": {
            "audit_ids": ["VcxU2JYqT8OzfUVvrjEITQ", "qNUTIJntTzO1-XUk5STybw"],
            "catalog": [
                {
                    "endpoints": [
                        {
                            "id": "39dc322ce86c4111b4f06c2eeae0841b",
                            "interface": "public",
                            "region": "RegionOne",
                            "url": "http://localhost:5000"
                        },
                        {
                            "id": "ec642f27474842e78bf059f6c48f4e99",
                            "interface": "internal",
                            "region": "RegionOne",
                            "url": "http://localhost:5000"
                        },
                        {
                            "id": "c609fc430175452290b62a4242e8a7e8",
                            "interface": "admin",
                            "region": "RegionOne",
                            "url": "http://localhost:35357"
                        }
                    ],
                    "id": "4363ae44bdf34a3981fde3b823cb9aa2",
                    "type": "identity",
                    "name": "keystone"
                }
            ],
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "is_domain": false,
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "project": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                },
                "id": "263fd9",
                "name": "project-x"
            },
            "roles": [
                {
                    "id": "76e72a",
                    "name": "admin"
                },
                {
                    "id": "f4f392",
                    "name": "member"
                }
            ],
            "service_providers": [
                {
                    "auth_url":"https://example.com:5000/v3/OS-FEDERATION/identity_providers/acme/protocols/saml2/auth",
                    "id": "sp1",
                    "sp_url": "https://example.com:5000/Shibboleth.sso/SAML2/ECP"
                },
                {
                    "auth_url":"https://other.example.com:5000/v3/OS-FEDERATION/identity_providers/acme/protocols/saml2/auth",
                    "id": "sp2",
                    "sp_url": "https://other.example.com:5000/Shibboleth.sso/SAML2/ECP"
                }
            ],
            "user": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                },
                "id": "0ca8f6",
                "name": "Joe",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        }
    }

A token scoped to a ``domain`` will also have a service ``catalog`` along with
the user's roles applicable to the ``domain``. Example response:

::

    Headers: X-Subject-Token

    X-Subject-Token: e80b74

    {
        "token": {
            "audit_ids": ["VcxU2JYqT8OzfUVvrjEITQ", "qNUTIJntTzO1-XUk5STybw"],
            "catalog": [
                {
                    "endpoints": [
                        {
                            "id": "39dc322ce86c4111b4f06c2eeae0841b",
                            "interface": "public",
                            "region": "RegionOne",
                            "url": "http://localhost:5000"
                        },
                        {
                            "id": "ec642f27474842e78bf059f6c48f4e99",
                            "interface": "internal",
                            "region": "RegionOne",
                            "url": "http://localhost:5000"
                        },
                        {
                            "id": "c609fc430175452290b62a4242e8a7e8",
                            "interface": "admin",
                            "region": "RegionOne",
                            "url": "http://localhost:35357"
                        }
                    ],
                    "id": "4363ae44bdf34a3981fde3b823cb9aa2",
                    "type": "identity",
                    "name": "keystone"
                }
            ],
            "domain": {
                "id": "1789d1",
                "name": "example.com"
            },
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "roles": [
                {
                    "id": "76e72a",
                    "name": "admin"
                },
                {
                    "id": "f4f392",
                    "name": "member"
                }
            ],
            "user": {
                "domain": {
                    "id": "1789d1",
                    "name": "example.com"
                },
                "id": "0ca8f6",
                "name": "Joe",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        }
    }

Authentication failures
'''''''''''''''''''''''

Several authentication errors are possible, including HTTP ``403 Forbidden``
and HTTP ``409 Conflict``, but here's an example of an HTTP
``401 Unauthorized`` response:

::

    Status: 401 Not Authorized

    {
        "error": {
            "code": 401,
            "message": "The request you have made requires authentication",
            "title": "Not Authorized"
        }
    }

Optionally, the Identity service implementation may return an
``authentication`` attribute to indicate the supported authentication methods.

::

    Status: 401 Not Authorized

    {
        "error": {
            "code": 401,
            "identity": {
                "methods": [
                    "password",
                    "token",
                    "challenge-response"
                ]
            },
            "message": "Need to authenticate with one or more supported methods",
            "title": "Not Authorized"
        }
    }

For authentication processes which require multiple round trips, the Identity
service implementation may return an HTTP ``401 Not Authorized`` with
additional information for the next authentication step.

For example:

::

    Status: 401 Not Authorized

    {
        "error": {
            "code": 401,
            "identity": {
                "challenge-response": {
                    "challenge": "What was the zip code of your birthplace?",
                    "session_id": "123456"
                },
                "methods": [
                    "challenge-response"
                ]
            },
            "message": "Additional authentications steps required.",
            "title": "Not Authorized"
        }
    }

Validate token and get service catalog
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /auth/tokens

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

To validate a token using the Identity API, pass your own token in the
``X-Auth-Token`` header, and the token to be validated in the
``X-Subject-Token`` header. The Identity service returns a service catalog in
the response. Example request:

::

    Headers:
        X-Auth-Token: 1dd7e3
        X-Subject-Token: c67580

No request body is required.

The Identity service will return the exact same response as when the subject
token was issued by ``POST /auth/tokens``.

Validate token
^^^^^^^^^^^^^^

::

    GET /auth/tokens?nocatalog

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

*New in version 3.2*

To validate a token using the Identity API without returning a service catalog
in the response. The request has the same format as ``GET /auth/tokens``.

The Identity service will return the exact same response as when the subject
token was issued by ``POST /auth/tokens?nocatalog``.

Check token
^^^^^^^^^^^

::

    HEAD /auth/tokens

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

This call is identical to ``GET /auth/tokens``, but no response body is
provided, even if an error occurs or the token is invalid.

Response:

::

    Status: 200 OK

Revoke token
^^^^^^^^^^^^

::

    DELETE /auth/tokens

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_tokens``

This call is identical to ``HEAD /auth/tokens`` except that the
``X-Subject-Token`` token is immediately invalidated, regardless of its
``expires_at`` attribute. An additional ``X-Auth-Token`` is not required. The
successful response status also differs from ``HEAD /auth/tokens``.

Response:

::

    Status: 204 No Content

Authentication Specific Routes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The key use cases we need to cover:

- Fetching a service catalog based upon the current authorization.

- Retrieve available scoping targets based upon the current authorization.

Get service catalog
^^^^^^^^^^^^^^^^^^^

::

    GET /auth/catalog

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_catalog``

*New in version 3.3*

This call returns a service catalog for the ``X-Auth-Token`` provided in the
request, even if the token does not contain a catalog itself (for example, if
it was generated using ``?nocatalog``).

The structure of the ``catalog`` object is identical to that contained in a
``token``.

Response:

::

    Status: 200 OK

    {
        "catalog": [
            {
                "endpoints": [
                    {
                        "id": "39dc322ce86c4111b4f06c2eeae0841b",
                        "interface": "public",
                        "region": "RegionOne",
                        "url": "http://localhost:5000"
                    },
                    {
                        "id": "ec642f27474842e78bf059f6c48f4e99",
                        "interface": "internal",
                        "region": "RegionOne",
                        "url": "http://localhost:5000"
                    },
                    {
                        "id": "c609fc430175452290b62a4242e8a7e8",
                        "interface": "admin",
                        "region": "RegionOne",
                        "url": "http://localhost:35357"
                    }
                ],
                "id": "4363ae44bdf34a3981fde3b823cb9aa2",
                "type": "identity",
                "name": "keystone"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/catalog",
            "previous": null,
            "next": null
        }
    }

Get available project scopes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /auth/projects

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_projects``

*New in version 3.3*

This call returns the list of projects that are available to be scoped to based
on the ``X-Auth-Token`` provided in the request.

The structure of the response is exactly the same as listing projects for a
user.

Response:

::

    Status: 200 OK

    {
        "projects": [
            {
                "domain_id": "1789d1",
                "enabled": true,
                "id": "263fd9",
                "links": {
                    "self": "https://identity:35357/v3/projects/263fd9"
                },
                "name": "Test Group"
            },
            {
                "domain_id": "1789d1",
                "enabled": true,
                "id": "50ef01",
                "links": {
                    "self": "https://identity:35357/v3/projects/50ef01"
                },
                "name": "Build Group"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/auth/projects",
            "previous": null,
            "next": null
        }
    }

Get available domain scopes
^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /auth/domains

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/auth_domains``

*New in version 3.3*

This call returns the list of domains that are available to be scoped to based
on the ``X-Auth-Token`` provided in the request.

The structure is the same as listing domains.

Response:

::

    Status: 200 OK

    {
        "domains": [
            {
                "description": "my domain description",
                "enabled": true,
                "id": "1789d1",
                "links": {
                    "self": "https://identity:35357/v3/domains/1789d1"
                },
                "name": "my domain"
            },
            {
                "description": "description of my other domain",
                "enabled": true,
                "id": "43e8da",
                "links": {
                    "self": "https://identity:35357/v3/domains/43e8da"
                },
                "name": "another domain"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/auth/domains",
            "previous": null,
            "next": null
        }
    }

Catalog
~~~~~~~

The key use cases we need to cover:

- CRUD for regions, services and endpoints

- Retrieving an endpoint URL by service, region, and interface

List regions
^^^^^^^^^^^^

::

    GET /regions

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/regions``

Optional query parameters:

- ``parent_region_id`` (string)

Response:

::

    Status: 200 OK

    {
        "regions": [
            {
                "description": "US East Region",
                "id": "us-east",
                "links": {
                    "self": "https://identity:35357/v3/regions/us-east",
                    "child_regions": "https://identity:35357/v3/regions?parent_region_id=us-east"
                },
                "parent_region_id": "us-east-coast"
            },
            ...
        ],
        "links": {
            "self": "https://identity:35357/v3/regions",
            "previous": null,
            "next": null
        }
    }

Get region
^^^^^^^^^^

::

    GET /regions/{region_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/region``

Response:

::

    Status: 200 OK

    {
        "region": {
            "description": "US Southwest Region",
            "id": "us-southwest",
            "links": {
                "self": "https://identity:35357/v3/regions/us-southwest",
                "child_regions": "http://identity:35357/v3/regions?parent_region_id=us-southwest"
            },
            "parent_region_id": "us-west-coast"
        }
    }

Create region
^^^^^^^^^^^^^

::

    POST /regions

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/regions``

Request:

::

    {
        "region": {
            "description": "US West Subregion 1",
            "parent_region_id": "829551"
        }
    }

Response:

::

    Status: 201 Created

    {
        "region": {
            "description": "US West Subregion 1",
            "id": "8ebd7f",
            "links": {
                "self": "https://identity:35357/v3/regions/8ebd7f",
                "child_regions": "https://identity:35357/v3/regions?parent_region_id=8ebd7f"
            },
            "parent_region_id": "829551"
        }
    }

- Adding a region with a parent\_region\_id that does not exist should fail
  with an HTTP ``404 Not Found``

- Adding a region with a parent\_region\_id that would form a circular
  relationship should fail with an HTTP ``409 Conflict``

Create region with specific ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /regions/{user_defined_region_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/region``

Request:

::

    {
        "region": {
            "description": "US Southwest Subregion 1",
            "parent_region_id": "us-south"
        }
    }

Response:

::

    Status: 201 Created

    {
        "region": {
            "description": "US Southwest Subregion 1",
            "id": "us-southwest-1",
            "links": {
                "self": "https://identity:35357/v3/regions/us-southwest-1",
                "child_regions": "https://identity:35357/v3/regions?parent_region_id=us-southwest-1"
            },
            "parent_region_id": "us-south"
        }
    }

- The {user\_defined\_region\_id} must be unique to the OpenStack deployment.
  If not, an HTTP ``409 Conflict`` should be returned.

- The {user\_defined\_region\_id} shall be urlencoded if the ID contains
  characters not permitted in a URI.

- Adding a region with a parent\_region\_id that does not exist should fail
  with an HTTP ``404 Not Found``

- Adding a region with a parent\_region\_id that would form a circular
  relationship should fail with an HTTP ``409 Conflict``

Update region
^^^^^^^^^^^^^

::

    PATCH /regions/{region_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/region``

Request:

::

    {
        "region": {
            "description": "US Southwest Subregion",
            "parent_region_id": "us-southwest"
        }
    }

Response:

::

    Status: 200 OK

    {
        "region": {
            "description": "US Southwest Subregion",
            "id": "us-southwest-1",
            "links": {
                "self": "https://identity:35357/v3/regions/us-southwest-1",
                "child_regions": "https://identity:35357/v3/regions?parent_region_id=us-southwest-1"
            },
            "parent_region_id": "us-southwest"
        }
    }

- Updating a region with a parent\_region\_id that does not exist should fail
  with an HTTP ``404 Not Found``

Delete region
^^^^^^^^^^^^^

::

    DELETE /regions/{region_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/region``

- Note: deleting a region with child regions should return an HTTP
  ``409 Conflict``

Response:

::

    Status: 204 No Content

List services
^^^^^^^^^^^^^

::

    GET /services

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/services``

Optional query parameters:

- ``name`` (string)

  *New in version 3.3*

- ``type`` (string)

Response:

::

    Status: 200 OK

    {
        "services": [
            {
                "description": "OpenStack Volume Service",
                "id": "ee057c",
                "links": {
                    "self": "https://identity:35357/v3/services/ee057c"
                },
                "name": "Cinder",
                "type": "volume"
            },
            {
                "description": "OpenStack Identity Service",
                "id": "5e70df",
                "links": {
                    "self": "https://identity:35357/v3/services/5e70df"
                },
                "name": "Keystone",
                "type": "identity"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/services",
            "previous": null,
            "next": null
        }
    }

Get service
^^^^^^^^^^^

::

    GET /services/{service_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/service``

Response:

::

    Status: 200 OK

    {
        "service": {
            "description": "OpenStack Volume Service",
            "id": "ee057c",
            "links": {
                "self": "https://identity:35357/v3/services/ee057c"
            },
            "name": "Cinder",
            "type": "volume"
        }
    }

Create service
^^^^^^^^^^^^^^

::

    POST /services

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/services``

Request:

::

    {
        "service": {
            "description": "OpenStack Compute Service",
            "name": "Nova",
            "type": "compute"
        }
    }

Response:

::

    Status: 201 Created

    {
        "service": {
            "description": "OpenStack Compute Service",
            "id": "520ec2",
            "links": {
                "self": "https://identity:35357/v3/services/520ec2"
            },
            "name": "Nova",
            "type": "compute"
        }
    }

Update service
^^^^^^^^^^^^^^

::

    PATCH /services/{service_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/service``

The request block is the same as the one for create service, except that only
the attributes that are being updated need to be included.

Response:

::

    Status: 200 OK

    {
        "service": {
            "description": "OpenStack Image Service",
            "id": "520ec2",
            "links": {
                "self": "https://identity:35357/v3/services/520ec2"
            },
            "name": "Glance",
            "type": "image"
        }
    }

Delete service
^^^^^^^^^^^^^^

::

    DELETE /services/{service_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/service``

- Note: deleting a service when endpoints exist should either 1) delete all
  associated endpoints or 2) fail until endpoints are deleted

Response:

::

    Status: 204 No Content

Endpoints
~~~~~~~~~

List endpoints
^^^^^^^^^^^^^^

::

    GET /endpoints

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/endpoints``

Optional query parameters:

- ``interface`` (string)

- ``service_id`` (string)

- ``region_id`` (string) *New in version 3.5*

Response:

::

    Status: 200 OK

    {
        "endpoints": [
            {
                "enabled": true,
                "id": "6fedc0",
                "interface": "public",
                "links": {
                    "self": "https://identity:35357/v3/endpoints/6fedc0"
                },
                "region_id": "us-east-1",
                "service_id": "ee057c",
                "url": "https://service.example.com:5000/"
            },
            {
                "enabled": true,
                "id": "d12b15",
                "interface": "admin",
                "links": {
                    "self": "https://identity:35357/v3/endpoints/d12b15"
                },
                "region_id": "us-east-2",
                "service_id": "8ef7de",
                "url": "https://service.example.com:35357/"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/endpoints",
            "previous": null,
            "next": null
        }
    }

Get endpoint
^^^^^^^^^^^^

::

    GET /endpoints/{endpoint_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/endpoint``

Response:

::

    Status: 200 OK

    {
        "endpoint": {
            "enabled": true,
            "id": "6fedc0",
            "interface": "public",
            "links": {
                "self": "https://identity:35357/v3/endpoints/6fedc0"
            },
            "region_id": "us-east-2",
            "service_id": "ee057c",
            "url": "https://service.example.com:5000/"
        }
    }

Create endpoint
^^^^^^^^^^^^^^^

::

    POST /endpoints

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/endpoints``

Request:

::

    {
        "endpoint": {
            "interface": "admin",
            "region_id": "us-east-2",
            "url": "https://service.example.com/",
            "service_id": "ee057c"
        }
    }

Response:

::

    Status: 200 OK

    {
        "endpoint": {
            "enabled": true,
            "id": "6fedc0",
            "interface": "admin",
            "links": {
                "self": "https://identity:35357/v3/endpoints/6fedc0"
            },
            "region_id": "us-east-2",
            "service_id": "ee057c",
            "url": "https://service.example.com:35357/"
        }
    }

Update endpoint
^^^^^^^^^^^^^^^

::

    PATCH /endpoints/{endpoint_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/endpoint``

The request block is the same as the one for create endpoint, except that only
the attributes that are being updated need to be included.

Response:

::

    Status: 200 OK

    {
        "endpoint": {
            "enabled": true,
            "id": "6fedc0",
            "interface": "public",
            "links": {
                "self": "https://identity:35357/v3/endpoints/6fedc0"
            },
            "region_id": "us-east-1",
            "service_id": "ee057c",
            "url": "https://service.example.com:5000/"
        }
    }

Delete endpoint
^^^^^^^^^^^^^^^

::

    DELETE /endpoints/{endpoint_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/endpoint``

Response:

::

    Status: 204 No Content

Domains
~~~~~~~

List domains
^^^^^^^^^^^^

::

    GET /domains

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domains``

Optional query parameters:

- ``enabled`` (key-only, no value expected)

- ``name`` (string)

Response:

::

    Status: 200 OK

    {
        "domains": [
            {
                "description": "my domain description",
                "enabled": true,
                "id": "1789d1",
                "links": {
                    "self": "https://identity:35357/v3/domains/1789d1"
                },
                "name": "my domain"
            },
            {
                "description": "description of my other domain",
                "enabled": true,
                "id": "43e8da",
                "links": {
                    "self": "https://identity:35357/v3/domains/43e8da"
                },
                "name": "another domain"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/domains",
            "previous": null,
            "next": null
        }
    }

Get domain
^^^^^^^^^^

::

    GET /domains/{domain_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain``

Response:

::

    Status: 200 OK

    {
        "domain": {
            "description": "my domain description",
            "enabled": true,
            "id": "1789d1",
            "links": {
                "self": "https://identity:35357/v3/domains/1789d1"
            },
            "name": "my domain"
        }
    }

Create domain
^^^^^^^^^^^^^

::

    POST /domains

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domains``

Request:

::

    {
        "domain": {
            "description": "my new domain for users",
            "enabled": true,
            "name": "my new domain"
        }
    }

Response:

::

    Status: 201 Created

    {
        "domain": {
            "description": "my new domain for users",
            "enabled": true,
            "id": "89b3e2",
            "links": {
                "self": "https://identity:35357/v3/domains/89b3e2"
            },
            "name": "my new domain"
        }
    }

Update domain
^^^^^^^^^^^^^

::

    PATCH /domains/{domain_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain``

The request block is the same as the one for create domain, except that only
the attributes that are being updated need to be included.

Request:

::

    {
        "domain": {
            "description": "my new domain for users and tenants"
        }
    }

Response:

::

    Status: 200 OK

    {
        "domain": {
            "description": "my new domain for users and tenants",
            "enabled": true,
            "id": "89b3e2",
            "links": {
                "self": "https://identity:35357/v3/domains/89b3e2"
            },
            "name": "my new domain"
        }
    }

Delete domain
^^^^^^^^^^^^^

::

    DELETE /domains/{domain_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain``

Deleting a domain will delete all the entities owned by it (Users, Groups, and
Projects), as well as any credentials and role grants that relate to these
entities.

In order to minimize the risk of an inadvertent deletion of a domain and its
entities, a domain must first be disabled (using the update domain API) before
a successful delete domain API call can be made. Attempting to delete an
enabled domain will result in an HTTP ``403 Forbidden`` response.

Response:

::

    Status: 204 No Content

*New in version 3.6*

- The deletion of a non-leaf domain in a domain hierarchy tree is prohibited
  and will fail with an HTTP ``400 Bad Request``


Domain configuration management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*New in version 3.4 (experimental)*

Keystone optionally supports the ability to manage domain specific
configuration options via the API, allowing configuration options to be
overriden for a given domain. In addition, *New in version 3.6 (experimental)*,
the default configuration options can also be retrieved.

Domain specific configuration options are structured within their group
objects. Currently only the ``identity`` and ``ldap`` groups are supported, and
these can be used to override the default configuration settings for the
storage of users and groups by the identity server. Attempting to read or
override configuration options for groups other than ``identity`` and ``ldap``
will result in an HTTP ``403 Forbidden``.

The default configuration settings for the options that can be overridden
can be retrieved.

::

    GET /domains/config/default

Relationship::
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config_default``

Response:

::

    Status: 200 OK

    {
        "config": {
            "identity": {
                "driver": "ldap"
            },
            "ldap": {
                "url": "ldap://localhost",
                "user": "",
                "suffix": "cn=example,cn=com".
                ....
            }
        }
    }

It is possible to read the default configuration settings for a specific group
or option.

::

    GET /domains/config/ldap/default

Relationship::
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config_default``

Response:

::

    Status: 200 OK

    {
        "ldap": {
            "url": "ldap://localhost",
            "user": "",
            "suffix": "cn=example,cn=com".
            ....
        }
    }

::

    GET /domains/config/identity/driver/default

Relationship::
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config_default``

Response:

::

    Status: 200 OK

    {
        "driver": "ldap"
    }

A similar form of URL can be used to retrieve the values of those options that
have been overriden for a domain by the API.

::

    GET /domains/{domain_id}/config

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config``

Response:

::

    Status: 200 OK

    {
        "config": {
            "identity": {
                "driver": "keystone.identity.backends.ldap.Identity"
            },
            "ldap": {
                "url": "http://myldap/root",
                "user_tree_dn": "ou=Users,dc=root,dc=org"
            }
        }
    }

The values of a specific group that has been overridden can also be read.

::

    GET /domains/{domain_id}/config/ldap

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config_group``

Response:

::

    Status: 200 OK

    {
        "ldap": {
            "url": "http://myldap/root",
            "user_tree_dn": "ou=Users,dc=root,dc=org"
        }
    }

An individual option may also be requested.

::

    GET /domains/{domain_id}/config/ldap/url

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config_option``

Response:

::

    Status: 200 OK

    {
        "url": "http://myldap/root",
    }

Domain specific configuration options can also be created, updated and deleted
using the PUT, PATCH and DELETE HTTP commands. When updating, it is only
necessary to include those options that are being updated.

::

    PATCH /domains/{domain_id}/config


Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_config``

Request:

::

    {
        "config": {
            "ldap": {
                "url": "http://myldap/my_new_root",
                "user_tree_dn": "ou=Users,dc=my_new_root,dc=org"
            }
        }
    }

Response:

::

    Status: 200 OK

    {
        "config": {
            "identity": {
                "driver": "keystone.identity.backends.ldap.Identity"
            },
            "ldap": {
                "url": "http://myldap/my_new_root",
                "user_tree_dn": "ou=Users,dc=my_new_root,dc=org"
            }
        }
    }

In a similar case to GET, an indiviudal option can be updated.

::

    PATCH /domains/{domain_id}/config/ldap/url

Request:

::

    {
        "url": "http://myldap/my_other_root",
    }

Response:

::

    Status: 200 OK

    {
        "config": {
            "identity": {
                "driver": "keystone.identity.backends.ldap.Identity"
            },
            "ldap": {
                "url": "http://myldap/my_other_root",
                "user_tree_dn": "ou=Users,dc=my_new_root,dc=org"
            }
        }
    }

In the above example, if the ``url`` option did not yet exist then an HTTP PUT
command would be required.

The Keystone API will not return options that are considered sensitive,
although these can be written/updated. The only option currently considered
sensitive is the ``password`` option within the ``ldap`` group. To aid those
situations where sensitive options are required to be included in other
options that otherwise would not considered sensitive, the API supports a
substitution ability for any sensitive options. For example, the
password can be included as part of the ``url`` option.

::

    PATCH /domains/{domain_id}/config/ldap/url

Request:

::

    {
        "url": "http://myldap/my_other_root/my_user/%(password)s",
    }

Response:

::

    Status: 200 OK

    {
        "config": {
            "identity": {
                "driver": "keystone.identity.backends.ldap.Identity"
            },
            "ldap": {
                "url": "http://myldap/my_other_root/my_user/$(password)s",
                "user_tree_dn": "ou=Users,dc=my_new_root,dc=org"
            }
        }
    }

In this example, Keystone will substitute the referenced ``password`` option
with its actual value when using the ``url`` to talk to the LDAP server. A
sensitive option that is referenced in this way must be in the same option
group as the referring option.

Projects
~~~~~~~~

List projects
^^^^^^^^^^^^^

::

    GET /projects

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/projects``

Optional query parameters:

- ``domain_id`` (string)

- ``enabled`` (key-only, no value expected)

- ``is_domain`` (boolean, defaults to false) *New in version 3.6*

  If this is not specified, then only projects that are not acting as a
  domain will be included in the collection. If it is specified as true, then
  only projects acting as a domain are included.

- ``name`` (string)

- ``parent_id`` (string) *New in version 3.4*

Response:

::

    Status: 200 OK

    {
        "projects": [
            {
                "domain_id": "1789d1",
                "enabled": true,
                "id": "263fd9",
                "links": {
                    "self": "https://identity:35357/v3/projects/263fd9"
                },
                "name": "Dev Group A",
                "parent_id": null
            },
            {
                "domain_id": "1789d1",
                "enabled": true,
                "id": "e56ad3",
                "links": {
                    "self": "https://identity:35357/v3/projects/e56ad3"
                },
                "name": "Dev Group B",
                "parent_id": null
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/projects",
            "previous": null,
            "next": null
        }
    }

Get project
^^^^^^^^^^^

::

    GET /projects/{project_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project``

*New in version 3.4*

- ``parents_as_list`` (key-only, no value expected)

- ``subtree_as_list`` (key-only, no value expected)

- ``parents_as_ids`` (key-only, no value expected)

- ``subtree_as_ids`` (key-only, no value expected)

Response:

::

    Status: 200 OK

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "https://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2"
        }
    }

If additional information about the project's hierarchy is required, this API
has two query parameters.

::

    GET /projects/{project_id}?parents_as_list

The parent hierarchy will be included as a list in the response. This list will
contain the projects found by traversing up the hierarchy to the top-level
project.

.. Note:: Server responses may vary depending on the level of authorization the
   user has against the projects in the hierarchy. The server may return an
   empty list if the user does not have any role assignments on the projects in
   the hierarchy, or only return projects that the user has role assignments
   on.

Response:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2",
            "parents": [
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "183ab2",
                        "links": {
                            "self": "identity:35357/v3/projects/183ab2"
                        },
                        "name": "Dev Group A Parent",
                        "parent_id": null
                    }
                }
            ]
        }
    }

::

    GET /projects/{project_id}?subtree_as_list

The child hierarchy will be included as a list in the response. This list will
contain the projects found by traversing down the hierarchy.

.. Note:: Server responses may vary depending on the level of authorization the
   user has against the projects in the hierarchy. The server may return an
   empty list if the user does not have any role assignments on the projects in
   the hierarchy, or only return projects that the user has role assignments
   on.

Response:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2",
            "subtree": [
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "9n1jhb",
                        "links": {
                            "self": "identity:35357/v3/projects/9n1jhb"
                        },
                        "name": "Dev Group A Child 1",
                        "parent_id": "263fd9"
                    }
                },
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "4b6aa1",
                        "links": {
                            "self": "identity:35357/v3/projects/4b6aa1"
                        },
                        "name": "Dev Group A Child 2",
                        "parent_id": "263fd9"
                    }
                },
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "b76eq8",
                        "links": {
                            "self": "identity:35357/v3/projects/b76xq8"
                        },
                        "name": "Dev Group A Grandchild",
                        "parent_id": "4b6aa1"
                    }
                }
            ]
        }
    }

::

    GET /projects/{project_id}?parents_as_ids

The entire parent hierarchy will be included as nested dictionaries in the
response. It will contain all projects ids found by traversing up the hierarchy
to the top-level project.

.. Note:: The server may return ids of all projects in the hierarchy without
   requiring the user to have role assignments on any projects in the
   hierarchy.

Response:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2",
            "parents": {
                "183ab2": {
                    "f53a4e": null
                }
            }
        }
    }


::

    GET /projects/{project_id}?subtree_as_ids

The entire child hierarchy will be included as nested dictionaries in the
response. It will contain all the projects ids found by traversing down the
hierarchy.

.. Note:: The server may return ids of all projects in the hierarchy without
   requiring the user to have role assignments on any projects in the
   hierarchy.

Response:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2",
            "subtree": {
                "9n1jhb": null,
                "4b6aa1": {
                    "b76eq8": null
                }
            }
        }
    }

Note that the subtree and parents query parameters are not mutually
exclusive. The same is not true for similar query params such as
``parents_as_list`` and ``parents_as_ids``, which can't be included at the same
time. If included, the server will fail with an HTTP ``400 Bad Request`` error.

::

    GET /projects/{project_id}?parents_as_list&subtree_as_list

Both the parents and subtree lists will be included in the response.

Response:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2",
            "parents": [
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "183ab2",
                        "links": {
                            "self": "identity:35357/v3/projects/183ab2"
                        },
                        "name": "Dev Group A Parent",
                        "parent_id": null
                    }
                }
            ],
            "subtree": [
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "9n1jhb",
                        "links": {
                            "self": "identity:35357/v3/projects/9n1jhb"
                        },
                        "name": "Dev Group A Child 1",
                        "parent_id": "263fd9"
                    }
                },
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "4b6aa1",
                        "links": {
                            "self": "identity:35357/v3/projects/4b6aa1"
                        },
                        "name": "Dev Group A Child 2",
                        "parent_id": "263fd9"
                    }
                },
                {
                    "project": {
                        "domain_id": "1789d1",
                        "enabled": true,
                        "id": "b76xq8",
                        "links": {
                            "self": "identity:35357/v3/projects/b76xq8"
                        },
                        "name": "Dev Group A Grandchild",
                        "parent_id": "4b6aa1"
                    }
                }
            ]
        }
    }

::

    GET /projects/{project_id}?parents_as_ids&subtree_as_ids

Both the parents and subtree hierarchies will be included in the response.

Response:

::

    {
        "project": {
            "domain_id": "1789d1",
            "enabled": true,
            "id": "263fd9",
            "links": {
                "self": "http://identity:35357/v3/projects/263fd9"
            },
            "name": "Dev Group A",
            "parent_id": "183ab2",
            "parents": {
                "183ab2": {
                    "f53a4e": null
                }
            },
            "subtree": {
                "9n1jhb": null,
                "4b6aa1": {
                    "b76eq8": null
                }
            }
        }
    }

Create project
^^^^^^^^^^^^^^

::

    POST /projects

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/projects``

Request:

::

    {
        "project": {
            "description": "Project space for Test Group",
            "domain_id": "1789d1",
            "enabled": true,
            "name": "Test Group",
            "parent_id": "7fa612"
        }
    }

Response:

::

    Status: 201 Created

    {
        "project": {
            "description": "Project space for Test Group",
            "domain_id": "1789d1",
            "enabled": true,
            "id": "d52e32",
            "links": {
                "self": "https://identity:35357/v3/projects/d52e32"
            },
            "name": "Test Group",
            "parent_id": "7fa612"
        }
    }

*New in version 3.4*

- Adding a project with a parent_id pointing to a project that does not exist
  fails with an HTTP ``404 Not Found``

Update project
^^^^^^^^^^^^^^

::

    PATCH /projects/{project_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project``

The request block is the same as the one for create project, except that only
the attributes that are being updated need to be included.

Request:

::

    {
        "project": {
            "description": "Project space for Build Group",
            "name": "Build Group"
        }
    }

Response:

::

    Status: 200 OK

    {
        "project": {
            "description": "Project space for Build Group",
            "domain_id": "1789d1",
            "enabled": true,
            "id": "d52e32",
            "links": {
                "self": "https://identity:35357/v3/projects/d52e32"
            },
            "name": "Build Group",
            "parent_id": "7fa612"
        }
    }

*New in version 3.4*

- The update of the parent_id is not allowed and will fail with an HTTP
  ``403 Forbidden``

- Disabling a project that has enabled projects in its subtree using this API
  will fail with an HTTP ``403 Forbidden``. See the
  ``Enable or disable subtree`` section for the appropriate API for this
  action.

- Enabling a project that has disabled parents will fail with an HTTP
  ``403 Forbidden``

Delete project
^^^^^^^^^^^^^^

::

    DELETE /projects/{project_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project``

::

    Status: 204 No Content

*New in version 3.4*

- The deletion of a project that is not a leaf in the project hierarchy (does
  not have children) using this API will fail with an HTTP ``403 Forbidden``.

Users
~~~~~

List users
^^^^^^^^^^

::

    GET /users

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/users``

Optional query parameters:

- ``domain_id`` (string)

- ``enabled`` (key-only, no value expected)

- ``name`` (string)

Response:

::

    Status: 200 OK

    {
        "users": [
            {
                "default_project_id": "263fd9",
                "description": "Admin user",
                "domain_id": "1789d1",
                "enabled": true,
                "id": "0ca8f6",
                "links": {
                    "self": "https://identity:35357/v3/users/0ca8f6"
                },
                "name": "admin",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            },
            {
                "default_project_id": "263fd9",
                "description": "John Smith's user",
                "domain_id": "1789d1",
                "enabled": true,
                "id": "9fe1d3",
                "links": {
                    "self": "https://identity:35357/v3/users/9fe1d3"
                },
                "name": "jsmith",
                "password_expires_at": null
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/users",
            "previous": null,
            "next": null
        }
    }

Get user
^^^^^^^^

::

    GET /users/{user_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/user``

Response:

::

    Status: 200 OK

    {
        "user": {
            "default_project_id": "263fd9",
            "description": "John Smith's user",
            "domain_id": "1789d1",
            "enabled": true,
            "id": "9fe1d3",
            "links": {
                "self": "https://identity:35357/v3/users/9fe1d3"
            },
            "name": "jsmith",
            "password_expires_at": null
        }
    }

List user projects
^^^^^^^^^^^^^^^^^^

::

    GET /users/{user_id}/projects

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/user_projects``

Optional query parameters:

- ``domain_id`` (string) *New in version 3.6*

- ``enabled`` (key-only, no value expected)

- ``name`` (string)

Response:

::

    Status: 200 OK

    {
        "projects": [
            {
                "domain_id": "1789d1",
                "enabled": true,
                "id": "263fd9",
                "links": {
                    "self": "https://identity:35357/v3/projects/263fd9"
                },
                "name": "Test Group"
            },
            {
                "domain_id": "1789d1",
                "enabled": true,
                "id": "50ef01",
                "links": {
                    "self": "https://identity:35357/v3/projects/50ef01"
                },
                "name": "Build Group"
            }
        ],
        "links": {
            "self": "https://identity:35357/v3/users/9fe1d3/projects",
            "previous": null,
            "next": null
        }
    }

List groups of which a user is a member
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /users/{user_id}/groups

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/user_groups``

Optional query parameters:

- ``name`` (string)

Response:

::

    Status: 200 OK

    {
        "groups": [
            {
                "description": "Developers cleared for work on all general projects"
                "domain_id": "1789d1",
                "id": "ea167b",
                "links": {
                    "self": "https://identity:35357/v3/groups/ea167b"
                },
                "name": "Developers"
            },
            {
                "description": "Developers cleared for work on secret projects"
                "domain_id": "1789d1",
                "id": "a62db1",
                "links": {
                    "self": "https://identity:35357/v3/groups/a62db1"
                },
                "name": "Secure Developers"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/users/9fe1d3/groups",
            "previous": null,
            "next": null
        }
    }

Create user
^^^^^^^^^^^

::

    POST /users

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/users``

Request:

::

    {
        "user": {
            "default_project_id": "263fd9",
            "description": "Jim Doe's user",
            "domain_id": "1789d1",
            "enabled": true,
            "name": "James Doe",
            "password": "secretsecret"
        }
    }

Response:

::

    Status: 201 Created

    {
        "user": {
            "default_project_id": "263fd9",
            "description": "Jim Doe's user",
            "domain_id": "1789d1",
            "enabled": true,
            "id": "ff4e51",
            "links": {
                "self": "https://identity:35357/v3/users/ff4e51"
            },
            "name": "jdoe",
            "password_expires_at": "2016-11-06T15:32:17.000000"
        }
    }

Update user
^^^^^^^^^^^

::

    PATCH /users/{user_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/user``

The request block is the same as the one for create user, except that only the
attributes that are being updated need to be included. Use this method attempt
to update user password or enable/disable the user. This may return a HTTP
``501 Not Implemented`` if the back-end driver does not allow for the
functionality.

Response:

::

    Status: 200 OK

    {
        "user": {
            "default_project_id": "263fd9",
            "description": "James Doe's user",
            "domain_id": "1789d1",
            "enabled": true,
            "id": "ff4e51",
            "links": {
                "self": "https://identity:35357/v3/users/ff4e51"
            },
            "name": "jamesdoe",
            "password_expires_at": "2016-11-06T15:32:17.000000"
        }
    }

Delete user
^^^^^^^^^^^

::

    DELETE /users/{user_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/user``

Response:

::

    Status: 204 No Content

Change user password
^^^^^^^^^^^^^^^^^^^^

::

    POST /users/{user_id}/password

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/user_change_password``

Request:

::

    {
        "user": {
            "password": "old_secretsecret",
            "original_password": "secretsecret"
        }
    }

Response:

::

    Status: 204 No Content

Groups
~~~~~~

Create group
^^^^^^^^^^^^

::

    POST /groups

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/groups``

Request:

::

    {
        "group": {
            "description": "--optional--",
            "domain_id": "--optional--",
            "name": "..."
        }
    }

Response:

::

    Status: 201 Created

    {
        "group": {
            "description": "Developers cleared for work on secret projects",
            "id": "--group-id--",
            "links": {
                "self": "http://identity:35357/v3/groups/--group-id--"
            },
            "name": "Secure Developers"
        }
    }

List groups
^^^^^^^^^^^

::

    GET /groups

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/groups``

Optional query parameters:

- ``domain_id`` (string)

- ``name`` (string)

Response:

::

    Status: 200 OK

    {
        "groups": [
            {
                "description": "Developers cleared for work on all general projects"
                "domain_id": "--domain-id--",
                "id": "--group-id--",
                "links": {
                    "self": "http://identity:35357/v3/groups/--group-id--"
                },
                "name": "Developers"
            },
            {
                "description": "Developers cleared for work on secret projects"
                "domain_id": "--domain-id--",
                "id": "--group-id--",
                "links": {
                    "self": "http://identity:35357/v3/groups/--group-id--"
                },
                "name": "Secure Developers"
            },
            {
                "description": "Testers cleared for work on all general projects"
                "domain_id": "--domain-id--",
                "id": "--group-id--",
                "links": {
                    "self": "http://identity:35357/v3/groups/--group-id--"
                },
                "name": "Testers"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/groups",
            "previous": null,
            "next": null
        }
    }

Get group
^^^^^^^^^

::

    GET /groups/{group_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group``

Response:

::

    Status: 200 OK

    {
        "group": {
            "description": "Developers cleared for work on secret projects",
            "id": "--group-id--",
            "links": {
                "self": "http://identity:35357/v3/groups/--group-id--"
            },
            "name": "Secure Developers"
        }
    }

List users who are members of a group
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /groups/{group_id}/users

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group_users``

Optional query parameters:

- ``enabled`` (key-only, no value expected)

- ``name`` (string)

Response:

::

    Status: 200 OK

    {
        "users": [
            {
                "default_project_id": "--default-project-id--",
                "description": "a user",
                "domain_id": "--domain-id--",
                "enabled": true,
                "id": "--user-id--",
                "links": {
                    "self": "http://identity:35357/v3/users/--user-id--"
                },
                "name": "admin",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            },
            {
                "default_project_id": "--default-project-id--",
                "description": "another user",
                "domain_id": "--domain-id--",
                "enabled": true,
                "id": "--user-id--",
                "links": {
                    "self": "http://identity:35357/v3/users/--user-id--"
                },
                "name": "someone",
                "password_expires_at": "2016-11-06T15:32:17.000000"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/groups/--group-id--/users",
            "previous": null,
            "next": null
        }
    }

Update group
^^^^^^^^^^^^

::

    PATCH /groups/{group_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group``

The request block is the same as the one for create group, except that only the
attributes that are being updated need to be included. This may return an HTTP
``501 Not Implemented`` if the back-end driver doesn't allow for the
functionality.

Response:

::

    Status: 200 OK

    {
        "group": {
            "description": "Developers cleared for work on secret projects",
            "id": "--group-id--",
            "links": {
                "self": "http://identity:35357/v3/groups/--group-id--"
            },
            "name": "Secure Developers"
        }
    }

Delete group
^^^^^^^^^^^^

::

    DELETE /groups/{group_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group``

Response:

::

    Status: 204 No Content

Add user to group
^^^^^^^^^^^^^^^^^

::

    PUT /groups/{group_id}/users/{user_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group_user``

Response:

::

    Status: 204 No Content

Remove user from group
^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /groups/{group_id}/users/{user_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group_user``

Response:

::

    Status: 204 No Content

Check if user is member of group
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /groups/{group_id}/users/{user_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/group_user``

Response:

::

    Status: 204 No Content

Credentials
~~~~~~~~~~~

The key use cases we need to cover:

- CRUD on a credential

Create credential
^^^^^^^^^^^^^^^^^

::

    POST /credentials

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/credentials``

This example shows creating an EC2 style credential where the credentials are a
combination of access\_key and secret. Other credentials (such as access\_key)
may be supported by simply changing the content of the key data.

Request:

::

    {
        "credential": {
            "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
            "project_id": "0211d7",
            "type": "ec2",
            "user_id": "f293ba"
        }
    }

Response:

::

    Status: 201 Created

    {
        "credential": {
            "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
            "id": "46322a",
            "links": {
                "self": "https://identity:35357/v3/credentials/46322a"
            },
            "project_id": "0211d7",
            "type": "ec2",
            "user_id": "f293ba"
        }
    }

List credentials
^^^^^^^^^^^^^^^^

::

    GET /credentials

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/credentials``

Optional query parameters:

- ``user_id`` (string)
- ``type`` (string) *New in version 3.5*

Response:

::

    Status: 200 OK

    {
        "credentials": [
            {
                "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
                "id": "10b182",
                "links": {
                    "self": "https://identity:35357/v3/credentials/10b182"
                },
                "project_id": "82cc2f",
                "type": "ec2",
                "user_id": "27a19b"
            },
            {
                "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
                "id": "85d995",
                "links": {
                    "self": "https://identity:35357/v3/credentials/85d995"
                },
                "project_id": "82cc2f",
                "type": "ec2",
                "user_id": "88770a"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/credentials",
            "previous": null,
            "next": null
        }
    }

Get credential
^^^^^^^^^^^^^^

::

    GET /credentials/{credential_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/credential``

Response:

::

    Status: 200 OK

    {
        "credential": {
            "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
            "id": "85d995",
            "links": {
                "self": "https://identity:35357/v3/credentials/85d995"
            },
            "project_id": "82cc2f",
            "type": "ec2",
            "user_id": "88770a"
        }
    }

Update credential
^^^^^^^^^^^^^^^^^

::

    PATCH /credentials/{credential_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/credential``

The request block is the same as the one for create credential, except
that only the attributes that are being updated need to be included.

Response:

::

    Status: 200 OK

    {
        "credential": {
            "blob": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY",
            "id": "85d995",
            "links": {
                "self": "https://identity:35357/v3/credentials/85d995"
            },
            "project_id": "6f20ed",
            "type": "ec2",
            "user_id": "2a64f5"
        }
    }

Delete credential
^^^^^^^^^^^^^^^^^

::

    DELETE /credentials/{credential_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/credential``

Response:

::

    Status: 204 No Content

Roles
~~~~~

The key use cases we need to cover:

- CRUD on a role

- CRUD for role inference rules

- Associating a role with a project or domain

Create role
^^^^^^^^^^^

::

    POST /roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/roles``

Request:

::

    {
        "role": {
            "name": "..."
        }
    }

Response:

::

    Status: 201 Created

    {
        "role": {
            "domain_id": null,
            "id": "--role-id--",
            "links": {
                "self": "http://identity:35357/v3/roles/--role-id--"
            },
            "name": "a role name"
        }
    }

To create a domain specific role, the request block and response would look
like:

::

    {
        "role": {
            "domain_id": "--domain-id--",
            "name": "my specific domain role"
        }
    }

Response:

::

    Status: 201 Created

    {
        "role": {
            "domain_id": "--domain-id--",
            "id": "--role-id--",
            "links": {
                "self": "http://identity:35357/v3/roles/--role-id--"
            },
            "name": "my specific domain role"
        }
    }

List roles
^^^^^^^^^^

::

    GET /roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/roles``

Optional query parameters:

- ``name`` (string)

- ``domain_id`` (string, defaults to null) *New in version 3.6*

  Since this defaults to null, listing roles without specifying ``domain_id``
  in the query string will result in a collection containing only global roles.

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "domain_id": null,
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "a role name"
            },
            {
                "domain_id": null,
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "a role name"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/roles",
            "previous": null,
            "next": null
        }
    }

Get role
^^^^^^^^

::

    GET /roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/role``

Response:

::

    Status: 200 OK

    {
        "role": {
            "domain_id": "--domain-id--",
            "id": "--role-id--",
            "links": {
                "self": "http://identity:35357/v3/roles/--role-id--"
            },
            "name": "a domain specific role"
        }
    }

Update role
^^^^^^^^^^^

::

    PATCH /roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/role``

The request block is the same as the one for create role, except that only the
attributes that are being updated need to be included.

Response:

::

    Status: 200 OK

    {
        "role": {
            "domain_id": null,
            "id": "--role-id--",
            "links": {
                "self": "http://identity:35357/v3/roles/--role-id--"
            },
            "name": "a global policy role name"
        }
    }

Delete role
^^^^^^^^^^^

Deleting a role also deletes all role inference rules where that role is
either the prior or implied role.

::

    DELETE /roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/role``

Response:

::

    Status: 204 No Content


Create role inference rule
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /roles/{prior_role_id}/implies/{implied_role_id}

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#createRoleInference``


Response:

::

    Status: 201 Created

    {
        "role_inference": {
            "prior_role": {
                "id": "--prior-role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--prior-role-id--"
                },
                "name": "prior role name"
            },
            "implies": {
                "id": "--implied-role1-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--implied-role1-id--"
                },
                "name": "implied role1 name"
            }
        },
        "links": {
            "self": "http://identity:35357/v3/roles/--prior-role-id--/implies/--implied-role-id--"
        }
    }


Get role inference rule
^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /roles/{prior_role_id}/implies/{implied_role_id}

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#getRoleInference``

Response:

::

    Status: 200 OK

    {
        "role_inference": {
            "prior_role": {
                "id": "--prior-role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--prior-role-id--"
                },
                "name": "prior role name"
            },
            "implies": {
                "id": "--implied-role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--implied-role-id--"
                },
                "name": "implied role name"
            }
        },
        "links": {
            "self": "http://identity:35357/v3/roles/--prior-role-id--/implies/--implied-role-id--"
        }
    }


Confirm a role inference rule
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /roles/{prior_role_id}/implies/{implied_role_id}

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#getRoleInference``

Response:

::

    Status: 204 No Content

Delete role inference rule
^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /roles/{prior_role_id}/implies/{implied_role_id}

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#deleteRoleInference``

Response:

::

    Status: 204 No Content


List implied roles for role
^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /roles/{prior_role_id}/implies

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#getRoleInference``


Response:

::

    Status: 200 OK
    {
        "role_inference": {
            "prior_role": {
                "id": "--prior-role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--prior-role-id--"
                },
                "name": "prior role name"
            },
            "implies": [
                {
                    "id": "--implied-role1-id--",
                    "links": {
                        "self": "http://identity:35357/v3/roles/--implied-role1-id--"
                    },
                    "name": "implied role1 name"
                },
                {
                    "id": "--implied-role2-id--",
                    "links": {
                        "self": "http://identity:35357/v3/roles/--implied-role2-id--"
                    },
                    "name": "implied role2 name"
                }
            ]
        },
        "links" : {
            "self": "http://identity:35357/v3/roles/--prior-role-id--/implies"
        }
    }



List all role inference rules
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /role_inferences/

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#getRoleInference``


Response:

::

    Status: 200 OK
    {
        "role_inferences": [
            {
                "prior_role": {
                    "id": "--prior-role-id1--",
                    "links": {
                        "self": "http://identity:35357/v3/roles/--prior-role-id--"
                    }
                    "name": "prior role name"
                },
                "implies": [
                    {
                        "id": "--implied-role1-id--",
                        "links": {
                            "self": "http://identity:35357/v3/roles/--implied-role1-id--"
                        },
                        "name": "implied role1 name"
                    },
                    {
                        "id": "--implied-role2-id--",
                        "links": {
                            "self": "http://identity:35357/v3/roles/--implied-role2-id--"
                        },
                        "name": "implied role2 name"
                    }
                ]
            },
            {
                "prior_role": {
                    "id": "--prior-role-id2--",
                    "links": {
                        "self" : "http://identity:35357/v3/roles/--prior-role-id--"
                    }
                    "name": "prior role name"
                },
                "implies": [
                    {
                        "id": "--implied-role3-id--",
                        "links": {
                            "self": "http://identity:35357/v3/roles/--implied-role1-id--"
                        },
                        "name": "implied role1 name"
                    },
                    {
                        "id": "--implied-role4-id--",
                        "links": {
                            "self": "http://identity:35357/v3/roles/--implied-role2-id--"
                        },
                        "name": "implied role2 name"
                    }
                ]
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/role_inferences"
        }
    }


Grant role to user on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /domains/{domain_id}/users/{user_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_user_role``

Response:

::

    Status: 204 No Content

Grant role to group on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /domains/{domain_id}/groups/{group_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_group_role``

Response:

::

    Status: 204 No Content

Grant role to user on projects owned by a domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /OS-INHERIT/domains/{domain_id}/users/{user_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#assignRoleToUser-domain``

The inherited role is only applied to the owned projects (both existing and
future projects), and will not appear as a role in a domain scoped token.

Response:

::

    Status: 204 No Content

Grant role to group on projects owned by a domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /OS-INHERIT/domains/{domain_id}/groups/{group_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#assignRoleToGroup-domain``

The inherited role is only applied to the owned projects (both existing and
future projects), and will not appear as a role in a domain scoped token.

Response:

::

    Status: 204 No Content

List user's roles on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /domains/{domain_id}/users/{user_id}/roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_user_roles``

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/domains/--domain_id--/users/--user_id--/roles",
            "previous": null,
            "next": null
        }
    }

List group's roles on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /domains/{domain_id}/groups/{group_id}/roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_group_roles``

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/domains/--domain_id--/groups/--group_id--/roles",
            "previous": null,
            "next": null
        }
    }

List user's inherited project roles on a domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /OS-INHERIT/domains/{domain_id}/users/{user_id}/roles/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#listRolesForUser-domain``

The list only contains those role assignments to the domain that were specified
as being inherited to projects within that domain.

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/OS-INHERIT/domains/--domain_id--/
                     users/--user_id--/roles/inherited_to_projects",
            "previous": null,
            "next": null
        }
    }

List group's inherited project roles on domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /OS-INHERIT/domains/{domain_id}/groups/{group_id}/roles/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#listRolesForGroup-domain``

The list only contains those role assignments to the domain that were specified
as being inherited to projects within that domain.

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/OS-INHERIT/domains/--domain_id--/
                     groups/--group_id--/roles/inherited_to_projects",
            "previous": null,
            "next": null
        }
    }

Check if user has role on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /domains/{domain_id}/users/{user_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_user_role``

Response:

::

    Status: 204 No Content

Check if group has role on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /domains/{domain_id}/groups/{group_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_group_role``

Response:

::

    Status: 204 No Content

Check if user has an inherited project role on domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /OS-INHERIT/domains/{domain_id}/users/{user_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#checkRoleForGroup-domain``

Response:

::

    Status: 204 No Content

Check if group has an inherited project role on domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /OS-INHERIT/domains/{domain_id}/groups/{group_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#checkRoleForGroup-domain``

Response:

::

    Status: 204 No Content

Revoke role from user on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /domains/{domain_id}/users/{user_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_user_role``

Response:

::

    Status: 204 No Content

Revoke role from group on domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /domains/{domain_id}/groups/{group_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/domain_group_role``

Response:

::

    Status: 204 No Content

Revoke an inherited project role from user on domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /OS-INHERIT/domains/{domain_id}/users/{user_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#revokeRoleFromUser-domain``

Response:

::

    Status: 204 No Content

Revoke an inherited project role from group on domain *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /OS-INHERIT/domains/{domain_id}/groups/{group_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#revokeRoleFromGroup-domain``

Response:

::

    Status: 204 No Content

Grant role to user on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /projects/{project_id}/users/{user_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_user_role``

Response:

::

    Status: 204 No Content

Grant role to group on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    PUT /projects/{project_id}/groups/{group_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_group_role``

Response:

::

    Status: 204 No Content

Grant role to user on projects in a subtree *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  PUT /OS-INHERIT/projects/{project_id}/users/{user_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#assignRoleToUser``

The inherited role assignment is anchored to a project and applied to its
subtree in the projects hierarchy (both existing and future projects).

* Note: It is possible for a user to have both a regular (non-inherited) and an
  inherited role assignment on the same project.
* Note: The request doesn't require a body, which will be ignored if provided.

Response:

::

    Status: 204 No Content

Grant role to group on projects in a subtree *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  PUT /OS-INHERIT/projects/{project_id}/groups/{group_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#assignRoleToGroup``

The inherited role assignment is anchored to a project and applied to its
subtree in the projects hierarchy (both existing and future projects).

* Note: It is possible for a group to have both a regular (non-inherited) and
  an inherited role assignment on the same project.
* Note: The request doesn't require a body, which will be ignored if provided.

Response:

::

    Status: 204 No Content

List user's roles on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /projects/{project_id}/users/{user_id}/roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_user_roles``

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/projects/--project_id--/users/--user_id--/roles",
            "previous": null,
            "next": null
        }
    }

List group's roles on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /projects/{project_id}/groups/{group_id}/roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_group_roles``

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/projects/--project_id--/groups/--group_id--/roles",
            "previous": null,
            "next": null
        }
    }

List user's inherited project roles on project *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  GET /OS-INHERIT/projects/{project_id}/users/{user_id}/roles/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#listRolesForUser``

The list only contains those roles assigned to this project that were specified
as being inherited to its subtree.

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/OS-INHERIT/projects/--project_id--/
                     users/--user_id--/roles/inherited_to_projects",
            "previous": null,
            "next": null
        }
    }

List group's inherited project roles on project *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  GET /OS-INHERIT/projects/{project_id)/groups/{group_id}/roles/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#listRolesForGroup``

The list only contains those roles assigned to this project that were specified
as being inherited to its subtree.

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--",
            },
            {
                "id": "--role-id--",
                "links": {
                    "self": "http://identity:35357/v3/roles/--role-id--"
                },
                "name": "--role-name--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/OS-INHERIT/projects/--project_id--/
                     groups/--group_id--/roles/inherited_to_projects",
            "previous": null,
            "next": null
        }
    }

Check if user has role on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /projects/{project_id}/users/{user_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_user_role``

Response:

::

    Status: 204 No Content

Check if group has role on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    HEAD /projects/{project_id}/groups/{group_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_group_role``

Response:

::

    Status: 204 No Content

Check if user has an inherited project role on project *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks if a user has a role assignment with the inherited_to_projects flag
on a project.

::

  HEAD /OS-INHERIT/projects/{project_id)/users/{user_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#checkRoleForUser``

Response:

::

    Status: 200 OK

Check if group has an inherited project role on project *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks if a group has a role assignment with the inherited_to_projects flag
on a project.

::

  HEAD /OS-INHERIT/projects/{project_id)/groups/{group_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#checkRoleForGroup``

Response:

::

    Status: 200 OK

Revoke role from user on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /projects/{project_id}/users/{user_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_user_role``

Response:

::

    Status: 204 No Content

Revoke role from group on project
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    DELETE /projects/{project_id}/groups/{group_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/project_group_role``

Response:

::

    Status: 204 No Content

Revoke an inherited project role from user on project *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  DELETE /OS-INHERIT/projects/{project_id)/users/{user_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#revokeRoleFromUser``

Response:

::

    Status: 204 No Content

Revoke an inherited project role from group on project *New in version 3.6*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  DELETE /OS-INHERIT/projects/{project_id)/groups/{group_id}/roles/{role_id}/inherited_to_projects

Relationship:
``http://developer.openstack.org/api-ref-identity-v3.html#revokeRoleFromGroup``

Response:

::

    Status: 204 No Content

List effective role assignments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    GET /role_assignments

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/role_assignments``

*New in version 3.1*

Optional query parameters:

- ``group.id`` (string)

- ``role.id`` (string)

- ``scope.OS-INHERIT:inherited_to`` (string) *New in version 3.6*

- ``scope.domain.id`` (string)

- ``scope.project.id`` (string)

- ``user.id`` (string)

- ``effective`` (key only, no value expected)

- ``include_subtree`` (boolean, defaults to false) *New in version 3.6*

- ``include_names`` (boolean, defaults to false) *New in version 3.6*

Get a list of role assignments.

If no query parameters are specified, then this API will return a list of all
role assignments.

Response:

::

    Status: 200 OK

    {
        "role_assignments": [
            {
                "links": {
                    "assignment": "http://identity:35357/v3/domains/--domain-id--/users/--user-id--/roles/--role-id--"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "domain": {
                        "id": "--domain-id--"
                    }
                },
                "user": {
                    "id": "--user-id--"
                }
            },
            {
                "group": {
                    "id": "--group-id--"
                },
                "links": {
                    "assignment": "http://identity:35357/v3/projects/--project-id--/groups/--group-id--/roles/--role-id--"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "project": {
                        "id": "--project-id--"
                    }
                }
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/role_assignments",
            "previous": null,
            "next": null
        }
    }

Since this list is likely to be very long, this API would typically always be
used with one of more of the filter queries. Some typical examples are:

``GET /role_assignments?user.id={user_id}`` would list all role assignments
involving the specified user.

``GET /role_assignments?scope.project.id={project_id}`` would list all role
assignments involving the specified project.

*New in version 3.6* It is also possible to list all role assignments within
a tree of projects:
``GET /role_assignments?scope.project.id={project_id}&include_subtree=true``
would list all role assignments involving the specified project and all
sub-projects. ``include_subtree=true`` can only be specified in conjunction
with ``scope.project.id``, specifiying it without this will result in an
HTTP ``400 Bad Request`` being returned.

Each role assignment entity in the collection contains a link to the assignment
that gave rise to this entity.

The scope section in the list response is extended to allow the representation
of role assignments that are inherited to projects.

Response:

::

    Status: 200 OK

    {
        "role_assignments": [
            {
                "links": {
                    "assignment": "http://identity:35357/v3/OS-INHERIT/
                                   domains/--domain-id--/users/--user-id--/
                                   roles/--role-id--/inherited_to_projects"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "domain": {
                        "id": "--domain-id--"
                    },
                    "OS-INHERIT:inherited_to": "projects"
                },
                "user": {
                    "id": "--user-id--"
                }
            },
            {
                "group": {
                    "id": "--group-id--"
                },
                "links": {
                    "assignment": "http://identity:35357/v3/projects/--project-id--/
                                   groups/--group-id--/roles/--role-id--"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "project": {
                        "id": "--project-id--"
                    }
                }
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/role_assignments",
            "previous": null,
            "next": null
        }
    }

The query filter ``scope.OS-INHERIT:inherited_to`` can be used to filter based
on role assignments that are inherited. The only value of
``scope.OS-INHERIT:inherited_to`` that is currently supported is ``projects``,
indicating that this role is inherited to all projects of the owning domain or
parent project.

If the query parameter ``effective`` is specified, rather than simply returning
a list of role assignments that have been made, the API returns a list of
effective assignments at the user, project and domain level, having allowed for
the effects of group membership, role inference rules as well as inheritance
from the parent domain or project. Since the effects of group membership have
already been allowed for, the group role assignment entities themselves will
not be returned in the collection. Likewise, since the effects of inheritance
have already been allowed for, the role assignment entities themselves that
specify the inheritance will also not be returned in the collection. This
represents the effective role assignments that would be included in a scoped
token. The same set of query parameters can also be used in combination with
the ``effective`` parameter.

For example:

``GET /role_assignments?user.id={user_id}&effective`` would, in other words,
answer the question "what can this user actually do?".

``GET
/role_assignments?user.id={user_id}&scope.project.id={project_id}&effective``
would return the equivalent set of role assignments that would be included in
the token response of a project scoped token.

An example response for an API call with the query parameter ``effective``
specified is given below:

Response:

::

    Status: 200 OK

    {
        "role_assignments": [
            {
                "links": {
                    "assignment": "http://identity:35357/v3/domains/--domain-id--/users/--user-id--/roles/--role-id--"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "domain": {
                        "id": "--domain-id--"
                    }
                },
                "user": {
                    "id": "--user-id--"
                }
            },
            {
                "links": {
                    "assignment": "http://identity:35357/v3/projects/--project-id--/groups/--group-id--/roles/--role-id--",
                    "membership": "http://identity:35357/v3/groups/--group-id--/users/--user-id--"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "project": {
                        "id": "--project-id--"
                    }
                },
                "user": {
                    "id": "--user-id--"
                }
            },
            {
                "links": {
                    "assignment": "http://identity:35357/v3/OS-INHERIT/
                                   domains/--domain-id--/users/--user-id--/
                                   roles/--role-id--/inherited_to_projects"
                },
                "role": {
                    "id": "--role-id--"
                },
                "scope": {
                    "project": {
                        "id": "--project-id--"
                    }
                },
                "user": {
                    "id": "--user-id--"
                }
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/role_assignments?effective",
            "previous": null,
            "next": null
        }
    }

The entity ``links`` section of a response using the ``effective`` query
parameter also contains, for entities that are included by virtue of group
membership, a url that can be used to access the membership of the group.

If the query parameter ``include_names`` is specified, rather than simply
returning the entity IDs in the role assignments, the collection will
additionally include the names of the entities. For example:

``GET /role_assignments?user.id={user_id}&effective&include_names=true``
would return:

Response:

::

    Status: 200 OK

    {
        "role_assignments": [
            {
                "links": {
                    "assignment": "http://identity:35357/v3/domains/--domain-id--/users/--user-id--/roles/--role-id--"
                },
                "role": {
                    "id": "--role-id--",
                    "name": "--role-name--"
                },
                "scope": {
                    "domain": {
                        "id": "--domain-id--",
                        "name": "--domain-name--"
                    }
                },
                "user": {
                    "domain": {
                        "id": "--domain-id--",
                        "name": "--domain-name--"
                    },
                    "id": "--user-id--",
                    "name": "--user-name--",
                    "password_expires_at": "--password-expires-at--"
                }
            },
            {
                "links": {
                    "assignment": "http://identity:35357/v3/projects/--project-id--/groups/--group-id--/roles/--role-id--",
                    "membership": "http://identity:35357/v3/groups/--group-id--/users/--user-id--"
                },
                "role": {
                    "id": "--role-id--",
                    "name": "--role-name--"
                },
                "scope": {
                    "project": {
                        "domain": {
                            "id": "--domain-id--",
                            "name": "--domain-name--"
                        }
                        "id": "--project-id--",
                        "name": "--project-name--"
                    }
                },
                "user": {
                    "domain": {
                        "id": "--domain-id--",
                        "name": "--domain-name--"
                    },
                    "id": "--user-id--",
                    "name": "--user-name--",
                    "password_expires_at": "--password-expires-at--"
                }
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/role_assignments?effective&include_names=true",
            "previous": null,
            "next": null
        }
    }

Policies
~~~~~~~~

The key use cases we need to cover:

- CRUD on a policy

Create policy
^^^^^^^^^^^^^

::

    POST /policies

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/policies``

Request:

::

    {
        "blob": "--serialized-blob--",
        "type": "--serialization-mime-type--"
    }

Response:

::

    Status: 201 Created

    {
        "policy": {
            "blob": "--serialized-blob--",
            "id": "--policy-id--",
            "links": {
                "self": "http://identity:35357/v3/policies/--policy-id--"
            },
            "type": "--serialization-mime-type--"
        }
    }

List policies
^^^^^^^^^^^^^

::

    GET /policies

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/policies``

Optional query parameters:

- ``type`` (string)

Response:

::

    Status: 200 OK

    {
        "policies": [
            {
                "blob": "--serialized-blob--",
                "id": "--policy-id--",
                "links": {
                    "self": "http://identity:35357/v3/policies/--policy-id--"
                },
                "type": "--serialization-mime-type--"
            },
            {
                "blob": "--serialized-blob--",
                "id": "--policy-id--",
                "links": {
                    "self": "http://identity:35357/v3/policies/--policy-id--"
                },
                "type": "--serialization-mime-type--"
            }
        ],
        "links": {
            "self": "http://identity:35357/v3/policies",
            "previous": null,
            "next": null
        }
    }

Get policy
^^^^^^^^^^

::

    GET /policies/{policy_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/policy``

Response:

::

    Status: 200 OK

    {
        "policy": {
            "blob": "--serialized-blob--",
            "id": "--policy-id--",
            "links": {
                "self": "http://identity:35357/v3/policies/--policy-id--"
            },
            "type": "--serialization-mime-type--"
        }
    }

Update policy
^^^^^^^^^^^^^

::

    PATCH /policies/{policy_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/policy``

The request block is the same as the one for create policy, except that only
the attributes that are being updated need to be included.

Response:

::

    Status: 200 OK

    {
        "policy": {
            "blob": "--serialized-blob--",
            "id": "--policy-id--",
            "links": {
                "self": "http://identity:35357/v3/policies/--policy-id--"
            },
            "type": "--serialization-mime-type--"
        }
    }

Delete policy
^^^^^^^^^^^^^

::

    DELETE /policies/{policy_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/rel/policy``

Response:

::

    Status: 204 No Content
