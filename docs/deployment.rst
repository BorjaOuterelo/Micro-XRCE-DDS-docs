.. _deployment_label:

Deployment example
==================

This part will show how to deploy a system using *eProsima Micro XRCE-DDS* in a real environment.
An example of this can be found into ``examples/Deployment`` folder.

Previous tutorials are based in `all in one` examples, that is, examples that create entities, publish or subscribe and then delete the resources.
One possible real purpose of this, consists in differentiate the logic of `creating entities` and the actions of `publishing and subscribing`.
This can be done creating two differents *Clients*.
One in charge of configure the entities in the *Agent*, and run possibly once, only for creating the entities at configuration time.
And other/s that logs in the same session as the configured *Client* (sharing the entities) and only publishes or subscribes data.

This way allows to easily create *Clients* in a real scenario only with the purpose of send and receive data.
Related to it, the concept of `profile` allows to build the *Client* library only with the chosen behavior (only publish or only subscribe, for example).
See :ref:`micro_xrce_dds_client_label` for more information about this.

Next diagram shows an example about how to configure the environment using a `configurator client`.

Initial state
-------------

    .. image:: images/deployment_0.svg
        :width: 600 px
        :align: center

The environment contains two *Agents* (is perfectly possible to use only one *Agent* too), and two *Clients*, one for publishing and another for subscribing.


Configurate the publisher
-------------------------

    .. image:: images/deployment_1.svg
        :width: 600 px
        :align: center

In this state a `configurator client` is connected to the *Agent* `A` with the `client key` that will be used by the future `publisher client` (0xAABBCCDD).
Once a session is logged in, the `configurator client` creates all the necessary entities for the `publisher client`.
This implies the creation of `participant`, `topic`, `publisher`, and `datawriter` entities.
These entities have a representation as DDS entities, and can be reached now from the DDS world.
That implies that a possible `subscriber DDS entity` could already be listening topics if it matches with a `publisher DDS entity` through `DDS` world.

Publish
-------
    .. image:: images/deployment_2.svg
        :width: 600 px
        :align: center

Then, the `publisher client` is connected to the *Agent* `A`.
This *Client* logs in session with its *Client* key (0xAABBCCDD).
At that moment, it can use all entities created related to this `client key`.
Because all entities that it used were successfully created by the `configurator client`, the `publisher client` can immediately publish to `DDS`.


Configurate the subscriber
--------------------------

    .. image:: images/deployment_3.svg
        :width: 600 px
        :align: center

Again, the `configurator client` connects and logs in, this time to *Agent* `B`, now with the subscriber's key (0x11223344).
In this case, the entities that the `configurator client` creates are a `participant`, a `topic`, a `subscriber`, and a `datareader`.
The entities created by the `configuraton client` will be available until the session is deleted.

Subscriber
----------

    .. image:: images/deployment_4.svg
        :width: 600 px
        :align: center

Once the subscriber is configured, the `subscriber client` logs in the *Agent* `B`.
As all their entities have been created previously, so it only needs to configure the read after log in.
Once the data request message has been sent, the subscriber will receive the topics from the publisher through `DDS` world.

