..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Enable Rebuild for Instances in cell0
=====================================
https://blueprints.launchpad.net/nova/+spec/something

This spec summarizes the changes needed for enabling the rebuilding of
instances that failed during creation.

Problem Description
===================

Currently the rebuild procedure is not supported for VMs that have never
booted before. If a user tries to rebuild an instance in cell0, the operation
fails with an InstanceInvalidState exception raised by the Compute API.

Use Cases
---------

As an operator I want to enable an external service to take actions after a
failed build request, such as removing preemptible resources. After the
succesful completion of the follow up actions, the service should be able to
issue a rebuild request for the server that failed in the first place.

Proposed Change
===============

In order to support the rebuild of instances that mapped to cell0, we need the
following changes:

?? Change the rebuild method to have the image parameter as optional.

#. Allow rebuild for instances that haver never booted.

#. Delete all the info from cell0 database, regarding the instance being
   rebuilt

#. When an instance fails during boot and before network allocations, its
   network_info field is empty. Currently there is no way to recover the
   NetworkRequest objects while trying to rebuild it. So we need to store the
   requested networks at the time of failure.

#. Create a new BuildRequest for the failed instance.

#. Update the extisting cell_mapping to not point to cell0.

#. Create copies of the info stored in cell0 regarding the instance.

#. Implement a `rebuild_instance_in_cell0` method in the conductor API that
   actually calls the `schedule_and_build_instnances` internally.

Alternatives
------------

None

Security Impact
---------------

None

Notifications Impact
--------------------

None

Other End User Impact
---------------------

The end user will be able to rebuild instances that failed to boot in the
first place.

Performance Impact
------------------

None

Other Deployer Impact
---------------------

Deployers may need to create a new project hierarchy to make use of this new
limit feature.

Developer Impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ttsiouts

Other contributors:
  johnthetubaguy
  strigazi
  belmoreira

Work Items
----------

See `Proposed change`_.

Dependencies
============

None

Documentation Impact
====================

We need to update the documentaion regarding rebuild operations.

References
==========

Discussed at the Dublin PTG:
#. https://etherpad.openstack.org/p/nova-ptg-rocky (Preemptible Instances)

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Rocky
     - Introduced
