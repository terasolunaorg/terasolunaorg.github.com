Session tutorial
********************************************************************************


.. only:: html

 .. contents:: Table of contents
    :depth: 3
    :local:

Introduction
================================================================================



Flow of learning
--------------------------------------------------------------------------------

In this tutorial, users will learn the how to design data for the session management and the specific method of implementation to use the session by creating a simple web application.
This tutorial will be implemented with the following flow.

#. To check the requirements of web application to be created
#. To check the method of implementation for the Controller and the procedure for designing data to meet the requirements
#. To implement on the basis of the design information


Lessons to be learnt in the tutorial
--------------------------------------------------------------------------------

* How to design data for the session management
    * Selecting data to be stored in the session
    * Discarding data in the session
* Specific method of using the session in this FW
    * How to use @SessionAttributes
    * How to use the Bean for session scope


Target Readers
--------------------------------------------------------------------------------

* Tutorial: Implements Todo application
* Tutorial: Implements Spring Security


Test environment
--------------------------------------------------------------------------------

This tutorial verifies the operation in the following environments.

.. tabularcolumns:: |p{0.30\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 30 70

    * - Type
      - Product
    * - OS
      - Windows 7
    * - JVM
      - `Java <http://www.oracle.com/technetwork/java/javase/downloads/index.html>`_ 1.8
    * - IDE
      - `Spring Tool Suite <http://spring.io/tools/sts/all>`_ 3.6.4.RELEASE (hereafter called as 'STS')
    * - Build Tool
      - `Apache Maven <http://maven.apache.org/download.cgi>`_ 3.3.3 (hereafter called as 'Maven')
    * - Application Server
      - `Pivotal tc Server <https://network.pivotal.io/products/pivotal-tcserver>`_ Developer Edition v3.1 (Included in STS)
    * - Web Browser
      - `Google Chrome <https://www.google.co.jp/chrome/browser/desktop/index.html>`_ 42.0.2311.90 m

Application overview and requirements
================================================================================


Overview
--------------------------------------------------------------------------------

Create a simple EC site.
User can perform the following operations on the EC site.


* Login to the account
* Create an account
* Change the information of the account created
* View the list of products handled on the EC site
* View the product details
* Register the products to be purchased in a cart
* Remove the products registered in the cart from the cart
* Place order for the products in the cart

Application overview is shown in the following diagram. XxxPages in the diagram indicate a set of screens.
In this tutorial, the interaction between the system and the user performed on 1 screen set is handled as 1 use case.

.. figure:: images/materialSessionTutorialOverview.png
   :alt: overview
   :width: 95%



Requirements
--------------------------------------------------------------------------------


Functional requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Implement the following function for each screen (use case) described earlier in this application.

 .. tabularcolumns::  |p{0.5\linewidth}|p{0.5\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 50 50
    
    * - Screen (use case)
      - Function
    * - | Login Pages
      - | Login function **(created)**
    * - | Account Create Pages
      - | Account creation function **(created)**
    * - | Account Update Pages
      - | Account information change function
    * - | Item View Pages
      - | Product list view function **(created)**
        | Product details view function **(created)**
        | Cart item registration function
    * - | Cart View Pages
      - | Cart item deletion function
    * - | Order Pages
      - | Product order function


Some functions have been created in advance in the project which are offered as the initial material for this tutorial.
This is done to reduce the cost to create the part that is not directly related to the session management.

Create unfinished function in this tutorial.
Further, implementation of domain layer/infrastructure layer has been created in unfinished function as well.
Therefore, create screens for unfinished function and application layer in this tutorial.




Non-functional requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is necessary to design and implement the application by considering the non-functional requirements required of that system while creating the real application.
Design/create the application in this tutorial with the assumption of non-functional requirements in this tutorial.
The specific numerical values for each requirement shown below are the hypothetical values used for learning.
It should be noted that it cannot be guaranteed that the application created in this tutorial will actually meet the requirements.


Availability

* Operation period: 24 hours
* A planned downtime of few days in a year
* Allow a downtime for about 1 hour
* Target the fault recovery within 1 business day
* Utilization: 99%

Usability

* Operation is not guaranteed on multiple browsers and tabs 

Performance

* Number of users: 10,000
* Number of concurrent access users: 200
* Number of online process records: 10,000/month
* The number of users/number of concurrent access users/number of online process records together are expected to increase 1.2 times in 1 year


It is necessary to consider the above requirements while reviewing the following items to design the session management.

 .. tabularcolumns::  |p{0.15\linewidth}|p{0.85\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 15 85
    
    * - Requirement
      - Items to be reviewed
    * - | Availability
      - * Status of replication in multiple-server operation
    * - | Usability
      - * Retention of data integrity
    * - | Performance
      - * Status of replication in multiple-server operation
        * Memory utilization

Further, passing of important information including personal information/credit card information also should be considered in the design of the session management other than the items mentioned above.


Configuration of platform
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Application created in this tutorial is to be operated on the following platforms.
The specific numerical values for the configuration shown below are the hypothetical values used for learning.

* Consider 2 configurations for each server of Web/AP/DB.
* Memory load of AP server is 8GB and 2 slots are free

It is necessary to consider the above configuration while reviewing the memory utilization and replication status to design the session management.


.. _development_policy:

Application design
================================================================================

Determine the policy to create the application based on the requirements described earlier.
Since the domain layer/infrastructure layer have been created in this tutorial,
consider only the items related to the application layer as the target.
Further, since this tutorial aims at learning the method to use the session,
description for the items those are not directly related to the session management is omitted.


.. warning::

    Note that an example of the process that uses the session is shown in this chapter.
    It is required to follow the work instructions/operating procedures for each project in the real development.

Screen definition
--------------------------------------------------------------------------------

Define the screen on which the application is displayed, based on the requirements.
Details for the screen definition process are omitted.

Image for the screen to be created in this tutorial defined in the end, is as follows.

.. figure:: images/materialSessionTutorialSpecificationOfUpdateAccountPages.png
   :alt: specification of Account Update Pages
   :width: 95%


.. figure:: images/materialSessionTutorialSpecificationsOfMainFlowPages.png
   :alt: specification of Main Flow Pages
   :width: 95%


Some of the transitions are given below which are omitted in the above diagram.

* When the user logins from the Login screen, the transition takes place to screen (5)
* When 'Home' button is clicked on each screen of Account Update Pages, the transition takes place to screen (5)
* When 'Update Account' button is clicked on each screen of Item View Pages, Cart View Pages and Order Pages, the transition takes place to screen (1)
* When 'Logout' button is clicked on each screen of Item View Pages, Cart View Pages and Order Pages, the transition takes place to Login screen



Extracting URL
--------------------------------------------------------------------------------

The application determines the URL that does processing, based on the screen image.

Set the URL and parameters for each event occurred from each screen.
Assign the respective names as per the following standards.

* URL:/<Use case name>
* Parameter:?<Process name>

Since the use cases are divided into account creation and update in this application,
set the URLs as /account/create and /account/update respectively.

Further, also determine the Controller to process each URL.
Basically process 1 use case in 1 Controller.

Finally the extracted URL can be arranged as follows.
The Controller that is mentioned as "Created", exists in the project provided as the initial material.
Further, the process wherein the path mentioned as "Created" is accessed, is already described within the created Controller mentioned earlier.


 .. tabularcolumns::  |p{0.05\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.20\linewidth}|p{0.25\linewidth}|p{0.20\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 20 10 20 25 20
    
    * - Sr. No.
      - Process name
      - HTTP method
      - Path
      - Controller name
      - Screen
    * - | (1)
      - | Account information change screen 1 display process
      - | GET
      - | /account/update?form1
      - | AccountUpdateController
      - | /account/updateForm1
    * - | (2)
      - | Account information change screen 2 display process
      - | POST
      - | /account/update?form2
      - | AccountUpdateController
      - | /account/updateForm2
    * - | (3)
      - | Account information change confirmation screen display process
      - | POST
      - | /account/update?confirm
      - | AccountUpdateController
      - | /account/updateConfirm
    * - | (4)
      - | Account information change process
      - | POST
      - | /account/update
      - | AccountUpdateController
      - | Redirect to Account information change completion screen display process
    * - | (5)
      - | Account information change completion screen display process
      - | GET
      - | /account/update?finish
      - | AccountUpdateController
      - | /account/updateFinish
    * - | (6)
      - | Process to return to Account information change screen 1
      - | POST
      - | /account/update?redoform1
      - | AccountUpdateController
      - | /account/updateForm1
    * - | (7)
      - | Process to return to Account information change screen 2
      - | POST
      - | /account/update?redoform2
      - | AccountUpdateController
      - | /account/updateForm2
    * - | (8)
      - | Process to return to Home
      - | GET
      - | /account/update?home
      - | AccountUpdateController
      - | Redirect to Product list screen display process
    * - | (9)
      - | Product list screen display process (default)
      - | GET
      - | /goods **(created)**
      - | GoodsController **(created)**
      - | /goods/showGoods
    * - | (10)
      - | Product list screen display process (while selecting category)
      - | GET
      - | /goods?categoryId **(created)**
      - | GoodsController **(created)**
      - | /goods/showGoods
    * - | (11)
      - | Product list screen display process (while selecting page)
      - | GET
      - | /goods?page **(created)**
      - | GoodsController **(created)**
      - | /goods/showGoods
    * - | (12)
      - | Product details screen display process
      - | GET
      - | /goods?{goodsId} **(created)**
      - | GoodsController **(created)**
      - | /goods/showGoodsDetail
    * - | (13)
      - | Process to add a product in the cart
      - | GET
      - | /addToCart
      - | GoodsController **(created)**
      - | Redirect to Product list screen display process
    * - | (14)
      - | Cart screen view process
      - | GET
      - | /cart
      - | CartController
      - | cart/viewCart
    * - | (15)
      - | Process to delete a product from the cart
      - | POST
      - | /cart
      - | CartController
      - | Redirect to cart screen display process
    * - | (16)
      - | Order confirmation screen display process
      - | GET
      - | /order?confirm
      - | OrderController
      - | order/confirm
    * - | (17)
      - | Order processing
      - | POST
      - | /order
      - | OrderController
      - | Redirect to order completion screen display process
    * - | (18)
      - | Order completion screen display process
      - | GET
      - | /order?finish
      - | OrderController
      - | order/finish




Designing input/output data
--------------------------------------------------------------------------------

Design the input/output data handled by the application based on the screen image.


Extracting data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Extract the input/output data handled by the application screen.
The following data can be extracted based on the screen image mentioned earlier.

 .. tabularcolumns::  |p{0.05\linewidth}|p{0.25\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 70 
    
    * - Sr. No.
      - Data item name
      - Data element
    * - | (1)
      - | Update account information
      - | Account name, mail address, birthdate, postal code, address, card number, validity, security code
    * - | (2)
      - | Account information
      - | Account name, mail address, password, birthdate, postal code, address, card number, validity, security code
    * - | (3)
      - | Product search information
      - | Selection category, page number
    * - | (4)
      - | Product information
      - | Product name, unit price, description, (product ID)
    * - | (5)
      - | Register cart information
      - | Quantity, (product ID)
    * - | (6)
      - | Cart information
      - | Product name, unit price, quantity, (product ID)
    * - | (7)
      - | Delete cart information
      - | Product ID list
    * - | (8)
      - | Order information
      - | Order ID, order date-time, (account ID), product name, unit price, quantity



Defining lifecycle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


Define the lifecycle of the data extracted in the previous paragraph.
Determine when the data will be generated and when it will be discarded in the definition of the lifecycle.

Also note that the data which is to be retained in multiple screens will have multiple discard timings as below.

* The task is terminated with the usual flow
* Task is cancelled during the process


If the above precautions are considered, the lifecycle of the data extracted in the previous paragraph can be defined as follows.

 .. tabularcolumns::  |p{0.05\linewidth}|p{0.25\linewidth}|p{0.70\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 5 25 70 
    
    * - Sr. No.
      - Data item name
      - Lifecycle
    * - | (1)
      - | Update account information
      - | Generate the data with the input from screen (1) and retain it during the transition from (1) to (3). Discard it when there is a transition except from screen (1) to (3).
    * - | (2)
      - | Account information
      - | Generate the data at the time of login and discard at the time of logout.
    * - | (3)
      - | Product search information
      - | Generate the data when there is a transition to screen (5) and retain during the transition from (1) to (8). Discard when there is a transition to screen (9).
    * - | (4)
      - | Product information
      - | Generate the data when there is a transition to screen (5) or (6) and retain only during that request.
    * - | (5)
      - | Register cart information
      - | Generate the data by the input from screen (5) or (6) and retain only during that request.
    * - | (6)
      - | Cart information
      - | Generate an empty object when there is a transition to screen (5) and retain during the transition from (1) to (8). Discard when there is a transition to screen (9).
    * - | (7)
      - | Delete cart information
      - | Generate the data by the input from screen (7) and retain only during that request.
    * - | (8)
      - | Order information
      - | Generate the data when there is a transition to screen (9) and retain only during that request.


Determining the status of using a session
--------------------------------------------------------------------------------

When the information needs to be retained on multiple screens, implementation is easy by using a session. At the same time, when the session is used, its demerits also need to be considered.
Refer to the guideline :doc:'../ArchitectureInDetail/WebApplicationDetail/SessionManagement' and
determine whether to use the session in this tutorial.

It is described in the guideline that it is recommended to use the session first and to store only the data that is necessary, in the session.
This tutorial also considers not using the session.



 .. tabularcolumns::  |p{0.25\linewidth}|p{0.85\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 15 85
    
    * - Data item
      - Review contents
    * - | Update account information
      - | The data needs to be passed using hidden since the Update account information is to be retained across 3 screens. However, the Update account information contains the important information like the card number. It is a security issue since the important information is written in the HTML source without masking the important information while passing data using hidden. Therefore, consider using the session in this tutorial.
    * - | Account information
      - | Since it is retained on all screens after login, the data needs to be passed using hidden. The process of passing data must be described on almost all screens created in this case. Therefore, consider using the session in this tutorial to reduce the implementation cost for screens as well.
    * - | Product search information
      - | The data needs to be passed using hidden since the Product search information is to be retained across 8 screens. The process of passing data must be described on almost all screens created in this case. Therefore, consider using the session in this tutorial to reduce the implementation cost for screens as well.
    * - | Product information
      - | Since the Delete cart information is used only in 1 screen, the data should be handled in the request scope.
    * - | Register cart information
      - | Since the Delete cart information is used only in 1 screen, the data should be handled in the request scope.
    * - | Cart information
      - | The data needs to be passed using hidden since the Cart information is to be retained across 8 screens. The process of passing data must be described on almost all screens created in this case. Therefore, consider using the session in this tutorial to reduce the implementation cost for screens as well.
    * - | Delete cart information
      - | Since the Delete cart information is used only in 1 screen, the data should be handled in the request scope.
    * - | Order information
      - | Since the Order information is used only in 1 screen, the data should be handled in the request scope.


From the above-mentioned, consider using the session for these 4 i.e. the Update account information, Account information, Cart information and Product search information.

Next, verify the demerits of using the session.
According to this verification, if it is determined that the impact of demerits cannot be ignored, do not use the session.

The following 3 major points can be mentioned as the demerits of using the session.

* When it is used on multiple tabs and multiple browsers, (it needs to be considered that) the integrity of data may be lost because of mutual operations.
* Since it is managed on the memory, the memory is likely to be exhausted because of the size of the data to be managed.
* The session replication needs to be considered when AP server multiplexing is done with the aim to implement scale-out or to achieve high availability. At that time, if large data is handled in the session, it may affect the performance etc.


Consider how to handle the respective risks or whether to allow the risks regarding the above-mentioned point of view.

 .. tabularcolumns::  |p{0.25\linewidth}|p{0.85\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 15 85
    
    * - Point of view
      - Review contents
    * - | Data integrity
      - | The operation on multiple browsers and tabs is not guaranteed in this application. Therefore, a countermeasure to secure the data integrity is not required.
    * - | Memory utilization
      - | Estimate the data size for which usage of session is considered. Assume maximum 100 characters 240 bytes (4 characters 8 bytes + initial 40 bytes) for the character string element, 24 bytes for data element and 16 bytes for the numerical value element. Further, the authentication information stored in the session at the time of login authentication also contains the size of \ "UserDetails"\ . \ "UserDetails"\  broadly contains ID, password and user rights. Multiple user rights can be specified, but assume here as 1. The result estimated for each item is as follows.
      
        * Account information (Character string: 7, Date: 2): maximum 1.7K bytes
        
        * Change account information (Character string: 8, Date: 2): maximum 2.0K bytes

        * Cart information (maximum 19 products x (Character string: 3, numerical value: 3)): maximum 14.6K bytes
        
        * Product search information (numerical value: 2): 32 bytes
        
        * \ "UserDetails"\ : (Character string: 3): 0.7K byte

        | 1 user uses maximum 19KB in total. 1 user uses approximately 21KB if safety factor of 10% is considered. Since the usage is about 210MB even if it is considered that 10000 people are simultaneously connected and the memory load is considerably below 8GB even if other memory utilization is considered, it is less likely that the memory exhaustion will happen.
    * - | AP server multiplexing
      - | Since high availability is not required in this application, use case continuation when failure happens is not required and redoing of use case because of re-login is allowed. Therefore, only take the countermeasure to set the load balancer so that all the requests occurred within the same session are distributed to the same AP server and do not implement the replication among the AP servers of the session.

.. warning::

    Tool (for example, like SizeOf) needs to be used to measure the object size for estimation of the object size. The calculation formula in this tutorial refers the trend in actually measured values in SizeOf, but note that ultimately it is a temporary value. It should be separately considered how to calculate it at the time of sizing in the real system development.

.. warning::

    The data to be stored in the session is basically restricted to the input data to avoid memory exhaustion. Since the size of the output data for search results tends to increase and on the other hand, often it is read-only that cannot be edited using screen operations, it is not suitable for storing in the session.

Increase in the management cost of session key is also 1 of the points to be considered other than the above-mentioned.
However, since the data quantity to be stored in the session is not large in the application created this time, it can be said that the management cost of the session key is limited.


It can be said from this result that the impact of demerits happening because of using session is not large.
Data to be stored in the session finally is as follows.

* Change account information
* Account information
* Product search information
* Cart information


This tutorial concludes that passing of data is implemented using the session.
However, it is also considered as a result of the investigation that it concludes that session should not be used.
Implement passing the data using hidden as an example when the session is not used.


Further, when the session is used, sometimes method to maintain the data integrity and replication settings are required.

The guideline mentions a method to avoid it using transaction token check. However, note that it becomes a low-usability application in this case. \ :doc:`../ArchitectureInDetail/WebApplicationDetail/DoubleSubmitProtection`\  should be referred for the specific implementation method.

Since replication settings depend on AP server, when replication needs to be considered, configuration of AP server needs to be checked.


.. warning::

    Sometimes there exists a data to be stored in the session other than the data determined here.
    Session is used when the following items among the items in the guideline are used.
    
    * It uses authentication/authorization/CSRF countermeasures using Spring Security
    * It uses transaction token check for prevention of double transmission


Method of implementation to use data during the session
--------------------------------------------------------------------------------

Method of implementation to use the data during the session for each data is determined in this section.

The guideline provides 2 implementation methods corresponding to the locations of using data.
:doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement` classifies the methods to be used depending on whether the data is contained within 1 Controller.
Therefore, the implementation method needs to be decided considering the lifecycle of data to be stored in the session and URL mapping.
Further, when the data is associated with the authentication information, session management should be implemented by the Spring Security function.

Considering these, the final result for which the data handled in the session is organized is as follows.

 .. tabularcolumns::  |p{0.30\linewidth}|p{0.30\linewidth}|p{0.40\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 30 30 40
    
    * - Data
      - Characteristics
      - Method of using data during session
    * - | Change account information
      - | Used only within 1 Controller
      - | Method wherein @SessionAttributes annotation is used
    * - | Account information
      - | Used among multiple Controllers
        | Used in authentication process
      - | Method wherein Spring Security function is used
    * - | Product search information
      - | Used among multiple Controllers
      - | Method wherein Bean for session scope of Spring is used
    * - | Cart information
      - | Used among multiple Controllers
      - | Method wherein Bean for session scope of Spring is used


Account information is already created in the project provided as initial material and
is managed using the Spring Security function.
Therefore, this tutorial does not describe any specific method of using.
:doc:'../Security/Authentication' should be referred for the specific method of using.


Considerations while using session
--------------------------------------------------------------------------------

Items mentioned hereafter need to be considered when it is decided to use the session.
Review the respective items.


Session synchronization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Object stored in the session can be simultaneously accessed by means of multiple requests of the same user.
Therefore, when session synchronization is not performed, it can cause unexpected error or operation.

Since the guideline mentions a method of implementing synchronization where BeanProcessor is used in :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`, use this in this tutorial.



Session time out
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Session time out needs to be set when session is used.
If timeout duration is too long, unnecessary resources are retained in the memory and
if timeout duration is too small, user friendliness is lowered.
Therefore, appropriate time needs to be set as per the requirement.

This tutorial is also well-equipped with memory resources, set the default value of AP server to 30 minutes.

Further, handling the requests after session timeout also needs to be reviewed.
The guideline mentions a method to handle the requests after session timeout in :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement`.


Settings are done so as to transit to the login screen after timeout in this tutorial.




Overall application design
--------------------------------------------------------------------------------

Overall image diagram of the final application design is shown below.


.. figure:: images/materialSessionTutorialDesignOverview.png
   :alt: overview of design
   :width: 95%


Project configuration
================================================================================


Project creation
--------------------------------------------------------------------------------

As stated already, this tutorial starts with the status that some functions are created.
Therefore, proceed with development using already created project.

Created project can be fetched with the following procedure.

#. Access to `tutorial-apps <https://github.com/terasolunaorg/tutorial-apps>`_.
#. Click 'Branch' button to select Branch of the required version and click 'Download ZIP' button to download zip file
#. Extract zip file and import the project in it.


Note that since the method to import the project is described in :doc:`./TutorialTodo`,
the description is omitted in this tutorial.


Project configuration
--------------------------------------------------------------------------------

It states about the configuration of the initial project fetched using git.
Only the differences between the project fetched and the blank project are shown below.


.. code-block:: console

    session-tutorial-init-domain
        └-- src
            └-- main
                 ├-- java
                 │   └-- com
                 │       └-- example
                 │           └-- session
                 │               └-- domain
                 │                   ├-- model  ... (1)
                 │                   │  ├-- Account.java  ... (2)
                 │                   │  ├-- Cart.java  ... (3)
                 │                   │  ├-- CartItem.java  ... (3)
                 │                   │  ├-- Goods.java
                 │                   │  ├-- Order.java  ... (4)
                 │                   │  └-- OrderLine.java  ... (4)
                 │                   ├-- repository  ... (5)
                 │                   │  ├-- account
                 │                   │  │  └-- AccountRepository.java
                 │                   │  ├-- goods
                 │                   │  │  └-- GoodsRepository.java
                 │                   │  └-- order
                 │                   │      └-- OrderRepository.java
                 │                   └-- service  ... (6)
                 │                       ├-- account
                 │                       │  └-- AccountService.java
                 │                       ├-- goods
                 │                       │  └-- GoodsService.java
                 │                       ├-- order
                 │                       │  ├-- EmptyCartOrderException.java
                 │                       │  ├-- InvalidCartOrderException.java
                 │                       │  └-- OrderService.java
                 │                       └-- userdetails
                 │                           ├-- AccountDetails.java
                 │                           └-- AccountDetailsService.java
                 └-- resources
                      ├-- com
                      │  └-- example
                      │      └-- session
                      │          └-- domain
                      │              └-- repository  ... (7)
                      │                  ├-- account
                      │                  │  └-- AccountRepository.xml
                      │                  ├-- goods
                      │                  │  └-- GoodsRepository.xml
                      │                  └-- order
                      │                      └-- OrderRepository.xml
                      └-- META-INF
                           ├-- dozer
                           │  └-- order-mapping.xml  ... (8)
                           └-- spring
                               └-- session-tutorial-init-codelist.xml  ... (9)




.. tabularcolumns::  |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80
   
   * - Sr. No.
     - Description
   * - | (1)
     - | Package to handle model used in this application.
       | It describes the model required to be understood for proceeding with the tutorial below in detail.
   * - | (2)
     - | Class to save user account information.
   * - | (3)
     - | Class to save information of product registered in cart by user.
       | 'Cart' manages the whole thing and 'CartItem' manages the individual products.
   * - | (4)
     - | Class to save information of product ordered by user.
       | 'Order' manages the whole thing and 'OrderLine' manages the individual products.
   * - | (5)
     - | Package to handle repository used in this application.
   * - | (6)
     - | Package to handle service used in this application.
   * - | (7)
     - | Directory to store mapping file used in repository.
   * - | (8)
     - | Mapping definition file of Dozer (Bean Mapper).
       | Conversion from 'Cart' to 'Order' is defined.
   * - | (9)
     - | Bean definition file in which code list used in this application is defined.





.. code-block:: console

    session-tutorial-init-env
        └-- src
            └-- main
                 └-- resources
                     └-- database  ... (1)
                         ├-- H2-dataload.sql
                         └-- H2-schema.sql



.. tabularcolumns::  |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80
   
   * - File name
     - Description
   * - | (1)
     - | Directory to store SQL in order to setup in-memory database (H2 Database) in this application.


.. code-block:: console

    session-tutorial-init-web
        └-- src
            └-- main
                 ├-- java
                 │   └-- com
                 │       └-- example
                 │           └-- session
                 │               └-- app  ... (1)
                 │                   ├-- account 
                 │                   │  ├-- AccountCreateController.java 
                 │                   │  ├-- AccountCreateForm.java 
                 │                   │  ├-- IlleagalOperationException.java  
                 │                   │  └-- IlleagalOperationExceptionHandler.java
                 │                   ├-- goods
                 │                   │  ├-- GoodsController.java  
                 │                   │  └-- GoodsViewForm.java
                 │                   ├-- login
                 │                   │  └-- LoginController.java
                 │                   └-- validation
                 │                       ├-- Confirm.java
                 │                       └-- ConfirmValidator.java
                 ├-- resources
                 │   ├-- i18n
                 │   │  └-- application-messages.properties  ... (2)
                 │   ├-- META-INF
                 │   │   └-- spring  ... (3)
                 │   │       ├-- spring-mvc.xml
                 │   │       └-- spring-security.xml
                 │   └-- ValidationMessages.properties  ... (2)
                 └-- webapp
                      ├-- resources  ... (4)
                      │  ├-- app
                      │  │  └-- css
                      │  │      └-- styles.css
                      │  └-- vendor
                      │      └-- bootstrap-3.0.0
                      │          └-- css
                      │              └-- bootstrap.css
                      └-- WEB-INF
                          └-- views  ... (5)
                              ├-- account
                              │  ├-- createConfirm.jsp
                              │  ├-- createFinish.jsp
                              │  └-- createForm.jsp
                              ├-- common
                              │  ├-- error
                              │  │  └-- illegalOperationError.jsp
                              │  └-- include.jsp
                              ├-- goods
                              │  ├-- showGoods.jsp
                              │  └-- showGoodsDetails.jsp
                              └-- login
                                  └-- loginForm.jsp


.. tabularcolumns::  |p{0.20\linewidth}|p{0.80\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 20 80
   
   * - Sr. No.
     - Description
   * - | (1)
     - | Package to store the class in application layer used in this application.
   * - | (2)
     - | Property file in which message used in this application is defined
   * - | (3)
     - | Bean definition file in which component used in this application is defined
   * - | (4)
     - | Static resource file used in this application
   * - | (5)
     - | Directory in which jsp used in this application is stored


Operation verification
--------------------------------------------------------------------------------

Check the operation of project fetched before the application development.
Start the application server with the project imported in STS as the target
The method to start the application server is omitted in this tutorial
since it is described in :doc:`./TutorialTodo`.

The following screen is displayed when `<http://localhost:8080/session-tutorial-init-web/loginForm>`_ is accessed after the application server is started.

.. figure:: images/materialSessionTutorialLoginPage.png
   :alt: Login Page
   :width: 40%
   
Account can be created when "here" link on the login screen is selected.

.. figure:: images/materialSessionTutorialCreateAccountPages.png
   :alt: Account Create Pages
   :width: 95%

When (E-mail="a@b.com", Password="demo") is input in the form on the login screen, login can be done.
Product list is displayed after the login.
The product details can be displayed when the product name is selected.

.. figure:: images/materialSessionTutorialViewItemPages.png
   :alt: Item View Pages
   :width: 65%
   
   

Creating simple EC site application
================================================================================




Create account information change function
--------------------------------------------------------------------------------

Create a function that allows the user to input the information and updates the account information.

Manage the Change account information using "@SessionAttributes annotation" as described in :ref:`development_policy`.

The information of the screen implemented in the Change account information function is shown below.

 .. tabularcolumns::  |p{0.30\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 30 15 25 30
    
    * - Process name
      - HTTP method
      - Path
      - Screen
    * - | Account information change screen 1 display process
      - | GET
      - | /account/update?form1
      - | /account/updateForm1
    * - | Account information change screen 2 display process
      - | GET
      - | /account/update?form2
      - | /account/updateForm2
    * - | Account information change confirmation screen display process
      - | GET
      - | /account/update?confirm
      - | /account/updateConfirm
    * - | Account information change process
      - | POST
      - | /account/update
      - | Redirect to Account information change completion screen display process
    * - | Account information change completion screen display process
      - | GET
      - | /account/update?finish
      - | /account/updateFinish
    * - | Process to return to Account information change screen 1
      - | GET
      - | /account/update?redoform1
      - | /account/updateForm1
    * - | Process to return to Account information change screen 2
      - | GET
      - | /account/update?redoform2
      - | /account/updateForm2
    * - | Process to return to Home
      - | GET
      - | /account/update?home
      - | Redirect to Home screen display process


Creating form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a class to retain the account change information.

"/session-tutorial-init-web/src/main/java/com/example/session/app/account/AccountUpdateForm.java"

.. code-block:: java
 
    package com.example.session.app.account;
     
    import java.io.Serializable;
    import java.util.Date;
     
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Size;
     
    import org.hibernate.validator.constraints.Email;
    import org.springframework.format.annotation.DateTimeFormat;
     
    public class AccountUpdateForm implements Serializable {  // (1)
     
        /**
         *
         */
        private static final long serialVersionUID = 1L;
     
        private String id;
     
        // (2)
        @NotNull(groups = { Wizard1.class })
        @Size(min = 1, max = 255, groups = { Wizard1.class })
        private String name;
     
        @NotNull(groups = { Wizard1.class })
        @Size(min = 1, max = 255, groups = { Wizard1.class })
        @Email(groups = { Wizard1.class })
        private String email;
     
        @NotNull(groups = { Wizard1.class })
        @DateTimeFormat(iso = DateTimeFormat.ISO.DATE)
        private Date birthday;
     
        @NotNull(groups = { Wizard1.class })
        @Size(min = 7, max = 7, groups = { Wizard1.class })
        private String zip;
     
        @NotNull(groups = { Wizard1.class })
        @Size(min = 1, max = 255, groups = { Wizard1.class })
        private String address;
     
        @Size(min = 16, max = 16, groups = { Wizard2.class })
        private String cardNumber;
     
        @DateTimeFormat(pattern = "yyyy-MM")
        private Date cardExpirationDate;
     
        @Size(min = 1, max = 255, groups = { Wizard2.class })
        private String cardSecurityCode;
     
        public String getId() {
            return id;
        }
     
        public void setId(String id) {
            this.id = id;
        }
     
        public String getName() {
            return name;
        }
     
        public void setName(String name) {
            this.name = name;
        }
     
        public String getEmail() {
            return email;
        }
     
        public void setEmail(String email) {
            this.email = email;
        }
     
        public Date getBirthday() {
            return birthday;
        }
     
        public void setBirthday(Date birthday) {
            this.birthday = birthday;
        }
     
        public String getZip() {
            return zip;
        }
     
        public void setZip(String zip) {
            this.zip = zip;
        }
     
        public String getAddress() {
            return address;
        }
     
        public void setAddress(String address) {
            this.address = address;
        }
     
        public String getCardNumber() {
            return cardNumber;
        }
     
        public void setCardNumber(String cardNumber) {
            this.cardNumber = cardNumber;
        }
     
        public Date getCardExpirationDate() {
            return cardExpirationDate;
        }
     
        public void setCardExpirationDate(Date cardExpirationDate) {
            this.cardExpirationDate = cardExpirationDate;
        }
     
        public String getCardSecurityCode() {
            return cardSecurityCode;
        }
     
        public void setCardSecurityCode(String cardSecurityCode) {
            this.cardSecurityCode = cardSecurityCode;
        }
     
        public String getLastFourOfCardNumber() {
            if (cardNumber == null) {
                return "";
            }
            return cardNumber.substring(cardNumber.length() - 4);
        }
     
        public static interface Wizard1 {
     
        }
     
        public static interface Wizard2 {
     
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Implement Serializable in advance to store the instance of this class in the session.
    * - | (2)
      - | Make validation groups to specify the target of the input check for each screen transition.
        | In the example above, 2 groups are created implement the input check corresponding to the input item on the 1st page and the input item on the 2nd page respectively.


Creating Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create the Controller.
The description in which the form that receives the input information is managed using "@SessionAttributes" annotation is required in the Controller.

"/session-tutorial-init-web/src/main/java/com/example/session/app/account/AccountUpdateController.java"

.. code-block:: java

    package com.example.session.app.account;

    import javax.inject.Inject;

    import org.dozer.Mapper;
    import org.springframework.beans.propertyeditors.StringTrimmerEditor;
    import org.springframework.security.core.annotation.AuthenticationPrincipal;
    import org.springframework.stereotype.Controller;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.WebDataBinder;
    import org.springframework.web.bind.annotation.InitBinder;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.SessionAttributes;
    import org.springframework.web.bind.support.SessionStatus;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import com.example.session.app.account.AccountUpdateForm.Wizard1;
    import com.example.session.app.account.AccountUpdateForm.Wizard2;
    import com.example.session.domain.model.Account;
    import com.example.session.domain.service.account.AccountService;
    import com.example.session.domain.service.userdetails.AccountDetails;

    @Controller
    @SessionAttributes(value = { "accountUpdateForm" }) // (1)
    @RequestMapping("account")
    public class AccountUpdateController {

        @Inject
        AccountService accountService;

        @Inject
        Mapper beanMapper;

        @InitBinder
        public void initBinder(WebDataBinder binder) {
            binder.registerCustomEditor(String.class, new StringTrimmerEditor(true));
        }

        @ModelAttribute(value = "accountUpdateForm") // (2)
        public AccountUpdateForm setUpAccountForm() {
            return new AccountUpdateForm();
        }

        @RequestMapping(value = "update", params = "form1")
        public String showUpdateForm1(
                @AuthenticationPrincipal AccountDetails userDetails,
                AccountUpdateForm form) { // (3)

            Account account = accountService.findOne(userDetails.getAccount()
                    .getEmail());
            beanMapper.map(account, form);

            return "account/updateForm1";
        }

        @RequestMapping(value = "update", params = "form2")
        public String showUpdateForm2(
                @Validated((Wizard1.class)) AccountUpdateForm form,
                BindingResult result) {

            if (result.hasErrors()) {
                return "account/updateForm1";
            }

            return "account/updateForm2";
        }

        @RequestMapping(value = "update", params = "redoForm1")
        public String redoUpdateForm1() {
            return "account/updateForm1";
        }

        @RequestMapping(value = "update", params = "confirm")
        public String confirmUpdate(
                @Validated(Wizard2.class) AccountUpdateForm form,
                BindingResult result) {

            if (result.hasErrors()) {
                return "account/updateForm2";
            }

            return "account/updateConfirm";
        }

        @RequestMapping(value = "update", params = "redoForm2")
        public String redoUpdateForm2() {
            return "account/updateForm2";
        }

        @RequestMapping(value = "update", method = RequestMethod.POST)
        public String update(
                @AuthenticationPrincipal AccountDetails userDetails,
                @Validated({ Wizard1.class, Wizard2.class }) AccountUpdateForm form,
                BindingResult result, RedirectAttributes attributes, SessionStatus sessionStatus) {

            if (result.hasErrors()) {
                ResultMessages messages = ResultMessages.error();
                messages.add("e.st.ac.5001");
                throw new IllegalOperationException(messages);
            }

            Account account = beanMapper.map(form, Account.class);
            accountService.update(account);
            userDetails.setAccount(account);
            attributes.addFlashAttribute("account", account);
            sessionStatus.setComplete();  // (4)

            return "redirect:/account/update?finish";
        }

        @RequestMapping(value = "update", method = RequestMethod.GET, params = "finish")
        public String finishUpdate() {
            return "account/updateFinish";
        }

        @RequestMapping(value = "update", method = RequestMethod.GET, params = "home")
        public String home(SessionStatus sessionStatus) {
            sessionStatus.setComplete();
            return "redirect:/goods";
        }

    }



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the attribute name of the object to be stored in the session, in the value attribute of "@SessionAttributes" annotation.
        | The object having the attribute name ``"accountUpdateForm"`` is stored in the session in the above example.
    * - | (2)
      - | Specify the attribute name to be stored in the Model object in the value attribute.
        | The returned object is stored in the session by the attribute name as ``"accountUpdateForm"`` in the above example.
        | Since the "@ModelAttribute" annotated method can no longer be called by the request after the object is stored in the session when the value attribute is specified, it has a merit that unnecessary object is not generated.
    * - | (3)
      - | In order to use the object managed by "@SessionAttributes" annotation, add argument in the method so that the object can be received.
        | Use "@Validated" annotation when the input check is required.
        | The object that contains ``"accountUpdateForm"`` that is the default attribute name of "AccountUpdateForm", in the attribute name is passed as an argument in the above example.
    * - | (4)
      - | Call "setComplete" method of "SessionStatus" object and delete the object from the session.
      

.. warning:: 

    The object managed using "@SessionAttributes" annotation continues to remain in the session unless it is explicitly deleted.
    Therefore, the data retained even when the transition has taken place outside the screen handled by the Controller and returned again, can be browsed.
    The data that is no longer required, should always be deleted to avoid the memory exhaustion.


.. warning::

    When the user goes back using the browser button, inputs the URL directly and moves from one screen to another, it is required to note the point that "setComplete" method is not called and the session remains active without being cleared.

Creating JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a screen to pass data to the form object managed by "@SessionAttributes" annotation.

Input screen on the 1st page

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/account/updateForm1.jsp"

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Account Update Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>

        <div class="container">
            <%-- (1) --%>
            <form:form action="${pageContext.request.contextPath}/account/update"
                method="post" modelAttribute="accountUpdateForm">

                <h2>Account Update Page 1/2</h2>
                <table>
                    <tr>
                        <td><form:label path="name" cssErrorClass="error-label">name</form:label></td>
                        <%-- (2) --%>
                        <td><form:input path="name" cssErrorClass="error-input" /> <form:errors
                                path="name" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td><form:label path="email" cssErrorClass="error-label">e-mail</form:label></td>
                        <td><form:input path="email" cssErrorClass="error-input" /> <form:errors
                                path="email" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td><form:label path="birthday" cssErrorClass="error-label">birthday</form:label></td>
                        <td><fmt:formatDate value="${accountUpdateForm.birthday}"
                                pattern="yyyy-MM-dd" var="formattedBirthday" /> <input
                            type="date" id="birthday" name="birthday"
                            value="${formattedBirthday}"> <form:errors path="birthday"
                                cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td><form:label path="zip" cssErrorClass="error-label">zip</form:label></td>
                        <td><form:input path="zip" cssErrorClass="error-input" /> <form:errors
                                path="zip" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td><form:label path="address" cssErrorClass="error-label">address</form:label></td>
                        <td><form:input path="address" cssErrorClass="error-input" />
                            <form:errors path="address" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td>&nbsp;</td>
                        <td><input type="submit" name="form2" id="next" value="next" /></td>
                    </tr>
                </table>
            </form:form>

            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="home" id="home" value="home" />
            </form>
        </div>
    </body>
    </html>



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the attribute name of the form object that receives the input data, in modelAttribute.
        | The object with the attribute name ``"accountUpdateForm"`` receives the input data in the above example.
    * - | (2)
      - | Specify the element name of the object that stores the input data in path attribute of form:input tag.
        | If this method is used, when data already exists in the element name of the specified object, that value becomes as the default value of the input form.



Input screen on 2nd page

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/account/updateForm2.jsp"

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Account Update Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>

        <div class="container">

            <form:form action="${pageContext.request.contextPath}/account/update"
                method="post" modelAttribute="accountUpdateForm">

                <h2>Account Update Page 2/2</h2>
                <table>
                    <tr>
                        <td><form:label path="cardNumber" cssErrorClass="error-label">your card number</form:label></td>
                        <td><form:input path="cardNumber" cssErrorClass="error-input" />
                            <form:errors path="cardNumber" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td><form:label path="cardExpirationDate"
                                cssErrorClass="error-label">expiration date of
                                your card</form:label></td>
                        <td><fmt:formatDate
                                value="${accountUpdateForm.cardExpirationDate}" pattern="yyyy-MM"
                                var="formattedCardExpirationDate" /><input type="month"
                            name="cardExpirationDate" id="cardExpirationDate"
                            value="${formattedCardExpirationDate}"> <form:errors
                                path="cardExpirationDate" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td><form:label path="cardSecurityCode"
                                cssErrorClass="error-label">security code of
                                your card</form:label></td>
                        <td><form:input path="cardSecurityCode"
                                cssErrorClass="error-input" /> <form:errors
                                path="cardSecurityCode" cssClass="error-messages" /></td>
                    </tr>
                    <tr>
                        <td>&nbsp;</td>
                        <td><input type="submit" name="redoForm1" id="back"
                            value="back" /><input type="submit" name="confirm" id="confirm"
                            value="confirm" /></td>
                    </tr>
                </table>
            </form:form>

            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="home" id="home" value="home" />
            </form>
        </div>
    </body>
    </html>


Confirmation screen

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/account/updateConfirm.jsp"

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Account Update Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div class="container">

            <form:form action="${pageContext.request.contextPath}/account/update"
                method="post">

                <h3>Your account will be updated with below information. Please
                    push "update" button if it's OK.</h3>
                <table>
                    <tr>
                        <td><label for="name">name</label></td>
                        <td><span id="name">${f:h(accountUpdateForm.name)}</span></td>
                    </tr>
                    <tr>
                        <td><label for="email">e-mail</label></td>
                        <td><span id="email">${f:h(accountUpdateForm.email)}</span></td>
                    </tr>
                    <tr>
                        <td><label for="birthday">birthday</label></td>
                        <td><span id="birthday"><fmt:formatDate
                                    value="${accountUpdateForm.birthday}" pattern="yyyy-MM-dd" /></span></td>
                    </tr>
                    <tr>
                        <td><label for="zip">zip</label></td>
                        <td><span id="zip">${f:h(accountUpdateForm.zip)}</span></td>
                    </tr>
                    <tr>
                        <td><label for="address">address</label></td>
                        <td><span id="address">${f:h(accountUpdateForm.address)}</span></td>
                    </tr>
                    <tr>
                        <td><label for="cardNumber">your card number</label></td>
                        <td><span id="cardNumber">****-****-****-${f:h(accountUpdateForm.lastFourOfCardNumber)}</span></td>
                    </tr>
                    <tr>
                        <td><label for="cardExpirationDate">expiration date of
                                your card</label></td>
                        <td><span id="cardExpirationDate"><fmt:formatDate
                                    value="${accountUpdateForm.cardExpirationDate}"
                                    pattern="yyyy-MM" /></span></td>
                    </tr>
                    <tr>
                        <td><label for="cardSecurityCode">security code of
                                your card</label></td>
                        <td><span id="cardSecurityCode">${f:h(accountUpdateForm.cardSecurityCode)}</span></td>
                    </tr>
                    <tr>
                        <td>&nbsp;</td>
                        <td><input type="submit" name="redoForm2" id="back"
                            value="back" /><input type="submit" id="update" value="update" /></td>
                    </tr>
                </table>
            </form:form>


            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="home" id="home" value="home" />
            </form>
        </div>
    </body>
    </html>


Completion screen

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/account/updateFinish.jsp"

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Account Update Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>
        <div class="container">

            <h3>Your account has updated.</h3>
            <table>
                <tr>
                    <td><label for="name">name</label></td>
                    <td><span id="name">${f:h(account.name)}</span></td>
                </tr>
                <tr>
                    <td><label for="email">e-mail</label></td>
                    <td><span id="email">${f:h(account.email)}</span></td>
                </tr>
                <tr>
                    <td><label for="birthday">birthday</label></td>
                    <td><span id="birthday"><fmt:formatDate
                                value="${account.birthday}" pattern="yyyy-MM-dd" /></span></td>
                </tr>
                <tr>
                    <td><label for="zip">zip</label></td>
                    <td><span id="zip">${f:h(account.zip)}</span></td>
                </tr>
                <tr>
                    <td><label for="address">address</label></td>
                    <td><span id="address">${f:h(account.address)}</span></td>
                </tr>
                <tr>
                    <td><label for="cardNumber">your card number</label></td>
                    <td><span id="cardNumber">****-****-****-${f:h(account.lastFourOfCardNumber)}</span></td>
                </tr>
                <tr>
                    <td><label for="cardExpirationDate">expiration date of
                            your card</label></td>
                    <td><span id="cardExpirationDate"><fmt:formatDate
                                value="${account.cardExpirationDate}" pattern="yyyy-MM" /></span></td>
                </tr>
                <tr>
                    <td><label for="cardSecurityCode">security code of your
                            card</label></td>
                    <td><span id="cardSecurityCode">${f:h(account.cardSecurityCode)}</span></td>
                </tr>
            </table>

            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="home" id="home" value="home" />
            </form>

        </div>
    </body>
    </html>

Checking operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The account information can be updated using the implementation so far.
The transition takes place to the "Update account information" screen by clicking 'Account Update' button in the upper part of the product list display screen.
At present, the information of account to which the user is logged in, is displayed in the form as the initial value.
Finally the account information is updated when the user changes the form value and proceeds to the next screen.

Since the form that receives the input value is stored in the session with the implementation so far,
passing of data can be easily implemented.
Further, the changed information is reset if the transition takes place to the Update account information screen after clicking 'home' button
since the session is discarded when 'home' button is clicked.




Create cart item registration function
--------------------------------------------------------------------------------

Create a function to register the products for a spe cified quantity in the cart.

Manage the cart information as a Bean for session scope as explained in :ref:`development_policy`.

The information of screen implemented in cart item registration function is shown below.

 .. tabularcolumns::  |p{0.30\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 30 15 25 30
    
    * - Process name
      - HTTP method
      - Path
      - Screen
    * - | Process to add a product to the cart
      - | POST
      - | /addToCart
      - | Redirect to Product list screen display process


Defining the Bean for session scope
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Object to retain the cart information is already created as "Cart.java".
Therefore, add the settings so as to enable handling this object as the Bean for session scope.

There are 2 types of configuration methods mentioned in :doc:`../ArchitectureInDetail/WebApplicationDetail/SessionManagement` as the methods to use the Bean for session scope.
Define the Bean using component-scan in this tutorial.


.. warning::
    
    The target object needs to be 'Serializable' to register as the Bean for session scope

To define the Bean for session scope using component-scan, 
the following annotation should be added to the class to be registered as Bean.


"/session-tutorial-init-domain/src/main/java/com/example/session/domain/model/Cart.java"

.. code-block:: java
    :emphasize-lines: 17-18

    package com.example.session.domain.model;

    import java.io.Serializable;
    import java.security.MessageDigest;
    import java.security.NoSuchAlgorithmException;
    import java.util.Collection;
    import java.util.LinkedHashMap;
    import java.util.Map;
    import java.util.Set;

    import org.springframework.context.annotation.Scope;
    import org.springframework.context.annotation.ScopedProxyMode;
    import org.springframework.security.crypto.codec.Base64;
    import org.springframework.stereotype.Component;
    import org.springframework.util.SerializationUtils;

    @Component // (1)
    @Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS) // (2)
    public class Cart implements Serializable {

        // omitted

    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ "@Component"\  annotation so that it becomes the target for component-scan.
    * - | (2)
      - | Set Bean scope to \ ``"session"``\ . Further, specify \ ``"ScopedProxyMode.TARGET_CLASS"``\  using proxyMode attribute and enable scoped-proxy.

Further, the base-package that is a target for component-scan, needs to be specified in the Bean definition file.
However, since the following is already mentioned in the Bean definition file created in this tutorial, it is not required to add a new description.

"/session-tutorial-init-domain/src/main/resources/META-INF/spring/session-tutorial-init-domain.xml"

.. code-block:: jsp

    <!-- (1) -->
    <context:component-scan base-package="com.example.session.domain" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the package used as a target for component-scan.


Creating the form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a class to retain the information of ordered products.

"/session-tutorial-init-web/src/main/java/com/example/session/app/goods/GoodAddForm.java"

.. code-block:: java

    package com.example.session.app.goods;

    import java.io.Serializable;

    import javax.validation.constraints.Min;
    import javax.validation.constraints.NotNull;

    public class GoodAddForm implements Serializable {

        /**
         *
         */
        private static final long serialVersionUID = 1L;

        @NotNull
        private String goodsId;

        @NotNull
        @Min(1)
        private int quantity;

        public String getGoodsId() {
            return goodsId;
        }

        public void setGoodsId(String goodsId) {
            this.goodsId = goodsId;
        }

        public int getQuantity() {
            return quantity;
        }

        public void setQuantity(int quantity) {
            this.quantity = quantity;
        }
    }


Creating Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create the Controller.

Since it is already created to process the partial request, the following code is added.

"/session-tutorial-init-web/src/main/java/com/example/session/app/goods/GoodsController.java"

.. code-block:: java
    :emphasize-lines: 9-10, 15-16, 18-19, 30-32, 57-75

    package com.example.session.app.goods;

    import javax.inject.Inject;

    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import com.example.session.domain.model.Cart;
    import com.example.session.domain.model.CartItem;
    import com.example.session.domain.model.Goods;
    import com.example.session.domain.service.goods.GoodsService;

    @Controller
    @RequestMapping("goods")
    public class GoodsController {

        @Inject
        GoodsService goodsService;

        // (1)
        @Inject
        Cart cart;

        @ModelAttribute(value = "goodViewForm")
        public GoodViewForm setUpCategoryId() {
            return new GoodViewForm();
        }

        @RequestMapping(value = "", method = RequestMethod.GET)
        String showGoods(GoodViewForm form, Pageable pageable, Model model) {

            Page<Goods> page = goodsService.findByCategoryId(form.getCategoryId(),
                    pageable);
            model.addAttribute("page", page);
            return "goods/showGoods";
        }

        @RequestMapping(value = "/{goodsId}", method = RequestMethod.GET)
        public String showGoodsDetail(@PathVariable String goodsId, Model model) {

            Goods goods = goodsService.findOne(goodsId);
            model.addAttribute(goods);

            return "/goods/showGoodsDetail";
        }

        @RequestMapping(value = "/addToCart", method = RequestMethod.POST)
        public String addToCart(@Validated GoodAddForm form, BindingResult result,
                RedirectAttributes attributes) {

            if (result.hasErrors()) {
                ResultMessages messages = ResultMessages.error()
                        .add("e.st.go.5001");
                attributes.addFlashAttribute(messages);
                return "redirect:/goods";
            }

            Goods goods = goodsService.findOne(form.getGoodsId());
            CartItem cartItem = new CartItem();
            cartItem.setGoods(goods);
            cartItem.setQuantity(form.getQuantity());
            cart.add(cartItem); // (2)

            return "redirect:/goods";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the Bean for session scope from DI container.
    * - | (2)
      - | Add the data in the Bean for session scope.
        | It is not necessary to add a object to Model to display the information in the screen.



Creating JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create JSP to display the contents of the cart.

Since JSP is also already created, the code shown below is added at the end of body tag.

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/goods/showGoods.jsp"

.. code-block:: jsp
    :emphasize-lines: 45, 53-59, 72-97

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Item List Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/vendor/bootstrap-3.0.0/css/bootstrap.css"
        type="text/css" media="screen, projection">
    </head>
    <body>

        <sec:authentication property="principal" var="userDetails" />
        <div style="display: inline-flex">
            welcome&nbsp;&nbsp; <span id="userName">${f:h(userDetails.account.name)}</span>
            <form method="post" action="${pageContext.request.contextPath}/logout">
                <input type="submit" id="logout" value="logout" />
                <sec:csrfInput />
            </form>
            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="form1" id="updateAccount"
                    value="Account Update" />
            </form>
        </div>
        <br>
        <br>

        <div class="container">
            <p>select a category</p>

            <form:form method="get"
                action="${pageContext.request.contextPath}/goods/"
                modelAttribute="goodViewForm">
                <form:select path="categoryId" items="${CL_CATEGORIES}" />
                <input type="submit" id="update" value="update" />
            </form:form>
            <br />
            <t:messagesPanel />
            <table>
                <tr>
                    <th>Name</th>
                    <th>Price</th>
                    <th>Quantity</th>
                </tr>
                <c:forEach items="${page.content}" var="goods" varStatus="status">
                    <tr>
                        <td><a id="${f:h(goods.name)}"
                            href="${pageContext.request.contextPath}/goods/${f:h(goods.id)}">${f:h(goods.name)}</a></td>
                        <td><fmt:formatNumber value="${f:h(goods.price)}"
                                type="CURRENCY" currencySymbol="&yen;" maxFractionDigits="0" /></td>
                        <td><form:form method="post"
                                action="${pageContext.request.contextPath}/goods/addToCart"
                                modelAttribute="goodAddForm">
                                <input type="text" name="quantity" id="quantity${status.index}" value="1" />
                                <input type="hidden" name="goodsId" value="${f:h(goods.id)}" />
                                <input type="submit" id="add${status.index}" value="add" />
                            </form:form></td>
                    </tr>
                </c:forEach>
            </table>
            <t:pagination page="${page}" outerElementClass="pagination" />
        </div>
        <div>
            <p>
                <fmt:formatNumber value="${page.totalElements}" />
                results <br> ${f:h(page.number + 1) } / ${f:h(page.totalPages)}
                Pages
            </p>
        </div>
        <div>
            <%-- (1) --%>
            <spring:eval var="cart" expression="@cart" />
            <form method="get" action="${pageContext.request.contextPath}/cart">
                <input type="submit" id="viewCart" value="view cart" />
            </form>
            <table>
                <%-- (2) --%>
                <c:forEach items="${cart.cartItems}" var="cartItem" varStatus="status">
                    <tr>
                        <td><span id="itemName${status.index}">${f:h(cartItem.goods.name)}</span></td>
                        <td><span id="itemPrice${status.index}"><fmt:formatNumber
                                    value="${cartItem.goods.price}" type="CURRENCY"
                                    currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                        <td><span id="itemQuantity${status.index}">${f:h(cartItem.quantity)}</span></td>
                    </tr>
                </c:forEach>
                <tr>
                    <td>Total</td>
                    <td><span id="totalPrice"><fmt:formatNumber
                                value="${f:h(cart.totalAmount)}" type="CURRENCY"
                                currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                    <td></td>
                </tr>
            </table>
        </div>

    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Store the Bean in a variable in order to display the contents of the Bean for session scope in a screen.
        | Cart object in session scope is stored in variable "cart" in the above example.
    * - | (2)
      - | Browse the contents of the Bean for session scope through the variable created in (1).
        | The contents of the Bean for session scope are browsed through variable "var" in the above example.

.. note::

     var attribute is not required if simply the contents of Bean are to be displayed alone without storing them in a variable.
     It can be displayed using "<spring:eval expression="@cart" />" in the above example.


"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/goods/showGoodsDetail.jsp"

.. code-block:: jsp
    :emphasize-lines: 44-51, 57-81

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Item List Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>

        <sec:authentication property="principal" var="userDetails" />
        <div style="display: inline-flex">
            welcome&nbsp;&nbsp; <span id="userName">${f:h(userDetails.account.name)}</span>
            <form:form method="post"
                action="${pageContext.request.contextPath}/logout">
                <input type="submit" id="logout" value="logout" />
            </form:form>
            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="form1" id="updateAccount"
                    value="Account Update" />
            </form>
        </div>
        <br>
        <br>

        <div class="container">

            <table>
                <tr>
                    <th>Name</th>
                    <td>${f:h(goods.name)}</td>
                </tr>
                <tr>
                    <th>Price</th>
                    <td><fmt:formatNumber value="${f:h(goods.price)}"
                            type="CURRENCY" currencySymbol="&yen;" maxFractionDigits="0" /></td>
                </tr>
                <tr>
                    <th>Description</th>
                    <td>${f:h(goods.description)}</td>
                </tr>
            </table>
            <form:form method="post"
                action="${pageContext.request.contextPath}/goods/addToCart"
                modelAttribute="AddToCartForm">
                Quantity<input type="text" id="quantity" name="quantity"
                    value="1" />
                <input type="hidden" name="goodsId" value="${f:h(goods.id)}" />
                <input type="submit" id="add" value="add" />
            </form:form>

            <form method="get" action="${pageContext.request.contextPath}/goods">
                <input type="submit" id="home" value="home" />
            </form>
        </div>
        <div>
            <spring:eval var="cart" expression="@cart" />
            <form method="get" action="${pageContext.request.contextPath}/cart">
                <input type="submit" value="view cart" />
            </form>
            <table>
                <c:forEach items="${cart.cartItems}" var="cartItem"
                    varStatus="status">
                    <tr>
                        <td><span id="itemName${status.index}">${f:h(cartItem.goods.name)}</span></td>
                        <td><span id="itemPrice${status.index}"><fmt:formatNumber
                                    value="${cartItem.goods.price}" type="CURRENCY"
                                    currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                        <td><span id="itemQuantity${status.index}">${f:h(cartItem.quantity)}</span></td>
                    </tr>
                </c:forEach>
                <tr>
                    <td>Total</td>
                    <td><span id="totalPrice"><fmt:formatNumber
                                value="${f:h(cart.totalAmount)}" type="CURRENCY"
                                currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                    <td></td>
                </tr>
            </table>
        </div>
    </body>
    </html>



Checking operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Products can be registered to the cart with the implementation so far.
Contents of the cart on the same page are displayed by clicking 'add' button for a product on the product list display screen.

Since the cart object is stored in the session with the implementation so far,
cart information is saved even though moved to the account information update screen and returned.


Create a mechanism to retain the product search information
--------------------------------------------------------------------------------

Products can be added to the cart with the implementation so far.
However, screen where the transition after adding products takes place, is usually the 1st page of 'book' category.

This tutorial has a specification to retain the product search information including the selection category and page number till the order is completed.
Therefore, modify the implementation so as to transit to the previous status after adding products or when returned from the account update screen.


Manage the product search information as the Bean for session scope as explained in :ref:`development_policy`.

Information of screen to be modified is shown below.

 .. tabularcolumns::  |p{0.30\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 30 15 25 30
    
    * - Process name
      - HTTP method
      - Path
      - Screen
    * - | Product list screen display process (default)
      - | GET
      - | /goods **(created)**
      - | /goods/showGoods
    * - | Product list screen display process (while selecting category)
      - | GET
      - | /goods?categoryId **(created)**
      - | /goods/showGoods
    * - | Product list screen display process (while selecting page)
      - | GET
      - | /goods?page **(created)**
      - | /goods/showGoods

Create session scope Bean
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create the session scope Bean to retain the product search information.
Define a bean using component-scan similar to the cart information.

"/session-tutorial-init-web/src/main/java/com/example/session/app/goods/GoodsSearchCriteria.java"

.. code-block:: java

    package com.example.session.app.goods;

    import java.io.Serializable;

    import org.springframework.context.annotation.Scope;
    import org.springframework.context.annotation.ScopedProxyMode;
    import org.springframework.stereotype.Component;

    @Component // (1)
    @Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS) // (2)
    public class GoodsSearchCriteria implements Serializable {

        /**
         * 
         */
        private static final long serialVersionUID = 1L;

        private int categoryId = 1;

        private int page = 0;

        public int getCategoryId() {
            return categoryId;
        }

        public void setCategoryId(int categoryId) {
            this.categoryId = categoryId;
        }

        public int getPage() {
            return page;
        }

        public void setPage(int page) {
            this.page = page;
        }

        public void clear() {
            categoryId = 1;
            page = 0;
        }

    }



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify \ "@Component"\  annotation so that it becomes the target for component-scan
    * - | (2)
      - | Set scope of Bean as \ ``"session"``\ . Further, specify \ ``"ScopedProxyMode.TARGET_CLASS"``\  in proxyMode attribute and enable scoped-proxy.


Further, it is required to specify base-package that is a target for component-scan in Bean definition file. However, since the following is already mentioned in the Bean definition file created in this tutorial, it is not required to add a new description.

"/session-tutorial-init-web/src/main/resources/META-INF/spring/spring-mvc.xml"

.. code-block:: xml

    <!-- (1) -->
    <context:component-scan base-package="com.example.session.app" />

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the package that is a target for component-scan.


Modifying Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modify the Controller so as to retain the product search information in the session and to use the product search information retained in the session.

"/session-tutorial-init-web/src/main/java/com/example/session/app/goods/GoodsController.java"

.. code-block:: java
    :emphasize-lines: 6, 34-36, 43-73

    package com.example.session.app.goods;

    import javax.inject.Inject;

    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    import org.springframework.data.domain.Pageable;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import com.example.session.domain.model.Cart;
    import com.example.session.domain.model.CartItem;
    import com.example.session.domain.model.Goods;
    import com.example.session.domain.service.goods.GoodsService;

    @Controller
    @RequestMapping("goods")
    public class GoodsController {

        @Inject
        GoodsService goodsService;

        @Inject
        Cart cart;

        // (1)
        @Inject
        GoodsSearchCriteria criteria;

        @ModelAttribute(value = "goodViewForm")
        public GoodViewForm setUpCategoryId() {
            return new GoodViewForm();
        }

        // (2)
        @RequestMapping(value = "", method = RequestMethod.GET)
        String showGoods(GoodViewForm form, Model model) {
            Pageable pageable = new PageRequest(criteria.getPage(), 3);
            form.setCategoryId(criteria.getCategoryId());
            return showGoods(pageable, model);
        }

        // (3)
        @RequestMapping(value = "", method = RequestMethod.GET, params = "categoryId")
        String changeCategoryId(GoodViewForm form, Pageable pageable, Model model) {
            criteria.setPage(pageable.getPageNumber());
            criteria.setCategoryId(form.getCategoryId());
            return showGoods(pageable, model);
        }

        // (4)
        @RequestMapping(value = "", method = RequestMethod.GET, params = "page")
        String changePage(GoodViewForm form, Pageable pageable, Model model) {
            criteria.setPage(pageable.getPageNumber());
            form.setCategoryId(criteria.getCategoryId());
            return showGoods(pageable, model);
        }

        // (5)
        String showGoods(Pageable pageable, Model model) {
            Page<Goods> page = goodsService.findByCategoryId(
                    criteria.getCategoryId(), pageable);
            model.addAttribute("page", page);
            return "goods/showGoods";
        }

        @RequestMapping(value = "/{goodsId}", method = RequestMethod.GET)
        public String showGoodsDetail(@PathVariable String goodsId, Model model) {

            Goods goods = goodsService.findOne(goodsId);
            model.addAttribute(goods);

            return "/goods/showGoodsDetail";
        }

        @RequestMapping(value = "/addToCart", method = RequestMethod.POST)
        public String addToCart(@Validated GoodAddForm form, BindingResult result,
                RedirectAttributes attributes) {

            if (result.hasErrors()) {
                ResultMessages messages = ResultMessages.error()
                        .add("e.st.go.5001");
                attributes.addFlashAttribute(messages);
                return "redirect:/goods";
            }

            Goods goods = goodsService.findOne(form.getGoodsId());
            CartItem cartItem = new CartItem();
            cartItem.setGoods(goods);
            cartItem.setQuantity(form.getQuantity());
            cart.add(cartItem);

            return "redirect:/goods";
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the Bean for session scope from DI container.
    * - | (2)
      - | Perform the pre-processing of the usual product list screen display process. Set the product category stored in the session to the form and the page number to \ "pageable"\ . The product category is set to the form in order to specify the product category displayed using the select box.
    * - | (3)
      - | Perform the pre-processing of the product list screen display process when the category is changed. Store the product category entered in the session. Specify the default 1st page of the page numbers in \ "pageable"\ .
    * - | (4)
      - | Perform the pre-processing of the product list screen display process when the page is changed. Store the page number entered in the session. Set the product category stored in the session in the form.
    * - | (5)
      - | Handle the common portion. Search a product based on product category managed in session and \ "pageable"\  fetched in the preprocessing.


Checking operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Product search information can be retained with the implementation so far.
For example, when a product is added to the cart on the 2nd page of 'music' category, the destination for transition remains the original 2nd page of 'music' category.
Further, when 'Account Update' button is clicked on the same screen, moved to the account update screen, 'home' button on the account update screen is clicked and returned, the destination for transition is just the original 2nd page of 'music' category.


Create cart item deletion function
--------------------------------------------------------------------------------

Create a function to delete the specified product from the cart.

Use checkbox to specify the product to be deleted.

The information of screen implemented using the cart item deletion function is shown below.

 .. tabularcolumns::  |p{0.30\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 30 15 25 30
    
    * - Process name
      - HTTP method
      - Path
      - Screen
    * - | Cart screen display process
      - | GET
      - | /cart
      - | cart/viewCart
    * - | Process to delete a product from the cart
      - | POST
      - | /cart
      - | Redirect to cart screen display process



Creating form object
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a class to retain ID of the product to be deleted.

"/session-tutorial-init-web/src/main/java/com/example/session/app/cart/CartForm.java"

.. code-block:: java

    package com.example.session.app.cart;

    import java.util.Set;

    import org.hibernate.validator.constraints.NotEmpty;

    public class CartForm {

        @NotEmpty
        private Set<String> removedItemsIds;

        public Set<String> getRemovedItemsIds() {
            return removedItemsIds;
        }

        public void setRemovedItemsIds(Set<String> removedItemsIds) {
            this.removedItemsIds = removedItemsIds;
        }
    }



Creating Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create Controller.

"/session-tutorial-init-web/src/main/java/com/example/session/app/cart/CartController.java"

.. code-block:: java

    package com.example.session.app.cart;

    import javax.inject.Inject;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.validation.BindingResult;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.ModelAttribute;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import com.example.session.domain.model.Cart;

    @Controller
    @RequestMapping("cart")
    public class CartController {

        // (1)
        @Inject
        Cart cart;

        @ModelAttribute
        CartForm setUpForm() {
            return new CartForm();
        }

        @RequestMapping(method = RequestMethod.GET)
        String viewCart(Model model) {
            return "cart/viewCart";
        }

        @RequestMapping(method = RequestMethod.POST)
        String removeFromCart(@Validated CartForm cartForm,
                BindingResult bindingResult, Model model) {
            if (bindingResult.hasErrors()) {
                ResultMessages messages = ResultMessages.error()
                        .add("e.st.ca.5001");
                model.addAttribute(messages);
                return viewCart(model);
            }
            cart.remove(cartForm.getRemovedItemsIds()); // (2)
            return "redirect:/cart";
        }
    }




.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the Bean for session scope from DI container.
    * - | (2)
      - | Delete the data in the Bean for session scope.


Creating JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Display the cart list and create JSP to select the product to be deleted.
Products can be ordered from this screen.

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/cart/viewCart.jsp"

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>View Cart Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>

        <sec:authentication property="principal" var="userDetails" />

        <div style="display: inline-flex">
            welcome ${f:h(userDetails.account.name)}
            <form:form method="post"
                action="${pageContext.request.contextPath}/logout">
                <input type="submit" id="logout" value="logout" />
            </form:form>
            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="form1" id="updateAccount"
                    value="Account Update" />
            </form>
        </div>
        <br>
        <br>

        <div>
            <spring:eval var="cart" expression="@cart" />
            <form:form method="post"
                action="${pageContext.request.contextPath}/cart"
                modelAttribute="cartForm">
                <form:errors path="removedItemsIds" cssClass="error-messages" />
                <t:messagesPanel />
                <table>
                    <tr>
                        <th>Name</th>
                        <th>Price</th>
                        <th>Quantity</th>
                        <th>Remove</th>
                    </tr>
                    <c:forEach items="${cart.cartItems}" var="cartItem"
                        varStatus="status">
                        <tr>
                            <td><span id="itemName${status.index}">${f:h(cartItem.goods.name)}</span></td>
                            <td><span id="itemPrice${status.index}"><fmt:formatNumber
                                        value="${cartItem.goods.price}" type="CURRENCY"
                                        currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                            <td><span id="itemQuantity${status.index}">${f:h(cartItem.quantity)}</span></td>
                            <%-- (1) --%>
                            <td><input type="checkbox" name="removedItemsIds"
                                id="removedItemsIds${status.index}"
                                value="${f:h(cartItem.goods.id)}" /></td>
                        </tr>
                    </c:forEach>
                    <tr>
                        <td>Total</td>
                        <td><span id="totalPrice"><fmt:formatNumber
                                    value="${f:h(cart.totalAmount)}" type="CURRENCY"
                                    currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                        <td></td>
                        <td></td>
                    </tr>
                </table>
                <input type="submit" id="remove" value="remove" />
            </form:form>
        </div>

        <div style="display: inline-flex">
            <form method="get" action="${pageContext.request.contextPath}/order">
                <input type="submit" id="confirm" name="confirm"
                    value="confirm your order" />
            </form>
            <form method="get" action="${pageContext.request.contextPath}/goods">
                <input type="submit" id="home" value="home" />
            </form>
        </div>
    </body>
    </html>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the product to be deleted using a checkbox.
        | When Delete button is clicked with the checkbox selected, ID of the corresponding product is sent to the server.

Checking operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Products registered in the cart can be deleted with the implementation so far.
Transition to the cart display screen takes place by clicking 'viewCart' button on the product list display screen.
Product can be deleted from the cart by checking the product to be deleted on the cart display screen and clicking 'remove' button.


Create the product order function
--------------------------------------------------------------------------------

Create a function to order the products registered in the cart.

Contents of the cart become blank after the order completion.

Information of the screen implemented in the product order function is shown below.

 .. tabularcolumns::  |p{0.30\linewidth}|p{0.15\linewidth}|p{0.25\linewidth}|p{0.30\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 30 15 25 30
    
    * - Process name
      - HTTP method
      - Path
      - Screen
    * - | Order confirmation screen display process
      - | GET
      - | /order?confirm
      - | order/confirm
    * - | Order processing
      - | POST
      - | /order
      - | Redirect to order completion screen display process
    * - | Order completion screen display process
      - | GET
      - | /order?finish
      - | order/finish


Creating Controller
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create the Controller.

"/session-tutorial-init-web/src/main/java/com/example/session/app/order/OrderController.java"

.. code-block:: java

    package com.example.session.app.order;

    import javax.inject.Inject;

    import org.springframework.http.HttpStatus;
    import org.springframework.security.core.annotation.AuthenticationPrincipal;
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.ResponseStatus;
    import org.springframework.web.servlet.ModelAndView;
    import org.springframework.web.servlet.mvc.support.RedirectAttributes;
    import org.terasoluna.gfw.common.exception.BusinessException;
    import org.terasoluna.gfw.common.message.ResultMessages;

    import com.example.session.app.goods.GoodsSearchCriteria;
    import com.example.session.domain.model.Cart;
    import com.example.session.domain.model.Order;
    import com.example.session.domain.service.order.EmptyCartOrderException;
    import com.example.session.domain.service.order.InvalidCartOrderException;
    import com.example.session.domain.service.order.OrderService;
    import com.example.session.domain.service.userdetails.AccountDetails;

    @Controller
    @RequestMapping("order")
    public class OrderController {

        @Inject
        OrderService orderService;

        // (1)
        @Inject
        Cart cart;

        @Inject
        GoodsSearchCriteria criteria;

        @RequestMapping(method = RequestMethod.GET, params = "confirm")
        String confirm(@AuthenticationPrincipal AccountDetails userDetails,
                Model model) {
            if (cart.isEmpty()) {
                ResultMessages messages = ResultMessages.error()
                        .add("e.st.od.5001");
                model.addAttribute(messages);
                return "cart/viewCart";
            }
            model.addAttribute("account", userDetails.getAccount());
            model.addAttribute("signature", cart.calcSignature());
            return "order/confirm";
        }

        @RequestMapping(method = RequestMethod.POST)
        String order(@AuthenticationPrincipal AccountDetails userDetails,
                @RequestParam String signature, RedirectAttributes attributes) {
            Order order = orderService.purchase(userDetails.getAccount(), cart,
                    signature); // (2)
            attributes.addFlashAttribute(order);
            criteria.clear(); // (3)
            return "redirect:/order?finish";
        }

        @RequestMapping(method = RequestMethod.GET, params = "finish")
        String finish() {
            return "order/finish";
        }

        // (4)
        @ExceptionHandler({ EmptyCartOrderException.class,
                InvalidCartOrderException.class })
        @ResponseStatus(HttpStatus.CONFLICT)
        ModelAndView handleOrderException(BusinessException e) {
            return new ModelAndView("common/error/businessError").addObject(e
                    .getResultMessages());
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Fetch the Bean for session scope from DI container.
    * - | (2)
      - | The contents of the Bean for session scope are made empty using the method of Service in the domain layer.
        | Accordingly, Bean for session scope is discarded.
        | Further, information in the Bean for session scope is used by the screen where the user transits after discarding Bean in this application.
        | Therefore, information that existed in the Bean for session scope is re-entered in a different object and added to the Flash scope.
    * - | (3)
      - | Product search information is returned to the default status.
    * - | (4)
      - | Since a Business exception may occur in the Service method, error handling is performed in this method.
        | Because of this, user transits to the specified error screen when a Business exception occurs.


.. warning::

    Method to discard the Bean for session scope is different from the method to discard the object managed by @SessionAttributes.
    Discarding the Bean for session scope should be assigned to DI container and should not be discarded by the application.
    Therefore, fields in the Bean for session scope just need to be reset to discard the Bean for session scope.
    Bean itself is discarded at the time of session timeout or logout.


Creating JSP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create JSP to display order details and payment information.

"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/order/confirm.jsp"

.. code-block:: jsp

    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Order Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>

        <sec:authentication property="principal" var="userDetails" />

        <div style="display: inline-flex">
            welcome ${f:h(userDetails.account.name)}
            <form:form method="post"
                action="${pageContext.request.contextPath}/logout">
                <input type="submit" id="logout" value="logout" />
            </form:form>
            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="form1" id="updateAccount"
                    value="Account Update" />
            </form>
        </div>
        <br>
        <br>

        <div>
            <spring:eval var="cart" expression="@cart" />

            <h3>Below items will be ordered. Please push "order" button if
                it's OK.</h3>
            <table>
                <tr>
                    <th>Name</th>
                    <th>Price</th>
                    <th>Quantity</th>
                </tr>
                <c:forEach items="${cart.cartItems}" var="cartItem"
                    varStatus="status">
                    <tr>
                        <td><span id="itemName${status.index}">${f:h(cartItem.goods.name)}</span></td>
                        <td><span id="itemPrice${status.index}"><fmt:formatNumber
                                    value="${cartItem.goods.price}" type="CURRENCY"
                                    currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                        <td><span id="itemQuantity${status.index}">${f:h(cartItem.quantity)}</span></td>
                    </tr>
                </c:forEach>
                <tr>
                    <td>Total</td>
                    <td><span id="totalPrice"><fmt:formatNumber
                                value="${f:h(cart.totalAmount)}" type="CURRENCY"
                                currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                    <td></td>
                </tr>
            </table>

            <table>
                <tr>
                    <td><label for="name">name</label></td>
                    <td><span id="name">${f:h(account.name)}</span></td>
                </tr>
                <tr>
                    <td><label for="email">e-mail</label></td>
                    <td><span id="email">${f:h(account.email)}</span></td>
                </tr>
                <tr>
                    <td><label for="zip">zip</label></td>
                    <td><span id="zip">${f:h(account.zip)}</span></td>
                </tr>
                <tr>
                    <td><label for="address">address</label></td>
                    <td><span id="address">${f:h(account.address)}</span></td>
                </tr>
                <tr>
                    <%-- (1) --%>
                    <td>payment</td>
                    <td><span id="payment"><c:choose>
                                <c:when test="${empty account.cardNumber}">
                                cash
                            </c:when>
                                <c:otherwise>
                                card (card number : ****-****-****-${f:h(account.lastFourOfCardNumber)})
                            </c:otherwise>
                            </c:choose></span></td>
                </tr>
            </table>
        </div>
        <div style="display: inline-flex">
            <form:form method="post"
                action="${pageContext.request.contextPath}/order">
                <input type="hidden" name="signature" value="${f:h(signature)}" />
                <input type="submit" id="order" value="order" />
            </form:form>
            <form method="get" action="${pageContext.request.contextPath}/cart">
                <input type="submit" id="back" value="back" />
            </form>
        </div>
        <div>
            <form method="get" action="${pageContext.request.contextPath}/goods">
                <input type="submit" id="home" value="home" />
            </form>
        </div>
    </body>
    </html>



.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr No.
      - Description
    * - | (1)
      - | Method of payment is card payment when card number is registered as the account information.
        | It is considered as cash payment when card number is not registered.


Create JSP to display the information after the confirmation of order.


"/session-tutorial-init-web/src/main/webapp/WEB-INF/views/order/finish.jsp"

.. code-block:: jsp


    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8" />
    <title>Order Page</title>
    <link rel="stylesheet"
        href="${pageContext.request.contextPath}/resources/app/css/styles.css">
    </head>
    <body>

        <sec:authentication property="principal" var="userDetails" />

        <div style="display: inline-flex">
            welcome ${f:h(userDetails.account.name)}
            <form:form method="post"
                action="${pageContext.request.contextPath}/logout">
                <input type="submit" id="logout" value="logout" />
            </form:form>
            <form method="get"
                action="${pageContext.request.contextPath}/account/update">
                <input type="submit" name="form1" id="updateAccount"
                    value="Account Update" />
            </form>
        </div>
        <br>
        <br>

        <div>

            <h3>Your order has been accepted</h3>
            <table>
                <tr>
                    <td><label for="orderNumber">order number</label></td>
                    <td><span id="orderNumber">${f:h(order.id)}</span></td>
                </tr>
                <tr>
                    <td><label for="orderDate">order date</label></td>
                    <td><span id="orderDate"><fmt:formatDate
                                value="${order.orderDate}" pattern="yyyy-MM-dd　hh:mm:ss" /></span></td>
                </tr>
            </table>
            <table>
                <tr>
                    <th>Name</th>
                    <th>Price</th>
                    <th>Quantity</th>
                </tr>
                <c:forEach items="${order.orderLines}" var="orderLine" varStatus="status">
                    <tr>
                        <td><span id="itemName${status.index}">${f:h(orderLine.goods.name)}</span></td>
                        <td><span id="itemPrice${status.index}"><fmt:formatNumber
                                    value="${orderLine.goods.price}" type="CURRENCY"
                                    currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                        <td><span id="itemQuantity${status.index}">${f:h(orderLine.quantity)}</span></td>
                    </tr>
                </c:forEach>
                <tr>
                    <td>Total</td>
                    <td><span id="totalPrice"><fmt:formatNumber
                                value="${f:h(order.totalAmount)}" type="CURRENCY"
                                currencySymbol="&yen;" maxFractionDigits="0" /></span></td>
                    <td></td>
                </tr>
            </table>
        </div>
        <div>
            <form method="get" action="${pageContext.request.contextPath}/goods">
                <input type="submit" id="home" value="home" />
            </form>
        </div>
    </body>
    </html>

Checking operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Products registered in the cart can be ordered with the implementation so far.
Transition to the order confirmation screen takes place by clicking 'confirm your order' button on the cart display screen.
Order is completed by clicking 'order' button on the order confirmation screen.

Cart object in the session when the order is completed, is deleted with the implementation so far.
Therefore, contents of the cart are cleared when the user returns to the product list screen after the order completion.


Settings for session synchronization and timeout
--------------------------------------------------------------------------------

Finally perform the settings for session synchronization and timeout.

Session synchronization is implemented using BeanProcessor.


"/session-tutorial-init-web/src/main/java/com/example/session/app/config/EnableSynchronizeOnSessionPostProcessor.java"

.. code-block:: java
    
    package com.example.session.app.config;

    import org.springframework.beans.BeansException;
    import org.springframework.beans.factory.config.BeanPostProcessor;
    import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter;

    public class EnableSynchronizeOnSessionPostProcessor implements
            BeanPostProcessor {

        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName)
                throws BeansException {
            return bean;
        }

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName)
                throws BeansException {
            if (bean instanceof RequestMappingHandlerAdapter) {
                RequestMappingHandlerAdapter adapter = (RequestMappingHandlerAdapter) bean;
                adapter.setSynchronizeOnSession(true); // (1)
            }
            return bean;
        }
    }


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Requests within the same session are synchronized by specifying true in the argument of setSynchronizeOnSession method.


"/session-tutorial-init-web/src/main/resources/META-INF/spring/spring-mvc.xml"

.. code-block:: xml
    
    <!-- Bean Processor -->
    <bean class="com.example.session.app.config.EnableSynchronizeOnSessionPostProcessor" />


Set the timeout time in web.xml.
Set the default value as 30 minutes.

"/session-tutorial-init-web/src/main/webapp/WEB-INF/web.xml" (set by default)

.. code-block:: xml
    
    <session-config>
        <!-- 30min -->
        <session-timeout>30</session-timeout>
    </session-config>


Use the Spring Security function for request detection after timeout.


"/session-tutorial-init-web/src/main/resources/META-INF/spring/spring-security.xml"

.. code-block:: xml
    
    <!-- (1) -->
    <sec:session-management invalid-session-url="/loginForm" />


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :widths: 10 90
    :header-rows: 1

    * - Sr. No.
      - Description
    * - | (1)
      - | Mention the destination for transition when request after the timeout is detected in invalid-session-url attribute of sec:session-management tag.




Conclusion
================================================================================

We have learnt the following contents in this tutorial.


* How to design the data for session management
    * Selecting data stored in the session
    * Example of the flow to determine whether to use the session
    * Discarding data during the session
* Specific method of using session in this FW
    * Method of using @SessionAttributes
    * Method of using the Bean for session scope
    * How to refer the data in the session in each method of usage
    * How to discard session in each method of usage



