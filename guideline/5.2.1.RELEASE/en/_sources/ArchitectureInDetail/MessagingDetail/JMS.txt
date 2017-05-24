JMS(Java Message Service)
==============================

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

.. _JMSOverview:

Overview
--------------------------------------------------------------------------------

This chapter explains how to send and receive messages which use components for JMS linking of JMS API and Spring Framework.


|

.. _JMSOverviewAboutJMS:

What is JMS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| JMS is a standard API for using MOM (Message Oriented Middleware) in Java.
| JMS architecture exchanges messages from client to client through JMS provider.
| Since JMS supports asynchronous messaging, a close coupling can be formed between the clients.
| Further, since the message can be stored in Queue by adopting a Point-to-Point model described later, messages can be received in accordance with the client performance.
| On the other hand, since the time lag is likely to occur in the messaging between clients, it is not suitable for the processes which require a real time response.
| For details of JMS, refer \ `Java Message Service (JMS) <http://www.oracle.com/technetwork/java/index-jsp-142945.html>`_\ .
|
| Asynchronous and synchronous messaging can be performed by using JMS.

.. note::

    Use of JMS1.1 is assumed in this guideline.

|

| While using, the delivery method and the method to send and receive messages described below must be selected in accordance with the requirements.


* **Delivery model**

 | Delivery model consists of two models - Point-to-Point (PTP) and Publisher-Subscriber (Pub/Sub).
 | A major difference between two models is in whether the sender and recipient ratio is 1:1 or 1:many, and should be selected based on the use.

 (1) Point-To-Point (PTP) model


  .. figure:: ./images_JMS/JMSQueue.png
     :alt: JMS Queue
     :width: 70%

  | PTP model consists of 2 clients wherein one of the clients (Producer) sends a message and the other client (Consumer) alone receives that message.
  | Destination of the message in PTP model is called as a Queue.
  | Producer sends the message to the Queue and the Consumer fetches the message from the Queue, and the process is carried out.
  | Message is fetched from the consumer or when the message reaches expiry period, the message is deleted from the Queue.
  |


 (2) Publisher-Subscriber (Pub/Sub) model

  .. figure:: ./images_JMS/JMSTopic.png
    :alt: JMS Topic
    :width: 70%

  | Pub/Sub model consists of 2 clients wherein one of the clients (Publisher) issues (Publishes) the message and delivers that message to multiple other clients (Subscribers).
  | Destination of the message in Pub/Sub model is called as a Topic.
  | Subscriber raises a subscription request for the Topic and the Publisher issues a message in Topic.
  | The message is delivered to all the subscribers who have raised a request for subscription.

 | **This guideline explains how to implement PTP model which is widely used in general.**


* **A method to send messages**

 | There are two types of processes for message sending - Synchronous sending method and asynchronous sending method for sending the messages to Queue or Topic however JMS1.1 supports only synchronous sending method.

 (1) Synchronous sending method
 
  | Message is processed and sent by explicitly calling a function to send messages.
  | Subsequent processing is blocked in order to wait until the response is received from JMS provider.
  |

 (2) Asynchronous sending method
 
  | Message is processed and sent by explicitly calling a function to send messages.
  | Subsequent processing is continued since it is not necessary to wait for the response from JMS provider.
  | For the details of asynchronous sending method, refer \ `Java Message Service(Version 2.0) <http://download.oracle.com/otndocs/jcp/jms-2_0-fr-eval-spec/>`_\ "7.3. Asynchronous send".



* **Message receiving method**

 | There are two types of methods for receiving messages - synchronous receiving method and asynchronous receiving method while implementing the process for receiving messages in Queue or Topic.
 | As described later, since use cases of synchronous receiving methods are limited, generally asynchronous receiving methods are widely used.
 

 (1) Asynchronous receiving method
 
  | When the message is received by Queue or Topic, the processing for the received message is initiated.
  | Since the processing for one message is initiated without terminating the processing for the other message, it is suitable for parallel processing.
  |

 (2) Synchronous receiving method
 
  | Receiving the message and related processing is initiated by explicitly calling the function which receives the message.
  | The function which receives the message waits till the message is received  when the message does not exist in Queue or Topic.
  | Hence, the waiting period for the message must be specified by configuring the timeout value.
  
  | As an example of synchronous receiving of message, it can be used when the message accumulated in the Queue, in the Web application is to be fetched and processed at any time like screen operation etc or
    when the messages are to be processed periodically in a batch.
  | 


| In JMS, the message consists of following parts.
| For details, refer "3. JMS Message Model" of \ `Java Message Service(Version 1.1) <http://download.oracle.com/otndocs/jcp/7195-jms-1.1-fr-spec-oth-JSpec/>`_\.

 .. tabularcolumns:: |p{0.20\linewidth}|p{0.80\linewidth}|
 .. list-table::
  :header-rows: 1
  :widths: 20 80
  
  * - Structure
    - Description
  * - | Header
    - | Control information of message like destination and identifier, extension header (JMSX) of JMS, JMS provider specific header and application specific header are stored for JMS provider and application.
  * - | Property
    - | Control information to be added to header is stored.
  * - | Payload
    - | Message body is stored.
      | As data types, 5 types of message types namely \ ``javax.jms.BytesMessage``\ , \ ``javax.jms.MapMessage``\ , \ ``javax.jms.ObjectMessage``\ , \ ``javax.jms.StreamMessage``\ and \ ``javax.jms.TextMessage``\ are offered.
      | \ ``ObjectMessage``\  is used while sending a JavaBean.
      | In such a case, JavaBean must be shared between the clients.


.. _JMSOverviewAPI:

Using JMS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When the process is to be implemented by using JMS, it can be executed by using JMS API defined in Java EE (hereafter referred to as JMS API).
| However, this guideline assumes the use of JMS linking component of Spring Framework wherein the merits are significant as compared to using JMS API as it is (easy description etc).
| Hence, details of JMS API are not described.
| For details, refer \ `Java API <https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html>`_\ .

 .. note::

   For JMS, Java API is standardized however physical protocols of messages are not standardized.

 .. note::

   Since JMS implementation is incorporated in Java EE server as a standard, it can be used as a default (restricted while using JMS provider incorporated in Java EE server), however JMS must be separately implemented in Java EE server like Apache Tomcat wherein JMS is not incorporated.

|
|

.. _JMSOverviewSpringJMS:

Using JMS which use component of Spring Framework
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Following are provided as the libraries for sending and receiving messages in Spring Framework.

* \ ``spring-jms``\
    | It offers a component for the messaging which use JMS.
    | By using the components included in this library, low level JMS API calling becomes unnecessary and the implementation becomes simplified.
    | \ ``spring-messaging``\  can be used.

* \ ``spring-messaging``\
    | It offers a component to abstract the infrastructure function which is required for creating application of messaging base.
    | It consists of a set of annotations to associate with the message and the method for processing the same.
    | By using the components included in the library, implementation style of messaging can be matched.


| Although implementation can be done only in \ ``spring-jms``\ , implementation method can be combined by using \ ``spring-messaging``\.
| This guideline recommends the use of \ ``spring-messaging``\  as well.

| Here, prior to explaining basic implementation method, it is explained how the messages are sent and received by the components for JMS linking offered by Spring Framework.
| At  first, the components registered in the description are introduced.
| Spring Framework sends and receives messages through JMS API using interfaces and classes shown below.

* \ ``javax.jms.ConnectionFactory``\
    | An interface for creating a connection with JMS provider.
    | It offers a function to create a connection to JMS provider from the application.

* \ ``javax.jms.Destination``\
    | An interface for indicating the address (Queue or topic).

* \ ``javax.jms.MessageProducer``\
    | An interface for sending messages.

* \ ``javax.jms.MessageConsumer``\
    | An interface for receiving messages.

* \ ``javax.jms.Message``\ 
    | An interface which shows the message that retains header and body.
    | Messages are sent and received by implementation class of the interface.

* \ ``org.springframework.messaging.Message``\ 
    | An interface which abstracts messages handled by various messaging systems.
    | It can also be used in JMS.
    | As mentioned earlier, basically \ ``org.springframework.messaging.Message``\  offered by spring-messaging is used to comply with implementation method of messaging.
    | However, \ ``javax.jms.Message``\  is used when it is preferable to use \ ``org.springframework.jms.core.JmsTemplate``\.

* \ ``org.springframework.jms.core.JmsMessagingTemplate``\  and \ ``org.springframework.jms.core.JmsTemplate``\
    | A class which creates templates for generation and release of resources for using JMS API.
    | The implementation can be simplified by using a message sending and message synchronous receiving function.
    | Basically, \ ``JmsMessagingTemplate``\  which can handle \ ``org.springframework.messaging.Message``\  is used.
    | Since \ ``JmsMessagingTemplate``\  wraps \ ``JmsTemplate``\, configuration can be done by using \ ``JmsTemplate``\  property.
    | However, sometimes it is preferable to use a \ ``JmsTemplate``\  as it is. Specific use cases are explained later.

* \ ``org.springframework.jms.listener.DefaultMessageListenerContainer``\
    | \ ``DefaultMessageListenerContainer``\  receives messages from Queue and initiates \ ``MessageListener``\  which processes the received messages.

* \ ``@org.springframework.jms.annotation.JmsListener``\
    | A marker annotation which indicates that it is a method to be handled as \ ``MessageListener``\  of JMS.
    | A \ ``@JmsListener``\  annotation is assigned for a method which performs a process while receiving messages.


.. _JMSOverviewSyncSend:

When messages are sent synchronously
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The flow of the process to send messages synchronously is explained using the figure.

 .. figure:: ./images_JMS/JMSSendOverview.png
    :alt: Send of Spring JMS
    :width: 70%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Execute the process in Service by passing "Destination name for sending" and "payload of messages to be sent" for \ ``JmsMessagingTemplate``\.
        | \ ``JmsMessagingTemplate``\  delegates the process to \ ``JmsTemplate``\.
    * - | (2)
      - | \ ``JmsTemplate``\ fetches \ ``javax.jms.Connection``\  from  \ ``ConnectionFactory``\  fetched through JNDI.
    * - | (3)
      - | \ ``JmsTemplate``\  passes \ ``Destination``\  and messages to ``MessageProducer``\.
        | \ ``MessageProducer``\  is generated from  \ ``javax.jms.Session``\. (\ ``Session``\  is generated from  \ ``Connection``\  fetched in (2).)
        | Further, \ ``Destination``\  is fetched through JNDI based on "Destination name for sending" passed in (1).
    * - | (4)
      - | \ ``MessageProducer``\  sends messages to \ ``Destination``\  for sending.


.. _JMSOverviewAsyncReceive:

When messages are received asynchronously
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The flow of the process to receive messages asynchronously is explained using a figure.

 .. figure:: ./images_JMS/JMSASyncOverview.png
    :alt: ASync of Spring JMS
    :width: 70%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch \ ``Connection``\  from  \ ``ConnectionFactory``\  fetched through JNDI.
    * - | (2)
      - | \ ``DefaultMessageListenerContainer``\  passes \ ``Destination``\  to \ ``MessageConsumer``\.
        | \ ``MessageConsumer``\  is generated from \ ``Session``\. (\ ``Session``\  is generated from \ ``Connection``\  fetched in (1))
        | Further, \ ``Destination``\  is fetched through JNDI based on "Destination name for receiving" specified in \ ``@JmsListener``\  annotation.
    * - | (3)
      - | \ ``MessageConsumer``\  receives messages from \ ``Destination``\.
    * - | (4)
      - | A method (listener method) configured by \ ``@JmsListener``\  annotation in \ ``MessageListener``\  is called using received message as an argument. Listener method is managed by \ ``DefaultMessageListenerContainer``\.
        | 

.. _JMSOverviewSyncReceive:

When the messages are received synchronously
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The flow of the process to receive messages synchronously is explained using a figure.

 .. figure:: ./images_JMS/JMSSyncOverview.png
    :alt: Sync of Spring JMS
    :width: 70%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Pass "Destination name for receiving" in Service for \ ``JmsMessagingTemplate``\.
        | \ ``JmsMessagingTemplate``\  delegates the process to \ ``JmsTemplate``\.
    * - | (2)
      - | \ ``JmsTemplate``\  fetches \ ``Connection``\  from \ ``ConnectionFactory``\  fetched through JNDI.
    * - | (3)
      - | \ ``JmsTemplate``\  passes \ ``Destination``\  and messages in \ ``MessageConsumer``\.
        | \ ``MessageConsumer``\  is generated from \ ``Session``\. (\ ``Session``\  is generated from \ ``Connection``\  fetched in (2).)
        | Further, \ ``Destination``\  is fetched through JNDI based on "Destination name for receiving" passed in (1).
    * - | (4)
      - | \ ``MessageConsumer``\  receives the messages from \ ``Destination``\.
        | Message is returned to the Service through \ ``JmsTemplate``\  or \ ``JmsMessagingTemplate``\.


.. _JMSOverviewAboutProjectConfiguration:

Regarding project configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Recommended configuration of the project while using JMS is explained.
| When serialised JavaBean is sent and received through \ ``ObjectMessage``\, the JavaBean must be shared by sending side and receiving side.
| In such a case, it is recommended to add a model project different from the existing blank project.


* **Sharing model**

 * When client of sending and receiving side does not provide a model

   model project is added and Jar file is distributed in the client for communication.

 * When client of sending and receiving side offer a model

   The model provided is added to library.

 | model project, and, relation between distributed archive file and existing project are as below.

  .. figure:: ./images_JMS/ProjectStructure.png
      :alt: Projects
      :width: 70%
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
  .. list-table::
      :header-rows: 1
      :widths: 10 30 60

      * - Sr. No.
        - Project name
        - Description
      * - | (1)
        - | web project
        - | Place the listener class to receive messages asynchronously.
      * - | (2)
        - | domain project
        - | Place the Service executed from listener class for receiving messages asynchronously.
          | Besides, Repository etc are same as used conventionally.
      * - | (3)
        - | model project or Jar file
        - | A class which is shared between the clients is used among the classes belonging to domain layer.

|


 | Following are implemented to add a model project.
 
  * Creating a model project
  * Adding a dependency from domain project to model project
 
 | For detail addition method, refer \ :ref:`SOAPAppendixAddProject` \  of \ :doc:`../WebServiceDetail/SOAP`\.
   shared by JavaBean in the same way.

.. _JMSHowToUse:

How to use
--------------------------------------------------------------------------------

.. _JMSHowToUseEnviromentSetting:

Settings that are common for both sending and receiving messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This section explains common settings required for sending and receiving messages.

.. _JMSHowToUseDependentLibrary:

Configuration of dependent library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``spring-jms``\  of Spring Framework is added to pom.xml of domain project in order to use component for linking JMS of Spring Framework.

- :file:`[projectName]-domain/pom.xml`

 .. code-block:: xml

    <dependencies>

         <!-- (1) -->
         <dependency>
             <groupId>org.springframework</groupId>
             <artifactId>spring-jms</artifactId>
         </dependency>

     </dependencies>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Add dependencies to \ ``spring-jms``\.
         | Since \ ``spring-jms``\  is dependent on \ ``spring-messaging``\, \ ``spring-messaging``\  is also added as a transitive dependent library.

 | A library of JMS provider is added to pom.xml in addition to \ ``spring-jms``\.
 | For additional examples of libraries for pom.xml, refer :ref:`JMSAppendixSettingsDependsOnJMSProvider`.

 .. note::

   In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.
   The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

|

.. _JMSHowToUseConnectionFactory:

Configuration of \ ``ConnectionFactory``\
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| \ ``ConnectionFactory``\  definition method consists of two methods - a method defined by application server and a method defined by Bean definition file.
| A method defined by application server is selected to make Bean definition file independent of JMS provider unless there is a specific reason to do otherwise.
| This chapter explains a method defined by application server only.
| Configuration for using a JavaBean fetched in Bean definition file through JNDI must be performed in order to use \ ``ConnectionFactory``\  defined in the application server.

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml

    <!-- (1) -->
    <jee:jndi-lookup id="connectionFactory" jndi-name="jms/ConnectionFactory"/>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify JNDI name of \ ``ConnectionFactory``\  offered by application server, in \ ``jndi-name``\  attribute.
        | Since \ ``resource-ref``\  attribute is set to \ ``true``\  by default, it is auto-assigned when no prefix exists for JNDI name (java:comp/env/).


 .. note:: **When ConnectionFactory for which a Bean is defined is used**

    When JNDI is not used, \ ``ConnectionFactory``\  can be used even when a Bean is defined for implementation class of \ ``ConnectionFactory``\.
    In such a case, implementation class of \ ``ConnectionFactory``\  is dependent on JMS provider. For details, refer :ref:`JMSAppendixSettingsDependsOnJMSProvider` "configuration when JNDI is not used".

.. _JMSHowToUseDestinationResolver:

Configuration of \ ``DestinationResolver``\
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Name resolution of Destination consists of 2 methods - Resolution by JNDI and Resolution by JMS provider.
| Resolution by JMS provider is performed by default, however resolution by JNDI is recommended for portability and control if there are no particular issues to address.

| Name resolution of Destination can be performed by JNDI name, by using \ ``org.springframework.jms.support.destination.JndiDestinationResolver``\.
| How to define \ ``JndiDestinationResolver``\  is shown below.

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml
   
    <!-- (1) -->
    <bean id="destinationResolver"
       class="org.springframework.jms.support.destination.JndiDestinationResolver">
       <property name="resourceRef" value="true" /> <!-- (2) -->
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``JndiDestinationResolver``\.
    * - | (2)
      - | When there is no prefix in JNDI name (java:comp/env/), set \ ``true``\ when it is auto-assigned. Default value is \ ``false``\.
        
        .. warning::

           Note that \ ``resource-ref``\  attribute of \ ``<jee:jndi-lookup/>``\  is different from the default value.


.. _JMSHowToUseSyncSendMessage:

A method to synchronously send messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| A method to synchronously send messages from  client (Producer) to JMS provider, for PTP model is explained below.

.. _JMSHowToUseSettingForSyncSend:

Basic synchronous sending
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Process of synchronous sending to JMS provider is achieved by using \ ``JmsMessagingTemplate``\.

| An implementation example wherein object of \ ``Todo``\  class is synchronously sent as a message.
| At first, how to configure \ ``JmsMessagingTemplate``\  is shown below.

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml

    <bean id="cachingConnectionFactory"
       class="org.springframework.jms.connection.CachingConnectionFactory"> <!-- (1) -->
       <property name="targetConnectionFactory" ref="connectionFactory" /> <!-- (2) -->
       <property name="sessionCacheSize" value="1" />  <!-- (3) -->
    </bean>

    <!-- (4) -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
       <property name="connectionFactory" ref="cachingConnectionFactory" />
       <property name="destinationResolver" ref="destinationResolver" />
    </bean>

    <!-- (5) -->
    <bean id="jmsMessagingTemplate" class="org.springframework.jms.core.JmsMessagingTemplate">
        <property name="jmsTemplate" ref="jmsTemplate"/>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``org.springframework.jms.connection.CachingConnectionFactory``\  which performs caching of \ ``Session``\  and \ ``MessageProducer/Consumer``\.
        | JMS provider specific \ ``ConnectionFactory``\  looked up by Bean definition or JNDI name is not used as it is.
          Cache function can be used by wrapping it in \ ``CachingConnectionFactory``\.
    * - | (2)
      - | Specify JMS provider specific \ ``ConnectionFactory``\  which is looked up by Bean definition or JNDI name.
    * - | (3)
      - | Specify cache count for \ ``Session``\. (Default value is 1)
        | Although 1 is specified in this example, number of caches must be changed appropriately corresponding to performance requirements.
        | When the session is required to be continued even after exceeding cache count, a new session is created and destroyed repeatedly without using a cache.
        | This is likely to cause deterioration of process efficiency resulting in performance degradation.
    * - | (4)
      - | Define a Bean for \ ``JmsTemplate``\.
        | \ ``JmsTemplate``\  alternates for a low level API handling (JMS API calling).
        | For attributes that can be configured, refer list of attributes of \ ``JmsTemplate``\  given below.
    * - | (5)
      - | Define a Bean for \ ``JmsMessagingTemplate``\. Inject \ ``JmsTemplate``\  which alternates as a synchronous sending process.


| Attributes of \ ``JmsTemplate``\  for synchronous sending are as below.
| It must be configured as per requirement.

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 20 50 15 10
    :class: longtable

    * - Sr. No.
      - Configuration item
      - Details
      - Mandatory
      - Default value
    * - 1.
      - \ ``connectionFactory``\
      - | Configure \ ``ConnectionFactory``\  to be used.
      - ○
      - Nil (since it is mandatory)
    * - 2.
      - \ ``pubSubDomain``\
      - | Configure for a messaging model.
        | Set \ ``false``\  for PTP (Queue) model and set \ ``true``\  for \ Pub/Sub (Topic).
      - \-
      - \ ``false``\ 
    * - 3.
      - \ ``sessionTransacted``\
      - | Specify whether the transaction control is performed in the session.
        | In this guideline, \ ``false``\  is recommended as a default for using transaction control described later.
      - \-
      - \ ``false``\ 
    * - 4.
      - \ ``messageConverter``\
      - | Specify message converter.
        | Default can be used for the scope described in the guideline.
      - \-
      - \ ``SimpleMessageConverter``\ (\*1) is used.
    * - 5.
      - \ ``destinationResolver``\
      - | Specify DestinationResolver.
        | In this guideline, it is recommended to use \ ``JndiDestinationResolver``\  which performs name resolution in JNDI.
      - \-
      - | \ ``DynamicDestinationResolver``\ (\*2) is used.
        | (When \ ``DynamicDestinationResolver``\  is used, name resolution of Destination is performed by JMS provider.)
    * - 6.
      - \ ``defaultDestination``\
      - | Specify existing Destination.
        | When Destination is not specified explicitly, this Destination is used.
      - \-
      - null (no existing destination)
    * - 7.
      - \ ``deliveryMode``\
      - | Select 1 (NON_PERSISTENT) or 2(PERSISTENT) for delivery mode.
        | 2(PERSISTENT) performs perpetuation of messages.
        | 1(NON_PERSISTENT) does not perform perpetuation of messages.
        | Hence, even though performance shows improvement, messages are likely to get lost if JMS provider is restarted.
        | In this guideline, it is recommended to use 2 (PERSISTENT) for avoiding loss of messages.
        | When this configuration is used, note that \ ``explicitQosEnabled``\  described later must be set to \ ``true``\.
      - \-
      - 2(PERSISTENT)
    * - 8.
      - \ ``priority``\
      - | Set priority for messages. Priority can be set from 0 to 9.
        | Higher the number, higher is the priority.
        | Priority is evaluated when the message is stored in the Queue at the time of synchronous sending. Messages are stored in such a way that messages with high priority are extracted before the messages with low priority.
        | FIFO (First-In-First-Out) system is employed for the messages with identical priorities.
        | When this configuration is used, note that \ ``explicitQosEnabled``\  described later must be set to \ ``true``\.
      - \-
      - 4
    * - 9.
      - \ ``timeToLive``\
      - | Specify validity period of messages in milliseconds.
        | When the message reaches the validity period, JMS provider deletes the message from  Queue
        | When this configuration is used, note that \ ``explicitQosEnabled``\  described later must be set to \ ``true``\.
      - \-
      - 0 (Unrestricted)
    * - 10.
      - \ ``explicitQosEnabled``\
      - | Specify \ ``true``\  when \ ``deliveryMode``\ , \ ``priority``\  and \ ``timeToLive``\  are enabled.
      - \-
      - \ ``false``\
 .. raw:: latex

    \newpage

(\*1)\ ``org.springframework.jms.support.converter.SimpleMessageConverter``\ 

(\*2)\ ``org.springframework.jms.support.destination.DynamicDestinationResolver``\ 

|

| Next, a JavaBean is created for sending.

- :file:`[projectName]-domain/src/main/java/com/example/domain/model/Todo.java`

 .. code-block:: java

    package com.example.domain.model;
    
    import java.io.Serializable;
    
    public class Todo implements Serializable { // (1)
    
        private static final long serialVersionUID = -1L;
    
        // omitted

        private String description;

        // omitted

        private boolean finished;

        // omitted
    
        public String getDescription() {
            return description;
        }
    
        public void setDescription(String description) {
            this.description = description;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Basically, a normal JavaBean can be used, however, a \ ``java.io.Serializable``\  interface must be implemented  for a serialised transmission.


| A process to perform actual synchronous sending is described in the end.
| An implementation example is shown below wherein \ ``Todo``\  object consisting of specified text is synchronously sent to Queue.

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject; 
    import org.springframework.jms.core.JmsMessagingTemplate;
    import org.springframework.stereotype.Service; 
    import com.example.domain.model.Todo;
    
    @Service
    public class TodoServiceImpl implements TodoService {

        @Inject
        JmsMessagingTemplate jmsMessagingTemplate;    // (1)

        @Override
        public void sendMessage(String message) {
       
           Todo todo = new Todo();
           // omitted
           
           jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo);  // (2)
          
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Inject \ ``JmsMessagingTemplate``\.
    * - | (2)
      - | By using \ ``convertAndSend``\  method of \ ``JmsMessagingTemplate``\, JavaBean of argument is converted to implementation class of \ ``org.springframework.messaging.Message``\  interface and the message is synchronously sent for a specified Destination.
        | \ ``org.springframework.jms.support.converter.SimpleMessageConverter``\  is used in the default conversion.
        | When \ ``SimpleMessageConverter``\  is used, the class which implements \ ``javax.jms.Message``\, \ ``java.lang.String``\, \ ``byte array``\, \ ``java.util.Map``\  and \ ``java.io.Serializable``\  interface can be sent.
          
 .. note:: **Exception handling of JMS in business logic**
    
    As covered in \ `JMS (Java Message Service) introduction <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/jms/core/JmsTemplate.html>`_\, exceptions are converted to run-time exceptions in Spring Framework.
    Hence, a run-time exception must be handled while handling JMS exception in business logic.

     .. tabularcolumns:: |p{0.20\linewidth}|p{0.60\linewidth}|p{0.20\linewidth}|
     .. list-table::
        :header-rows: 1
        :widths: 20 60 20

        * - Template class
          - A method to convert exceptions
          - Exception after conversion
        * - | \ ``JmsMessagingTemplate``\ 
          - | \ ``convertJmsException``\  method of \ ``JmsMessagingTemplate``\
          - | \ ``MessagingException``\ (\*1) and its sub-exception
        * - | \ ``JmsTemplate``\ 
          - | \ ``convertJmsAccessException``\  method of \ ``JmsAccessor``\
          - | \ ``JmsException``\ (\*2) and its sub-exception

    (\*1) \ ``org.springframework.messaging.MessagingException``\ 

    (\*2) \ ``org.springframework.jms.JmsException``\ 

|

.. _JMSHowToUseSettingForSendWithHeader:

When message header is to be edited and sent synchronously
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Header attribute can be edited and then sent synchronously by specifying header attribute and value of Key-Value format in \ ``convertAndSend``\  method argument of \ ``JmsMessagingTemplate``\.
For header details, refer \ `javax.jms.Messages  <https://docs.oracle.com/javaee/7/api/javax/jms/Message.html>`_\.
An implementation example wherein \ ``JMSCorrelationID``\  of role which links sending and response messages is specified at the time of synchronous sending.


- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java
 
  package com.example.domain.service.todo;
  
  import java.util.Map;
  import javax.inject.Inject;
  import org.springframework.jms.core.JmsMessagingTemplate;
  import org.springframework.stereotype.Service;
  import org.springframework.jms.support.JmsHeaders;
  import com.example.domain.model.Todo;
  
  @Service
  public class TodoServiceImpl implements TodoService {
  
  @Inject
  JmsMessagingTemplate jmsMessagingTemplate;
  
    public void sendMessageWithCorrelationId(String correlationId) {
    
      Todo todo = new Todo();
      // omitted
      
      Map<String, Object> headers = new HashMap<>();
      headers.put(JmsHeaders.CORRELATION_ID, correlationId);// (1)
      
      jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue",
              todo, headers); // (2)
      
    }
  }
  
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Create header information by configuring header attribute name and its value, for implementation class of \ ``Map``\.
    * - | (2)
      - | Synchronously send the message assigned with header information created in (2) by using \ ``convertAndSend``\  method of \ ``JmsMessagingTemplate``\.

 .. warning:: **Regarding header attributes that can be edited**
 
   Components of header attributes (\ ``JMSDestination``\, \ ``JMSDeliveryMode``\, \ ``JMSExpiration``\, \ ``JMSMessageID``\, \ ``JMSPriority``\, \ ``JMSRedelivered``\ and \ ``JMSTimestamp``\ ) are handled as read-only while converting messages using \ ``SimpleMessageConverter``\  of Spring Framework.
   Hence, even though read-only header attributes are configured as shown in the implementation example, they are not stored in the header of sent messages. (retained as message properties.)
   Among read-only header attributes, \ ``JMSDeliveryMode``\  and \ ``JMSPriority``\  can be configured in \ ``JmsTemplate``\  unit.
   For details, refer list of attributes of \ ``JmsTemplate``\  of :ref:`JMSHowToUseSettingForSyncSend`.



.. _JMSHowToUseSettingForSyncSendTransactionManagement:

Transaction control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| When data consistency is required to be guaranteed, a transaction control function is used.
| An implementation example wherein "Declaration type transaction control" is used is recommended in this guideline.
| For details of "Declaration type transaction control", refer \ :ref:`service_transaction_management` \.
|
| \ ``org.springframework.jms.connection.JmsTransactionManager``\  is used to achieve transaction control.
| Configuration example is shown at first.

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="sendJmsTransactionManager"
       class="org.springframework.jms.connection.JmsTransactionManager">
       <!-- (2) -->  
       <property name="connectionFactory" ref="cachingConnectionFactory" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``JmsTransactionManager``\.
      
        .. note:: **Regarding bean name of TransactionManager**

            When \ ``@Transactional``\  annotation is assigned, a Bean registered by Bean name \ ``transactionManager``\  is used as a default.
            (For details, refer \ :ref:`DomainLayerAppendixTransactionManagement` \)
            
            In Blank project, since \ ``DataSourceTransactionManager``\  is defined by Bean name called \ ``transactionManager``\, Bean is defined with an alias in the configuration above.
            
            Hence, when only one \ ``TransactionManager``\  is used in the application, specification of \ ``transactionManager``\  attribute in \ ``@Transactional``\  annotation can be omitted by using \ ``transactionManager``\  as a Bean name.
       
       
    * - | (2)
      - | Specify \ ``CachingConnectionFactory``\  which controls a transaction.

An implementation example is shown below wherein \ ``Todo``\  object is synchronously sent to Queue by performing transaction control.

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.inject.Inject; 
    import org.springframework.jms.core.JmsMessagingTemplate;
    import org.springframework.stereotype.Service; 
    import org.springframework.transaction.annotation.Transactional; 
    import com.example.domain.model.Todo;
    
    @Service
    @Transactional("sendJmsTransactionManager")  // (1)
    public class TodoServiceImpl implements TodoService {
       @Inject
       JmsMessagingTemplate jmsMessagingTemplate;

       @Override
       public void sendMessage(String message) {
          
          Todo todo = new Todo();
          // omitted
          
          jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo);  // (2)
       }

    }
    
 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Declare transaction boundary by using \ ``@Transactional``\  annotation.
        | Accordingly, transaction starts while starting each method in the class and transaction is committed while terminating a method.
    * - | (2)
      - | Send message synchronously to Queue.
        | However, note that the message is actually sent to Queue within the timing wherein the transaction is committed.

|

In the application wherein DB transaction control is performed, a transaction control policy must be determined after reviewing relation between JMS and DB transaction based on business requirements.

* **When JMS and DB transactions are to be separated and then committed and rolled back**

  | A transaction boundary is declared individually when JMS and DB transactions are to be separated.
  | An implementation example is shown below wherein \ ``sendJmsTransactionManager``\  of \ :ref:`JMSHowToUseSettingForSyncSendTransactionManagement`\  is used in JMS transaction control whereas \ ``transactionManager``\  defined in default configuration of Blank project is used in DB transaction control.
  
 - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TransactionalTodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;

      import javax.inject.Inject; 
      import org.springframework.jms.core.JmsMessagingTemplate;
      import org.springframework.stereotype.Service; 
      import org.springframework.transaction.annotation.Transactional; 
      import com.example.domain.model.Todo;
      
      @Service
      @Transactional("sendJmsTransactionManager")  // (1)
      public class TransactionalTodoServiceImpl implements TransactionalTodoService {
         @Inject
         JmsMessagingTemplate jmsMessagingTemplate;

         @Inject
         TodoService todoService;

         @Override
         public void sendMessage(String message) {
             Todo todo = new Todo();
             // omitted
             
             jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo);

             // omitted
             todoService.update(todo);
         }

      }


 - :file:`src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

   .. code-block:: java

      import org.springframework.stereotype.Service;
      import org.springframework.transaction.annotation.Transactional;
      import com.example.domain.model.Todo;

      @Transactional // (2)
      @Service
      public class TodoServiceImpl implements TodoService {
          
          @Override
          public void update(Todo todo) {
              // omitted
          }
      }


   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Declare transaction boundary of JMS by using \ ``@Transactional``\  annotation.
          | Specify \ ``sendJmsTransactionManager``\  which performs transaction control of JMS.
      * - | (2)
        - | Declare transaction boundary of DB by using \ ``@Transactional``\  annotation.
          | Since value is omitted, Bean name \ ``transactionManager``\  is referred as a default.
          | For details of \ ``@Transactional``\  annotation, refer \ :ref:`service_transaction_management`\  of \ :doc:`../../ImplementationAtEachLayer/DomainLayer`\.

|

* **When JMS and DB transactions are to committed and rolled back together**


  A method which uses global transaction by JTA exists for linking JMS and DB transactions, however "Best Effort 1 Phase Commit" is recommended since overheads are likely to occur for performance, among protocol characteristics. Refer below for details.

  | \ `Distributed transactions in Spring, with and without XA <http://www.javaworld.com/article/2077963/open-source-tools/distributed-transactions-in-spring--with-and-without-xa.html>`_\ 
  | \ `Spring Distributed transactions using Best Effort 1 Phase Commit <http://gharshangupta.blogspot.jp/2015/03/spring-distributed-transactions-using_2.html>`_\ 


  .. warning:: **When transaction process results are not returned in JMS provider due to issues like loss of connection with JMS provider after receiving a message**
    
    When transaction process results are not returned in JMS provider due to issues like loss of connection with JMS provider after receiving a message, transaction handling depends on JMS provider.
    Hence, \ **a design considering loss of received messages**\  must be carried out.
    Especially, when loss of messages is absolutely not permissible, \ **providing a system to compensate for the loss of messages or using a global transaction etc**\  must be adopted.

  | "Best Effort 1 Phase Commit" can be achieved by using \ ``org.springframework.data.transaction.ChainedTransactionManager``\.
  | An implementation example is shown below wherein \ ``sendJmsTransactionManager``\  of \ :ref:`JMSHowToUseSettingForSyncSendTransactionManagement`\  is used in JMS transaction management whereas \ ``transactionManager``\  defined in default configuration of blank project is used in DB transaction management.

  - :file:`xxx-env.xml`

   .. code-block:: xml
     
      <!-- (1) -->
      <bean id="sendChainedTransactionManager" class="org.springframework.data.transaction.ChainedTransactionManager">
          <constructor-arg>
              <list>
                  <!-- (2) -->
                  <ref bean="sendJmsTransactionManager" />
                  <ref bean="transactionManager" />
              </list>
          </constructor-arg>
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90
     
      * - Sr. No.
        - Description
      * - | (1)
        - | Define a Bean for \ ``ChainedTransactionManager``\.
      * - | (2)
        - | Specify JMS and DB transaction manager.
          | Transaction starts in a registered sequence and transaction is committed in the reverse sequence.

  Implementation example using the configuration given above is shown below.


 - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/ChainedTransactionalTodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;

      import javax.inject.Inject; 
      import org.springframework.jms.core.JmsMessagingTemplate;
      import org.springframework.stereotype.Service; 
      import org.springframework.transaction.annotation.Transactional; 
      import com.example.domain.model.Todo;
      
      @Service
      @Transactional("sendChainedTransactionManager")  // (1)
      public class ChainedTransactionalTodoServiceImpl implements ChainedTransactionalTodoService {
         @Inject
         JmsMessagingTemplate jmsMessagingTemplate;

         @Inject
         TodoSharedService todoSharedService;

         @Override
         public void sendMessage(String message) {
             Todo todo = new Todo();
             // omitted
             
             jmsMessagingTemplate.convertAndSend("jms/queue/TodoMessageQueue", todo); // (2)

             // omitted
             todoSharedService.insert(todo); // (3)
         }

      }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Control JMS and DB transactions by specifying \ ``sendChainedTransactionManager``\  in \ ``@Transactional``\  annotation.
          | For details of \ ``@Transactional``\  annotation, refer \ :ref:`service_transaction_management`\  of \ :doc:`../../ImplementationAtEachLayer/DomainLayer`\.
      * - | (2)
        - | Send messages synchronously.
      * - | (3)
        - | Execute process associated with DB access. In this example, SharedService is executed along with DB update.


   .. note::

      If it is necessary to manage multiple transactions together like JMS and DB, for business, a global transaction is considered.
      For global transactions, refer "when transaction control (global transaction control) is necessary for multiple DB (multiple resources)" of \ :ref:`service_enable_transaction_management`\.

|


.. _JMSHowToUseAsyncReceiveMessage:

A method to receive messages asynchronously
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| As described in "message receiving method" of \ :ref:`JMSOverviewAboutJMS`\, asynchronous receiving is generally used for receiving messages.
| Asynchronous receiving can be achieved by registering a listener method assigned by \ ``@JmsListener``\  annotation for \ ``DefaultMessageListenerContainer``\  responsible for asynchronous receiving function
| Roles of listener method which perform s processing at the time of asynchronous receiving are shown below.

#. | **Provide a method to receive messages.**
   | Messages can be received by executing a method assigned by \ ``@JmsListener``\  annotation.
#. | **Call business process.**
   | Business process is not implemented by listener method. It is delegated to Service method.
#. | **Handle exceptions occurring in business logic.**
   | Business exceptions and library exceptions occurred during normal operations are handled.
#. | **Send process results as messages.**
   | In the methods wherein response messages are required to be sent, process results of listener method and business logic should be sent as messages for a specified Destination, by using \ ``org.springframework.jms.listener.adapter.JmsResponse``\.

.. _JMSHowToUseListenerContainer:

Basic asynchronous receiving
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| An asynchronous receiving method using \ ``@JmsListener``\  annotation is explained.
| Following configuration is required for the implementation of asynchronous receiving.

* Define JMS Namespace.
* Enable \ ``@JmsListener``\  annotation.
* Specify \ ``@JmsListener``\  annotation in the method of component managed by DI container.

| Respective detail implementation methods are described below.

- :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

 .. code-block:: xml

    <!-- (1) -->
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jms="http://www.springframework.org/schema/jms"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms.xsd">

        <!-- (2) -->
        <jms:annotation-driven />


        <!-- (3) -->
        <jms:listener-container
            factory-id="jmsListenerContainerFactory"
            destination-resolver="destinationResolver"
            concurrency="1"
            cache="consumer"
            transaction-manager="jmsAsyncReceiveTransactionManager"/>

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.21\linewidth}|p{0.74\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 26 64
    :class: longtable

    * - Sr. No.
      - Attribute name
      - Details
    * - | (1)
      - xmlns:jms
      - | Define JMS Namespace.
        | Specify \ ``http://www.springframework.org/schema/jms``\  as a value.
        | For details of JMS Namespace, refer \ `JMS Namespace Support <http://docs.spring.io/autorepo/docs/spring-framework/4.2.7.RELEASE/spring-framework-reference/html/jms.html#jms-namespace>`_\.
    * - 
      - xsi:schemaLocation
      - | Specify URL of schema.
        | Add \ ``http://www.springframework.org/schema/jms``\  and \ ``http://www.springframework.org/schema/jms/spring-jms.xsd``\  to values.
    * - | (2)
      - \-
      - | Enable JMS related annotation function of \ ``@JmsListener``\  annotation and \ ``@SendTo``\  annotation by using \ ``<jms:annotation-driven />``\.
    * - | (3)
      - \-
      - | Configure \ ``DefaultMessageListenerContainer``\  by assigning parameters to the factory which generate \ ``DefaultMessageListenerContainer``\, by using \ ``<jms:listener-container/>``\.
        | \ ``connection-factory``\  attribute which can specify a Bean of \ ``ConnectionFactory``\  to be used, exists in the \ ``<jms:listener-container/>``\ attribute. Default value of \ ``connection-factory``\  attribute is \ ``connectionFactory``\.
        | In this example, since Bean (Bean name is \ ``connectionFactory``\) of \ ``ConnectionFactory``\  shown in \ :ref:`JMSHowToUseConnectionFactory`\  is used, \ ``connection-factory``\  attribute is omitted.
        | \ ``<jms:listener-container/>``\  also consists of attributes other than introduced here.
        | For details, refer \ `Attributes of the JMS <listener-container> element <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/html/jms.html#jms-namespace-listener-container-tbl>`_\.

        .. warning::

            Since \ ``DefaultMessageListenerContainer``\  is equipped with an independent cache function, \ ``CachingConnectionFactory``\  should not be used in case of asynchronous receiving.
            For details, refer \ `Javadoc of DefaultMessageListenerContainer <http://docs.spring.io/autorepo/docs/spring-framework/4.2.7.RELEASE/javadoc-api/org/springframework/jms/listener/DefaultMessageListenerContainer.html>`_\.
            For above, \ ``ConnectionFactory``\  defined in  \ :ref:`JMSHowToUseConnectionFactory`\  should be specified in \ ``connection-factory``\  attribute of \ ``<jms:listener-container/>``\.

    * - 
      - \ ``concurrency``\
      - | Specify upper limit of parallel number of each listener method managed by \ ``DefaultMessageListenerContainer``\.
        | Default value for \ ``concurrency``\  attribute is 1.
        | Lower limit and upper limit of parallel numbers can also be specified. For example, specify "5-10" when lower limit is 5 and upper limit is 10.
        | When the parallel number of listener method has reached specified upper limit, parallel processing is not done and a "waiting" state is reached.
        | A value should be specified as required.

        .. note::

          When you want to specify parallel number in listener method unit, \ ``concurrency``\  attribute of \ ``@JmsListener``\  annotation can be used.

    * - 
      - \ ``destination-resolver``\
      - | Specify Bean name of \ ``DestinationResolver``\  which is used to resolve Destination name at the time of asynchronous receiving.
        | For Bean definition of \ ``DestinationResolver``\, refer \ :ref:`JMSHowToUseDestinationResolver`\.
        | When \ ``destination-resolver``\  attribute is not specified, \ ``DynamicDestinationResolver``\  generated in \ ``DefaultMessageListenerContainer``\  is used.
    * - 
      - \ ``factory-id``\
      - | Specify name of \ ``DefaultJmsListenerContainerFactory``\  which defines a Bean.
        | Since \ ``@JmsListener``\  annotation refers Bean name \ ``jmsListenerContainerFactory``\  as a default, it is recommended to consider Bean name as \ ``jmsListenerContainerFactory``\  in case of a single \ ``<jms:listener-container/>``\.
    * - 
      - \ ``cache``\
      - | Specify cache level to determine cache targets like \ ``Connection``\, \ ``Session``\  or \ ``Consumer``\  etc.
        | Default is \ ``auto``\.
        | When Connection is not pooled in the application server while configuring \ ``transaction-manager``\  attribute described later, specifying \ ``consumer``\  is recommended for performance improvement.
        
        .. note::
        
           In case of \ ``auto``\, when \ ``transaction-manager``\  attribute is not configured, behaviour is same as \ ``consumer``\ (\ ``Consumer``\  is cached).
           However, behaviour is same as \ ``none``\ (invalid cache) \  considering the pooling in the application server using global transaction at the time of ``transaction-manager``\  attribute configuration.

    * - 
      - \ ``transaction-manager``\
      - | Specify Bean name which performs transaction control at the time of asynchronous receiving. For details, refer \ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\.

 .. raw:: latex

    \newpage



 Messages can be asynchronously received from specified Destination by specifying \ ``@JmsListener``\  annotation in the component method managed by DI container.
 Implementation method is as shown below.


- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    package com.example.listener.todo;

    import org.springframework.jms.annotation.JmsListener;
    import org.springframework.stereotype.Component;
    import com.example.domain.model.Todo; 
    @Component
    public class TodoMessageListener {
   
       @JmsListener(destination = "jms/queue/TodoMessageQueue")   // (1)
       public void receive(Todo todo) {
          // omitted
       }
   
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``@JmsListener``\  annotation for a method, for asynchronous receiving. Specify Destination name for receiving in \ ``destination``\  attribute.
      


 A list of main attributes of \ ``@JmsListener``\  annotation is shown below.
 For details and other attributes, refer \ `Javadoc of @JmsListener annotation <http://docs.spring.io/spring-framework/docs/4.2.7.RELEASE/javadoc-api/org/springframework/jms/annotation/JmsListener.html#destination-->`_\.


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - Sr. No.
      - Fields
      - Details
    * - 1.
      - \ ``destination``\
      - | Specify Destination to be received.
    * - 2.
      - \ ``containerFactory``\
      - | Specify Bean name of \ ``DefaultJmsListenerContainerFactory``\ which manages the listener method.
        | Default is \ ``jmsListenerContainerFactory``\.
    * - 3.
      - \ ``selector``\
      - | Specify a message selector which acts as a condition for restricting the message to be received.
        | When the value is not explicitly specified, default is ""(blank character) and all the messages can be received.
        | For how to use, refer \ :ref:`JMSHowToUseMessageSelectorForAsyncReceive`\.
    * - 3.
      - \ ``concurrency``\
      - | Specify upper limit of parallel numbers for listener method.;
        | Default value for \ ``concurrency``\  attribute is 1.
        | Lower limit and upper limit of parallel numbers can be specified. For example, specify "5-10" when lower limit is 5 and upper limit is 10.
        | When the parallel number of listener method has reached specified upper limit, parallel processing is not done and a "waiting" state is reached.
        | Value should be specified as required.

.. _JMSHowToUseListenerContainerGetHeader:

Fetching header information of messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When the header information of messages is to be used in the listener method such as sending process results of asynchronous receiving to the Destination specified on Producer side (Value of header attribute \ ``JMSReplyTo``\), \ ``@org.springframework.messaging.handler.annotation.Header``\  annotation is used.
| Implementation example is shown below.

- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    @JmsListener(destination = "jms/queue/TodoMessageQueue")
    public JmsResponse<Todo> receiveAndResponse(
            Todo todo, @Header("jms_replyTo") Destination storeResponseMessageQueue) { // (1)

        // omitted

         return JmsResponse.forDestination(todo, storeResponseMessageQueue);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ ``@Header``\  annotation to fetch value of header attribute \ ``JMSReplyTo``\  of receiving message.
        | For a key specified while fetching JMS standard header attribute, refer \ `JmsHeaders constants definition <https://static.javadoc.io/org.springframework/spring-jms/4.2.7.RELEASE/constant-values.html#org.springframework.jms.support.JmsHeaders.CORRELATION_ID>`_\.


.. _JMSHowToUseListenerContainerReSendMessage:

Process results after asynchronous receiving are sent as messages
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| A method which sends the process results of method defined in \ ``@JmsListener``\  annotation, to Destination as a response message.
| Following 2 methods can be listed for specifying sending destination of process results.

* When sending destination of process results is specified statically
* When sending destination of process results is specified dynamically

Respective descriptions are as below.

* **When sending destination of process results is specified statically**
     | Process results can be sent as messages to a fixed destination by defining \ ``@SendTo``\  annotation which specifies a destination, for a method defined by \ ``@JmsListener``\  annotation.
     | Implementation example is as shown below.

 - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

  .. code-block:: java

     @JmsListener(destination = "jms/queue/TodoMessageQueue")
     @SendTo("jms/queue/ResponseMessageQueue") // (1)
     public Todo receiveMessageAndSendTo(Todo todo) {

         // omitted
         return todo; // (2)
     }


  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Default sending destination of process results can be specified by defining \ ``@SendTo``\  annotation.
     * - | (2)
       - | Return data to be sent to Destination defined in \ ``@SendTo``\  annotation.
         | Permissible return value types are the classes which implement \ ``org.springframework.messaging.Message``\ , \ ``javax.jms.Message``\ , \ ``String``\ , \ ``byte``\  array, \ ``Map``\  and \ ``Serializable``\  interface.


* **When sending destination of process results is changed dynamically**

 | When sending destination is to be changed dynamically, \ ``forDestination``\  or \ ``forQueue``\  methods of \ ``JmsResponse``\  class are used
 | Process results can be sent to any Destination by dynamically changing Destination for sending or Destination name. Implementation example is as shown below.
 
 - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

  .. code-block:: java

     @JmsListener(destination = "jms/queue/TodoMessageQueue")
     public JmsResponse<Todo> receiveMessageJmsResponse(Todo todo) {
   
         // omitted
   
         String resQueue = null;
   
         if (todo.isFinished()) {
             resQueue = "jms/queue/FinihedTodoMessageQueue";
         } else {
             resQueue = "jms/queue/ActiveTodoMessageQueue";
         }
   
         return JmsResponse.forQueue(todo, resQueue); // (1)
     }

  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | When Queue for sending is to be changed in accordance with the process details, \ ``forDestination``\  or \ ``forQueue``\  methods of \ ``JmsResponse``\  class are used.
         | In this example, messages are sent from Destination name by using \ ``forQueue``\  method.
         
         .. note::
         
            When \ ``forQueue``\  method of \ ``JmsResponse``\  class is used, Destination name is used as a string.
            For resolving destination name, \ ``DestinationResolver``\  specified in \ ``DefaultMessageListenerContainer``\  is used.


.. note:: **When sending destination of process results is specified on Producer side**

   Using the implementation below, messages of process results can be sent to any destination specified on Producer side.

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Implementation location
         - Implementation details
       * - | Producer side
         - | Specify Destination in header attribute \ ``JMSReplyTo``\  of messages in accordance with JMS standards.
           | For editing of header attribute, refer \ :ref:`JMSHowToUseSettingForSendWithHeader`\.
       * - | Consumer side
         - | Return objects which send a message.

   Header attribute \ ``JMSReplyTo``\  is given priority over the default Destination specified on the Consumer side.
   For details, refer \ `Response management <http://docs.spring.io/spring/docs/4.2.7.RELEASE/spring-framework-reference/htmlsingle/#jms-annotated-response>`_\ .


.. _JMSHowToUseMessageSelectorForAsyncReceive:

When messages which are asynchronously received are to be restricted
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The messages to be received can be restricted by specifying a message selector at the time of receiving.


- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    @JmsListener(destination = "jms/queue/MessageQueue" , selector = "TodoStatus = 'deleted'")    // (1)
    public void receive(Todo todo) {
        // omitted
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Conditions for receiving can be set by using \ ``selector``\  attribute.
        | \ ``TodoStatus``\  of header attribute receives only \ ``deleted``\  messages.
        | Message selector is based on subset of SQL92 conditional expression syntax.
        | For details, refer \ `Message Selectors <http://docs.oracle.com/javaee/7/api/javax/jms/Message.html>`_\.


.. _JMSHowToUseValidationForAsyncReceive:

Input check for the messages which are received asynchronously
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| An input check must be carried out to ensure that the messages which retain invalid data are not processed in the business logic from a security viewpoint.
| Input check is implemented in Service method by using Method Validation and the exception at the time of input check error is handled by the listener method.
| This is done to avoid the occurrence of unnecessary rollback process for the exception occurring at the time of input check error, while performing transaction control. For transaction control, refer \ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\.
| For details of how to configure and implement Method Validation, refer \ :ref:`MethodValidation`\  of \ :doc:`../WebApplicationDetail/Validation`\.
| An implementation example is shown below wherein input check is performed for \ ``Todo``\  object shown in \ :ref:`JMSHowToUseSettingForSyncSend`\.

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;

    import javax.validation.Valid;
    import org.springframework.validation.annotation.Validated;
    import com.example.domain.model.Todo;
    
    @Validated // (1)
    public interface TodoService {

        void updateTodo(@Valid Todo todo); // (2)

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Declare the interface as a target for input check by attaching a \ ``@Validated``\  annotation.
    * - | (2)
      - | Specify a constraint annotation of Bean Validation as an argument for the method.


- :file:`[projectName]-domain/src/main/java/com/example/domain/model/Todo.java`

 .. code-block:: java

    package com.example.domain.model;
    
    import java.io.Serializable;
    import javax.validation.constraints.Null;
    
    // (1)
    public class Todo implements Serializable {
    
        private static final long serialVersionUID = -1L;
    
        // omitted

        @Null
        private String description;

        // omitted

        private boolean finished;

        // omitted
    
        public String getDescription() {
            return description;
        }
    
        public void setDescription(String description) {
            this.description = description;
        }
    
        public boolean isFinished() {
            return finished;
        }
    
        public void setFinished(boolean finished) {
            this.finished = finished;
        }
    
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define input check of JavaBean in Bean Validation.
        | In this example, \ ``@Null``\  annotation is set as an example.
        | For details, refer "\ :doc:`../WebApplicationDetail/Validation`\ ".


- :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`

 .. code-block:: java

    @Inject
    TodoService todoService;

    @JmsListener(destination = "jms/queue/MessageQueue")
    public void receive(Todo todo) {
        try {
            todoService.updateTodo(todo); // (1)
        } catch (ConstraintViolationException e) { // (2) 
            // omitted
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Implement Service method which performs input check.
    * - | (2)
      - | Capture \ ``ConstraintViolationException``\  which occurs at the time of constraint violation.
        | Any process can be performed after capturing the exception.
        | For examples of sending messages to another Queue like using a Queue for storing logical error messages, refer \ :ref:`JMSHowToUseExceptionHandlingForAsyncReceive`\.

|

.. _JMSHowToUseTransactionManagementForAsyncReceive:

Transaction control
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| A transaction control function is used when data consistency is required to be guaranteed.
| Transactions can be controlled at the time of asynchronous message receiving by setting a transaction manager for \ ``<jms:listener-container/>``\.

    .. note:: 

       When message returns to Queue, it is asynchronously received once again. Since the cause of error is not resolved, the operations of rollback and asynchronous receiving are repeated.
       A threshold value for number of resends after rollback can be set depending on JMS provider and when resend count exceeds the threshold value, the message is stored in Dead Letter Queue.


How to configure is shown below.

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="jmsAsyncReceiveTransactionManager"
       class="org.springframework.jms.connection.JmsTransactionManager">
       <!-- (2) -->  
       <property name="connectionFactory" ref="connectionFactory" />
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``JmsTransactionManager``\  of asynchronous receiving.
    * - | (2)
      - | Specify \ ``ConnectionFactory``\  which manages a transaction. Note that, \ ``CachingConnectionFactory``\  cannot be used at the time of asynchronous receiving.


- :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

 .. code-block:: xml

    <!-- (1) -->
    <jms:listener-container
        factory-id="jmsListenerContainerFactory"
        destination-resolver="destinationResolver"
        concurrency="1"
        error-handler="jmsErrorHandler"
        cache="consumer"
        transaction-manager="jmsAsyncReceiveTransactionManager"/>

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.26\linewidth}|p{0.69\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 26 64

    * - Sr. No.
      - Attribute name
      - Details
    * - | (1)
      - \ ``cache``\ 
      - | Specify cache level to determine cache targets such as \ ``Connection``\, \ ``Session``\  or \ ``Consumer``\.
        | Default is \ ``auto``\.
        | As described earlier in \ :ref:`JMSHowToUseListenerContainer`\, \ ``consumer``\  is specified when connection is not pooled in the application server.
    * - 
      - \ ``transaction-manager``\ 
      - | Specify Bean name for \ ``JmsTransactionManager``\  to be used.
        | Note that, \ ``JmsTransactionManager``\  does not manage \ ``CachingConnectionFactory``\.
        
 .. warning:: 
 
    Since \ ``Connection``\  or \ ``Session``\  caching in the application is likely to be prohibited depending on the application server, cache validation and invalidation should be determined in accordance with the specifications of application server to be used.

|

.. note:: **A method which performs exception handling other than roll back process in case of specific exceptions** 

   When transaction control is enabled, the message returns to Queue due to roll back if the exception occurred in input check is thrown without getting captured.
   Since listener method asynchronously receives the message again which has returned in Queue, the sequence asynchronous receiving Error occurrence Rollback is repeated a number of times for JMS provider configuration.
   In case of an error for which the cause of the error is not resolved even after retry, the error handling is done so as not to throw an exception from listener method after capturing, to restrain futile processes mentioned above.
   For details, refer \ :ref:`JMSHowToUseExceptionHandlingForAsyncReceive`\.


In the application wherein DB transaction control is required, a transaction control policy must be determined after reviewing the relation between JMS and DB transactions based on business requirements.


* **When the transaction is to be committed and rolled back by separating JMS and DB transactions**

  | In some cases, JMS transaction is rolled back whereas only DB transaction is committed.
  | In such a situation, it is necessary to manage JMS and DB transactions separately.
  | JMS and DB transactions can be separately controlled by defining a \ ``@Transactional``\  annotation in the Service class called by listener method.
  | Implementation example is given below wherein \ ``jmsListenerContainerFactory``\  of \ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\  is used for transaction control of JMS and \ ``transactionManager``\  defined in default configuration of Blank project is used for transaction control of DB.
  
  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
   
      package com.example.listener.todo;
      
      import javax.inject.Inject;
      import org.springframework.jms.annotation.JmsListener;
      import org.springframework.stereotype.Component;
      import com.example.domain.service.todo.TodoService;
      import com.example.domain.model.Todo; 
      @Component
      public class TodoMessageListener {
          @Inject
          TodoService todoService;

          @JmsListener(destination = "TransactedQueue") // (1)
          public void receiveTransactedMessage(Todo todo) {

              todoService.update(todo);

          }
      }

  - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`
  
   .. code-block:: java
  
      package com.example.domain.service.todo;
  
      import org.springframework.stereotype.Service;
      import org.springframework.transaction.annotation.Transactional;
      import com.example.domain.model.Todo;
      
      @Transactional // (2)
      @Service
      public class TodoServiceImpl implements TodoService {
          
          @Override
          public void update(Todo todo) {
              // omitted
          }
      }


   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Define \ ``@JmsListener``\  annotation and specify \ ``DefaultJmsListenerContainerFactory``\  which enables transaction control of JMS.
          | Since \ ``@JmsListener``\  annotation refers to Bean name \ ``jmsListenerContainerFactory``\  as a default, \ ``containerFactory``\  attribute is omitted.
      * - | (2)
        - | Define transaction boundary of DB.
          | Since value is omitted, Bean name \ ``transactionManager``\  is referred as a default.
          | For details of \ ``@Transactional``\  annotation, refer \ :ref:`service_transaction_management`\  of \ :doc:`../../ImplementationAtEachLayer/DomainLayer`\.

   .. note::

      Nesting sequence for transaction boundary depends on the business requirements and JMS provider is often used for linking with external systems.
      In such a case, JMS transaction boundary is kept outside DB transaction boundary and recovery becomes easier when inward DB transaction is completed earlier.
      When DB transaction is committed and JMS transaction is rolled back, the message returns to the Queue. Hence the same message is processed again.
      It should be designed so as to allow DB update process to be re-tried at the time of reexecution of business process.

|

* **When JMS and DB transactions are to be committed and rolled back together**

  A method which uses global transaction by JTA exists for linking JMS and DB transactions, however "Best Effort 1 Phase Commit" is recommended since overheads are likely to occur for performance, among protocol characteristics. Refer below for details.

  | \ `Distributed transactions in Spring, with and without XA <http://www.javaworld.com/article/2077963/open-source-tools/distributed-transactions-in-spring--with-and-without-xa.html>`_\ 
  | \ `Spring Distributed transactions using Best Effort 1 Phase Commit <http://gharshangupta.blogspot.jp/2015/03/spring-distributed-transactions-using_2.html>`_\ 

  .. warning:: **When transaction process results are not returned in JMS provider due to issues like loss of connection with JMS provider after receiving a message**
    
    When transaction process results are not returned in JMS provider due to issues like loss of connection with JMS provider after receiving a message, transaction handling depends on JMS provider.
    Hence, \ **a design considering loss of received messages, reprocessing of messages due to roll back**\  must be performed.
    Especially, when loss of messages is absolutely not permissible, \ **providing a system to compensate for the loss of messages or using a global transaction etc**\  must be adopted.

  | "Best Effort 1 Phase Commit" can be achieved by using \ ``org.springframework.data.transaction.ChainedTransactionManager``\.
  | An implementation example is shown below wherein \ ``jmsAsyncReceiveTransactionManager``\  of \ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\  is used in JMS transaction control and \ ``transactionManager``\  defined in default setting of Blank project is used in DB transaction control.

  - :file:`[projectName]-env/src/main/resources/META-INF/spring/[projectName]-env.xml`

   .. code-block:: xml
     
      <!-- (1) -->
      <bean id="chainedTransactionManager" class="org.springframework.data.transaction.ChainedTransactionManager">
          <constructor-arg>
              <list>
                  <!-- (2) -->
                  <ref bean="jmsAsyncReceiveTransactionManager" />
                  <ref bean="transactionManager" />
              </list>
          </constructor-arg>
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90
     
      * - Sr. No.
        - Description
      * - | (1)
        - | Define a Bean for \ ``ChainedTransactionManager``\.
      * - | (2)
        - | Specify a transaction manager of JMS and DB.
          | Transaction starts in a registered sequence and transaction is committed in the reverse sequence.

  - :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

   .. code-block:: xml
     
      <!-- (1) -->
      <jms:listener-container
          factory-id="chainedTransactionJmsListenerContainerFactory"
          destination-resolver="destinationResolver"
          concurrency="1"
          error-handler="jmsErrorHandler"
          cache="consumer"
          transaction-manager="chainedTransactionManager"
          acknowledge="transacted"/>

   .. tabularcolumns:: |p{0.05\linewidth}|p{0.26\linewidth}|p{0.69\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 26 64

      * - Sr. No.
        - Attribute name
        - Details
      * - | (1)
        - \-
        - | Define \ ``<jms:listener-container/>``\  for using \ ``ChainedTransactionManager``\.
      * - 
        - \ ``factory-id``\ 
        - | Specify Bean name of \ ``DefaultJmsListenerContainerFactory``\.
          | In this example, Bean name is set to \ ``chainedTransactionJmsListenerContainerFactory``\  by considering its combined use with \ ``<jms:listener-container/>``\  of \ :ref:`JMSHowToUseListenerContainer`\.
      * - 
        - \ ``transaction-manager``\ 
        - | Specify Bean name for \ ``ChainedTransactionManager``\.
      * - 
        - \ ``acknowledge``\ 
        - | Specify \ ``transacted``\  in check response mode in order to enable the transaction. Default is \ ``auto``\.
        
          .. note::
             
             In \ ``DefaultMessageListenerContainer``\, when Bean of implementation class of \ ``org.springframework.transaction.support.ResourceTransactionManager``\  is specified in \ ``transaction-manager``\ attribute, the transaction control which uses this Bean is enabled.
             However, since \ ``ChainedTransactionManager``\  does not implement \ ``ResourceTransactionManager``\, transaction control is not enabled.
             \ ``transacted``\  must be specified in \ ``acknowledge``\  attribute in order to enable transaction control.
             Accordingly, Session acting as a target for transaction control is generated and transaction control for \ ``ChainedTransactionManager``\  is enabled.

  Implementation example which uses the configuration above is shown below.

  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
     
     @Inject
     TodoService todoService;

     @JmsListener(containerFactory = "chainedTransactionJmsListenerContainerFactory", destination = "jms/queue/TodoMessageQueue") // (1)
     public void receiveTodo(Todo todo) {
         // omitted
     }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Since the Bean name uses \ ``DefaultJmsListenerContainerFactory``\  of \ ``chainedTransactionJmsListenerContainerFactory``\, \ ``chainedTransactionJmsListenerContainerFactory``\  is specified in \ ``containerFactory``\  attribute.


  Behaviour when an application is created in accordance with the configuration and implementation example above is explained.

  * **When processing of listener method is successfully completed**

   | JMS and DB transactions are committed together.
   | Transaction starts in the sequence of JMS, DB, and transaction ends in the sequence of DB, JMS after executing listener method.

    .. figure:: ./images_JMS/JMSDBTransactionAllCommit.png
        :alt: JMS/DB Transaction
        :width: 80%
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Start JMS transaction.
       * - | (2)
         - | Start DB transaction.
       * - | (3)
         - | Terminate listener method successfully.
       * - | (4)
         - | Commit DB transaction and terminate DB transaction.
       * - | (5)
         - | Commit JMS transaction and terminate JMS transaction.


  * **When an unexpected exception occurs in listener method or business logic**

   When exception occurs in listener method

    .. figure:: ./images_JMS/JMSDBTransactionAllRollback.png
        :alt: JMS/DB Transaction
        :width: 80%
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Start JMS transaction.
       * - | (2)
         - | Start DB transaction.
       * - | (3)
         - | An unexpected exception occurs in listener method or business logic.
       * - | (4)
         - | Roll back DB transaction and terminate DB transaction.
       * - | (5)
         - | Roll back JMS transaction and terminate JMS transaction.
           | Since JMS transaction is rolled back, the message is returned to the Queue.

  * **When connection with JMS provider is lost after receiving the message and only DB transaction is committed**

   | JMS transaction handling is dependent on JMS provider, however, since DB transaction is committed, an inconsistency is likely to occur in JMS and DB status.
   | Considering the possibility of rollback of JMS transaction, integrity of the data must be ensured when same message is received multiple times.
   | An example to ensure data integrity is shown below.

   * When the process after asynchronous receiving is executed multiple times, process must be designed to ensure same status after the processing.
   * Design so as to record \ ``JMSMessageID``\. \ ``JMSMessageID``\  recorded earlier and \ ``JMSMessageID``\  of received message are compared for each received message and when they match, received message is destroyed.

    .. figure:: ./images_JMS/JMSDBTransactionUnexpectedError.png
        :alt: JMS/DB Transaction
        :width: 80%
    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Start JMS transaction.
       * - | (2)
         - | Start DB transaction.
       * - | (3)
         - | Successfully terminate listener method.
       * - | (4)
         - | Commit DB transaction and terminate DB transaction.
       * - | (5)
         - | An unexpected error like JMS provider disconnected occurs.
       * - | (6)
         - | An error is likely to occur in JMS transaction commit.
           | Hence, a system to ensure consistency must be provided for loss of messages etc.

    .. note::

       When it is necessary to stringently control multiple transactions like JMS and DB by avoiding events given above, use of global transaction is considered.
       For global transaction, refer various product manuals.

|

.. _JMSHowToUseExceptionHandlingForAsyncReceive:

Exception handling at the time of asynchronous receiving
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| While performing transaction control, exceptions must be handled considering the rollback process.
| For details of transaction control, refer \ :ref:`JMSHowToUseTransactionManagementForAsyncReceive`\ .
| Exception handling of JMS is classified in 2 patterns given below based on the objective.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.40\linewidth}|p{0.25\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|
 .. list-table:: **Table-Exception handling patterns**
    :header-rows: 1
    :widths: 10 40 25 10 15

    * - Sr. No.
      - Objective of handling
      - An example of exception for handling
      - Handling method
      - Handling unit
    * - | (1)
      - | When exceptions occurring in business layer are to be individually handled
      - | Business exception like input check error
      - | Listener method
        | (try-catch)
      - | Listener method unit
    * - | (2)
      - | When the exceptions thrown by listener method are to be handled uniformly
      - | System exceptions like input output error etc
      - | \ ``ErrorHandler``\ 
      - | JMSListenerContainer unit


* **When exceptions occurred in the business layer are to be handled individually**

  | The exceptions occurring in the business layer like "message contents are invalid" are trapped (try-catch) by listener method and handled by listener method unit.
  | While performing transaction control, since exceptions must be thrown in \ ``DefaultMessageListenerContainer``\  for the cases which require rollback, the captured exception must be thrown again.
  | Implementation example is shown below.
  
  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
     
     @Inject
     TodoService todoService;

     @JmsListener(destination = "jms/queue/TodoMessageQueue")
     public JmsResponse<Todo> receiveTodo(Todo todo) {
         try {
             todoService.insertTodo(todo);
         } catch (BusinessException e) {
             return JmsResponse.forQueue(todo, "jms/queue/ErrorMessageQueue"); // (1)
         }
         return null; // (2)
     }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | An object can be sent to a Queue for storing a logical error message by using \ ``forQueue``\  method of \ ``JmsResponse``\  class.
          | In this example, since \ ``BusinessException``\  which outputs the log in AOP is captured, log output process is not described explicitly. However, exceptions must be handled so as not to eliminate the cause of exception.
          | When the messages are to be processed after roll back, by performing transaction control, the exception thus captured must be thrown.
      * - | (2)
        - | When the messages are not sent, set return value to \ ``null``\.

* **When the exceptions thrown by listener method are to be handled uniformly**

  | While commonly handling exceptions, implementation class of \ ``ErrorHandler``\  defined in \ ``error-handler``\  attribute of \ ``<jms:listener-container/>``\  is used.
  | Configuration method is shown below.

  - :file:`[projectName]-web/src/main/resources/META-INF/spring/applicationContext.xml`

   .. code-block:: xml

       <!-- (1) -->
       <jms:listener-container
           factory-id="jmsListenerContainerFactory"
           destination-resolver="destinationResolver"
           concurrency="1"
           error-handler="jmsErrorHandler"
           cache="consumer"
           transaction-manager="jmsAsyncReceiveTransactionManager"/>

       <!-- (2) -->
       <bean id="jmsErrorHandler"
           class="com.example.domain.service.todo.JmsErrorHandler">
       </bean>


   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Define a Bean name of error handling class in \ ``error-handler``\  attribute of \ ``<jms:listener-container/>``\.
      * - | (2)
        - | Define a Bean for error handling class.
       

  Implementation method is as shown below.

  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/JmsErrorHandler.java`
  
   .. code-block:: java
  
      package com.example.listener.todo;
      
      import org.springframework.util.ErrorHandler;
      import org.terasoluna.gfw.common.exception.SystemException;
     
      public class JmsErrorHandler implements ErrorHandler {  // (1)
             
         @Override
          public void handleError(Throwable t) { // (2)
              // omitted
              if (t.getCause() instanceof SystemException) {  // (3)
               
                  // omitted system error handling
               
              } else {
                  // omitted error handling
              }
          }
      }


   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Create an error handling class which implements \ ``ErrorHandler``\  interface.
      * - | (2)
        - | Exception occurring in the listener method is wrapped in \ ``org.springframework.jms.listener.adapter.ListenerExecutionFailedException``\  and passed as an argument.
      * - | (3)
        - | Determine any exception class and implement error handling associated with the exception.
          | \ ``t.getCause()``\  must be executed to fetch the exception occurred in the application.

|

.. _JMSHowToUseSyncReceiveMessage:

A method wherein the messages are synchronously received
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| Synchronous receiving in JMS provider is achieved by using \ ``JmsMessagingTemplate``\ .
| Messages can be received within any timing by using synchronous receiving.
| Architecture should be determined after thoroughly examining the implementation method which does not use synchronous receiving.

| Configuration of Bean definition file for synchronous receiving is shown below.

- :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

 .. code-block:: xml

    <bean id="cachingConnectionFactory"
       class="org.springframework.jms.connection.CachingConnectionFactory"> <!-- (1) -->
       <property name="targetConnectionFactory" ref="connectionFactory" /> <!-- (2) -->
       <property name="sessionCacheSize" value="1" />  <!-- (3) -->
    </bean>

    <!-- (4) -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
       <property name="connectionFactory" ref="cachingConnectionFactory" />
       <property name="destinationResolver" ref="destinationResolver" />
    </bean>

    <!-- (5) -->
    <bean id="jmsMessagingTemplate" class="org.springframework.jms.core.JmsMessagingTemplate">
        <property name="jmsTemplate" ref="jmsTemplate"/>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Define a Bean for \ ``org.springframework.jms.connection.CachingConnectionFactory``\  which performs caching of \ ``Session``\  and \ ``MessageProducer/Consumer``\.
        | Cache function can be used by using JMS provider specific \ ``ConnectionFactory``\  which is looked up in Bean definition or JNDI name,
          by wrapping it in \ ``CachingConnectionFactory``\  instead of using it as it is.
    * - | (2)
      - | Specify \ ``ConnectionFactory``\ which is wrapped up in Bean definition or JNDI name.
    * - | (3)
      - | Set cache number of \ ``Session``\ . (default value is 1)
        | Although 1 is specified in this example, cache number should be changed appropriately corresponding to performance requirement.
        | If a session is required to be continued even after exceeding this cache number, a new session is created and destroyed repeatedly without using a cache.
        | It is likely to cause reduction in process efficiency resulting in performance degradation.
    * - | (4)
      - | Define a Bean for \ ``JmsTemplate``\.
        | \ ``JmsTemplate``\  alternates for a low level API handling (JMS API calling).
        | For the attributes that can be configured, refer list of attributes of \ ``JmsTemplate``\  shown below.
    * - | (5)
      - | Define a Bean for \ ``JmsMessagingTemplate``\ . Inject \ ``JmsTemplate``\  which alternates as a synchronous receiving process.


| List of attributes of \ ``JmsTemplate``\  related to synchronous receiving are shown below.
| Configuration must be done as required.

 .. tabularcolumns:: |p{0.05\linewidth}|p{0.20\linewidth}|p{0.50\linewidth}|p{0.15\linewidth}|p{0.10\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 20 50 15 10
    :class: longtable

    * - Sr. No.
      - Configuration item
      - Details
      - Mandatory
      - Default value
    * - 1.
      - \ ``connectionFactory``\
      - | Set \ ``ConnectionFactory``\ to be used.
      - ○
      - Nil (since it is mandatory)
    * - 2.
      - \ ``pubSubDomain``\
      - | Set for messaging model.
        | Set \ ``false``\ for PTP (Queue) model and \ ``true``\ for Pub/Sub (Topic).
      - \-
      - \ ``false``\ 
    * - 3.
      - \ ``sessionTransacted``\
      - | Set whether the transaction is controlled in the session.
        | In this guideline, since transaction control described later is used, default \ ``false``\  is recommended.
      - \-
      - \ ``false``\ 
    * - 4.
      - \ ``sessionAcknowledgeMode``\
      - | Set confirmation response mode of session for \ ``sessionAcknowledgeMode``\.
        | For details, refer \ `JavaDoc of JMSTemplate <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/jms/core/JmsTemplate.html>`_\ .

        .. todo::

           Details of sessionAcknowledgeMode will be added later.
           
      - \-
      - | 1 
    * - 5.
      - \ ``receiveTimeout``\
      - | Set timeout period (milliseconds) at the time of synchronous receiving. If it is not set, wait till the message is received.
        | If the timeout period is not set, it impacts the subsequent processes, hence an appropriate timeout period must always be set.
      - \-
      - | 0 
    * - 6.
      - \ ``messageConverter``\
      - | Set message converter.
        | Default can be used in the range introduced in the guideline.
      - \-
      - \ ``SimpleMessageConverter``\ (\*1) is used.
    * - 7.
      - \ ``destinationResolver``\
      - | Set DestinationResolver.
        | This guideline recommends setting \ ``JndiDestinationResolver``\  which performs name resolution by JNDI.
      - \-
      - | \ ``DynamicDestinationResolver``\ (\*2) is used.
        | (If \ ``DynamicDestinationResolver``\  is used, name resolution of Destination is performed by JMS provider.)
    * - 8.
      - \ ``defaultDestination``\
      - | Specify existing Destination.
        | When the Destination is not specified explicitly, this Destination is used.
      - \-
      - null(no existing Destination)

 .. raw:: latex

    \newpage

(\*1)\ ``org.springframework.jms.support.converter.SimpleMessageConverter``\ 

(\*2)\ ``org.springframework.jms.support.destination.DynamicDestinationResolver``\ 


Messages are received synchronously by \ ``receiveAndConvert``\  message of \ ``JmsMessagingTemplate``\  class. Implementation example is shown below.

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

    package com.example.domain.service.todo;
    
    import javax.inject.Inject; 
    import org.springframework.jms.core.JmsMessagingTemplate;
    import org.springframework.stereotype.Service; 
    import com.example.domain.model.Todo;
    
    @Service
    public class TodoServiceImpl implements TodoService {
        @Inject
        JmsMessagingTemplate jmsMessagingTemplate;
        
        @Override
        public String receiveTodo() {
       
           // omitted
           Todo retTodo = jmsMessagingTemplate.receiveAndConvert("jms/queue/TodoMessageQueue", Todo.class);   // (1)

        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Receive message from specified Destination using \ ``receiveAndConvert``\  method of \ ``JmsMessagingTemplate``\ .
        | \ ``receiveAndConvert``\  method can fetch the class for which type conversion is done, by specifying the class for conversion, in the second argument.
        | A header item can be fetched by \ ``Message``\  object of Spring Framework, by using \ ``receive``\  method.

|

Appendix
--------------------------------------------------------------------------------

.. _JMSAppendixSettingsDependsOnJMSProvider:

JMS provider dependent configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Configuration varies for each JMS provider.
Configuration for each JMS provider is explained below.


While using Apache ActiveMQ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Configuration while using Apache ActiveMQ is explained.

* **JMS provider specific configuration for application server**

  | JMS provider requires specific configuration.
  | In Apache ActiveMQ, environment variable must be added to starting variable of application server to ensure that it consists of objects wherein payload of received messages is permissible.
  | For details, refer \ `ObjectMessage <http://activemq.apache.org/objectmessage.html>`_\ .
  | Configuration example while using Apache Tomcat is shown below. Refer \ `Service Configuration <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Installation_Guide/sect-Service_Configuration.html>`_\  for JBoss and \ `Starting Managed Servers with a Startup Script <http://docs.oracle.com/middleware/1221/wls/START/overview.htm#START120>`_\  for Weblogic.

  - :file:`$CATALINA_HOME/bin/setenv.sh`

   .. code-block:: properties

      # omitted 
      # (1)
      -Dorg.apache.activemq.SERIALIZABLE_PACKAGES=java.lang,java.util,org.apache.activemq,org.fusesource.hawtbuf,com.thoughtworks.xstream.mapper,com.example.domain.model
      # omitted 

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Add package for an arbitrary object to be allowed. \ ``java.lang``\ ,\ ``java.util``\ , \ ``org.apache.activemq``\ , \ ``org.fusesource.hawtbuf``\  and \ ``com.thoughtworks.xstream.mapper``\  are the settings required while using Apache ActiveMQ.
           | "com.example.domain.model" is added as a required configuration value in this sample.

* **Adding a library**

  | JMS API is not included in the \ ``spring-jms``\  library.
  | Although JMS API is normally included in the JMS provider library, JMS API is added to pom.xml if JMS API is not included in JMS provider library.


  | \ ``activemq-client``\  is added to pom.xml of domain project and web project as a library for build.
  | Further, \ ``activemq-client``\  and its dependent libraries are added to the application server.

  - :file:`[projectName]-domain/pom.xml`
  - :file:`[projectName]-web/pom.xml`

   .. code-block:: xml

       <dependencies>

           <!-- (1) -->
           <dependency>
               <groupId>org.apache.activemq</groupId>
               <artifactId>activemq-client</artifactId>
               <scope>provided</scope>
           </dependency>

       </dependencies>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
       :header-rows: 1
       :widths: 10 90

       * - Sr. No.
         - Description
       * - | (1)
         - | Add client library of Apache ActiveMQ to dependencies as a build. Since JMS API is also incorporated in \ ``activemq-client``\  library, it is not necessary to add JMS API as a library.

 .. note::

   In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.
   The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ .

|

 .. warning::

   The version of the library used while establishing a connection with Apache ActiveMQ is defined in \ `Spring IO Platform <http://platform.spring.io/platform/>`_\  which is used by TERASOLUNA Server Framework for Java.
   Hence, care must be taken while determining Apache ActiveMQ version.
   Note that, library and middleware versions are likely to be inconsistent while upgrading the version of TERASOLUNA Server Framework for Java.

|


* **JNDI registration to application server**

  | For registration of JNDI to application server, refer \ `Manually integrating Tomcat and ActiveMQ <http://activemq.apache.org/tomcat.html>`_\ .


* **Configuration when JNDI is not used.**

  | Although this guideline recommends a method to resolve names using JNDI,
  | JNDI is not used while connecting to JMS provider during the implementation of a simple test which is not run on the application server.
  | In such a case, a Bean of implementation class of \ ``ConnectionFactory``\  must be generated.
  | Further, use of JNDI is also specified for Queue, however, if a Queue specified in the Destination by using a JMS provider function does not exist, a Queue of specified name can be dynamically generated.
  | Internal Broker of Apache ActiveMQ must be used for establishing a connection without passing through the application server.
  | For configuration of internal Broker for Apache ActiveMQ, refer \ `How do I embed a Broker inside a Connection  <http://activemq.apache.org/how-do-i-embed-a-broker-inside-a-connection.html>`_\ .
  | Following configuration should be added to the context for the testing.

  - :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

   .. code-block:: xml

      <!-- (1) -->
      <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory"> 
          <constructor-arg value="tcp://localhost:61616"/>  <!-- (2) -->
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Define a Bean for \ ``ConnectionFactory``\  of Apache ActiveMQ.
      * - | (2)
        - | Specify beginning URL of Apache ActiveMQ. Beginning URL sets the value for each environment.

 .. note::
 
   When the configuration method of ConnectionFactory is to be changed by JNDI and bean definition, based on development phase,
   configuration should be described in \ ``[projectName]-env/src/main/resources/META-INF/spring/[projectName]-env.xml``\.

.. _JMSAppendixSendManySameMessages:

Mass-mailing of identical messages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| If \ ``JmsMessageTemplate``\  is to be used for mass mailing of identical message, memory usage is likely to increase.
| Hence, a method wherein implementation is done by using \ ``send``\  method of \ ``JmsTemplate``\  class must be considered.
| Reason being, an instance of \ ``org.springframework.jms.core.MessageCreator``\  class is generated while sending a message in \ ``JmsMessageTemplate``\.
| In order to prevent the generation of unnecessary instance, the messages are sent by \ ``send``\  method of \ ``JmsTemplate``\  class wherein \ ``MessageCreator``\  instance is not generated during sending thus reducing the amount of memory used.
| An example of code wherein a string is sent 100 times to an identical destination is shown below.

- :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

 .. code-block:: java

     package com.example.domain.service.todo;
     
     import java.io.IOException;
     import javax.inject.Inject;
     import javax.jms.JMSException;
     import javax.jms.Message;
     import javax.jms.Session;
     import javax.jms.TextMessage;
     import org.springframework.jms.core.JmsTemplate;
     import org.springframework.jms.core.MessageCreator;
     import org.springframework.stereotype.Service; 
     
     @Service
     public class TodoServiceImpl implements TodoService {

        @Inject
        JmsTemplate jmsTemplate; // (1)

        @Override
        public void sendManyMessage(final String messageStr) throws IOException {
            MessageCreator mc = new MessageCreator() { // (2)
                public Message createMessage(Session session) throws JMSException {
                    TextMessage message = session.createTextMessage(); // (3)
                    message.setText(messageStr);

                    // omitted
                    return message;
                }
            };
            for (int i = 0; i < 100; i++) {
                jmsTemplate.send("jms/queue/TodoMessageQueue", mc); // (4)
            }
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When \ ``JmsMessagingTemplate``\  is used, \ ``MessageCreater``\  is generated at the time of sending. Hence, \ ``JmsTemplate``\  which can define generation of \ ``MessageCreater``\  by isolating it from sending, is used.
    * - | (2)
      - | Generate instance of \ ``MessageCreator``\  for creating \ ``Message``\  of JMS.
    * - | (3)
      - | When messages are sent by \ ``send``\  method of \ ``JmsTemplate``\  class, instances of \ ``MessageCreator``\  are no longer generated for each loop
          and the amount of memory can be reduced.


.. _JMSAppendixSendLargeData:

Sending and receiving large size data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
While handling data of large size like Image data (Data of size 1MB or more as a rough indication), \ ``OutOfMemoryError``\  is likely to occur due to number of concurrent transactions and heap size.
In standard API of JMS, only \ ``StreamMessage``\  which sends primitive type data and \ ``ByteMessage``\  which can send uninterpreted byte stream can handle large size data as a stream.
Hence, a specific API offered for each JMS provider is used instead of JMS API.


While using Apache ActiveMQ
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
A message of large size can be received or sent by using \ `Blob Message <http://activemq.apache.org/blob-messages.html>`_\ . Implementation example is shown below.


 .. note::
    Since Apache ActiveMQ specific API is used while using \ ``org.apache.activemq.BlobMessage``\,
    \ ``Message``\  and \ ``CachingConnectionFactory``\  offered by Spring Framework cannot be used.
    It is recommended to separately define \ ``JmsTemplate``\  for \ ``BlobMessage``\  while using \ ``BlobMessage``\  considering the impact on the performance.


* **Configuration**

  While sending a message by using \ ``BlobMessage``\, the message is stored in the server which is temporarily run by Apache ActiveMQ, instead of heap area.
  Definition example for storage destination for the messages is shown below.

  - :file:`[projectName]-domain/src/main/resources/META-INF/spring/[projectName]-infra.xml`

   .. code-block:: xml

      <bean id="connectionFactory"
         class="org.apache.activemq.ActiveMQConnectionFactory">
          <property name="brokerURL">
            <!-- (1) -->
            <value>tcp://localhost:61616?jms.blobTransferPolicy.uploadUrl=/tmp</value>
          </property>
      </bean>

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Define a directory of Apache ActiveMQ server which temporarily stores messages.
          | \ ``http://localhost:8080/uploads/``\  is set as a default in \ ``jms.blobTransferPolicy.uploadUrl``\  wherein location for temporary file can be specified by overloading default or ``brokerURL``.
          | For example, the file is temporarily stored in \ ``/tmp``\ . 


* **Sending**

  An implementation example of sending class which use \ ``Blob Message``\  is shown below.


  - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;
      
      import java.io.IOException;
      import java.io.InputStream;
      import java.nio.file.Files;
      import java.nio.file.Path;
      import java.nio.file.Paths;
      import javax.inject.Inject;
      import javax.jms.JMSException;
      import javax.jms.Message;
      import javax.jms.Session;
      import org.apache.activemq.ActiveMQSession;
      import org.apache.activemq.BlobMessage;
      import org.springframework.jms.core.JmsTemplate;
      import org.springframework.jms.core.MessageCreator;
      import org.springframework.stereotype.Service;
      
      @Service
      public class TodoServiceImpl implements TodoService {
          @Inject
          JmsTemplate jmsTemplate;
          
          @Override
          public void sendBlobMessage(String inputFilePath) throws IOException {

              Path path = Paths.get(inputFilePath);
              try (final InputStream inputStream = Files.newInputStream(path)) {

                  jmsTemplate.send("jms/queue/TodoMessageQueue", new MessageCreator() {
                      public Message createMessage(Session session) throws JMSException {

                          ActiveMQSession activeMQSession = (ActiveMQSession) session;  // (1)

                          BlobMessage blobMessage = activeMQSession.createBlobMessage(inputStream);  // (2)
                          return blobMessage;
                      }
                  });
              }
          }
      }
     
   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 10 90

      * - Sr. No.
        - Description
      * - | (1)
        - | Use \ ``org.apache.activemq.ActiveMQSession``\  - a Apache ActiveMQ specific API in \ ``BlobMessage``\.
      * - | (2)
        - | Generate \ ``BlobMessage``\  from \ ``ActiveMQSession``\ by specifying sending data.
          | Arguments of \ ``createBlobMessage``\  method can be specified by \ ``File``\, \ ``InputStream``\  and \ ``URL``\  class.


* **Receiving**

  An implementation example of receiving class is shown below.


  - :file:`[projectName]-web/src/main/java/com/example/listener/todo/TodoMessageListener.java`
  
   .. code-block:: java
   
      package com.example.listener.todo;
      
      import java.io.IOException;
      import javax.inject.Inject;
      import javax.jms.JMSException;
      import org.apache.activemq.BlobMessage;
      import org.springframework.jms.annotation.JmsListener;
      import org.springframework.stereotype.Component;
      import com.example.domain.service.todo.TodoService;
      @Component
      public class TodoMessageListener {
          @Inject
          TodoService todoService;
          @JmsListener(destination = "jms/queue/TodoMessageQueue")
          public void receiveBlobMessage(BlobMessage message) throws IOException, JMSException {
           todoService.fileInputBlobMessage(message);
              // omitted
          }
      }

  - :file:`[projectName]-domain/src/main/java/com/example/domain/service/todo/TodoServiceImpl.java`

   .. code-block:: java

      package com.example.domain.service.todo;
      
      import java.io.IOException;
      import java.io.InputStream;
      import java.nio.file.Files;
      import java.nio.file.Path;
      import java.nio.file.Paths;
      import org.apache.activemq.BlobMessage;
      import org.springframework.stereotype.Service;
      
      @Service
      public class TodoServiceImpl implements TodoService {
      
          @Override
          public void fileInputBlobMessage(BlobMessage message) throws IOException {
              try(InputStream is =  message.getInputStream()){   // (1)
                  Path path = Paths.get("outputFilePath");
                  Files.copy(is, path);
                  // omitted
              }
          }
      }

   .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
   .. list-table::
      :header-rows: 1
      :widths: 13 87

      * - Sr. No.
        - Description
      * - | (1)
        - | Fetch data from received \ ``BlobMessage``\  as \ ``InputStream``\ .


.. raw:: latex

   \newpage



