..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Introduce Pending VM state
==========================
https://blueprints.launchpad.net/nova/+spec/something

This feature adds support for the PENDING server state. Instead of setting a
server to the ERROR state when the Placement returns no valid hosts for the
requested server, the PENDING state is used if the operator wishes so. This
will allow the execution of subsequent actions transparently to the end user.

Problem description
===================

Use Cases
---------

As an operator, I want to provide preemptible servers using an external -to
Nova- service. As soon as a server's build request fails due to NoValidHosts,
the external service will try to free up the requested resources. In order to
achieve that, transparently to the user, the instance should not be set to the
ERROR state but to the new PENDING state.

Proposed change
===============

#. Add the PENDING state in the InstanceState object.

#. Add the PENDING state in compute vm_states.

#. Add the PENDING state in the server ViewBuilder as a new progress status.

#. Add a configuration option that defaults to ``False`` in the conductor to
   enable the use of PENDING vm_state on NoValidHost events::

        CONF.conductor.use_pending_state

#. Add the following code in the conductor manager ``_bury_in_cell0`` method
   to make sure that the a vm is set to PENDING only when the operator has
   chosen so and the failure reported by the scheduler is a NoValidHost::

        verify = isinstance(exc, exception.NoValidHost)

        if CONF.conductor.use_pending_state and verify:
            vm_state = vm_states.PENDING
        else:
            vm_state = vm_states.ERROR

        updates = {'vm_state': vm_state, 'task_state': None}

#. Add an `obj_make_compatible` method in the instance object, mapping the
   PENDING state to ERROR.

#. Add a new vensioned notification ``nova.error.build_failed`` (naming?).

#. Emit the new versioned from the ``_bury_in_cell0`` method.

Alternatives
------------

Follow the vendor data example and perform an asynchronous REST API call from
the Nova Conductor to the external service when enabled by the operator.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

The end user will have servers in PENDING state if the operator chooses to
enable the configuration option in the Nova conductor.

Performance Impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
----------------

None

Upgrade impact
--------------

In previous versions the 'pending' state would lead to UNKNOWN shown
to the user... Any idea on how to handle?

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

Testing
=======

Updating existing unit and functional tests should be enough.

Documentation Impact
====================

#. The new configuration option as well as the meaning of the PENDING state
   should be documented.

#. Update the state transitions documentation to include::

        BUILD to PENDING
        PENDING to REBUILD (maybe in the other spec?)
        PENDING to ERROR (maybe in the other spec?)

References
==========
#. https://etherpad.openstack.org/p/nova-ptg-rocky (topic Preemptible Servers)

History
=======

.. list-table:: Revisions
   :header-rows: 1

   * - Release Name
     - Description
   * - Rocky
     - Introduced
