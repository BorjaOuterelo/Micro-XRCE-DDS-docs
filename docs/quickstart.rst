.. _quickstart_label:

Quick start
===========
*Prosima Micro XRCE-DDS* provides a C API which allows the creation of *eProsima Micro XRCE-DDS Clients* that publish and/or subscribe to topics from DDS Global Data Space.
The following example shows how to create a simple *eProsima Micro XRCE-DDS Client* and *eProsima Micro XRCE-DDS Agent* for publishing and subscribing to the DDS world, using this HelloWorld.idl: ::

    struct HelloWorld
    {
        unsigned long index;
        string message;
    };

First of all, we launch the `Agent`. For this example, the `Client` - `Agent` communication will be done through UDP: ::

    $ cd /usr/local/bin && MicroXRCEAgent udp -p 2019 -r <references-file>

Along with the `Agent`, the `PublishHelloWorldClient` example provided in the source code is launched.
This `Client` example will publish in the DDS World the HelloWorld topic. ::

    $ examples/uxr/client/PublishHelloWorld/PublishHelloWorldClient 127.0.0.1 2019

The code of the *PublishHelloWorldClient* is the following:

.. code-block:: C

    // Copyright 2019 Proyectos y Sistemas de Mantenimiento SL (eProsima).
    //
    // Licensed under the Apache License, Version 2.0 (the "License");
    // you may not use this file except in compliance with the License.
    // You may obtain a copy of the License at
    //
    //     http://www.apache.org/licenses/LICENSE-2.0
    //
    // Unless required by applicable law or agreed to in writing, software
    // distributed under the License is distributed on an "AS IS" BASIS,
    // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    // See the License for the specific language governing permissions and
    // limitations under the License.

    #include "HelloWorld.h"

    #include <uxr/client/client.h>
    #include <ucdr/microcdr.h>

    #include <stdio.h>
    #include <string.h> //strcmp
    #include <stdlib.h> //atoi

    #define STREAM_HISTORY  8
    #define BUFFER_SIZE     UXR_CONFIG_UDP_TRANSPORT_MTU * STREAM_HISTORY

    int main(int args, char** argv)
    {
        // CLI
        if(3 > args || 0 == atoi(argv[2]))
        {
            printf("usage: program [-h | --help] | ip port [<topics>]\n");
            return 0;
        }

        char* ip = argv[1];
        uint16_t port = (uint16_t)atoi(argv[2]);
        uint32_t max_topics = (args == 4) ? (uint32_t)atoi(argv[3]) : UINT32_MAX;

        // Transport
        uxrUDPTransport transport;
        uxrUDPPlatform udp_platform;
        if(!uxr_init_udp_transport(&transport, &udp_platform, ip, port))
        {
            printf("Error at create transport.\n");
            return 1;
        }

        // Session
        uxrSession session;
        uxr_init_session(&session, &transport.comm, 0xAAAABBBB);
        if(!uxr_create_session(&session))
        {
            printf("Error at create session.\n");
            return 1;
        }

        // Streams
        uint8_t output_reliable_stream_buffer[BUFFER_SIZE];
        uxrStreamId reliable_out = uxr_create_output_reliable_stream(&session, output_reliable_stream_buffer, BUFFER_SIZE, STREAM_HISTORY);

        uint8_t input_reliable_stream_buffer[BUFFER_SIZE];
        uxr_create_input_reliable_stream(&session, input_reliable_stream_buffer, BUFFER_SIZE, STREAM_HISTORY);

        // Create entities
        uxrObjectId participant_id = uxr_object_id(0x01, UXR_PARTICIPANT_ID);
        const char* participant_xml = "<dds>"
                                          "<participant>"
                                              "<rtps>"
                                                  "<name>default_xrce_participant</name>"
                                              "</rtps>"
                                          "</participant>"
                                      "</dds>";
        uint16_t participant_req = uxr_buffer_create_participant_xml(&session, reliable_out, participant_id, 0, participant_xml, UXR_REPLACE);

        uxrObjectId topic_id = uxr_object_id(0x01, UXR_TOPIC_ID);
        const char* topic_xml = "<dds>"
                                    "<topic>"
                                        "<name>HelloWorldTopic</name>"
                                        "<dataType>HelloWorld</dataType>"
                                    "</topic>"
                                "</dds>";
        uint16_t topic_req = uxr_buffer_create_topic_xml(&session, reliable_out, topic_id, participant_id, topic_xml, UXR_REPLACE);

        uxrObjectId publisher_id = uxr_object_id(0x01, UXR_PUBLISHER_ID);
        const char* publisher_xml = "";
        uint16_t publisher_req = uxr_buffer_create_publisher_xml(&session, reliable_out, publisher_id, participant_id, publisher_xml, UXR_REPLACE);

        uxrObjectId datawriter_id = uxr_object_id(0x01, UXR_DATAWRITER_ID);
        const char* datawriter_xml = "<dds>"
                                         "<data_writer>"
                                             "<topic>"
                                                 "<kind>NO_KEY</kind>"
                                                 "<name>HelloWorldTopic</name>"
                                                 "<dataType>HelloWorld</dataType>"
                                             "</topic>"
                                         "</data_writer>"
                                     "</dds>";
        uint16_t datawriter_req = uxr_buffer_create_datawriter_xml(&session, reliable_out, datawriter_id, publisher_id, datawriter_xml, UXR_REPLACE);

        // Send create entities message and wait its status
        uint8_t status[4];
        uint16_t requests[4] = {participant_req, topic_req, publisher_req, datawriter_req};
        if(!uxr_run_session_until_all_status(&session, 1000, requests, status, 4))
        {
            printf("Error at create entities: participant: %i topic: %i publisher: %i darawriter: %i\n", status[0], status[1], status[2], status[3]);
            return 1;
        }

        // Write topics
        bool connected = true;
        uint32_t count = 0;
        while(connected && count < max_topics)
        {
            HelloWorld topic = {count++, "Hello DDS world!"};

            ucdrBuffer mb;
            uint32_t topic_size = HelloWorld_size_of_topic(&topic, 0);
            uxr_prepare_output_stream(&session, reliable_out, datawriter_id, &mb, topic_size);
            HelloWorld_serialize_topic(&mb, &topic);

            connected = uxr_run_session_time(&session, 1000);
            if(connected)
            {
                printf("Sent topic: %s, id: %i\n", topic.message, topic.index);
            }
        }

        // Delete resources
        uxr_delete_session(&session);
        uxr_close_udp_transport(&transport);

        return 0;
    }

After it, we will launch the *SubscriberHelloWorldClient*. This `Client` example will subscribe to HelloWorld topic from the DDS World. ::

    $ examples/uxr/client/SubscriberHelloWorld/SubscribeHelloWorldClient 127.0.0.1 2019

The code of the *SubscriberHelloWorldClient* is the following:

.. code-block:: C

    // Copyright 2019 Proyectos y Sistemas de Mantenimiento SL (eProsima).
    //
    // Licensed under the Apache License, Version 2.0 (the "License");
    // you may not use this file except in compliance with the License.
    // You may obtain a copy of the License at
    //
    //     http://www.apache.org/licenses/LICENSE-2.0
    //
    // Unless required by applicable law or agreed to in writing, software
    // distributed under the License is distributed on an "AS IS" BASIS,
    // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    // See the License for the specific language governing permissions and
    // limitations under the License.

    #include "HelloWorld.h"

    #include <uxr/client/client.h>
    #include <string.h> //strcmp
    #include <stdlib.h> //atoi
    #include <stdio.h>

    #define STREAM_HISTORY  8
    #define BUFFER_SIZE     UXR_CONFIG_UDP_TRANSPORT_MTU * STREAM_HISTORY

    void on_topic(uxrSession* session, uxrObjectId object_id, uint16_t request_id, uxrStreamId stream_id, struct ucdrBuffer* mb, void* args)
    {
        (void) session; (void) object_id; (void) request_id; (void) stream_id;

        HelloWorld topic;
        HelloWorld_deserialize_topic(mb, &topic);

        printf("Received topic: %s, id: %i\n", topic.message, topic.index);

        uint32_t* count_ptr = (uint32_t*) args;
        (*count_ptr)++;
    }

    int main(int args, char** argv)
    {
        // CLI
        if(3 > args || 0 == atoi(argv[2]))
        {
            printf("usage: program [-h | --help] | ip port [<topics>]\n");
            return 0;
        }

        char* ip = argv[1];
        uint16_t port = (uint16_t)atoi(argv[2]);
        uint32_t max_topics = (args == 4) ? (uint32_t)atoi(argv[3]) : UINT32_MAX;

        // State
        uint32_t count = 0;

        // Transport
        uxrUDPTransport transport;
        uxrUDPPlatform udp_platform;
        if(!uxr_init_udp_transport(&transport, &udp_platform, ip, port))
        {
            printf("Error at create transport.\n");
            return 1;
        }

        // Session
        uxrSession session;
        uxr_init_session(&session, &transport.comm, 0xCCCCDDDD);
        uxr_set_topic_callback(&session, on_topic, &count);
        if(!uxr_create_session(&session))
        {
            printf("Error at create session.\n");
            return 1;
        }

        // Streams
        uint8_t output_reliable_stream_buffer[BUFFER_SIZE];
        uxrStreamId reliable_out = uxr_create_output_reliable_stream(&session, output_reliable_stream_buffer, BUFFER_SIZE, STREAM_HISTORY);

        uint8_t input_reliable_stream_buffer[BUFFER_SIZE];
        uxrStreamId reliable_in = uxr_create_input_reliable_stream(&session, input_reliable_stream_buffer, BUFFER_SIZE, STREAM_HISTORY);

        // Create entities
        uxrObjectId participant_id = uxr_object_id(0x01, UXR_PARTICIPANT_ID);
        const char* participant_xml = "<dds>"
                                          "<participant>"
                                              "<rtps>"
                                                  "<name>default_xrce_participant</name>"
                                              "</rtps>"
                                          "</participant>"
                                      "</dds>";
        uint16_t participant_req = uxr_buffer_create_participant_xml(&session, reliable_out, participant_id, 0, participant_xml, UXR_REPLACE);

        uxrObjectId topic_id = uxr_object_id(0x01, UXR_TOPIC_ID);
        const char* topic_xml = "<dds>"
                                    "<topic>"
                                        "<name>HelloWorldTopic</name>"
                                        "<dataType>HelloWorld</dataType>"
                                    "</topic>"
                                "</dds>";
        uint16_t topic_req = uxr_buffer_create_topic_xml(&session, reliable_out, topic_id, participant_id, topic_xml, UXR_REPLACE);

        uxrObjectId subscriber_id = uxr_object_id(0x01, UXR_SUBSCRIBER_ID);
        const char* subscriber_xml = "";
        uint16_t subscriber_req = uxr_buffer_create_subscriber_xml(&session, reliable_out, subscriber_id, participant_id, subscriber_xml, UXR_REPLACE);

        uxrObjectId datareader_id = uxr_object_id(0x01, UXR_DATAREADER_ID);
        const char* datareader_xml = "<dds>"
                                         "<data_reader>"
                                             "<topic>"
                                                 "<kind>NO_KEY</kind>"
                                                 "<name>HelloWorldTopic</name>"
                                                 "<dataType>HelloWorld</dataType>"
                                             "</topic>"
                                         "</data_reader>"
                                     "</dds>";
        uint16_t datareader_req = uxr_buffer_create_datareader_xml(&session, reliable_out, datareader_id, subscriber_id, datareader_xml, UXR_REPLACE);

        // Send create entities message and wait its status
        uint8_t status[4];
        uint16_t requests[4] = {participant_req, topic_req, subscriber_req, datareader_req};
        if(!uxr_run_session_until_all_status(&session, 1000, requests, status, 4))
        {
            printf("Error at create entities: participant: %i topic: %i subscriber: %i datareader: %i\n", status[0], status[1], status[2], status[3]);
            return 1;
        }

        // Request topics
        uxrDeliveryControl delivery_control = {0};
        delivery_control.max_samples = UXR_MAX_SAMPLES_UNLIMITED;
        uint16_t read_data_req = uxr_buffer_request_data(&session, reliable_out, datareader_id, reliable_in, &delivery_control);

        // Read topics
        bool connected = true;
        while(connected && count < max_topics)
        {
            uint8_t read_data_status;
            connected = uxr_run_session_until_all_status(&session, UXR_TIMEOUT_INF, &read_data_req, &read_data_status, 1);
        }

        // Delete resources
        uxr_delete_session(&session);
        uxr_close_udp_transport(&transport);

        return 0;
    }

At this moment, the subscriber will receive the topics that are sending by the publisher.

In order to see the messages from the DDS Global Data Space point of view, you can use *eProsima Fast RTPS* HelloWorld example running a subscriber
(`Fast RTPS HelloWorld <http://eprosima-fast-rtps.readthedocs.io/en/latest/introduction.html#building-your-first-application>`_): ::

    $ cd /usr/local/examples/C++/HelloWorldExample
    $ sudo make && cd bin
    $ ./HelloWorldExample subscriber

Learn More
----------

To learn more about DDS and *eProsima Fast RTPS*: `eProsima Fast RTPS <http://eprosima-fast-rtps.readthedocs.io>`_

To learn how to install *eProsima Micro XRCE-DDS* read: :ref:`installation_label`

To learn more about *eProsima Micro XRCE-DDS* read: :ref:`user`

To learn more about *eProsima Micro XRCE-DDS Gen* read: :ref:`microxrceddsgen_label`

