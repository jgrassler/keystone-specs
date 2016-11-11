..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================
 Allow Standalone Trusts without Trustee User
=============================================

`bp standalone-trusts <https://blueprints.launchpad.net/keystone/+spec/standalone-trusts>`_

The existing trusts implementation requires a *Trustee user* that a trust is
assigned to. This effectively limits the use of a trust to service users who
have access to both the regular user's account through the keystone token used
to authenticate against the service and an acocunt they control themselves (a
service account or a dedicated trustee account specifically created for the
purpose). Optionally allowing an API key rather than a trustee user to consume
a trust would enable users to create trusts on their own and generally reduce
overhead in handling trusts.

Problem Description
===================

Trusts are currently tied to a trustee user. This is undesirable for the
following reasons:

* A user always needs to use their credentials to access their user account.
  This is fine as long as the account is only used on the user's personal
  machine but becomes a security problem as soon as people use their
  credentials for purposes such as a statistics gathering tool running on a
  remote machine with shared access. The user's credentials would give full,
  uncontrollable access to a user's account if that machine is compromised,
  while a trust could still be revoked if the breach is discovered, allowing
  for a measure of damage control. Currently a user cannot create such a trust
  because that user would have to create or at least have access to a second
  user account to use as a trustee user.

* In a scenario where trust credentials need to be passed into an instance, the
  services creating trusts for this purpose need to create a dedicated trustee
  user, which requires passing three separate pieces of data (user name,
  password and trust ID) to the entity using them, where one (an API key) would
  suffice with standalone trusts in place. Once the trust is no longer needed,
  there's additional bookkeeping overhead in removing the trustee user.

Proposed Change
===============

This change extends the trusts API by allowing trusts to be created without
specifying a trustee. When a user creates such a trust they will receive an API
key that can then be used to acquire tokens scoped to the trust in question
throughout the life time of the trust. The API key is sufficient to obtain the
trust scoped token: the trust ID does not need to be supplied along with it.

This change does explicitely not remove the ability to tie a trust to a trustee
user, for this can be used to provide additional security in cases where such a
user is available anyway (e.g. when a trust is delegated to a service user).

Alternatives
------------

Continue requiring a trustee user for trust creation.

Security Impact
---------------

Once this feature (along with [0]) widely known and adopted, it will arguably
improve security since users no longer need to use their login credentials to
grant applications or automation tools access to their OpenStack account.
Instead they can provide these applications and automation tools with a trust
scoped API key.

* First and formeost, this change adds a new way of acquiring a token scoped to
  a trust: the trust's API key will now be sufficient if the trust has been
  created using the method provided by this change. For scenarios where the
  association with a trustee user is desired, this feature must be configurable
  (just as trusts themselves are configurable).

* The API key itself needs to be generated in a secure manner since it is used
  for both identification and authentication. In the simplest case it would be
  a unique, long random string. The exact manner of creating this string should
  be discussed in the course of spec review.

Notifications Impact
--------------------

None

Other End User Impact
---------------------

python-keystoneclient and python-openstackclient will need to allow trust
creation without specifying trustee user.

Performance Impact
------------------

Keystone needs to be able to look up a trust from the database by API key, so
the column holding a trust's API key should be a primary key for the table.

Other Deployer Impact
---------------------

This feature will be configurable through the setting `trust/standalone_trusts`
which will be `True` by default. The rationale behind this is giving as many
users as possible the option to use lightweight trusts (thus keeping them from
picking the worse option of providing their login credentials to applications
and automation tools) while still allowing operators to prohibit the use of
API keys.

Developer Impact
----------------

None. This feature can be implemented in a way that does not break the way
trusts *with* a trustee user are created.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Johannes Grassler <jgr-launchpad>

Other assignees:
  Konstantin Baikov (kbaikov)
  Sayali Lunkad (sayalilunkad)

Work Items
----------

1. Implement support for creating a standalone trust in keystone.

2. Implement support for creating a standalone trust in python-keystoneclient
   and python-openstackclient.

3. Implement an API key authentication method in keystone.

Dependencies
============

None

Documentation Impact
====================

The existence of this feature needs to be documented in both the admin (to
allow operators to decide on enabling/disabling it) and user guides (to inform
users who might otherwise provide their login credentials to
applications/automation tools).

In both cases there needs to be some discussion of the scenarios where its use
is appropriate. In admin guide there needs to be information on how to
enable/disable it. Since it is enabled by default, the Ocata release notes
should mention it.

References
==========


[0] https://blueprints.launchpad.net/keystone/+spec/trust-scope-extensions

    Allows fine-grained restrictions (capabilities) on trusts.
