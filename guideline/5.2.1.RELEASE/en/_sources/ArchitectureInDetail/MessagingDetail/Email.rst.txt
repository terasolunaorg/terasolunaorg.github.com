Sending E-mail (SMTP)
================================================================================

.. only:: html

 .. contents:: Index
    :depth: 3
    :local:

Overview
--------------------------------------------------------------------------------

This chapter explains how to send an E-mail using SMTP.

In this guideline, it is assumed that a component for email coordination offered by API and Spring Framework of JavaMail is used.

.. note::

    The description covers only the part related to sending an email.
    The processing method related to sending an email is not mentioned.
    (An example is introduced for \ :ref:`email-processing-method`\ .)

|

Regarding JavaMail
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

\ `JavaMail <https://java.net/projects/javamail/pages/Home>`_\  offers an API to send and receive emails in Java.
Although it is included in Java EE, it can also be used in Java SE as an add-on package.
By using JavaMail, the email function can be easily incorporated in Java application.

Also, since it is assumed that an email coordination component of Spring Framework is used in this guideline, the details related to JavaMail API are not covered.
Refer \ `JavaMail API Design Specification <http://download.oracle.com/otn-pub/jcp/java_mail-1_5-mrel2-eval-spec/JavaMail-1.5.pdf>`_\  for JavaMail API specifications.

.. note:: **Mail Session**

   A mail session (\ `Session <http://docs.oracle.com/javaee/7/api/javax/mail/Session.html>`_\ ) manages the information required for connecting to a mail server.
   
   Following methods are used to fetch a mail session.
   
   * Fetch the mail session managed by Java EE container through JNDI for a typical enterprise application.
   * Fetch the mail session defined by resource factory through JNDI in case of Tomcat.
   * Fetch the mail session for which a Bean is defined, from DI container using a static factory method.
   * Fetch directly from Java source using static factory method of \ ``Session``\ .
   
   Note that, if \ ``JavaMailSenderImpl``\  of Spring described later is used, it is possible to connect to a mail server without  handling a mail session directly.
   
   Implementation examples using two methods given below are introduced in this guideline.

   * A method wherein a mail session is fetched through JNDI
   * A method wherein connection information is specified in \ ``JavaMailSenderImpl``\  property without handling a session directly

|

Regarding component of Spring Framework for email coordination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Framework offers a component (\ ``org.springframework.mail``\  package) for sending an email.
The component included in the package conceals the detail logic related to sending an email and carry out low level API handling (API calling of JavaMail).

The method by which the component offered by Spring Framework for email transmission sends an email is explained before the explanation of basic implementation methods.

.. figure:: ./images_Email/EmailOverview.png
    :alt: Constitution of Spring Mail
    :width: 100%

.. raw:: latex

   \newpage

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.60\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 60
    :class: longtable

    * - Sr. No.
      - Component
      - Description
    * - | (1)
      - | Application
      - | Call \ ``JavaMailSender``\  method and request to send an email.
        |
        | \* While sending a simple message, an email can also be sent by generating \ ``SimpleMailMessage``\  and specifying address and body text.
    * - | (2)
      - | \ ``JavaMailSender``\
      - | Call \ ``MimeMessagePreparator``\  (callback interface to create \ ``MimeMessage``\  of JavaMail) specified by the application and request to create a message for sending an email (\ ``MimeMessage``\ ).
        |
        | This process is not called while sending the message using \* \ ``SimpleMailMessage``\ .
    * - | (3)
      - | Application
        | (\ ``MimeMessagePreparator``\)
      - | Create a message (\ ``MimeMessage``\ ) for sending the email using \ ``MimeMessageHelper``\  method.
        |
        | This process is not called while sending the message using \* \ ``SimpleMailMessage``\ .
    * - | (4)
      - | \ ``JavaMailSender``\
      - | Request to send the email using API of JavaMail.
    * - | (5)
      - | JavaMail
      - | Send a message to the email server.

.. raw:: latex

   \newpage

\

The method to implement a process for sending an email using interface and class below is explained in this guideline.

* \ ``JavaMailSender``\
    | An interface for JavaMail to send an email.
    | It supports both \ `MimeMessage <http://docs.oracle.com/javaee/7/api/javax/mail/internet/MimeMessage.html>`_\  of JavaMail and \ ``SimpleMailMessage``\  of Spring.
    | Further, since \ ``Session``\  of JavaMail is managed by implementation class of \ ``JavaMailSender``\  , it is not necessary to handle \ ``Session``\  directly while writing the code for the process to send an email.

* \ ``JavaMailSenderImpl``\
    | An implementation class of \ ``JavaMailSender``\  interface.
    | This class supports a method wherein DI is applied to configured \ ``Session``\  and a method wherein \ ``Session``\  is created from connection information specified in property.

* \ ``MimeMessagePreparator``\
    | A callback interface for creating \ ``MimeMessage``\  of JavaMail.
    | It is called from \ ``send``\  method of \ ``JavaMailSender``\ .
    | The exception generated in \ ``prepare``\  method of \ ``MimeMessagePreparator``\  is wrapped in \ ``MailPreparationException``\  (runtime exception) and thrown again.

* \ ``MimeMessageHelper``\
    | A helper class to facilitate creation of \ ``MimeMessage``\  of JavaMail.
    | \ ``MimeMessageHelper``\  provides a number of convenient methods to specify a value in \ ``MimeMessage``\ .

* \ ``SimpleMailMessage``\
    | A class to create a simple email message.
    | It can be used to create a plain text email in English.
    | A \ ``MimeMessage``\  of JavaMail must be used for creating rich messages such as specifying specific encoding like UTF-8, sending HTML emails and emails with attachments or associating personal names with email addresses.

How to use
--------------------------------------------------------------------------------

Regarding dependent library
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a component of Spring Framework for email coordination is used, following libraries must be added.

* `JavaMail <https://java.net/projects/javamail/pages/Home>`_

| Add a dependency relation for library above to \ :file:`pom.xml`\ .
| In case of a multi-project configuration, it is added to \ :file:`pom.xml`\  (:file:`projectName-domain/pom.xml`) of domain project.

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>com.sun.mail</groupId>
            <artifactId>javax.mail</artifactId>
        </dependency>

    </dependencies>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Add JavaMail libraries to dependencies.
        | Set \ ``<scope>``\  to \ ``provided``\  while using a mail session offered by application server.

.. note::

    In the above setting example, since it is assumed that the dependent library version is managed by the parent project  terasoluna-gfw-parent, specifying the version in pom.xml is not necessary.
    The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\.

|

How to configure JavaMailSender
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Define a Bean for applying DI to \ ``JavaMailSender``\ .

.. note::

    In case of a multi-project configuration, it is recommended to set in \ :file:`projectName-env.xml`\  of env project.
    Note that, in this guideline, it is recommended to adopt a multi-project configuration.


When a mail session offered by application server is used
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A configuration example while using a mail session offered by application server is given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Mail session offered by Application Server**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Application server
      - Refer page
    * - 1.
      - Apache Tomcat 8
      - | Refer \ `Apache Tomcat 8 User Guide(JNDI Resources HOW-TO) <http://tomcat.apache.org/tomcat-8.0-doc/jndi-resources-howto.html#JavaMail_Sessions>`_\  (JavaMail Sessions).
    * - 2.
      - Oracle WebLogic Server 12c
      - Refer \ `Oracle WebLogic Server 12.2.1.0 Documentation <http://docs.oracle.com/middleware/1221/wls/WLACH/taskhelp/mail/CreateMailSessions.html>`_\ .
    * - 3.
      - IBM WebSphere Application Server Version 8.5
      - Refer \ `WebSphere Application Server Version 8.5.5 documentation <https://www.ibm.com/support/knowledgecenter/en/SSD28V_8.5.5/com.ibm.websphere.wlp.core.doc/ae/twlp_admin_javamail.html>`_\ .
    * - 4.
      - Red Hat JBoss Enterprise Application Platform Version 6.4
      - Refer \ `Product Documentation <https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.4/html/Administration_and_Configuration_Guide/chap-Mail_subsystem.html>`_\ .


Carry out setup for registering a mail session fetched through JNDI, as a Bean.

.. code-block:: xml

   <jee:jndi-lookup id="mailSession" jndi-name="mail/Session" /> <!-- (1) -->

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify JNDI name of mail session offered by application server in \ ``jndi-name``\  attribute of \ ``<jee:jndi-lookup>``\  element.


Next, define a Bean for \ ``JavaMailSender``\ .

.. code-block:: xml

   <!-- (1) -->
   <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
       <property name="session" ref="mailSession" /> <!-- (2) -->
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a Bean for \ ``JavaMailSenderImpl``\ .
   * - | (2)
     - | Specify a Bean of configured mail session in \ ``session``\  property.


When a mail session offered by application server is not used (no authentication)
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A configuration example wherein authentication is not required is given below.

Define a Bean for \ ``JavaMailSender``\ .

.. code-block:: xml

   <!-- (1) -->
   <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
       <property name="host" value="${mail.smtp.host}"/> <!-- (2) -->
       <property name="port" value="${mail.smtp.port}"/> <!-- (3) -->
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a Bean for \ ``JavaMailSenderImpl``\ .
   * - | (2)
     - | Specify a host name of SMTP server in \ ``host``\  property.
       | In this example, a value defined in the property file (value corresponding to key "\ ``mail.smtp.host``\ ") is specified.
   * - | (3)
     - | Specify a port number of SMTP server in \ ``port``\  property.
       | In this example, a value defined in the property file (value corresponding to key "\ ``mail.smtp.port``\ ") is specified.

.. note::

   Refer :doc:`../GeneralFuncDetail/PropertyManagement` for details of property file.


When a mail session offered by application server is not used (authenticated)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

A configuration example wherein authentication is required is given below.

Define a Bean for \ ``JavaMailSender``\ .

.. code-block:: xml

   <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
       <property name="host" value="${mail.smtp.host}"/>
       <property name="port" value="${mail.smtp.port}"/>
       <property name="username" value="${mail.smtp.user}"/> <!-- (1) -->
       <property name="password" value="${mail.smtp.password}"/> <!-- (2) -->
       <property name="javaMailProperties">
           <props>
               <prop key="mail.smtp.auth">true</prop> <!-- (3) -->
           </props>
       </property>
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Specify a user name of SMTP server in \ ``username``\  property.
       | In this example, a value defined in property file (value corresponding to key "\ ``mail.smtp.user``\ ") is specified.
   * - | (2)
     - | Specify a password of SMTP server in \ ``password``\  property.
       | In this example, a value defined in the property file (value corresponding to key "\ ``mail.smtp.password``\ ") is specified.
   * - | (3)
     - | Set \ ``true``\  in \ ``javaMailProperties``\  property as a key "\ ``mail.smtp.auth``\ ".

.. note::

   Refer :doc:`../GeneralFuncDetail/PropertyManagement` for details of property file.

.. tip::

   When the connection using TLS is necessary, set \ ``true``\  in \ ``javaMailProperties``\  property as a key "\ ``mail.smtp.starttls.enable``\ ".
   Note that, when SMTP server does not support STARTTLS even when specified as below, plain text is used for communication.
   When \ ``true``\  is set in \ ``javaMailProperties``\  property as a key "\ ``mail.smtp.starttls.required``\ " whenever required, an error can occur if it is not possible to use STARTTLS.

|

How to send an email using SimpleMailMessage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a plain text email (an email which does not require encode specification or attachments) is to be sent in English, \ ``SimpleMailMessage``\  class offered by Spring is used.

A method to send an email using \ ``SimpleMailMessage``\  class is explained below.

**Example of Bean definition**

.. code-block:: xml

   <!-- (1) -->
   <bean id="templateMessage" class="org.springframework.mail.SimpleMailMessage">
       <property name="from" value="info@example.com" /> <!-- (2) -->
       <property name="subject" value="Registration confirmation." /> <!-- (3) -->
   </bean>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Define a Bean for \ ``SimpleMailMessage``\  as a template.
       | Although it is not mandatory to use \ ``SimpleMailMessage``\  of template, if a fixed location is specified (for e.g. sender email address etc) in the email message as a template, it is not necessary to individually specify it later in the email message.
   * - | (2)
     - | Specify details of From header in \ ``from``\  property.
   * - | (3)
     - | Specify details of Subject header in \ ``subject``\  property.

**Implementation example of Java class**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    @Inject
    SimpleMailMessage templateMessage; // (2)

    public void register(User user) {
        // omitted
        
        // (3)
        SimpleMailMessage message = new SimpleMailMessage(templateMessage);
        message.setTo(user.getEmailAddress());
        String text = "Hi "
                + user.getUserName()
                + ", welcome to EXAMPLE.COM!\r\n"
                + "If you were not an intended recipient, Please notify the sender.";
        message.setText(text);
        mailSender.send(message);
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject \ ``JavaMailSender``\ .
   * - | (2)
     - | Inject \ ``SimpleMailMessage``\  as a template for which a Bean is defined.
   * - | (3)
     - | Generate a \ ``SimpleMailMessage``\  instance by using Bean of template, specify To header and body text, and send the message.

.. note::

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}||p{0.60\linewidth}|
    .. list-table:: **Properties that can be set in SimpleMailMessage**
       :header-rows: 1
       :widths: 10 30 60

       * - Sr. No.
         - Property
         - Description
       * - | 1.
         - | \ ``from``\ 
         - | Specify From header.
       * - | 2.
         - | \ ``to``\ 
         - | Specify To header.
       * - | 3.
         - | \ ``cc``\ 
         - | Specify Cc header.
       * - | 4.
         - | \ ``bcc``\ 
         - | Specify Bcc header.
       * - | 5.
         - | \ ``subject``\ 
         - | Specify Subject header.
       * - | 6.
         - | \ ``replyTo``\ 
         - | Specify Reply-To header.
       * - | 7.
         - | \ ``sentDate``\ 
         - | Specify Date header.
           | Note that, if it is not explicitly set, system time （\ ``new Date()``\ ）is set automatically at the time of sending an email.
       * - | 8.
         - | \ ``text``\ 
         - | Specify body text.

   When multiple addresses are to be specified in To, Cc and Bcc, specify addresses in an array.
   

.. warning::

   While setting an email header, an email header injection must be considered.
   Refer \ :ref:`email-header-injection`\  for details.

|

How to send an email using MimeMessage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a non-English text email, HTML email and attachments are to be sent,
\ ``javax.mail.internet.MimeMessage``\  class is used.
In this guideline, a method to create \ ``MimeMessage``\  by using \ ``MimeMessageHelper``\  class is recommended.

In this section, the methods to send an email using \ ``MimeMessageHelper``\  class are explained below.

* :ref:`email-text`
* :ref:`email-html`
* :ref:`email-attachment`
* :ref:`email-inline-resource`

.. _email-text:

Sending a text email
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An implementation example wherein a text email is sent using \ ``MimeMessageHelper``\  class is given below.

**Implementation example of Java class**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String text = "Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!\r\n"
                        + "If you were not an intended recipient, Please notify the sender.";
                helper.setText(text); // (7)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject \ ``JavaMailSender``\ .
   * - | (2)
     - | Send an email using \ ``send``\  method of \ ``JavaMailSender``\ .
       | Define an anonymous inner class that implements \ ``MimeMessagePreparator``\ , in the argument.
   * - | (3)
     - | Specify character code and generate \ ``MimeMessageHelper``\  instance.
       | In this example, UTF-8 is specified in the character code.
   * - | (4)
     - | Specify details of From header.
       | In this example, it is set in "Name <Address>" format.
   * - | (5)
     - | Specify details of To header.
   * - | (6)
     - | Specify details of Subject header.
   * - | (7)
     - | Specify details of body text.

.. warning::

   While setting an email header, an email header injection must be considered.
   Refer \ :ref:`email-header-injection`\  for details.

.. note::

   While sending an email in Japanese, ISO-2022-JP can also be used in encoding if it is also necessary to support a mail client which does not support UTF-8.
   Refer \ :ref:`email-iso-2022-jp`\  for the points that should be considered while using ISO-2022-JP in encoding.

.. _email-html:

Sending a HTML email
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An implementation example wherein a HTML email is sent using \ ``MimeMessageHelper``\  class is shown below.

**Implementation example of Java class**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String text = "<html><body><h3>Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!</h3>"
                        + "If you were not an intended recipient, Please notify the sender.</body></html>";
                helper.setText(text, true); // (7)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject \ ``JavaMailSender``\ .
   * - | (2)
     - | Send an email using \ ``send``\  method of \ ``JavaMailSender``\ .
       | Define an anonymous inner class that implements \ ``MimeMessagePreparator``\ , in the argument.
   * - | (3)
     - | Specify character code and generate \ ``MimeMessageHelper``\  instance.
       | In this example, UTF-8 is specified in the character code.
   * - | (4)
     - | Specify details of From header.
       | In this example, it is set in the "Name <Address>" format.
   * - | (5)
     - | Specify details of To header.
   * - | (6)
     - | Specify details of Subject header.
   * - | (7)
     - | Specify details of body text. Content-Type changes to text/html by specifying \ ``true``\  in the second argument of \ ``setText``\  method.

.. warning::

   When a value entered externally is used while generating HTML for email text, countermeasures for XSS attack should be employed.


.. _email-attachment:

Sending an email with attachment
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An implementation example wherein an email with the attachment is sent using \ ``MimeMessageHelper``\  class is shown below.

**Implementation example of Java class**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        true, StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String text = "Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!\r\n"
                        + "Please find attached the file.\r\n\r\n"
                        + "If you were not an intended recipient, Please notify the sender.";
                helper.setText(text); // (7)
                ClassPathResource file = new ClassPathResource("doc/quickstart.pdf");
                helper.addAttachment("QuickStart.pdf", file); // (8)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject \ ``JavaMailSender``\ .
   * - | (2)
     - | Send an email using \ ``send``\  method of \ ``JavaMailSender``\ .
       | Define an anonymous inner class that implements \ ``MimeMessagePreparator``\ , in the argument.
   * - | (3)
     - | Specify character code and generate \ ``MimeMessageHelper``\  instance.
       | In this example, UTF-8 is specified in the character code.
       | It becomes multi-part mode (default \ ``MULTIPART_MODE_MIXED_RELATED``\ ) by specifying \ ``true``\  in the second argument of the constructor of \ ``MimeMessageHelper``\ . 
   * - | (4)
     - | Specify details of From header.
   * - | (5)
     - | Specify details of To header.
   * - | (6)
     - | Specify details of Subject header.
   * - | (7)
     - | Specify details of body text.
   * - | (8)
     - | Specify attachment name and identify file to be attached.
       | In this example, \ ``"QuickStart.pdf"``\  denotes the file name and \ :file:`doc/quickstart.pdf`\  file on the class path is attached.


.. _email-inline-resource:

Sending an email with inline resource
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

An implementation example wherein an email with inline resource is sent using \ ``MimeMessageHelper``\  class is shown below.

**Implementation example of Java class**

.. code-block:: java

    @Inject
    JavaMailSender mailSender; // (1)

    public void register(User user) {
        // omitted
        
        // (2)
        mailSender.send(new MimeMessagePreparator() {

            @Override
            public void prepare(MimeMessage mimeMessage) throws Exception {
                MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                        true, StandardCharsets.UTF_8.name()); // (3)
                helper.setFrom("EXAMPLE.COM <info@example.com>"); // (4)
                helper.setTo(user.getEmailAddress()); // (5)
                helper.setSubject("Registration confirmation."); // (6)
                String cid = "identifier1234";
                String text = "<html><body><img src='cid:"
                        + cid
                        + "' /><h3>Hi "
                        + user.getUserName()
                        + ", welcome to EXAMPLE.COM!\r\n</h3>"
                        + "If you were not an intended recipient, Please notify the sender.</body></html>";
                helper.setText(text, true); // (7)
                ClassPathResource res = new ClassPathResource("image/logo.jpg");
                helper.addInline(cid, res); // (8)
            }
        });
        
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Inject \ ``JavaMailSender``\ .
   * - | (2)
     - | Send an email using \ ``send``\  method of \ ``JavaMailSender``\ .
       | Define an anonymous inner class that implements \ ``MimeMessagePreparator``\  , in the argument.
   * - | (3)
     - | Specify character code and generate \ ``MimeMessageHelper``\  instance.
       | In this example, UTF-8 is specified in the character code.
       | It becomes a multi-part mode by specifying \ ``true``\  in the second argument of constructor of \ ``MimeMessageHelper``\ .
   * - | (4)
     - | Specify details of From header.
   * - | (5)
     - | Specify details of To header.
   * - | (6)
     - | Specify details of Subject header.
   * - | (7)
     - | Specify details of body text. Content-Type changes to text/html by specifying \ ``true``\  in the second argument of \ ``setText``\  method.
   * - | (8)
     - | Specify inline resource contents ID and set inline resource.
       | In this example, \ ``"identifier1234"``\  denotes a content ID and \ :file:`image/logo.jpg`\  file on the class path is set.

.. note::

   \ ``addInline``\  method should be called after ``setText``\  method.
   If done otherwise, a mail client cannot view the inline resource correctly.

|

Regarding exceptions while sending an email
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The exception that occurs while sending an email using \ ``send``\  method of \ ``JavaMailSender``\  is an exception which inherits \ ``org.springframework.mail.MailException``\ .
The exception class that inherits \ ``MailException``\  and occurrence conditions of respective exceptions are shown in the table below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.35\linewidth}|p{0.55\linewidth}|
 .. list-table:: **Exceptions at the time of sending an email**
    :header-rows: 1
    :widths: 10 35 55

    * - Sr. No.
      - Exception class
      - Occurrence conditions
    * - 1.
      - `MailAuthenticationException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailAuthenticationException.html>`_
      - | Occurs during authentication failure.
    * - 2.
      - `MailParseException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailParseException.html>`_
      - | Occurs when an invalid value is set in the properties of email message.
    * - 3.
      - `MailPreparationException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailPreparationException.html>`_
      - | Occurs if an unexpected error occurs while creating an email message.
          Unexpected errors, for example, are the errors that occur in the template library.
        | Exceptions occurring in \ ``MimeMessagePreparator``\  are wrapped in \ ``MailPreparationException``\  and thrown.
    * - 4.
      - `MailSendException <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/mail/MailSendException.html>`_
      - | Occurs when an error occurs while sending an email.

.. note::

   Refer :doc:`../WebApplicationDetail/ExceptionHandling` for transition to error screen corresponding to specific exceptions.

|

How to extend
--------------------------------------------------------------------------------

How to create an email text using a template
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is not recommended to  build an email text directly in Java source as shown in the implementation examples above for the reasons given below.

* Building the email text in Java source causes poor readability and it may cause an error.
* Boundary between display logic and business logic become ambiguous.
* It becomes necessary to modify, compile and deploy Java source in order to change the email text design.

Hence, it is recommended to use a template library to define an email text design.
A template library must especially be used when the email text is particularly complex.

Creating the email text using FreeMarker
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In this guideline, a method that uses \ `FreeMarker <http://freemarker.org/>`_\  as a template library is described.

* Set a dependent library for using FreeMarker.

    **Configuration example of pom.xml**
    
    .. code-block:: xml
    
        <dependencies>
    
            <!-- (1) -->
            <dependency>
                <groupId>org.freemarker</groupId>
                <artifactId>freemarker</artifactId>
            </dependency>
    
        </dependencies>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
        :header-rows: 1
        :widths: 10 90
    
        * - Sr. No.
          - Description
        * - | (1)
          - | Add a FreeMarker library to dependencies.
          
.. note::  
 
       In the above setting example, since it is assumed that the dependent library version is managed by the parent project terasoluna-gfw-parent , specifying the version in pom.xml is not necessary.  
       The above dependent library used by terasoluna-gfw-parent is defined by \ `Spring IO Platform <http://platform.spring.io/platform/>`_\ . 

* Define a Bean for FactoryBean to generate \ ``freemarker.template.Configuration``\ .

    **Configuration example of Bean definition file**
    
    .. code-block:: xml
    
       <!-- (1) -->
       <bean id="freemarkerConfiguration"
           class="org.springframework.ui.freemarker.FreeMarkerConfigurationFactoryBean">
           <property name="templateLoaderPath" value="classpath:/META-INF/freemarker/" /> <!-- (2) -->
           <property name="defaultEncoding" value="UTF-8" /> <!-- (3) -->
       </bean>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Define a Bean for \ ``FreeMarkerConfigurationFactoryBean``\ .
       * - | (2)
         - | Specify the location of template file storage in \ ``templateLoaderPath``\  property.
           | In this example, \ :file:`META-INF/freemarker/`\  directory on the class path is specified.
       * - | (3)
         - | Specify default encoding in \ ``defaultEncoding``\  property.
           | In this example, UTF-8 is specified.

    .. note::

       Refer to \ `JavaDoc of FreeMarkerConfigurationFactoryBean <http://docs.spring.io/spring/docs/4.2.7.RELEASE/javadoc-api/org/springframework/ui/freemarker/FreeMarkerConfigurationFactoryBean.html>`_\  for the setup other than mentioned above.
       Also, refer \ `FreeMarker Manual (Programmer's Guide / The Configuration) <http://freemarker.org/docs/pgui_config.html>`_\  for setup of FreeMarker itself.

* Create a template file for email text.

    **Configuration example of template file**
    
    .. code-block:: text
    
       <#escape x as x?html> <#-- (1) -->
       <html>
           <body>
               <h3>Hi ${userName}, welcome to TERASOLUNA.ORG!</h3> <#-- (2) -->
    
               <div>
                   If you were not an intended recipient, Please notify the sender.
               </div>
           </body>
       </html>
       </#escape>

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Specify to apply HTML escape as a countermeasure for XSS attack.
       * - | (2)
         - | Embed the value of \ ``userName``\  specified in the data model.

    .. note::

       Refer \ `FreeMarker Manual (Template Language Reference) <http://freemarker.org/docs/ref.html>`_\  for details of template language (FIL).

* Generate an email text using a template and send email.

    **Implementation example of Java class**
    
    .. code-block:: java
    
        @Inject
        JavaMailSender mailSender;
    
    	@Inject
    	Configuration freemarkerConfiguration; // (1)
    	
        public void register(User user) {
            // omitted
            
            mailSender.send(new MimeMessagePreparator() {

                @Override
                public void prepare(MimeMessage mimeMessage) throws Exception {
                    MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                            StandardCharsets.UTF_8.name());
                    helper.setFrom("EXAMPLE.COM <info@example.com>");
                    helper.setTo(user.getEmailAddress());
                    helper.setSubject("Registration confirmation.");
                    Template template = freemarkerConfiguration
                            .getTemplate("registration-confirmation.ftl"); // (2)
                    String text = FreeMarkerTemplateUtils
                            .processTemplateIntoString(template, user); // (3)
                    helper.setText(text, true);
                }
            });
            
            // omitted
        }

    .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
    .. list-table::
       :header-rows: 1
       :widths: 10 90
    
       * - Sr. No.
         - Description
       * - | (1)
         - | Inject \ `Configuration <http://freemarker.org/docs/api/freemarker/template/Configuration.html>`_\ .
       * - | (2)
         - | Fetch \ `Template <http://freemarker.org/docs/api/freemarker/template/Template.html>`_\  using \ ``getTemplate``\  method of \ ``Configuration``\ .
           | In this example, "registration-confirmation.ftl" is specified as a template file.
       * - | (3)
         - | Based on fetched \ ``Template``\ , generate a string from the template using \ ``processTemplateIntoString``\  method of \ ``org.springframework.ui.freemarker.FreeMarkerTemplateUtils``\ .
           | In this example, \ ``User``\  object (JavaBeans) consisting of \ ``userName``\  property is specified as a data model.
             Accordingly, value of \ ``userName``\  property is embedded in the location of \ ``${userName}``\  of template file.

|

Appendix
--------------------------------------------------------------------------------

.. _email-iso-2022-jp:

Considerations for ISO-2022-JP encoding
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When sending an email in Japanese, if the email client that receives the sent mail cannot be restricted, it is necessary to consider use of ISO-2022-JP in encoding.
This is because the legacy email client does not support UTF-8.

When encoding based on character set of JIS X 0208 including ISO-2022-JP is set for the string entered by MS932,
garbling occurs for seven characters described in the table below.

.. tabularcolumns:: |p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 2
   :widths: 20 15 15 15 15 20

   * - Before conversion
     - 
     - 
     - After conversion
     - 
     - 
   * - | MS932
       | Input character
     - | Input value
       | （SJIS）
     - | Unicode
       | （UTF-16）
     - | Unicode
       | （UTF-16）
     - | ISO-2022-JP
       | （JIS）
     - | JIS X 0208
       | Alternative character
   * - | ―(Double byte hyphen)
     - | 815D
     - | U+2015
     - | U+2014
     - | 213E
     - | —(EM dash)
   * - | －(Hyphen-minus)
     - | 817C
     - | U+FF0D
     - | U+2212
     - | 215D
     - | −(Double byte minus)
   * - | ～(Double byte tilde)
     - | 8160
     - | U+FF5E
     - | U+301C
     - | 2141
     - | 〜(Tilde)
   * - | ∥(Parallel symbol)
     - | 8161
     - | U+2225
     - | U+2016
     - | 2142
     - | ‖(Pipe sumbol)
   * - | ￠(Double byte cent symbol)
     - | 8191
     - | U+FFE0
     - | U+00A2
     - | 2171
     - | ￠(Cent symbol)
   * - | ￡(Double byte pound symbol)
     - | 8192
     - | U+FFE1
     - | U+00A3
     - | 2172
     - | ￡(Pound symbol)
   * - | ￢(Double byte negation symbol)
     - | 81CA
     - | U+FFE2
     - | U+00AC
     - | 224C
     - | ￢(Negation symbol)

This issue occurs during character code conversion through Unicode due to the presence of characters that exist in MS932 but do not exist in JIS X 0208.
In order to avoid garbling, the measures such as replacing character codes for the garbled characters with alternate characters must be employed.
Note that, conversion process is not necessary while using x-windows-iso2022jp described later.

Implementation example of conversion process is shown below.

.. code-block:: java

    public static String convertISO2022JPCharacters(String targetStr) {

        if (targetStr == null) {
            return null;
        }

        char[] ch = targetStr.toCharArray();

        for (int i = 0; i < ch.length; i++) {
            switch (ch[i]) {

            // '-'(Double byte hyphen)
            case '\u2015':
                ch[i] = '\u2014';
                break;
            // '－'(Double byte minus)
            case '\uff0d':
                ch[i] = '\u2212';
                break;
            // '～'(Tilde)
            case '\uff5e':
                ch[i] = '\u301c';
                break;
            // '∥'(Pipe)
            case '\u2225':
                ch[i] = '\u2016';
                break;
            // '￠'(Cent symbol)
            case '\uffe0':
                ch[i] = '\u00A2';
                break;
            // '￡'(Pound symbol)
            case '\uffe1':
                ch[i] = '\u00A3';
                break;
            // '￢' (Negation symbol)
            case '\uffe2':
                ch[i] = '\u00AC';
                break;
            default:
                break;
            }
        }

        return String.valueOf(ch);
    }

.. note::

   Since it is an issue that occurs during mapping in Unicode, conversion is necessary regardless of the character code of input value.
   Header and body text which may contain strings with Japanese text are used for conversion.
   From, To, Cc, Bcc, Reply-To, Subject etc are examples of the header that may contain Japanese text and are frequently used in general.

Also, when ISO-2-22-JP is set as encoding, extended characters which are not in scope are garbled.

.. figure:: ./images_Email/EmailOutofEscapeCharacter.png
    :alt: Out of EscapeCharacter
    :width: 100%
    :align: center
    
    **Fig.- Examples of extended characters that are not in scope**

These characters essentially should not be used.
If it is necessary to use these characters, settings can be done as follows as JVM start-up options.
Moreover, when ISO-2022-JP encoding is set, these characters can be replaced so that they can be mapped with x-windows-iso2022jp.

.. code-block:: text

   -Dsun.nio.cs.map=x-windows-iso2022jp/ISO-2022-JP

.. warning::

   x-windows-iso2022jp is a ISO-2022-JP implementation which includes mapping (MS932 base) that is different from ISO-2022-JP standards.
   When ISO-2022-JP encoding is specified in the email header, whether the out-of-scope extension characters are handled in the implementation will depend on the email client.
   Hence it cannot be guaranteed that there will not be any garbling in all email clients even though mapping is done using x-windows-iso2022jp.

When extension characters can also be converted to alternate characters, a method wherein conversion is done in the application independently should also be reviewed similar to seven characters described earlier.

|

.. _email-note-of-when-sending:

Precautions while using multibyte characters in JavaMail
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In JavaMail, body text of mail to be sent ends with multibyte characters, additional characters  ("?", "w" etc) are likely to be output.  
Following methods can be used to avoid these occurrences.  

* Change end character of mail body text to single byte character.  
* Change end of mail body text to linefeed code (CRLF)  

|

.. _email-header-injection:

Email header injection countermeasures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If email header injection attack is successful, an email is sent to an unintended address
and thus can end up as "junk mail".
When a string that has been entered externally is used in the email header (Subject etc) contents, it becomes necessary to take countermeasures against email header injection attack.

For example, if the following string is set by \ ``setSubject``\  method of \ ``MimeMessageHelper``\ , the text can be manipulated by adding a Bcc header.

.. code-block:: text

   Notification\r\nBcc: attacker@exapmle.com\r\n\r\nManipulated body.

Following methods can be considered as countermeasures for email header injection attack.

* Set contents to be specified in email header as a fixed value and output entire string that has been entered externally in the email body text.
* Check that no linefeed character is included in the contents specified in the email header.

|

.. _email-processing-method:

Processing method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since sending an email is a time consuming process, response time can become longer if an email is sent during a web application request.
Hence, an email is usually not sent during a web application request and a method is adopted wherein an email is sent asynchronously.
Although a process to send the email is not described here in detail, the example is given below which can be used as a reference.

Send an email based on the email information stored in database or message queue
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Incorporate the functions given below in the application when you want to send an email based on the email information stored in database or message queue.

* Register the information of the email to be sent (address, body text, attachment etc) in the database (or message queue).
* Fetch the email information that has not been sent from database (or message queue) periodically and send the email through SMTP.
* Register the transmission results in database (or message queue).

Note that, following points must be taken into consideration during review.

* How to check registered email information and email sending results
* Handling in case of an error while sending an email

.. tip::

   When an email is sent continuously by a mailing service, it is determined as spam mail.
   A method can be considered as a countermeasure wherein the sending sequence is set as random so that emails are not sent continuously for the same domain.

|

.. _email-test-with-greenmail:

Test using GreenMail
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A method wherein \ `GreenMail <http://www.icegreen.com/greenmail/>`_\  is used as a fake server to test the email sending function is introduced.
GreenMail can also be used by deploying war file besides using it as a library.

Implementation example of test code using GreenMail is shown below.

**Configuration example of pom.xml**

.. code-block:: xml

    <dependencies>

        <!-- (1) -->
        <dependency>
            <groupId>com.icegreen</groupId>
            <artifactId>greenmail</artifactId>
            <version>1.4.1</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr.No.
      - Description
    * - | (1)
      - | Add GreenMail library to dependencies.

**Implementation example of JUnit source**

.. code-block:: java

    @Inject
    EmailService emailService;

    @Rule
    public final GreenMailRule greenMail = new GreenMailRule(
            ServerSetupTest.SMTP); // (1)

    @Test
    public void testSend() {

        String from = "info@example.com";
        String to = "foo@example.com";
        String subject = "Registration confirmation.";
        String text = "Hi "
                + to
                + ", welcome to EXAMPLE.COM!\r\n"
                + "If you were not an intended recipient, Please notify the sender.";
        emailService.send(from, to, subject, text);

        assertTrue(greenMail.waitForIncomingEmail(3000, 1)); // (2)

        Message[] messages = greenMail.getReceivedMessages(); // (3)

        assertNotNull(messages);
        assertEquals(1, messages.length);
        // omitted
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Set \ ``GreenMailRule``\  as a rule which specifies \ ``ServerSetupTest.SMTP``\ .
       | SMTP port number is set to \ ``3025``\  as a default.
   * - | (2)
     - | Wait for arrival of email by using \ ``waitForIncomingEmail``\  method.
       | It is used when an email is sent asynchronously by another thread.
       | In this example, it is assumed that the email is sent asynchronously and waiting time for an email arrival is maximum 3 seconds.
   * - | (3)
     - | Fetch all received emails by using \ ``getReceivedMessages``\  method.
       | Emails sent by GreenMail are all received by GreenMail regardless of the address.

.. raw:: latex

   \newpage

