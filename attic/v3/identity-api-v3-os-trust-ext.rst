OpenStack Identity API v3 OS-TRUST
==================================

Trusts provide project-specific role delegation between users, with optional
impersonation.

API Resources
-------------

Trusts
~~~~~~

A trust represents a user's (the *trustor*) authorization to delegate roles to
another user (the *trustee*), and optionally allow the trustee to impersonate
the trustor. After the trustor has created a trust, the trustee can specify the
trust's ``id`` attribute as part of an authentication request to then create a
token representing the delegated authority.

The trust contains constraints on the delegated attributes. A token created
based on a trust will convey a subset of the trustor's roles on the specified
project. Optionally, the trust may only be valid for a specified time period,
as defined by ``expires_at``. If no ``expires_at`` is specified, then the trust
is valid until it is explicitly revoked.

The ``impersonation`` flag allows the trustor to optionally delegate
impersonation abilities to the trustee. To services validating the token, the
trustee will appear as the trustor, although the token will also contain the
``impersonation`` flag to indicate that this behavior is in effect.

A ``project_id`` may not be specified without at least one role, and vice
versa. In other words, there is no way of implicitly delegating all roles to a
trustee, in order to prevent users accidentally creating trust that are much
more broad in scope than intended. A trust without a ``project_id`` or any
delegated roles is unscoped, and therefore does not represent authorization on
a specific resource.

Trusts are immutable. If the trustee wishes to modify the attributes of the
trust, they should create a new trust and delete the old trust. If a trust is
deleted, any tokens generated based on the trust are immediately revoked.

If the trustor loses access to any delegated attributes, the trust becomes
immediately invalid and any tokens generated based on the trust are immediately
revoked.

Additional required attributes:

- ``trustor_user_id`` (string)

  Represents the user who created the trust, and who's authorization is being
  delegated.

- ``trustee_user_id`` (string)

  Represents the user who is capable of consuming the trust.

- ``impersonation``: (boolean)

  If ``impersonation`` is set to ``true``, then the ``user`` attribute of
  tokens token's generated based on the trust will represent that of the
  trustor rather than the trustee, thus allowing the trustee to impersonate the
  trustor. If ``impersonation`` is set to ``false``, then the token's ``user``
  attribute will represent that of the trustee.

Optional attributes:

- ``project_id`` (string)

  Identifies the project upon which the trustor is delegating authorization.

- ``roles``: (list of objects)

  Specifies the subset of the trustor's roles on the ``project_id`` to be
  granted to the trustee when the token is consumed. The trustor must
  already be granted these roles in the project referenced by the
  ``project_id`` attribute. If redelegation is used (when trust-scoped token
  is used and consumed trust has ``allow_redelegation`` set to ``true``)
  this parameter should contain redelegated trust's roles only.

  Roles are only provided when the trust is created, and are subsequently
  available as a separate read-only collection. Each role can be specified by
  either ``id`` or ``name``.

- ``expires_at`` (string, ISO 8601 extended format date time with microseconds)

  Specifies the expiration time of the trust. A trust may be revoked ahead
  of expiration. If the value represents a time in the past, the trust is
  deactivated. In the redelegation case it must not exceed the value of the
  corresponding ``expires_at`` field of the redelegated trust or it may be
  omitted, then the ``expires_at`` value is copied from the redelegated trust.

- ``remaining_uses`` (integer or null)

  Specifies how many times the trust can be used to obtain a token. This
  value is decreased each time a token is issued through the trust. Once
  it reaches 0, no further tokens will be issued through the trust. The
  default value is null, meaning there is no limit on the number of tokens
  issued through the trust. If redelegation is enabled it must not be set.

- ``allow_redelegation`` (boolean)

  If ``allow_redelegation`` is set to ``true`` then a trust between a
  ``trustor`` and any third-party user may be issued by the ``trustee``
  just like a regular trust. If set to ``false``, stops further redelegation.
  ``false`` by default, not stored in the trust.

- ``redelegation_count`` (integer or null)

  Specifies the maximum remaining depth of the redelegated trust chain. Each
  subsequent trust has this field decremented by 1 automatically.
  The initial ``trustor`` issuing new trust that can be redelegated, must set
  ``allow_redelegation`` to ``true`` and may set ``redelegation_count`` to an
  integer value less than or equal to ``max_redelegation_count`` configuration
  parameter in order to limit the possible length of derivated trust chains.
  The trust issued by the ``trustor`` using a regular token (not redelegating),
  in which ``allow_redelegation`` is set to ``true`` (the new trust is
  redelegatable), will be populated with the value specified in the
  ``max_redelegation_count`` configuration parameter if ``redelegation_count``
  is not set or set to ``null``.
  If ``allow_redelegation`` is set to ``false`` then ``redelegation_count``
  will be set to 0 in the trust on the server side.

  If the trust is being issued by the ``trustee`` of a redelegatable
  trust-scoped token (redelegation case) then ``redelegation_count`` should not
  be set, as it will automatically be set to the value in the redelegatable
  trust-scoped token decremented by 1. Note, if the resulting value is 0, this
  means that the new trust will not be redelegatable, regardless of the value
  of ``allow_redelegation``.

- ``redelegated_trust_id`` (string, read-only)

  Returned with redelegated trust provides information about the predecessor
  in the trust chain. Specifying this field in a trust manipulation request
  has no effect.

Example entity:

::

    {
        "trust": {
            "id": "987fe7",
            "impersonation": true,
            "project_id": "0f1233",
            "remaining_uses": null,
            "allow_redelegation": true,
            "redelegation_count": 2,
            "links": {
                "self": "http://identity:35357/v3/trusts/987fe7"
            },
            "trustee_user_id": "fea342",
            "trustor_user_id": "56aed3"
        }
    }

Tokens
~~~~~~

Additional attributes:

- ``trust`` (object)

  If present, indicates that the token was created based on a trust. This
  attribute identifies both the trustor and trustee, and indicates whether the
  token represents the trustee impersonating the trustor.

API
---

Consuming a trust
~~~~~~~~~~~~~~~~~

::

    POST /auth/tokens

Consuming a trust effectively assumes the scope as delegated in the trust. No
other scope attributes may be specified.

The user specified by ``authentication`` must match the trust's
``trustee_user_id`` attribute.

If the trust has the ``impersonation`` attribute set to ``true``, then the
resulting token's ``user`` attribute will also represent the trustor, rather
than the authenticating user (the trustee).

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
            },
            "scope": {
                "OS-TRUST:trust": {
                    "id": "de0945a"
                }
            }
        }
    }

A token created from a trust will have a ``trust`` section containing the
``id`` of the trust, the ``impersonation`` flag, the ``trustee_user_id`` and
the ``trustor_user_id``. Example response:

::

    Headers: X-Subject-Token

    X-Subject-Token: e80b74

    {
        "token": {
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "OS-TRUST:trust": {
                "id": "fe0aef",
                "impersonation": false,
                "links": {
                    "self": "http://identity:35357/v3/trusts/fe0aef"
                },
                "trustee_user": {
                    "id": "0ca8f6",
                    "links": {
                        "self": "http://identity:35357/v3/users/0ca8f6"
                    }
                },
                "trustor_user": {
                    "id": "bd263c",
                    "links": {
                        "self": "http://identity:35357/v3/users/bd263c"
                    }
                }
            },
            "user": {
                "domain": {
                    "id": "1789d1",
                    "links": {
                        "self": "http://identity:35357/v3/domains/1789d1"
                    },
                    "name": "example.com"
                },
                "email": "joe@example.com",
                "id": "0ca8f6",
                "links": {
                    "self": "http://identity:35357/v3/users/0ca8f6"
                },
                "name": "Joe"
            }
        }
    }

A token created from a redelegated trust will have an  ``OS-TRUST:trust``
section containing the same fields as a regular trust token, only
``redelegated_trust_id`` and ``redelegation_count`` are added.
Example response:

::

    Headers: X-Subject-Token

    X-Subject-Token: e80b74

    {
        "token": {
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "issued_at": "2013-02-27T16:30:59.999999Z",
            "methods": [
                "password"
            ],
            "OS-TRUST:trust": {
                "id": "fe0aef",
                "impersonation": false,
                "redelegated_trust_id": "3ba234",
                "redelegation_count": 2,
                "links": {
                    "self": "http://identity:35357/v3/trusts/fe0aef"
                },
                "trustee_user": {
                    "id": "0ca8f6",
                    "links": {
                        "self": "http://identity:35357/v3/users/0ca8f6"
                    }
                },
                "trustor_user": {
                    "id": "bd263c",
                    "links": {
                        "self": "http://identity:35357/v3/users/bd263c"
                    }
                }
            },
            "user": {
                "domain": {
                    "id": "1789d1",
                    "links": {
                        "self": "http://identity:35357/v3/domains/1789d1"
                    },
                    "name": "example.com"
                },
                "email": "joe@example.com",
                "id": "0ca8f6",
                "links": {
                    "self": "http://identity:35357/v3/users/0ca8f6"
                },
                "name": "Joe"
            }
        }
    }

Create trust
~~~~~~~~~~~~

::

    POST /OS-TRUST/trusts

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trusts``

Request:

::

    {
        "trust": {
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "impersonation": true,
            "allow_redelegation": true,
            "project_id": "ddef321",
            "roles": [
                {
                    "name": "member"
                }
            ],
            "trustee_user_id": "86c0d5",
            "trustor_user_id": "a0fdfd"
        }
    }

Response:

::

    Status: 201 Created

    {
        "trust": {
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "id": "1ff900",
            "impersonation": true,
            "redelegation_count": 10,
            "links": {
                "self": "http://identity:35357/v3/OS-TRUST/trusts/1ff900"
            },
            "project_id": "ddef321",
            "remaining_uses": null,
            "roles": [
                {
                    "id": "ed7b78",
                    "links": {
                        "self": "http://identity:35357/v3/roles/ed7b78"
                    },
                    "name": "member"
                }
            ],
            "roles_links": {
                "next": null,
                "previous": null,
                "self": "http://identity:35357/v3/OS-TRUST/trusts/1ff900/roles"
            },
            "trustee_user_id": "86c0d5",
            "trustor_user_id": "a0fdfd"
        }
    }

List trusts
~~~~~~~~~~~

::

    GET /OS-TRUST/trusts

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trusts``

Optional query strings:

- ``page``

- ``per_page`` (default 30)

- ``trustee_user_id``

- ``trustor_user_id``

Response:

::

    Status: 200 OK

    {
        "trusts": [
            {
                "id": "1ff900",
                "expires_at": "2013-02-27T18:30:59.999999Z",
                "impersonation": true,
                "redelegation_count": 10,
                "links": {
                    "self": "http://identity:35357/v3/OS-TRUST/trusts/1ff900"
                },
                "project_id": "0f1233",
                "trustee_user_id": "86c0d5",
                "trustor_user_id": "a0fdfd"
            },
            {
                "id": "f4513a",
                "impersonation": true,
                "redelegation_count": 1,
                "redelegated_trust_id": "34fc39",
                "links": {
                    "self": "http://identity:35357/v3/OS-TRUST/trusts/f4513a"
                },
                "project_id": "0f1233",
                "trustee_user_id": "86c0d5",
                "trustor_user_id": "3cd2ce"
            }
        ]
    }

In order to list trusts for a given trustor, filter the collection using a
query string (e.g., ``?trustor_user_id={user_id}``).

Request:

::

    GET /OS-TRUST/trusts?trustor_user_id=a0fdfd

Response:

::

    Status: 200 OK

    {
        "trusts": [
            {
                "id": "1ff900",
                "expires_at": "2013-02-27T18:30:59.999999Z",
                "impersonation": false,
                "links": {
                    "self": "http://identity:35357/v3/OS-TRUST/trusts/1ff900"
                },
                "project_id": "0f1233",
                "trustee_user_id": "86c0d5",
                "trustor_user_id": "a0fdfd"
            }
        ]
    }

In order to list trusts for a given trustee, filter the collection using a
query string (e.g., ``?trustee_user_id={user_id}``).

Request:

::

    GET /OS-TRUST/trusts?trustee_user_id=86c0d5

Response:

::

    Status: 200 OK

    {
        "trusts": [
            {
                "id": "1ff900",
                "expires_at": "2013-02-27T18:30:59.999999Z",
                "impersonation": true,
                "links": {
                    "self": "http://identity:35357/v3/OS-TRUST/trusts/1ff900"
                },
                "project_id": "0f1233",
                "trustee_user_id": "86c0d5",
                "trustor_user_id": "a0fdfd"
            },
            {
                "id": "f4513a",
                "impersonation": false,
                "links": {
                    "self": "http://identity:35357/v3/OS-TRUST/trusts/f45513a"
                },
                "project_id": "0f1233",
                "trustee_user_id": "86c0d5",
                "trustor_user_id": "3cd2ce"
            }
        ]
    }

Get trust
~~~~~~~~~

::

    GET /OS-TRUST/trusts/{trust_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trust``

Response:

::

    Status: 200 OK

    {
        "trust": {
            "id": "987fe8",
            "expires_at": "2013-02-27T18:30:59.999999Z",
            "impersonation": true,
            "links": {
                "self": "http://identity:35357/v3/OS-TRUST/trusts/987fe8"
            },
            "roles": [
                {
                    "id": "ed7b78",
                    "links": {
                        "self": "http://identity:35357/v3/roles/ed7b78"
                    },
                    "name": "member"
                }
            ],
            "roles_links": {
                "next": null,
                "previous": null,
                "self": "http://identity:35357/v3/OS-TRUST/trusts/1ff900/roles"
            },
            "project_id": "0f1233",
            "trustee_user_id": "be34d1",
            "trustor_user_id": "56ae32"
        }
    }

Delete trust
~~~~~~~~~~~~

::

    DELETE /OS-TRUST/trusts/{trust_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trust``

Response:

::

    Status: 204 No Content

List roles delegated by a trust
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    GET /OS-TRUST/trusts/{trust_id}/roles

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trust_roles``

Response:

::

    Status: 200 OK

    {
        "roles": [
            {
                "id": "c1648e",
                "links": {
                    "self": "http://identity:35357/v3/roles/c1648e"
                },
                "name": "manager"
            },
            {
                "id": "ed7b78",
                "links": {
                    "self": "http://identity:35357/v3/roles/ed7b78"
                },
                "name": "member"
            }
        ]
    }

Check if role is delegated by a trust
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    HEAD /OS-TRUST/trusts/{trust_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trust_role``

Response:

::

    Status: 200 OK

Get role delegated by a trust
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    GET /OS-TRUST/trusts/{trust_id}/roles/{role_id}

Relationship:
``http://docs.openstack.org/api/openstack-identity/3/ext/OS-TRUST/1.0/rel/trust_role``

Response:

::

    Status: 200 OK

    {
        "role": {
            "id": "c1648e",
            "links": {
                "self": "http://identity:35357/v3/roles/c1648e"
            },
            "name": "manager"
        }
    }

