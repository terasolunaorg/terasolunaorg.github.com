Password Hashing
================================================================================

.. only:: html

 .. contents:: Table of contents
    :local:

Overview
--------------------------------------------------------------------------------
| Password hashing is one of the key considerations when designing a secure application.
| In a normal system, hashing is mandatory as it is not possible to register a password in plaintext.
| However, if a weak algorithm is selected, the original data of hashed password can be easily cracked by
| "Offline Brute Force Attack" or "Rainbow Crack" etc.
| 
| Spring Security provides \ ``org.springframework.security.crypto.password.PasswordEncoder``\  interface, as the password hashing mechanism.
| The following classes,

* \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\
* \ ``org.springframework.security.crypto.password.StandardPasswordEncoder``\

| etc. are provided as its implementation classes.
| 
| As \ ``PasswordEncoder``\  mechanism, password is hashed by \ ``encode(String rawPassword)``\  method and the encoded password is verified by
| the \ ``matches (String rawPassword, String encodedPassword)``\  method.

.. figure:: ./images/PasswordEncoder_class.png 
   :alt: PasswordEncoder Class Diagram
   :width: 80%
   :align: center

   **Picture - PasswordEncoder Class Diagram**

|

How to use
--------------------------------------------------------------------------------
| Implementation classes of PasswordEncoder provided by Spring Security, are explained in this section.

**List of PasswordEncoder implementation classes**

.. tabularcolumns:: |p{0.50\linewidth}|p{0.50\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - PasswordEncoder implementation classes
     - Overview
   * - | \ ``org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder``\
     - | Encoder that performs hashing using "bcrypt" algorithm
   * - | \ ``org.springframework.security.crypto.password.StandardPasswordEncoder``\
     - | Encoder that performs hashing with "SHA-256" algorithm + 1024 rounds of stretching
   * - | ``org.springframework.security.crypto.password.NoOpPasswordEncoder``
     - | Encoder that does not perform hashing (for testing)

It is recommended to use \ ``BCryptPasswordEncoder``\  when hashing is not required.
However, the calculation time taken by \ ``BCryptPasswordEncoder``\  to improve counter-attacks being more,
if the performance requirements at the time of authentication are not fulfilled, \ ``StandardPasswordEncoder``\  should be considered.

In case of any restrictions concerning salt and hashed algorithm owing to the relation with existing system,
implementation class of \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\  interface, which will be described later, should be used.
For details, refer \ :ref:`authenticationPasswordEncoder`\ .

BCryptPasswordEncoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``BCryptPasswordEncoder``\  is the class that implements \ ``PasswordEncoder``\  and provides password hashing.
| It is the encoder which uses random 16 bytes of salt and bcrypt algorithm.

.. note::

  In Bcrypt algorithm, number of calculations have been intentionally increased to more than the calculations of the standard algorithms. Therefore,
  compared to standard algorithms (SHA, MD5), it has strong features to resist "offline brute force attack".

.. _BCryptPasswordEncoder:

Configuration example of BCryptPasswordEncoder
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* applicationContext.xml

  .. code-block:: xml

    <bean id="passwordEncoder"
        class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" />    <!-- (1) -->
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90

     * - Sr. No.
       - Description
     * - | (1)
       - | Specify \ ``BCryptPasswordEncoder``\  in passwordEncoder class.
         | 
         | Number of salt hash rounds can be specified as constructor argument. Values from 4 to 31 can be set.
         | Longer the salt value, stronger the hashing. However, as number of calculations increase exponentially, it is necessary to exercise caution from performance perspective.
         | When no value is specified, "10" is set.
       
  .. tip::

    It is described later in 'How to extend'; however, DaoAuthenticationProvider can set the implementation class of \ ``org.springframework.security.crypto.password.PasswordEncoder``\ 
    as well as the implementation class of \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ .
    Therefore, when the existing PasswordEncoder (authentication package) is changed to a new PasswordEncoder,
    it can be handled only by changing the passwordEncoder of DaoAuthenticationProvider, after changing the user password.
  
  .. warning::
  
    When \ ``DaoAuthenticationProvider``\  is set in authentication provider and \ ``UsernameNotFoundException``\  is thrown, without letting the person operating the system know that user does not exist,
    password is intentionally hashed after \ ``UsernameNotFoundException``\  is thrown. (Side channel attack countermeasure)
  
    To create values for this hashing, \ ``encode``\  method is once executed internally when starting the application.
  
  .. warning::
  
    When SecureRandom is used in Linux environment, the process may be delayed or timeout may occur.
    The cause of this issue is random number generation and is described in the following Java Bug Database.
  
    http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6202721
    
    It has been corrected in the JDK 7 version b20 and above.
  
    http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6521844
  
    This issue can be avoided by setting the following as JVM boot arguments.
  
    -Djava.security.egd=file:/dev/./urandom

* Java class

  .. code-block:: java
  
        @Inject
        PasswordEncoder passwordEncoder;  // (1)
  
        public String register(Customer customer, String rawPassword) {
            // omitted
            // Password Hashing
            String password = passwordEncoder.encode(rawPassword); // (2)
            customer.setPassword(password);
            // omitted
        }
  
        public boolean matches(Customer customer, String rawPassword) {
            return passwordEncoder.matches(rawPassword, customer.getPassword()); // (3)
        }
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Inject \ ``PasswordEncoder``\  for which bean definition is carried out.
     * - | (2)
       - | Password hashing example
         | By specifying the plaintext password as an argument of encode method, hashed password is returned.
     * - | (3)
       - | Password verification example
         | By specifying plaintext password as the first argument and hashed password as the second argument,
         | 'matches' method checks whether both the passwords match.

StandardPasswordEncoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``StandardPasswordEncoder``\  uses SHA-256 as the hashing algorithm and performs 1024 rounds of stretching.
| It assigns randomly generated salt of 8 bytes.


| \ ``encode(String rawPassword)``\  method and \ ``matches(String rawPassword, String encodedPassword)``\  method 
| of the \ ``StandardPasswordEncoder``\  are described below.

**encode(String rawPassword) method**

.. figure:: ./images/standard_password_encoder_encode.png
   :alt: encode method
   :width: 50%
   :align: center
 
   **Picture - encode method**

| Hashing is carried out by randomly generated salt of 8 bytes + secret key + the password specified in argument.
| Return value of method is the value wherein, salt used for hashing is assigned in the beginning of the above hashed value.

**matches(String rawPassword, String encodedPassword) method**

.. figure:: ./images/standard_password_encoder_matches.png
   :alt: matches method
   :width: 60%
   :align: center
 
   **Picture - matches method**

| The salt passed from argument and assigned at the beginning of encodedPassword, is split and the two values namely, the value hashed by salt + secret + rawPassword
| and the value without salt assigned at the beginning of encodedPassword, are compared.


Configuration example of StandardPasswordEncoder
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

* applicationContext.xml

  .. code-block:: xml
  
    <bean id="passwordEncoder"
      class="org.springframework.security.crypto.password.StandardPasswordEncoder">
      <!-- from properties file -->
      <constructor-arg value="${passoword.encoder.secret}"/> <!-- (1) -->
    </bean>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Specify the secret key for hashing.
         | When specified, password is hashed with "internally generated salt" + "specified secret key" + "password".
         | It is recommended to specify secret key, as the strength against rainbow table attack reduces if not specified.
         | 
         | **About secret key**
         | Secret key should be handled as confidential information.
         | Therefore, instead of specifying it directly in the Spring Security configuration file, fetch it from properties file or environment variable etc.
         | Here, example of fetching the secret key from properties file is enabled. Further, care should be taken regarding the storage location of properties file in a production environment.

  .. tip::

    **When secret key is fetched from environment variables**

   It can be fetched by performing the following settings in \ ``<constructor-arg>``\  of StandardPasswordEncoder bean definition.
    
      .. code-block:: xml
      
        <bean id="passwordEncoder"
          class="org.springframework.security.crypto.password.StandardPasswordEncoder">
          <!-- from environment variable -->
          <constructor-arg value="#{systemEnvironment['PASSWORD_ENCODER_SECRET']}" /> <!-- (1) -->
        </bean>
      
      .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
      .. list-table::
         :header-rows: 1
         :widths: 10 90
         
         * - Sr. No.
           - Description
         * - | (1)
           - | Fetch value from environment variable PASSWORD_ENCODER_SECRET.



  | Refer to \ :ref:`BCryptPasswordEncoder`\ , as example of Java class is the same as \ ``BCryptPasswordEncoder``\ .

NoOpPasswordEncoder
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| \ ``NoOpPasswordEncoder``\  is the encoder that returns the specified value as a string without any change.
| It must not be used except for the un-hashed strings to be used at the time of unit testing etc.

| As its configuration example is same as that of BCryptPasswordEncoder, it is omitted here.

.. _authenticationPasswordEncoder:

How to extend
--------------------------------------------------------------------------------
| Depending on business requirements, it may not be possible to implement password hashing using the class that implements \ ``PasswordEncoder``\  mentioned above.
| Especially when the hashing system used in the existing account information is to be followed, often the \ ``PasswordEncoder``\  mentioned above, does not fulfill the requirements.

For example we may consider a case wherein the existing hashing system is as follows:
 * Algorithm used is SHA-512.
 * There are 1000 rounds of stretching.
 * Salt is stored in account table column and needs to be passed externally from \ ``PasswordEncoder``\ .

| In this case, it is recommended to use the class that implements \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\  of a different package
| rather than the class that implements \ ``org.springframework.security.crypto.password.PasswordEncoder``\ .

 .. warning::

     In versions prior to Spring Security 3.1.4, the class that implements  \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\  was used for hashing. However,
     it has been deprecated from Spring Security version 3.1.4 onwards.
     Therefore, it differs from the pattern recommended by Spring.

Example where ShaPasswordEncoder is used
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When business requirements are as follows wherein,
| SHA-512 algorithm is used and 1000 rounds of stretching are performed.
| It is explained here with an example of authentication process that uses DaoAuthenticationProvider
| described in \ :doc:`Authentication`\ .

* applicationContext.xml

  .. code-block:: xml
  
    <bean id ="passwordEncoder"
        class="org.springframework.security.authentication.encoding.ShaPasswordEncoder"> <!-- (1) -->
        <constructor-arg value="512" /> <!-- (2) -->
        <property name="iterations" value="1000" /> <!-- (3) -->
    </bean>
  
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | Specify \ ``org.springframework.security.authentication.encoding.ShaPasswordEncoder``\  as the passwordEncoder.
         | The class to be specified in passwordEncoder should change according to the algorithm to be used.
     * - | (2)
       - | Set the SHA algorithm type as constructor argument.
         | The values "1, 256, 384, 512" can be set. When omitted, "1" is set.
     * - | (4)
       - | Specify the number of stretching rounds at the time of hashing.
         | When omitted, it is 0.

* spring-mvc.xml

  .. code-block:: xml
  
    <bean id="authenticationProvider"
        class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
        <!-- omitted -->
        <property name="saltSource" ref="saltSource" /> <!-- (1) -->
        <property name="userDetailsService" ref="userDetailsService" />
        <property name="passwordEncoder" ref="passwordEncoder" /> <!-- (2) -->
    </bean>
  
    <bean id="saltSource"
        class="org.springframework.security.authentication.dao.ReflectionSaltSource"> <!-- (3) -->
        <property name="userPropertyToUse" value="username" /> <!-- (4) -->
    </bean>
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | When salt is to be defined externally, set BeanId of the class that implements
         | \ ``org.springframework.security.authentication.dao.SaltSource``\ .
         | In this example, \ ``org.springframework.security.authentication.dao.ReflectionSaltSource``\
         | that fetches the value set in user information class by reflection, is defined.
     * - | (2)
       - | Specify \ ``org.springframework.security.authentication.encoding.ShaPasswordEncoder``\  as the passwordEncoder.
         | The class to be specified in passwordEncoder should change according to the algorithm to be used.
     * - | (3)
       - | Specify \ ``org.springframework.security.authentication.dao.SaltSource``\  that decides how to create salt.
         | Here, \ ``ReflectionSaltSource``\  resource that fetches \ ``UserDetails``\  object property by reflection, is used.
     * - | (4)
       - | \ ``username``\  property of \ ``UserDetails``\  object is used as salt.

* Java class
       
  .. code-block:: java
  
      @Inject
      PasswordEncoder passwordEncoder;
  
      public String register(Customer customer, String rawPassword, String userSalt) {
          // omitted
          String password = passwordEncoder.encodePassword(rawPassword,
                  userSalt); // (1)
          customer.setPassword(password);
          // omitted
      }
  
      public boolean matches(Customer customer, String rawPassword, String userSalt) {
          return passwordEncoder.isPasswordValid(customer.getPassword(),
                     rawPassword, userSalt); // (2)
      }
  
  .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
  .. list-table::
     :header-rows: 1
     :widths: 10 90
  
     * - Sr. No.
       - Description
     * - | (1)
       - | To hash password,
         | specify password and salt string as the argument of \ ``encodePassword``\  method
         | of the class that implements \ ``org.springframework.security.authentication.encoding.PasswordEncoder``\ .
     * - | (2)
       - | To verify password,
         | Using \ ``isPasswordValid``\  method, hashed password, plain text password and
         | salt string are specified as the argument and the hashed password and plaintext passwords are compared.

Appendix
--------------------------------------------------------------------------------
       
.. note::    **About stretch**

    By repeating the hash function computation, information regarding password to be stored can be repeatedly encoded.
    This is done to prolong the time required to crack a password, and thus acts as a countermeasure against the brute force attack.
    However, since stretching affects system performance, it is necessary to decide the stretch count on considering the system performance.
       
       
.. note::    **About salt**

  Salt is the string assigned to the original data to be encoded.
  By assigning salt to a password, the length of the password is increased and thus makes it difficult to crack passwords using rainbow attacks etc.
  Further, if the same salt is used for multiple users and if there are users who have the same password,
  it will be obvious from the hash value that same password is used.
  Therefore, it is recommended to use a different salt (random value etc.) for each user.

.. raw:: latex

   \newpage
