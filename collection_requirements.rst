*****************************
Ansible Collections Checklist
*****************************

.. contents:: Topics

Overview
========
This document is for maintainers of collections to provide them help, advice, and guidance on making sure their collections are correct.

**Warning:** Subject to frequent updates
       This is a "living document", expect it to change as we progress with the Collections work over the next few months.

Feedback
========

As with any project it's very important that we get feedback from users, contributors and maintainers. We recognize that the move to Collections is a big change in how Ansible is developed and delivered.

Please raise feedback by:

* Discussing in ``#ansible-community`` in Freenode IRC
* Adding to the `Community Working Group IRC meeting <https://github.com/ansible/community/issues/539>`_
* Creating `GitHub Issues <https://github.com/ansible-collections/overview/issues>`_ against this repository

Keeping informed
================

Be sure you're subscribed to:

* `Changes impacting Collections <https://github.com/ansible-collections/overview/issues/45>`_ to track changes that Collection maintainers should be aware of
* The Bullhorn, a newsletter for the Ansible developer community, `back issues and how to add content <https://github.com/ansible/community/issues/546>`_

Why is this needed
===================


Repo structure
===============

galaxy.yml
----------

* The ``tags`` field MUST be set
* Collection dependencies used are expected to be stable, hence MUST be set to ``'>=1.0.0'``

  * This means that all collection dependencies have to specify lower bounds on the versions, and these lower bounds should be stable releases, and not versions of the form 0.x.y.
  * When creating new collections where collection dependencies are also under development, you need to watch out since Galaxy checks whether dependencies exist in the required versions:

    1. Assume that ``foo.bar`` depends on ``foo.baz``
    2. First release ``foo.baz`` as 1.0.0.
    3. Then modify ``foo.bar``'s ``galaxy.yml`` to specify ``'>=1.0.0'`` for ``foo.baz``
    4. Finally release ``foo.bar`` as 1.0.0

meta/runtime.yml
----------------
Example: `meta/runtime.yml <https://github.com/ansible-collections/collection_template/blob/main/meta/runtime.yml>`_

* MUST define the minimum version of Ansible which this collection works with

  * If the collection works with Ansible 2.9, then this should be set to `>=2.9.10`
  * It's usually better to avoid adding `<2.11` as a restriction, since this for example makes it impossible to use the collection with the current ansible-base devel branch (which has version 2.11.0.dev0)

Modules & Plugins
------------------

Documentation
~~~~~~~~~~~~~~
All module and plugin ``DOCUMENTATION`` and ``RETURN`` MUST:

* Use the FQCN for ``M(...)`` and ``- module:`` references of ``seealso`` subsections. See `Linking within module documentation <https://docs.ansible.com/ansible/devel/dev_guide/developing_modules_documenting.html#linking-within-module-documentation>`_
* Use the field ``version_added`` to document the version of the collection for which an option, module or plugin was added.

  * Use collection version numbers for ``version_added``, and not Ansible version numbers or other unrelated version numbers.
  * If you for some reason really have to specify version numbers of Ansible or of another collection, you have to provide ``version_added_collection``. We strongly recommend to NOT do this.
  * Not every option, module or plugin must have ``version_added``. You should use it to mark when new content (modules, plugins, options) were added to the collection. The values are shown in the documentation, and this can be very useful for your users.

All module and plugin ``EXAMPLES`` MUST:

* Use FQCN for module (or plugin) name.
* For modules (or plugins) left in ansible-base use ``ansible.builtin.`` as a FQCN prefix, for example, ``ansible.builtin.template``

Other items:

* You MUST Use the FQCN for ``extends_documentation_fragment:``, unless you are referring to doc_fragments from ansible-base


Contributor Workflow
====================

Changelogs
----------

To give a consistent feel for changelogs across collections, and ensure for collections included in the ``ansible`` package we suggest you use `antsibull-changelog <https://github.com/ansible-community/antsibull-changelog>`_


Preferred (in descending order):

1. Use antsibull-changelog (preferred)
2. Provide ``changelogs/changelog.yaml`` in the `correct format <https://github.com/ansible-community/antsibull-changelog/blob/main/docs/changelog.yaml-format.md>`_
3. Provide a link to the changelog file (self-hosted) (not recommended)

Please note that the porting guide is compiled from ``changelogs/changelog.yaml`` (sections ``breaking_changes``, ``major_changes``, ``deprecated_features``, ``removed_features``). So if you use option 3, you will not be able to add something to the porting guide.

Versioning and deprecation
~~~~~~~~~~~~~~~~~~~~~~~~~~

* To preserve backward compatibility for users, every ansible minor version series (2.10.x) will keep the major version of a collection constant. If ansible 2.10.0 includes ``community.general`` 1.2.0, then each 2.10.x release will include the latest ``community.general`` 1.y.z release available at build time. Ansible 2.10.x will **never** include a ``community.general`` 2.y.x release, even if it is available. Major collection version changes will be included in the next ansible minor release (2.11.0, 2.12.0, and so on).
* Therefore, please make sure that the current major release of your collection included in 2.10.0 receives at least bugfixes as long new 2.10.x releases are produced.
* Since new minor releases are included, you can include new features, modules and plugins. You must make sure that you do not break backwards compatibility! (See `semantic versioning <https://semver.org/>`_.) This means in particular:

  * You can fix bugs in patch releases, but not add new features or deprecate things.
  * You can add new features and deprecate things in minor releases, but not remove things or change behavior of existing features.
  * You can only remove things or make breaking changes in major releases.
* We recommend to make sure that if a deprecation is added in a collection version that is included in 2.10.x, but not in 2.10.0, that the removal itself will only happen in a collection version included in 2.12.0 or later, but not in a collection version included in 2.11.0.
* Content moved from ansible/ansible that was scheduled for removal in 2.11 or later MUST NOT be removed in the current major release  available when ansible 2.10.0 is released. Otherwise it would already be removed in 2.10, unexpectedly for users! Deprecation cycles can be shortened (since they are now uncoupled from ansible or ansible-base versions), but existing ones must not be unexpectedly terminated.
* We recommend to announce your policy of releasing, versioning and deprecation to contributors and users in some way. For an example of how to do this, see `the announcement in community.general <https://github.com/ansible-collections/community.general/issues/582>`_. You could also do this in the README.


Naming
======

For collections under ansible-collections the repository SHOULD be named ``NAMESPACE.COLLECTION``.

`Namespace limitations <https://galaxy.ansible.com/docs/contributing/namespaces.html#galaxy-namespace-limitations>`_  lists requirements for namespaces in Galaxy.

For collections created for working with a particular entity, they should contain the entity name, for example ``community.mysql``.

For corporate maintained collections, the repository can be named ``COMPANY_NAME.PRODUCT_NAME``, for example ``ibm.db2``.

We should avoid FQCN / repository names:

* which are unnecessary long: try to make it compact but clear
* contain the same words / collocations in ``NAMESPACE`` and ``COLLECTION`` parts, for example ``my_system.my_system``


Repository management
=====================

Branch name and configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All new repositories under `ansible-collections <https://github.com/ansible-collections>`_ MUST have ``main`` as the default branch.

Existing repositories SHOULD be converted to use ``main``

Repository Protections:

* Allow merge commits: disallowed

Branch protections MUST be enforced:

* Require linear history
* Include administrators

CI Testing
===========

At a minimum ``ansible-test sanity`` MUST be run from the `latest stable ansible-base branch <https://github.com/ansible/ansible/branches/all?query=stable->`_. We suggest to *additionally* run ``ansible-test sanity`` from the ansible/ansible ``devel`` branch so that you find out about new linting requirements earlier.

For most repository GitHub actions are sufficient, see `example <https://github.com/ansible-collections/collection_template/tree/main/.github/workflows>`_

FIXME to write a guide "How to write CI tests" (from scratch / add to existing) and put the reference here

Unit Testing
============


Collections and Working Groups
==============================

* Working group page(s) on a corresponding wiki (if needed. Makes sense if there is a group of modules for working with one common entity, e.g. postgresql, zabbix, grafana, etc.)
* Issue for agenda (or pinboard if there aren't regular meetings) as pinned issue in the repository

When moving modules between collections
=======================================

All related entities must be moved / copied including:

* related plugins/module_utils/ files (moving be sure it is not used by other modules, otherwise copy)
* CI and unit tests
* corresponding documentation fragments from plugins/doc_fragments

Also:

* change M(), examples, seealso, extended_documentation_fragments to use actual FQCNs (in moved content and in other collections that have references to the content)
* move all related issues / pull requests / wiki pages

See `Migrating content to a different collection <https://docs.ansible.com/ansible/devel/dev_guide/developing_collections.html#migrating-ansible-content-to-a-different-collection>`_ for complete details.


Other things
============

* ansible-base's runtime.yml
