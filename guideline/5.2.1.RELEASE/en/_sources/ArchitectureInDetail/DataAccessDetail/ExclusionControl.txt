Exclusive Control
================================================================================

.. only:: html

 .. contents:: Table of Contents
   :depth: 3
   :local:

|

Overview
--------------------------------------------------------------------------------
Exclusive control is a process to maintain data consistency when same data is to be updated simultaneously by multiple transactions.

Exclusive control should be performed when same data is likely to get updated simultaneously by multiple transactions.
These transactions are not necessarily restricted to database transactions only; they also include long transactions.

.. note:: **Long transactions**

    Long transactions are the transactions where data fetch and data update are performed as separate database transactions.

    To illustrate a specific example, the transactions are observed in an application where the fetched data is displayed on the Edit screen and the value edited on the screen is updated in database.

| This chapter explains about exclusive control for the data stored in the database.
| However, it should also be performed in a similar way for the data stored in data-store apart from database (such as memory, files etc.).

.. _ExclusionControl-Necessity:

Necessity of exclusive control
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To understand why exclusive control is necessary, let us first see the issues that occur in the absence of exclusive control using the examples given below.

Problem 1
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example below of receiving an order for Tea from users on a shopping site.

 .. figure:: ./images/ExclusionControl-problem1.png
   :alt: Exclusive Control problem1
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - UserA
      - UserB
      - Description
    * - 1.
      - 〇
      - \-
      - User A confirms that the quantity of Tea on Product screen is 5nos.
    * - 2.
      - \-
      - 〇
      - User B confirms that the quantity of Tea on Product screen is 5nos.
    * - 3.
      - \-
      - 〇
      - User B orders 5nos. of Tea. Stock of Tea in the DB reduces by 5 and becomes 0.
    * - 4.
      - 〇
      - \-
      - User A orders 5nos. of Tea. Stock of Tea in the DB reduces by 5 and becomes -5.

 | **Order from User A is accepted however an apology is conveyed due to non-availability of actual stock.**
 | **The stock quantity of Tea stored in the table also shows a value (minus value) different from actual stock quantity of Tea.**

Problem 2
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example below wherein the staff that manages the stock quantity of Tea on shopping site, displays the stock quantity of Tea, calculates the added stock quantity at Client side and updates the stock quantity of Tea.

 .. figure:: ./images/ExclusionControl-problem2.png
   :alt: Exclusive Control problem2
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - UserA
      - UserB
      - Description
    * - 1.
      - 〇
      - \-
      - Staff A confirms that the quantity of Tea is 5nos.
    * - 2.
      - \-
      - 〇
      - Staff B confirms that the quantity of Tea is 5nos.
    * - 3.
      - \-
      - 〇
      - Staff B adds stock of 10nos. of Tea and updates the stock quantity on Client side as 5+10=15.
    * - 4.
      - 〇
      - \-
      - Staff A adds stock of 20nos. of Tea and updates the stock quantity on Client side as 5+20=25.

 **The stock of tea 10nos. added in process 3 is lost and there is a mismatch in actual stock quantity (35nos.).**

Problem 3
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example below wherein the data locked by batch processing is updated by online processing.

 .. figure:: ./images/ExclusionControl-problem4.png
   :alt: Exclusive Control problem4
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - UserA
      - Batch
      - Description
    * - 1.
      - \-
      - 〇
      - Batch locks the table row (temporarily all rows) to be updated and does not allow the update by other processes.
    * - 2.
      - 〇
      - \-
      - User A searches the updated information. Since batch is not committed at this point, the information prior to batch update can be fetched.
    * - 3.
      - 〇
      - \-
      - User A requests for update and waits as it is locked in batch.
    * - 4.
      - \-
      - 〇
      - Batch terminates the process to release the lock.
    * - 5.
      - 〇
      - \-
      - The update process for which user A was waiting can now be executed.

 | **User A executes update process after waiting for the batch to release the lock. However, data fetched by User A prior to waiting is the data before batch update and batch processing may overwrite the data with the updated data.**
 | **Batch takes a longer time as compared to online processing and user has to wait longer.**

Exclusive control according to the isolation level of transaction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The easiest way to resolve all the 3 problems given in :ref:`ExclusionControl-Necessity` is to sequentially execute the database processes.
| When the processes are sequentially executed, there is no effect on transactions.
| However, when the processes are sequentially executed, the number of transactions  that can be executed in a unit of time shows reduction resulting in deterioration in the performance.

In ANSI/ISO SQL standards, the guidelines indicating the isolation level (extent of impact by each transaction) are defined.
The 4 isolation levels of transaction are given below. Events that occur at each isolation level are explained below.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|p{0.20\linewidth}|
.. list-table::
   :header-rows: 1
   :widths: 10 20 20 20 20

   * - | Sr. No.
     - | Isolation levels
     - | 
       | DIRTY READ
     - | 
       | NON-REPEATABLE READ
     - | 
       | PHANTOM READ
   * - | 1.
     - | READ UNCOMMITTED
     - Yes
     - Yes
     - Yes
   * - | 2.
     - | READ COMMITTED
     - No
     - Yes
     - Yes
   * - | 3.
     - | REPEATABLE READ
     - No
     - No
     - Yes
   * - | 4.
     - | SERIALIZABLE
     - No
     - No
     - No

.. tip:: **DIRTY READ**

     Dirty Read occurs when the data written by uncommitted transaction is read by other transaction.

.. tip:: **NON-REPEATABLE READ**

     When the same record is likely to be read twice in the same transaction, if other transaction is committed during the first read and second read, the details read at first time and those at second time may differ.
     The multiple data readings may vary based on commit timing of other transaction.

.. tip:: **PHANTOM READ**

     In Phantom Read, when a same record is being read twice in the same transaction, if other transaction adds or deletes the record, it causes difference in the number of records (details) fetched during the first read and second read.

| The isolation level defined in above table gets higher as we go down.
| If the isolation level is high, the data can be protected; however it increases the locking overhead resulting in deterioration of the performance.
| It is not desirable to select SERIALIZABLE unless the access frequency is fairly low.
| This is because all the data is accessed sequentially one by one including SELECT.

| The relationship between level of isolation and degree of concurrency between transactions is a Trade-off relationship.
| In other words, high isolation level leads to reduction in concurrency and vice versa.
| Thus, it is necessary to balance the level of isolation and degree of concurrency of transactions in accordance with application requirements.

| The supported isolation level differs depending on the database to be used, it is necessary to understand the characteristics of the database to be used.
| Isolation levels supported by each database and their default values are shown below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|p{0.15\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 15 15 15 15

   * - | Sr. No.
     - | Database
     - | READ UNCOMMITTED
     - | READ COMMITTED
     - | REPEATABLE READ
     - | SERIALIZABLE
   * - | 1.
     - | Oracle
     - | ×
     - | 〇 (default)
     - | ×
     - | 〇
   * - | 2.
     - | PostgreSQL
     - | ×
     - | 〇 (default)
     - | ×
     - | 〇
   * - | 3.
     - | DB2
     - | 〇
     - | 〇 (default)
     - | 〇
     - | 〇
   * - | 4.
     - | MySQL InnoDB
     - | 〇
     - | 〇
     - | 〇 (default)
     - | 〇

**When a balance is to be maintained between isolation and concurrency while maintaining data consistency, it is necessary to perform exclusive control using Database Locking which is described below.**

Exclusive control using database locking
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| It is necessary to lock the data to be updated using an appropriate method due to the following reasons:

* To maintain consistency of data stored in the database
* To prevent conflicts in update process

| The three types of methods to lock the data stored in the database are as follows: 
| The Architect should adequately understand the characteristics of such locking and use the appropriate locking method in accordance with the characteristics of application.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.40\linewidth}|p{0.35\linewidth}|
 .. list-table:: Types of locking
   :header-rows: 1
   :widths: 10 15 40 35

   * - Sr. No.
     - Types of locking
     - Applicable cases
     - Characteristics
   * - 1.
     - Automatic locking by RDBMS
     - * When the conditions necessary to ensure data consistency can be specified as update conditions for data.
       * When concurrency for the same data is less and update process also takes less time.
     - * Effective since check and update process are executed using single SQL.
       * Conditions for ensuring data consistency need to be analyzed separately as compared to optimistic locking.
   * - 2.
     - Optimistic locking
     - * When the already fetched data is being updated by other transaction and if the updated contents need to be verified.
       * When concurrency for the same data is less and update process also takes less time.
     - * Ensures that the fetched data is not updated by other transaction.
       * Column to manage versions needs to be defined in the table.
   * - 3.
     - Pessimistic locking
     - * When the data that is likely to remain in locked state for a longer period is updated.
       * When data consistency check needs to be carried out since optimistic locking cannot be used (column cannot be defined to manage versions).
       * When concurrency for the same data is more and update process takes longer time.
     - * Possibility of a process failure due to process results of other transaction is eliminated.
       * Costly since it is necessary to execute SELECT statement for obtaining pessimistic lock.

.. note:: **Standards for adopting types of locking**

    The Architect should decide the type of locking to be used based on the functional and performance requirements.

    * Optimistic locking is necessary to ensure that the database transactions such as returning and changing the data on screen are cut off and the data remains unchanged in subsequent transaction.
    * When locking is needed in a single transaction, both pessimistic and optimistic locking can be implemented; however when pessimistic locking is used, lock is implemented at database level thus resulting in the possible increase in database processing cost. It is always preferable to use optimistic locking unless there are any specific issues.
    * If optimistic locking is used in a process with higher update frequency wherein multiple tables are to be updated in a single transaction, the waiting time for obtaining a lock can be minimized. However the possibility of error occurrences increases since an exclusive error may occur in between the process.
      If pessimistic locking is used, the waiting time for obtaining a lock is likely to increase; however since exclusive error does not occur once lock is obtained, it reduces the possibility of error occurrences.

.. tip:: **Business transaction**

    In actual application development, there could also be cases where exclusive control is necessary for the transactions at business flow level.
    A typical example of business flow level transactions could be an application used at a travel agency for making reservations while talking to the customer.

    While making travel reservations, means of transport such as railway, accommodation facilities and additional scheme, etc. are discussed.
    At this point, the reserved accommodation and additional scheme should remain unavailable for other users.
    In such a case, the status of table should be updated from 'Temporarily Reserved' to 'Reserved'. Even if the reservation is in process, other users should not be able to make the corresponding reservation.

    Description of exclusive control for business transaction is skipped in this chapter since it should be analyzed and designed under business process design or functional design.

Exclusive control using row lock function of the database
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In most databases, when a record is updated (UPDATE, DELETE), a row lock is obtained to prevent updates by other transactions till the primary transaction is committed or rolled back.
| Thus, if the number of updated records is as anticipated, the data consistency can be ensured.

| Exclusive control can be performed by using this characteristic and specifying the conditions to ensure data consistency for WHERE clause at the time of update.
| The support status of row lock at the time of update for each database is shown below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.10\linewidth}|p{0.15\linewidth}|p{0.35\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 10 15 35

   * - Sr. No.
     - Database
     - Version
     - Default setting lock
     - Remarks
   * - 1.
     - Oracle
     - 11
     - Row lock
     - Increased memory usage due to locking.
   * - 2.
     - PostgreSQL
     - 9
     - Row lock
     - There is no maximum limit on the number of rows that can be locked simultaneously since information for the modified rows is not stored in the memory. However, it is necessary to perform VACUUM periodically to write to the table.
   * - 3.
     - DB2
     - 9
     - Row lock
     - Increased memory usage due to locking.
   * - 4.
     - MySQL InnoDB
     - 5
     - Row lock
     - Increased memory usage due to locking.

| Exclusive control using row lock function of database can be used when it is not necessary to verify the contents updated by other transaction.
| For example, it can be used in the purchase process of a shopping site wherein the purchased product quantity is deducted from the records that manage product stock quantity.
| Exclusive control is not recommended in the processes used for status management since the earlier status is important in such processes.

See the example below illustrating a specific scenario.

* On a shopping site, both User A and User B are displayed a purchase screen of the same product at the same time.
  A stock quantity fetched from Stock Table is also displayed at the same time.
* 5 products were purchased at the same time; but since User A clicked "Purchase" button a bit earlier, User A buys the product first and then User B.

 .. figure:: ./images/update-for-db-line-lock.png
   :alt: update for db line lock
   :width: 90%
   :align: center

 .. raw:: latex

    \newpage

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80
    :class: longtable

    * - Sr. No.
      - UserA
      - UserB
      - Description
    * - 1.
      - 〇
      - \-
      - User A displays a purchase screen of the product. A stock quantity of 100nos. is displayed on the screen.

        .. code-block:: sql

          select quantity from Stock where ItemId = '01'

    * - 2.
      - \-
      - 〇
      - User B displays a purchase screen of the product. A stock quantity of 100nos. is displayed on the screen.

        .. code-block:: sql

          select quantity from Stock where ItemId = '01'

    * - 3.
      - 〇
      - \-
      - User A purchases 5 products of ItemId=01. 5nos. are deducted from Stock Table.

        .. code-block:: sql

          Update from Stock set quantity = quantity - 5
                                where ItemId='01' and quantity >= 5

    * - 4.
      - \-
      - 〇
      - User B purchases 5 products of ItemId=01. The system tries to deduct 5 products from Stock Table; however since transaction of User A is not yet completed, the purchase process of User B is kept on hold.
    * - 5.
      - 〇
      - \-
      - Transaction of user A is committed.
    * - 6.
      - \-
      - 〇
      - | The transaction of user A is now committed, hence purchase for User B which was kept on hold in step 4 is now resumed.
        | If a stock screen is viewed at this point, the stock quantity is now 95 instead of 100. However, since the balance stock quantity is more than the purchase quantity (in the above example, 5),  5 is deducted from Stock Table.

        .. code-block:: sql

          Update from Stock set quantity = quantity - 5
                                where ItemId='01' and quantity >= 5

    * - 7.
      - \-
      - 〇
      - Transaction of user B is committed.

 .. raw:: latex

    \newpage

 .. note:: **Important**

    It is important to specify the deduction ( ``"quantity - 5"`` ) and update condition ( ``"and quantity >= 5"`` ) in SQL.

| In the similar scenario as above, if the stock quantity when a product purchase screen is displayed is 9, the stock quantity when the User B resumes update process becomes 4. As it does not satisfy \ ``quantity >= 5``\ condition, update count becomes 0.
| If the update count in the application is 0, purchase is rolled back and User B is prompted for re-execution.

 .. figure:: ./images/update-for-db-line-lock-not-enough.png
   :alt: update for db line lock not enough
   :width: 90%
   :align: center

 .. note:: **Important**

    It is important to verify the update count in the application and an error should occur if it is different from the expected count and transaction should be rolled back.

**When this method is used for locking, the process can be continued depending on conditions even if there is a change in the referred information and data consistency can be ensured by using database function.**

Exclusive control using optimistic locking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Optimistic locking is a method for ensuring data consistency wherein the data is not locked for transaction and is updated only after checking whether it is same as fetched data.
| When optimistic locking is to be used, a Version column for managing versions is created to determine if the data to be updated is same as fetched data. 
| Data consistency can be ensured by keeping both the versions at the time of data fetch and data update same as a condition for update.

.. note:: **Version column**

    This column is used in optimistic locking for managing the update count of a record. It is set to 0 when a record is inserted and then it is incremented with each successful update.
    The Version column can be also be substituted with the latest update timestamp in place of a number.
    However, when timestamp is used, uniqueness when processes are executed simultaneously cannot be ensured.
    Hence, in order to ensure uniqueness, it is necessary to use a number in Version column.

| Exclusive control with optimistic locking is used when it is necessary to verify the contents updated by other transaction.
| For example, consider a case of workflow application wherein an applicant and an approver perform concurrent operations (withdrawal and approval).
| In this case, using exclusive control with optimistic locking, it is possible to notify the applicant and the approver that the operation is yet to be completed since the status before and after the operation are different.

.. warning::

    When an optimistic locking is to be performed, it is not appropriate to update/delete the record by adding conditions other than ID and Version.
    This is because when the data cannot be updated, it is difficult to determine whether the reason for update failure is version mismatch or condition mismatch.
    When the conditions for update are different, it is necessary to check whether previous process meets all the conditions.

See the example below illustrating a specific scenario.

* Staff (Staff A, Staff B) who manage the stock quantity of shopping site add the product stock. Let us assume that Staff A adds 5nos. and Staff B adds 15nos.
* A stock management screen is displayed in order to reflect the added stock in the stock management system. The stock quantity being stored in Stock Management System is then displayed.
* Against the displayed stock quantity, the value calculated by summing up the quantity added by the staff is entered in the Update form and the total stock is updated.

 .. figure:: ./images/Optimistic-lock-flow.png
   :alt: Optimistic lock flow
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - StaffA
      - StaffB
      - Description
    * - 1.
      - 〇
      - \-
      - Staff A displays stock management screen of the product. The stock quantity of 10nos. is displayed on the screen. Version of the referred data is ``1``.
    * - 2.
      - \-
      - 〇
      - Staff B displays stock management screen of the product. The stock quantity of 10nos. is displayed on the screen. Version of the referred data is ``1``.
    * - 3.
      - 〇
      - \-
      - Staff A adds the stock of 5 nos. against the stock quantity of 10 nos. displayed on the screen and updates stock quantity as 15nos. Version of the referred data is included as an update condition.

        .. code-block:: sql

           UPDATE Stock SET quantity = 15, version = version + 1
                        WHERE itemId = '01' and version = 1

    * - 4.
      - \-
      - 〇
      - Although Staff B adds 15nos. to the stock quantity of 10 nos. displayed on the screen and attempts to update the stock quantity to 25 nos.; however the transaction is kept on hold since the transaction of Staff A is not completed yet. Version of the referred data is included as an update condition.
    * - 5.
      - 〇
      - \-
      - Transaction of Staff A is committed. **Version becomes 2 at this point.**
    * - 6.
      - 〇
      - \-
      - Update process of Staff B which was kept on hold in step 4 is now resumed since the transaction of Staff A is committed. At this time, since the version of Stock Table data is ``2``, update result is 0 records. When the update result is 0 records, an exclusive error occurs.

        .. code-block:: sql

           UPDATE Stock SET quantity = 25, version = version + 1
                        WHERE itemId = '01' and version = 1

    * - 7.
      - 〇
      - \-
      - The transaction of Staff B is rolled back.

.. note:: **Points**

    Version (``"version + 1"``) should be incremented and update condition (``"and version = 1"``) should be specified in SQL.

Exclusive control using pessimistic locking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Pessimistic locking is a method to lock the data to be updated at the time of fetching so that it is not updated by other transaction.
| When pessimistic locking is to be used, lock is obtained for the record to be updated immediately after starting a transaction.
| Since the locked record is not updated by other transaction till the transaction is committed or rolled back, data consistency can be ensured.

.. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.30\linewidth}|
.. list-table:: RDBMS-wise pessimistic lock acquisition method
   :header-rows: 1
   :widths: 10 15 30

   * - Sr. No.
     - Database
     - Pessimistic locking method
   * - 1.
     - Oracle
     - FOR UPDATE
   * - 2.
     - PostgreSQL
     - FOR UPDATE
   * - 3.
     - DB2
     - FOR UPDATE WITH
   * - 4.
     - MySQL
     - FOR UPDATE

.. note:: **About pessimistic locking timeout**

     At the time of obtaining a pessimistic lock, if a lock is obtained by some other transaction, then the expected behavior is specified as an option in some cases.
     In case of Oracle,

     * Default value is \ ``select for update [wait]``\ , and it waits till lock is released.
     * If set to \ ``select for update nowait``\ , it immediately gives a resource busy error, if locked by other transaction.
     * If set to \ ``select for update wait 5``\ , it waits for 5 seconds, and gives a resource busy error, if the lock is not released within 5 seconds.

     Although there are variations in the functions as per DB, it is necessary to analyze the method to be used when using pessimistic locking.

.. note:: **When using JPA (Hibernate)**

     Although the method of fetching a pessimistic lock varies for each database, such differences are absorbed by JPA (Hibernate).
     Refer to `Hibernate Developer Guide <http://docs.jboss.org/hibernate/orm/4.3/devguide/en-US/html_single/#d5e233>`_ for RDBMS that supports Hibernate.

Exclusive control with pessimistic locking is used when it is applicable to any of the 3 cases given below.

#. | Data to be updated is managed by dividing it in multiple tables.
   | When the data to be updated is divided into multiple tables, it is necessary to ensure that there are no updates by other transaction, till the update of each table is completed.

#. | Status of the fetched data needs to be checked before performing update.
   | After completing the checks, it is necessary to ensure that there are no updates by other transaction.

#. | Online processing may be executed during batch execution.
   | In batch processing, locks are collectively obtained for the data to be updated so as to ensure that exclusive error does not occur in between the execution.
   | In case of locks which are obtained collectively, the locking period for online processing may be longer. In such a case, it is advisable to use pessimistic locking by specifying a timeout period.

See the example below illustrating specific scenario.

* Batch processing has already started execution, and data to be updated online is locked by pessimistic locking.
* Timeout period of 10 seconds is specified for the online processing and lock is obtained for the data to be updated.
* Batch processing is terminated after 5 seconds (before timeout).

 .. figure:: ./images/Pessimistic-lock.png
   :alt: Pessimistic lock
   :width: 90%
   :align: center

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - Online
      - Batch
      - Description
    * - 1.
      - \-
      - 〇
      - Batch processing obtains the pessimistic lock for the data to be updated in online processing.
    * - 2.
      - 〇
      - \-
      - Online processing tries to perform pessimistic locking for the data to be updated; however it is kept on hold since the pessimistic locking is performed by the transaction of batch processing.

        .. code-block:: sql

           SELECT * FROM Stock WHERE quantity < 5 FOR UPDATE WAIT 10

    * - 3.
      - \-
      - 〇
      - Batch processing updates data.
    * - 4.
      - \-
      - 〇
      - Batch processing transaction is committed.
    * - 5.
      - 〇
      - \-
      - Since the batch processing transaction is committed, online processing is resumed. As the fetched data reflects the update results of batch processing, data inconsistency does not occur.
    * - 6.
      - 〇
      - \-
      - Online processing updates data.
    * - 7.
      - 〇
      - \-
      - Online processing transaction is committed.

 .. raw:: latex

    \newpage

| The flow at the time of timeout is given below.
| Exclusive error occurs without waiting for the batch processing to end.

 .. figure:: ./images/Pessimistic-lock-timeout.png
   :alt: Pessimistic lock
   :width: 90%
   :align: center

 .. raw:: latex

    \newpage

| The flow given below illustrates a case wherein pessimistic lock is being obtained by other transaction in case of "pessimistic lock no wait" setting.
| An exclusive error occurs immediately without waiting for the release of pessimistic lock.

 .. figure:: ./images/Pessimistic-lock-nowait.png
   :alt: Pessimistic lock
   :width: 90%
   :align: center

**When there is a possibility of conflict between batch processing and online processing and if batch processing is going to take longer time, it is recommended to specify timeout period of pessimistic exclusive locking.**
**The timeout period should be determined based on the online processing requirements.**

Prevention of deadlock
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| When using database locks, it should be noted that if multiple records are updated in same transaction, 2 deadlocks shown below are likely to occur.

*  :ref:`Dead-Lock-Record`
*  :ref:`Dead-Lock-Table`

.. _Dead-Lock-Record:

Deadlock in table
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
This deadlock occurs when the records of the same table are locked by multiple transactions as shown in the flow of (1)-(5) below.

 .. figure:: ./images/Dead-Lock-Record.png
   :alt: Dead Lock Record
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - Program A
      - Program B
      - Description
    * - | (1)
      - 〇
      - \-
      - Program A obtains the lock for Record X.
    * - | (2)
      - 〇
      - \-
      - Program B obtains the lock for Record Y.
    * - | (3)
      - 〇
      - \-
      - Program A tries to obtain the lock for Record Y that is locked by the transaction of Program B, however since lock of (2) is not released, the status is changed to "Waiting for release".
    * - | (4)
      - \-
      - 〇
      - Program B tries to obtain the lock for Record X that is locked by the transaction of Program A, however since lock of (1) is not released, the status is changed to "Waiting for release".
    * - | (5)
      - \-
      - \-
      - Since Program A and Program B both have the "Waiting for release" status for each other, it results into a deadlock. When a deadlock occurs, it is detected by the database and error is thrown.

 .. note:: **How to resolve a deadlock**

    The deadlock can be resolved by timeout or retry; however, it is important to determine the rules for the update sequence of records in the same table.
    When rows are to be updated one by one, the rules such as updating in ascending order of PK (PRIMARY KEY) should be set.

    Let us say if both Program A and Program B follow the rule of starting the update from Record X, the deadlock shown in figure \ :ref:`Dead-Lock-Record`\  above no longer occurs.

.. _Dead-Lock-Table:

Deadlock between tables
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| This deadlock occurs when records of different tables are locked by multiple transactions as shown in the flow of (1)-(5) below.
| The basic concept is same as :ref:`Dead-Lock-Record`.

 .. figure:: ./images/Dead-Lock-Table.png
   :alt: Dead Lock Table
   :width: 90%
   :align: center

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.05\linewidth}|p{0.05\linewidth}|p{0.80\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 5 5 80

    * - Sr. No.
      - Program A
      - Program B
      - Description
    * - | (1)
      - 〇
      - \-
      - Program A obtains the lock for Record X of Table A.
    * - | (2)
      - 〇
      - \-
      - Program B obtains the lock for Record Y of Table B.
    * - | (3)
      - 〇
      - \-
      - Program A tries to obtain the lock for Record Y of Table B that is locked by the transaction of Program B, however since lock of (2) is not released, the status is changed to "Waiting for release".
    * - | (4)
      - \-
      - 〇
      - Program B tries to obtain the lock of Record X of Table A that is locked by the transaction of Program A, however since lock of (1) is not released, the status is changed to "Waiting for release".
    * - | (5)
      - \-
      - \-
      - Since Program A and Program B both have the "Waiting for release" status for each other, it results into a deadlock. When a deadlock occurs, it is detected by the database and error is thrown.

.. note:: **How to resolve a deadlock**

    Although deadlocks can be resolved by a timeout or retry; it is important to define rules for update sequence across the tables.

   Let us say if both Program A and Program B follow the rule of starting the update from Table A, the deadlock shown in figure \ :ref:`Dead-Lock-Table`\  above no longer occurs.

.. warning::

   A deadlock might still occur due to sequence of locking records even after adopting either of the methods as a precaution.
   The rules should be defined for the lock sequence of tables and records.

|

How to use
--------------------------------------------------------------------------------

From here, implementation method of exclusive control using O/R Mapper is described.

First confirm the implementation method of O/R Mapper to be used.

* :ref:`ExclusionControlHowToUseMyBatis3`
* :ref:`ExclusionControlHowToUseJpa`

For method of handling exclusive error, refer to:

* :ref:`ExclusionControlHowToUseExceptionHandling`

|

.. _ExclusionControlHowToUseMyBatis3:

How to implement while using MyBatis3
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Row lock function of RDBMS
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When exclusive control is to be performed using a row lock function of RDBMS, the following should be considered in SQL.

* Update contents to be specified in SET clause
* Update conditions to be specified in WHERE clause

|

- Define a method in Repository interface.

 .. code-block:: java

     public interface StockRepository {
        // (1)
        boolean decrementQuantity(@Param("itemCode") String itemCode,
                                  @Param("quantity") int quantity);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - In Repository interface,
        define a method to update data using row lock function of RDBMS.

        In above example, a method to decrease stock quantity is defined.
        When stock quantity is decreased, \ ``true``\  is returned.

|

- Define SQL wherein exclusive control using a row lock function of RDBMS becomes valid.

 .. code-block:: xml

    <!-- (2) -->
    <update id="decrementQuantity">
    <![CDATA[
        UPDATE
            m_stock
        SET
            /* (3) */
            quantity = quantity - #{quantity}
        WHERE
            item_code = #{itemCode}
        AND
            /* (4) */
            quantity >= #{quantity}
    ]]>
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - Define statement (SQL) to update the data using row lock function of RDBMS.

        In above example, SQL to decrease stock quantity is defined.

        When a row lock function of RDBMS is to be used, the data can be safely updated because of the operations given below.

        * When other transaction has obtained lock for the same data,
          SQL is executed after releasing (commit or rollback) the lock.

        * When stock quantity is decreased successfully,
          row lock of RDBMS is fetched, and update from other transactions gets locked.

    * - | (3)
      - Subtract the stock quantity (\ ``quantity = quantity - #{quantity}``\ ) in SQL.
    * - | (4)
      - As an update condition, add "stock quantity should be greater than or equal to the order quantity (\ ``quantity >= #{quantity}``\ ).

|

- Call the Repository method to safely update the data using row lock function of RDBMS.

 .. code-block:: java

    // (5)
    boolean updated = stockRepository.decrementQuantity(itemCode, quantityOfOrder);
    // (6)
    if (!updated) {
        // (7)
        ResultMessages messages = ResultMessages.error().add(ResultMessage
                .fromText("Not enough stock. Please, change quantity."));
        throw new BusinessException(messages);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (5)
      - Call the Repository method to perform update.
    * - | (6)
      - Call the Repository method to determine result.

        In case of \ ``false``\ , stock quantity will be insufficient as update conditions are not fulfilled.
    * - | (7)
      - Business error occurs.

        In above example, only business rules are checked (stock quantity check) while performing exclusive control.
        Therefore, when update conditions are not fulfilled, it is being treated as business error and not exclusive error.

        Business errors should be appropriately handled in Controller.

|

Optimistic locking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In MyBatis3, a mechanism to perform optimistic locking as library is not provided.
| Therefore, when performing optimistic locking, the version should be considered in SQL.

- Define a property for version control in entity.

 .. code-block:: java

    public class Stock implements Serializable {
        private static final long serialVersionUID = 1L;

        private String itemCode;
        private int quantity;
        // (1)
        private long version;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Create a property for version control in entity.

|

- Define a method in Repository interface.

 .. code-block:: java

    public interface StockRepository {
        // (2)
        Stock findOne(String itemCode);
        // (3)
        boolean update(Stock stock);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - Define a method to fetch entity, in Repository interface.
    * - | (3)
      - Define a method to update data using optimistic lock function, in Repository interface.

        In above example, a method to update records with specified entity details is defined.
        When update is successful, \ ``true``\  is returned.

|

- Define SQL in mapping file.

 .. code-block:: xml

    <!-- (4) -->
    <select id="findOne" parameterType="string" resultType="Stock">
        SELECT
            item_code,
            quantity,
            version
        FROM
            m_stock
        WHERE
            item_code = #{itemCode}
    </select>

    <!-- (5) -->
    <update id="update" parameterType="Stock">
        UPDATE
            m_stock
        SET
            quantity = #{quantity},
            /* (6) */
            version = version + 1
        WHERE
            item_code = #{itemCode}
        AND
            /* (7) */
            version = #{version}
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (4)
      - Define statement (SQL) to fetch entity.

        When using optimistic lock, it is necessary to obtain version at the time of fetching entity.
    * - | (5)
      - Define statement (SQL) to update data using optimistic lock function.

        In above example, SQL to update records with specified entity details is defined.
    * - | (6)
      - Update the version (\ ``version = version + 1``\ ) in SQL.
    * - | (4)
      - As an update condition, add "version should not be changed (\ ``version = #{version}``\ ).

|

- Call the Repository method to safely update the data using optimistic lock function.

 .. code-block:: java

    // (5)
    Stock stock = stockRepository.findOne(itemCode);
    if (stock == null) {
        ResultMessages messages = ResultMessages.error().add(ResultMessage
                .fromText("Stock not found. itemCode : " + itemCode));
        throw new ResourceNotFoundException(messages);
    }

    // (6)
    stock.setQuantity(stock.getQuantity() + addedQuantity);

    // (7)
    boolean updated = stockRepository.update(stock);
    if(!updated) {
        // (8)
        throw new ObjectOptimisticLockingFailureException(Stock.class, itemCode);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (5)
      - Call findOne method of Repository interface to fetch the entity.
    * - | (6)
      - Specify the value to be updated for the entity fetched in step (5).

        In above example, procured stock quantity is added.
    * - | (7)
      - Call the update method of Repository interface
        to reflect the entity updated in step (5) in persistence layer (DB).
    * - | (8)
      - Determine the update result. In case of \ ``false``\ ,
        entity gets updated by other transaction, thereby
        leading to optimistic lock error (\ ``org.springframework.orm.ObjectOptimisticLockingFailureException``\ ).

|

When performing optimistic lock for long transactions, points given below should be noted.

 .. warning::

    When performing optimistic locking for long transactions, apart from checking version at the time of update,
    it should also be checked at the time of fetching data.


Example of implementation is given below.

- Check version even at the time of fetching data.

 .. code-block:: java

    Stock stock = stockRepository.findOne(itemCode);
    if (stock == null || stock.getVersion() != version) {
        // (9)
        throw new ObjectOptimisticLockingFailureException(Stock.class, itemCode);
    }

    stock.setQuantity(stock.getQuantity() + addedQuantity);
    boolean updated = stockRepository.update(stock);
    // ...

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (9)
      - Compare the version of entity fetched in (5) with
        that of the entity fetched in other database transaction.

        If version differs, the data is updated by other transaction, thereby
        leading to optimistic lock error (\ ``org.springframework.dao.ObjectOptimisticLockingFailureException``\ ). 

        It is also necessary to consider a case wherein data does not exist (\ ``stock == null``\  ),
        The implementation should be as per the application specifications.
        In above example, it is being treated as optimistic locking error.


|

Following points should be noted when application uses optimistic lock function in combination with row lock function of RDBMS.

 .. warning::

    In case of application in which a process to perform exclusive control using row lock function of RDBMS and
    a process to perform exclusive control using optimistic lock function of RDBMS coexist, 
    **version should be updated (incremented)** in SQL using row lock function of RDBMS.

    If version is not updated in SQL that performs exclusive control using row lock function of RDBMS,
    the data may get overwritten by SQL that performs exclusive control using optimistic lock function.

Example of implementation is shown below.

- Update the version in SQL.

 .. code-block:: xml

    <update id="decrementQuantity">
    <![CDATA[
        UPDATE
            m_stock
        SET
            quantity = quantity - #{quantity},
            /* (10) */
            version = version + 1
        WHERE
            item_code = #{itemCode}
        AND
            quantity >= #{quantity}
    ]]>
    </update>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (10)
      - Update (increment) the version.

|

Pessimistic locking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| In MyBatis3, a mechanism to perform pessimistic locking as library is not provided.
| Therefore, in case of pessimistic locking, a keyword should be specified to fetch lock in SQL.

- Specify the keyword to fetch lock in SQL.

 .. code-block:: xml

    <select id="findOneForUpdate" parameterType="string" resultType="Stock">
        SELECT
            item_code,
            quantity,
            version
        FROM
            m_stock
        WHERE
            item_code = #{itemCode}
        /* (1) */
        FOR UPDATE
    </select>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - For SQL in which pessimistic lock needs to be fetched, specify the keyword to fetch the same.

        Keyword and location to specify the keyword differ depending on database.

|

.. _ExclusionControlHowToUseJpa:

How to implement while using JPA (Spring Data JPA)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Row lock function of RDBMS
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When exclusive control is to be performed using a row lock function of RDBMS, Query method is added to Repository interface for implementation.
| For Query method, refer to \ :ref:`data-access-jpa_how_to_use_querymethod`\  and \ :ref:`data-access-jpa_howtouse_querymethod_modifying`\ .

- Repository interface

 .. code-block:: java
   :emphasize-lines: 5, 7

     public interface StockRepository extends JpaRepository<Stock, String> {

        @Modifying
        @Query("UPDATE Stock s"
                + " SET s.quantity = s.quantity - :quantity"
                + " WHERE s.itemCode = :itemCode"
                + " AND :quantity <= s.quantity")  // (1)
        public int decrementQuantity(@Param("itemCode") String itemCode,
                @Param("quantity") int quantity);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | When the stock quantity is equal to or more than the order quantity, JPQL is specified in Query method to reduce stock quantity.
        | Since it is necessary to check the update count, \ ``int``\  is specified as the return value of Query method.

- Service

 .. code-block:: java

    String itemCodeOfOrder = "ITM0000001";
    int quantityOfOrder = 31;

    int updateCount = stockRepository.decrementQuantity(itemCodeOfOrder, quantityOfOrder); // (2)
    if (updateCount == 0) { // (3)
        ResultMessages message = ResultMessages.error();
        message.add(ResultMessage
                .fromText("Not enough stock. Please, change quantity."));
        throw new BusinessException(message); // (4)
    }

 .. code-block:: sql

    update m_stock set quantity=quantity-31
                   where item_code='ITM0000001' and 31<=quantity -- (5)


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - Call the Query method.
    * - | (3)
      - Determine the call results of Query method. In case of \ ``0``\ , the stock quantity becomes inadequate, since update conditions are not satisfied.
    * - | (4)
      - | Store the message notifying "No stock" or "Not enough stock" to generate a business error.
        | The generated error should be handled appropriately in Controller as per the requirements.
        | In the above example, only business rules are checked while performing exclusive control; hence when update conditions are not satisfied, it is treated as business error and not exclusive error.
        | For error handling methods, refer to \ :ref:`exception-handling-how-to-use-codingpoint-controller-label`\ .
    * - | (5)
      - SQL that is executed while calling the Query method.

Optimistic locking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In JPA, optimistic locking can be performed by specifying \ ``@javax.persistence.Version``\ annotation in property for version control.

- Entity

 .. code-block:: java
   :emphasize-lines: 11

    @Entity
    @Table(name = "m_stock")
    public class Stock implements Serializable {

        @Id
        @Column(name = "item_code")
        private String itemCode;

        private int quantity;

        @Version // (1)
        private long version;

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify the \ ``@Version``\  annotation in property for version control.

- Service

 .. code-block:: java

    String itemCode = "ITM0000001";
    int newQuantity = 30;

    Stock stock = stockRepository.findOne(itemCode); // (2)
    if (stock == null) {
        ResultMessages messages = ResultMessages.error().add(ResultMessage
                .fromText("Stock not found. itemCode : " + itemCode));
        throw new ResourceNotFoundException(messages);
    }

    stock.setQuantity(newQuantity); // (3)

    stockRepository.flush(); // (4)

 .. code-block:: sql

    update m_stock set quantity=30, version=7
                   where item_code='ITM0000001' and version=6 -- ( 5)

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (2)
      - Call findOne method of Repository interface to fetch the entity.
    * - | (3)
      - Specify the value to be updated for the entity fetched in step (2).
    * - | (4)
      - | Reflect the changes of (3) in the persistence layer (DB). This process is usually not required since it is performed for the description purpose.
        | Normally, it is reflected automatically when the transaction is committed.
        | In the above example, when the version held by the entity fetched in step (2) and the version stored in persistence layer (DB) do not match, optimistic locking error (\ ``org.springframework.dao.OptimisticLockingFailureException``\ ) occurs.
    * - | (5)
      - SQL that is executed while reflecting to persistence layer (DB) of step (4).

It is important to note the following points while performing optimistic locking for long transactions.

.. warning::

  It is not sufficient to simply assign  \ ``@Version``\  annotation for optimistic locking which is to be performed for long transactions.
  When optimistic locking is to be performed for long transactions, version check should also be carried out while fetching the data to be updated, in addition to the check at the time of update performed using JPA function.

An implementation example is given below.

- Service

 .. code-block:: java

    long version = 12;
    String itemCode = "ITM0000001";
    int newQuantity = 30;

    Stock stock = stockRepository.findOne(itemCode); // (1)
    if (stock == null || stock.getVersion() != version) { // (2)
        throw new ObjectOptimisticLockingFailureException(Stock.class, itemCode); // (3)
    }

    stock.setQuantity(newQuantity);

    stockRepository.flush();

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Fetch an entity from persistence layer (DB).
    * - | (2)
      - | Compare the version of the entity fetched by a different database transaction in advance with the latest version of persistence layer (DB) fetched in step (1).
        | If versions match, optimistic locking which uses \ ``@Version``\  annotation becomes valid in subsequent processes.
    * - | (3)
      - If the versions are different, generate the optimistic locking error (\ ``org.springframework.dao.ObjectOptimisticLockingFailureException``\ ).

.. warning:: **Setting a value in property for Version control**

    Entity fetched using Repository interface is called "Managed entity".

     **For "Managed entity", it should be noted that a value cannot be set in a process for the property for Version control.**

    Even if a process as shown below is carried out, the value of version that is set to "Managed entity" is not reflected. Hence it is not used to fetch an optimistic lock. The version when a value is fetched using findOne method is used for optimistic locking.

     .. code-block:: java
        :emphasize-lines: 11

        long version = 12;
        String itemCode = "ITM0000001";
        int newQuantity = 30;

        Stock stock = stockRepository.findOne(itemCode);
        if (stock == null) {
            ResultMessages messages = ResultMessages.error().add(ResultMessage
                    .fromText("Stock not found. itemCode : " + itemCode));
            throw new ResourceNotFoundException(messages);
        }
        stock.setVersion(version); // ★ Invalid Processing
        stock.setQuantity(newQuantity);

        stockRepository.flush();

    For example, even if the value of the version sent from the screen is overwritten, it is not reflected in entity. Hence the exclusive control can no longer be appropriately performed.


.. note:: **Standardization of optimistic locking for long transactions**

  When optimistic locking for long transactions becomes necessary in multiple processes, it is desirable to standardize the processes of (1) ~ (3) described above.
  For standardization method, refer to \ :ref:`data-access-jpa_how_to_extends_custommethod`\  .

It is important to note the following points when both row lock function of RDBMS and optimistic locking function are to be used.

.. warning::

  \ **Version must always be updated**\  in Query method that uses row lock function of RDBMS
  in case of applications wherein a process performing exclusive control using a row lock function of RDBMS
  and a process performing exclusive control using optimistic locking function co-exist, for the same data.

  If version is not updated in Query method that performs exclusive control using row lock function of RDBMS,
  the contents updated by Query method may be overwritten by the process of a different transaction. Hence, the exclusive control is not performed properly.

An implementation example is given below.

- Repository interface

 .. code-block:: java
   :emphasize-lines: 5

    public interface StockRepository extends JpaRepository<Stock, String> {

        @Modifying
        @Query("UPDATE Stock s SET s.quantity = s.quantity - :quantity"
                + ", s.version = s.version + 1" // (1)
                + " WHERE s.itemCode = :itemCode"
                + " AND :quantity <= s.quantity")
        public int decrementQuantity(@Param("itemCode") String itemCode,
                @Param("quantity") int quantity);

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Version needs to be updated (\ ``s.version = s.version + 1``\ ).

Pessimistic locking
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
In Spring Data JPA, the pessimistic lock can be performed by specifying \ ``@org.springframework.data.jpa.repository.Lock``\  annotation.

- Repository interface

 .. code-block:: java

    public interface StockRepository extends JpaRepository<Stock, String> {

        @Lock(LockModeType.PESSIMISTIC_WRITE) // (1)
        @Query("SELECT s FROM Stock s WHERE s.itemCode = :itemCode")
        Stock findOneForUpdate(@Param("itemCode") String itemCode);

    }

 .. code-block:: sql

    -- (2)
    SELECT
            stock0_.item_code AS item1_5_
            ,stock0_.quantity AS quantity2_5_
            ,stock0_.version AS version3_5_
        FROM
            m_stock stock0_
        WHERE
            stock0_.item_code = 'ITM0000001'
        FOR UPDATE;

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify \ ``@Lock``\  annotation in Query method.
    * - | (2)
      - Executed SQL. In the above example, the SQL executed while using PostgreSQL is given.

The types of pessimistic locking that can be specified using \ ``@Lock``\  annotation are as given below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.25\linewidth}|p{0.45\linewidth}|p{0.20\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 25 45 20

   * - Sr. No.
     - LockModeType
     - Description
     - Issued SQL
   * - 1.
     - PESSIMISTIC_READ
     - | Pessimistic lock for read is obtained. It acts as a shared lock rather than an exclusive lock depending on the database.
       | Lock is released at the time of commit or rollback
     - | select ... for update / select ... for share
   * - 2.
     - PESSIMISTIC_WRITE
     - | Pessimistic lock for update is obtained and an exclusive lock is applied.
       | In case of exclusive lock, if a lock has already been applied, the entity is fetched after waiting for the lock to be released.
       | Lock is released at the time of commit or rollback
     - | select ... for update
   * - 3.
     - PESSIMISTIC_FORCE_INCREMENT
     - | Exclusive lock is applied to the target data when entity is fetched. Version is forcibly updated immediately after fetching the entity.
       | Lock is released at the time of commit or rollback
     - | select ... for update + update

 .. note:: **Lock timeout period**

    Timeout period can be specified by specifying \ ``"javax.persistence.lock.timeout"``\  as JPA (\ ``EntityManager``\ ) settings or Query hint.

There are 2 methods for specifying the locking timeout period: 1. Method wherein it is specified for the entire process 2. Method wherein it is specified for each Query.

Method wherein it is specified for the entire process is as follows:

- :file:`xxx-infra.xml`

 .. code-block:: xml

     <bean id="entityManagerFactory"
         class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
         <property name="packagesToScan" value="xxxxxx.yyyyyy.zzzzzz.domain.model" />
         <property name="dataSource" ref="dataSource" />
         <property name="jpaVendorAdapter" ref="jpaVendorAdapter" />
         <property name="jpaPropertyMap">
             <util:map>
                 <!-- ... -->
                 <entry key="javax.persistence.lock.timeout" value="1000" /> <!-- (1) -->
             </util:map>
         </property>
     </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify the timeout in milliseconds. If \ ``1000``\  is specified, it equals 1 second.

 .. note:: **nowait support**

    When \ ``0``\  is specified for Oracle and PostgreSQL, \ ``nowait``\ is added, and when locked by another transaction, an exclusive error occurs without waiting for release of lock.

 .. warning:: **Restrictions of PostgreSQL**

    Although nowait can be specified in PostgreSQL, it is not possible to specify waiting time.
    Therefore, the measures such as separately providing timeout for Query etc. should be implemented.

Method wherein it is specified for each Query is as follows:


- Repository interface

 .. code-block:: java

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @QueryHints(@QueryHint(name = "javax.persistence.lock.timeout", value = "2000")) // (1)
    @Query("SELECT s FROM Stock s WHERE s.itemCode = :itemCode")
    Stock findOneForUpdate(@Param("itemCode") String itemCode);

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - | Specify the timeout in milliseconds. If \ ``2000``\  is specified, it equals 2 seconds.
        | All the specified values are overwritten.

.. _ExclusionControlHowToUseExceptionHandling:

How to handle an exclusive error
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Error handling in case of optimistic locking failure
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When optimistic locking fails, \ ``org.springframework.dao.OptimisticLockingFailureException``\ occurs;
hence it is necessary to handle it appropriately in Controller.

| The handling method varies with the operation specifications of application when optimistic locking error occurs.

When there is no need to change the operation at request level, it is handled by using \ ``@ExceptionHandler``\  annotation.

 .. code-block:: java

    @ExceptionHandler(OptimisticLockingFailureException.class) // (1)
    public ModelAndView handleOptimisticLockingFailureException(
            OptimisticLockingFailureException e) {
        // (2)
        ExtendedModelMap modelMap = new ExtendedModelMap();
        ResultMessages resultMessages = ResultMessages.warning();
        resultMessages.add(ResultMessage.fromText("Other user updated!!"));
        modelMap.addAttribute(setUpForm());
        modelMap.addAttribute(resultMessages);
        String viewName = top(modelMap);
        return new ModelAndView(viewName, modelMap);
    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Specify \ ``OptimisticLockingFailureException.class``\  in the value attribute of \ ``@ExceptionHandler``\  annotation.
    * - | (2)
      - | Carry out error handling. Generate the message to notify error and information required for screen display (form or other model) and return \ ``ModelAndView``\  specifying the destination.
        | For details on error handling, refer to \ :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`\ .

If there is a need to change the operation at request level, it is to be handled using \ ``try - catch``\  in the handler method of Controller.

 .. code-block:: java

    @RequestMapping(value = "{itemId}/update", method = RequestMethod.POST)
    public String update(StockForm form, Model model, RedirectAttributes attributes){

        // ...

        try {
            stockService.update(...);
        } catch (OptimisticLockingFailureException e) { // (1)
            // (2)
            ResultMessages resultMessages = ResultMessages.warn();
            resultMessages.add(ResultMessage.fromText("Other user updated!!"));
            model.addAttribute(resultMessages);
            return updateRedo(modelMap);
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Catch ``OptimisticLockingFailureException``.
    * - | (2)
      - | Carry out error handling. Generate the message to notify error and information required for screen display (form or other model) and return the destination view name.
        | For details on error handling, refer to \ :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`\ .

Error handling in case of pessimistic locking failure
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

When pessimistic locking fails, \ ``org.springframework.dao.PessimisticLockingFailureException``\  occurs; hence it is necessary to handle it appropriately in Controller.

The handling method varies with the operation specifications of the application when pessimistic locking error occurs.

If there is no need to change the operation at request level, it is handled using \ ``@ExceptionHandler``\  annotation.

 .. code-block:: java

    @ExceptionHandler(PessimisticLockingFailureException.class) // (1)
    public ModelAndView handlePessimisticLockingFailureException(
            PessimisticLockingFailureException e) {
        // (2)
        ExtendedModelMap modelMap = new ExtendedModelMap();
        ResultMessages resultMessages = ResultMessages.warning();
        resultMessages.add(ResultMessage.fromText("Other user updated!!"));
        modelMap.addAttribute(setUpForm());
        modelMap.addAttribute(resultMessages);
        String viewName = top(modelMap);
        return new ModelAndView(viewName, modelMap);
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - SR. No.
      - Description
    * - | (1)
      - Specify \ ``PessimisticLockingFailureException .class``\  in value attribute of \ ``@ExceptionHandler``\  annotation.
    * - | (2)
      - | Carry out error handling. Generate the message to notify error and information required for screen display (form or other model) and return \ ``ModelAndView``\  specifying the destination.
        | For details on error handling, refer to \ :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`\ .

If there is need to change the operation at request level, it is to be handled using \ ``try - catch``\  in the handler method of Controller.

 .. code-block:: java

    @RequestMapping(value = "{itemId}/update", method = RequestMethod.POST)
    public String update(StockForm form, Model model, RedirectAttributes attributes){

        // ...

        try {
            stockService.update(...);
        } catch (PessimisticLockingFailureException e) { // (1)
            // (2)
            ResultMessages resultMessages = ResultMessages.warn();
            resultMessages.add(ResultMessage.fromText("Other user updated!!"));
            model.addAttribute(resultMessages);
            return updateRedo(modelMap);
        }

        // ...

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
    :header-rows: 1
    :widths: 10 90

    * - Sr. No.
      - Description
    * - | (1)
      - Catch \ ``PessimisticLockingFailureException``\ .
    * - | (2)
      - | Carry out error handling. Generate the message to notify error and information required for screen display (form or other model) and return destination view name.
        | For details on error handling, refer to \ :ref:`exception-handling-how-to-use-codingpoint-controller-usecase-label`\ .


.. raw:: latex

   \newpage
