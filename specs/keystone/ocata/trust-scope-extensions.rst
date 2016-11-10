..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================================
Add Fine Grained Restrictions to Keystone Trusts
================================================

`bp <https://blueprints.launchpad.net/keystone/+spec/trust-scope-extensions>`_

Currently Keystone trusts are mostly unrestricted. Restrictions can only be
imposed on redelegation of the trust and impersonation. Other than that they
allow unfettered use of the roles being delegated in the project the trust is
created for. This renders trusts questionable anywhere a least-privilege
delegation is desired. Technically it is possible to store additional
restrictions for a trust which other OpenStack services would then enforce.
This spec outlines an approach for storing and handling such restrictions.

Problem Description
===================

Keystone trusts have been around for quite a while now (they were introduced in
the Grizzly release), but adoption remains low. Right now they are used by the
following projects among others (list may not be complete):

* Heat: operations on behalf of the user at times when a token may have expired
        already.
* Magnum: access to Magnum's certificate signing API and other OpenStack APIs
          from inside a cluster's instances where the container orchestration
          engine requires it (e.g. Glance as backend for docker-registry or
          cinder as backend for docker-volume)

Other projects (neutron-lbaas, Barbican) hesitate to employ trusts since they
are an all-or-nothing approach: they grant full access to all OpenStack APIs in
the scope (roles and project) they are created for. In order to provide
least-privilege access, these services implement ACLs on their own (Barbican,
Swift) or rely on other services' ACLs to grant limited access to resources
(neutron-lbaas uses Barbican's ACLs to grant its service user access to secret
containers holding SSL keys).

The existence of these project specific ACL implementations shows there is a
real need for fine-grained delegation of access. The implementation of Keystone
trusts as it exists right now cannot serve this need, though: a trust can only
be used to grant full access within the trust's scope, but it cannot be used to
give access limited to just one particular resource or access that only allows
the creation of new resources, but not the modification/deletion of existing
resources.

With fine-grained restrictions in place, trusts can be used to perform the task
service specific ACLs are currently used (and still being implemented) for, in
a reliable, consistent manner.

Proposed Change
===============

The approach to implementing fine-grained permissions for trusts is
three-pronged. Permission data is stored in Keystone and enforced by
oslo.policy and keystonemiddleware as follows:

1) Alongside a trust, capability triples are stored. A capability triple
   consists of a service (e.g. Nova) an oslo.policy target (e.g. "compute:get")
   and a permission level. The latter can be

     (a) a object UUID for restriction to a specific object
     (b) the keyword 'user' for restriction to objects owned by the trustee
         user
     (c) unspecified for unrestricted access to that target with the trust's
         role/project scope

   This list is a white list, e.g. access to an unspecified target is denied by
   default. Keystone does not validate the content of capability triples since
   that would require awareness of service specific policy rules.

   In addition to capability triples, a list of endpoints the trust is valid
   for can optionally be specified. An empty list means access to any endpoint.

2) The service receives the additional permission information stored alongside
   the trust in the course of token validation. Based on the endpoint list
   keystonemiddleware performs the first check: it checks the list of endpoints
   against the endpoint the token is being used for. If the list is empty or
   the current endpoint is in the list, it allows the request to proceed.
   Otherwise it denies the request.

3) If keystonemiddleware's endpoint check passes, oslo.policy performs the
   second check: it verifies if the trust's capabilities (or rather the subset
   applying to the service in question) allow the request being made. If not,
   it denies the request. Only if the trust's capabilities allow the request,
   oslo.policy proceeds to the regular policy checks as they exist today.

Alternatives
------------

One alternative to this exists already: internal ACL implementations by various
OpenStack services. This situation is undesirable for several reasons, some of
which are:

1) Auditability: authorization information is stored in multiple locations, all
                 of which need to be checked to find out who is authorized to
                 perform what operation. From an auditability perspective it
                 would be preferable to have a central source of truth.

2) Maintenance: when there are multiple independent implementations a lot of
                code is duplicated and bugs may be duplicated as well as new
                projects implement their own ACL system.

3) Consistency: with multiple sources of truth, an individual service's ACLs
                may well end up overriding a cloud-wide policy permitting or
                denying an operation.

Security Impact
---------------

This change tightens security by providing a means to restrict the permissions
granted by trusts. That being said, its implementation does have various
security critical aspects:

* This change adds additional information to the token data retrieved by
  keystonemiddleware upon token validation.

* Capability rules contain user-supplied strings which will eventually be
  parsed by oslo.policy. These user supplied strings should be sanitized to
  guard against format string attacks in this part of the implementation.

* It might be a good idea to limit the length/number of capability rules per
  trust to prevent denial of service against the Keystone database (by filling
  it with bogus rules) or the Keystone API (via large validation payloads).
  Another reason to introduce such a limit is the possibility to slow down a
  service by creating trusts with a large number of capabilities, all of which
  have to be evaluated. Likewise, introducing a limit on the number of trusts
  would also be a good idea to prevent creating such a large list by dividing
  it between a large number of trusts. The number of endpoints should also be
  limitable.

* This change is unlikely to allow privilege escalation since it only adds
  additional failing criteria to token validation and policy enforcement. These
  failing criteria need to be carefully tested for false positives, though.


Notifications Impact
--------------------

None

Other End User Impact
---------------------

Since this changes adds extra information to trusts, both python-keystoneclient
and python-openstackclient need to be extended to handle that extra
information.

Performance Impact
------------------

The performance impact upon trust creation is probably neglible, since all that
happens is that a small amount of data is stored along with the trust.

That small amount of data may not be so small during the token validation,
though, resulting in multiple/more packets being sent in response to validation
request, causing congestion and/or increasing latency. This can be mitigated by
limiting the number of capabilities allowed per trust.

This mitigation strategy can also be used to limit performance impact on
the oslo.policy side: by having a limited number of capability rules, there is
also a limit on the amount of processing overhead they incur.

Other Deployer Impact
---------------------

This change will introduce the following settings for Keystone:

* `trust/capabilities` [Default: `True`] This setting configures capabilities
  for trusts globally. If it is changed to `False`, Keystone will not allow new
  trusts with capabilities to be created and it will not send capability
  information upon token validation. For existing trusts with capabilities,
  capability information is still visible when showing the trust. This setting
  gives operators the ability to disable capabilities in cases where they
  cause performance problems or are unwanted for other reasons.

* `trust/endpoints` [Default: `True`] This setting configures endpoint limits
  for trusts globally. If it is changed to `False`, Keystone will not allow new
  trusts with endpoint limits to be created and it will not send endpoint
  limit information upon token validation. For existing trusts with
  endpoint limits, that information is still visible when showing the trust.
  This setting gives operators the ability to disable endpoint limits in cases
  where they cause performance problems or are unwanted for other reasons.

Developer Impact
----------------

This change provides developers across all OpenStack services with a means to
create trusts with fine-grained permissions, allowing them to get delegate
access to their service users according to the principle of least privilege.

As far as the trusts API is concerned, it will be fully backwards compatible,
since specifying capabilities and endpoints when creating a trust is optional.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Johannes Grassler (jgr-launchpad)

Other assignees:
  Konstantin Baikov (kbaikov)
  Sayali Lunkad (sayalilunkad)

Work Items
----------

1. Extend the trusts API and database schema in Keystone to allow for endpoints
   and capabilities. Strictly speaking, an extension is not neccessary, since
   any kwargs being passed upon trust creation will end up in the `extra`
   column, but this is very ugly.

2. Implement handling for endpoint and capability information in
   python-keystoneclient and python-openstackclient.

3. Extend the Keystone token validation API to pass capability and endpoint
   lists upon token validation.

4. Implement the endpoint list check in keystonemiddleware.

5. Implement the capability check in oslo.policy.

Dependencies
============

None

Documentation Impact
====================

* The existence of capability and endpoint limitation for trusts needs to be
  documented in the release notes and the admin guide.

* The capability "language" outlined in the *Proposed Change* section needs
  to be documented in the admin guide.

References
==========

* Etherpad with original proposal from the Barcelona 2016 summit: https://etherpad.openstack.org/p/ocata-keystone-authorization

* Documentation on Barbican ACLs: http://developer.openstack.org/api-guide/key-manager/acls.html

* Documentation on Swift ACLs: https://www.swiftstack.com/docs/cookbooks/swift_usage/container_acl.html

