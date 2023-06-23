Double Submit Protection
================================================================================

.. only:: html

 .. contents:: Table of Contents
    :depth: 4
    :local:

Overview
--------------------------------------------------------------------------------

Problems
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If any of the operations given below is performed in a Web application with screens, it leads to same process being executed multiple times.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.30\linewidth}|p{0.60\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 30 60

   * - Sr. No.
     - Operation
     - Operation Overview
   * - | (1)
     - | Double clicking of 'Update' button
     - | Repeatedly clicking the button to perform update process.
   * - | (2)
     - | Reloading of screen after completing the update process
     - | Using the 'Refresh' button of the browser to reload the screen after completion of update process.
   * - | (3)
     - | Invalid screen transition using 'Back' button of the browser
     - | Using 'Back' button of the browser to go back to the previous page from update process completion screen and clicking the button again to perform update process.

Each problem is described in detail below.

Double clicking of 'Update' button
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The following problems occur when the button to perform update process is clicked repeatedly.
| The example of "Product Purchase" on a shopping site is given below to explain the problems that are likely to occur if the necessary measures are not taken.

 .. figure:: ./images/duplicate-double-click.png
   :alt: duplicate double click
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Buyer clicks 'Order' button on Product Purchase screen.
   * - | (2)
     - | Buyer accidentally clicks 'Order' button again before receiving the response of (1).
   * - | (3)
     - | Server updates DB based on purchase of the product received through request (1).
   * - | (4)
     - | Server updates DB based on purchase of the product received through request (2).
   * - | (5)
     - | Server sends response with a "Purchase Complete" screen for the product received through request (2).

 .. warning::

    In the above case, since buyer accidentally clicks 'Order' button again, **it results in duplicate purchase of same product.**
    Although the problem can be attributed to erroneous operation by the buyer, it is desirable to have the application design such that the above problems do not occur.

Reloading of screen after completion of update process
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The following problems occur when the screen is reloaded after completion of update process.
| The example of "Product Purchase" on a shopping site is given below to explain the problems that are likely to occur if the necessary measures are not taken.

 .. figure:: ./images/duplicate-reload.png
   :alt: duplicate reload
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Buyer clicks 'Order' button on "Product Purchase" screen.
   * - | (2)
     - | Server updates DB based on purchase of the product received through request (1).
   * - | (3)
     - | Server sends response with a "Purchase complete" screen for the product received through request (1).
   * - | (4)
     - | Buyer accidently executes Reload functionality of the browser.
   * - | (5)
     - | Server updates DB based on the purchase of the product received through request (4).
   * - | (6)
     - | Server sends response with a "Purchase Complete" screen for the product received through request (4).

 .. warning::

    In the above case, since buyer accidentally executes Reload functionality of the browser, **it results in duplicate purchase of same product.**
    Although the problem can be attributed to erroneous operation by the buyer, it is desirable to have the application design such that the above problems do not occur.

Invalid screen transition using 'Back' button of the browser
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| The following problems occur if invalid screen transition is performed using 'Back' button of the browser.
| The example of "Product Purchase" on a shopping site is given below to explain the problems that are likely to occur if the necessary measures are not taken.

 .. figure:: ./images/duplicate-invalid-screenflow.png
   :alt: duplicate invalid screen flow
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Buyer clicks 'Order' button on "Product Purchase" screen.
   * - | (2)
     - | Server updates DB based on the purchase of the product received through request (1).
   * - | (3)
     - | Server sends response with a "Purchase complete" screen of the product received through request (1).
   * - | (4)
     - | Buyer uses 'Back' button of the browser to go back to "Product Purchase" screen.
   * - | (5)
     - | Buyer again clicks 'Order' button on "Product Purchase" screen which is re-displayed by clicking 'Back' button of the browser.
   * - | (6)
     - | Server updates DB based on the purchase of the product received through request (5).
   * - | (7)
     - | Server sends response with a "Purchase Complete" screen for the product received through request (5).

 .. note::
 
    In the above case, since buyer does not perform any erroneous operation, the problem is not attributed to the buyer.

|

However, if update process gets executed even after performing invalid screen operations, the following problems occur.

 .. figure:: ./images/duplicate-allow-malicious-request.png
    :alt: duplicate allow a malicious request
    :width: 100%
    
 .. warning::

    As described above, update process getting executed even after performing invalid screen operations increases the risk of direct updates by a malicious attacker bypassing a valid route.
    
        .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
        .. list-table::
           :header-rows: 1
           :widths: 10 90

           * - Sr. No.
             - Description
           * - | (1)
             - | Attacker executes request for processing a direct product purchase without going through a valid screen transition.
           * - | (2)
             - | Server cannot detect that the request is getting executed through an invalid route; hence it updates DB based on the purchase of the product received through that request.

    Execution of purchase process through an invalid request increases the load on each server resulting in inability to purchase products through a valid route.
    As a result, the problem causes a ripple effect for the users who purchase the products through valid routes. Hence, it is desirable to have the application design such that the above problems do not occur.

Solutions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The following measures should be taken to resolve the problems described above.
| In view of malicious operations such as tampering with requests, **(3) "Applying transaction token check" is mandatory.**

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - Sr. No.
     - Solution
     - Overview
   * - | (1)
     - | Preventing double clicking of a button using JavaScript
     - | When a button to perform update process is clicked, the button control using JavaScript prevents the submission of a request if the button is clicked again.
   * - | (2)
     - | Applying PRG (Post-Redirect-Get) pattern
     - | A redirect command is returned as a response to the request for performing update process (request by POST method) and then a screen for transition is returned as a response of GET method which is automatically requested from a browser.
       | When a PRG pattern is used, the request generated while reloading the page after the screen is displayed is a GET method; hence re-execution of update process can be prevented.
   * - | (3)
     - | Applying transaction token check
     - | Issue a token value for each screen transition and compare the token value sent from browser with the token value stored on the Server to make sure that invalid screen operations do not occur in the transaction.
       | Implementation of transaction token check can prevent re-execution of update process after the page is reloaded using 'Back' button of the browser.
       | Deleting the token value stored on the Server after performing the token check can prevent double submission as a Server side process.

 .. note::

    When only transaction token check is performed, even a simple operational mistake can lead to transaction token error which in turn results in a low-usability application for the user.
    
    To ensure usability as well as to prevent the problems that occur due to double submission, measures such as "Preventing double clicking of a button using JavaScript" and "Applying PRG (Post-Redirect-Get) pattern" are necessary.
    
    ** Although this guideline recommends that you implement all the measures, the decision should be taken depending on application requirements.**

 .. Warning::

   In Ajax and Web services, since it is difficult to transfer transaction tokens which change for each request, transaction token check need not be used.
   In Ajax, double submit protection should be performed using only one of the above measures i.e. "Preventing double clicking of a button using JavaScript".

 .. todo::
 
    **TBD**

    There is further scope for reviewing the check methods in Ajax and Web services.


Preventing double clicking of a button using JavaScript
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| Prevent double clicking of buttons like button to perform update process or button which is used to perform time-consuming search process.
| When a button is clicked, use JavaScript to disable that button or link.
| Typical examples of control used for disabling a button or link are given below

#. By disabling the button or link so that it cannot be clicked
#. By maintaining a flag for tracking process status and displaying notification "Process in progress" when the button or link is clicked in the middle of the process.



The image when a button is disabled will be as follows:

 .. figure:: ./images/prevent-double-click.png
   :alt: prevent double click
   :width: 60%

 .. warning::
 
    If all the buttons and links on the screen are disabled, the screen operations can no longer be performed if there is no response from the Server.
    Therefore, it is recommended not to disable buttons or links that execute events such as "Return to previous screen" or "Go to top screen" etc.

.. _DoubleSubmitProtectionAboutPRG:

About PRG (Post-Redirect-Get) pattern
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| A redirect command is returned as a response to the request for performing update process (request by POST method) and then a screen for transition is returned as a response of GET method which is automatically requested from a browser.
| When a PRG pattern is used, the request generated while reloading the page after the screen is displayed is a GET method; hence re-execution of update process can be prevented.

 .. figure:: ./images/prevent-double-submit-reload.png
   :alt: prevent double submit by reload
   :width: 100%


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Buyer clicks 'Order' button on "Product Purchase" screen.
       | **The request is submitted using POST method.**
   * - | (2)
     - | Server updates DB based on the purchase of the product received through request (1).
   * - | (3)
     - | **Server sends a redirect response for the URL to display the "Purchase Complete" screen for the product.**
   * - | (4)
     - | Browser submits request for the URL to display the "Purchase Complete" screen for the product.
       | **The request is submitted using GET method.**
   * - | (5)
     - | Server sends response with a "Purchase Complete" screen for the product.
   * - | (6)
     - | Buyer accidentally executes Reload functionality of the browser.
       | The request called by Reload functionality displays "Purchase Complete" screen of the product; hence **update process is not re-executed.**
   * - | (7)
     - | Server sends response with a "Purchase Complete" screen.

 .. note::
 
    It is recommended to use \ :abbr:`PRG (Post-Redirect-Get)`\  pattern for the processes associated with update process and implement a control so that a request of GET method is sent when 'Refresh' button of the browser is clicked.

 .. warning::
 
    In the \ :abbr:`PRG (Post-Redirect-Get)`\  pattern, the re-execution of update process cannot be prevented by clicking 'Back' button of the browser on Completion screen.
    A transaction token check must be performed to prevent re-execution of update process after an invalid screen transition using 'Back' button of the browser.
    
.. _double-submit_transactiontokencheck:

Transaction Token Check
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Transaction token check consists of the following 3 processes.

* When a request is received from Client, the Server stores a value (hereafter referred to as transaction token) for uniquely identifying a transaction on the Server.
* Server passes the transaction token to the Client. In case of a Web application with screens, it passes the transaction token to the Client using hidden tag of form.
* When submitting the next request, Client sends the transaction token that was passed from the Server. Server compares the transaction token received from the Client and the transaction token stored on the Server.

When the transaction token sent in the request does not match with the transaction token stored on the Server, it is treated as invalid request and an error is returned.

 .. warning::
 
    Misuse of transaction token check leads to poor usability of the application; hence the scope of its usage should be defined by considering the following points.

    * | It is not necessary to include reference-type requests that do not involve data update and requests that perform only screen transitions in the scope of transaction token check.
      | If the scope of transactions is extended unnecessarily, transaction token errors are more likely to occur which in turn reduces the usability of the application.
    * | Transaction token check is not mandatory for the processes wherein there is no problem even if the data gets updated multiple times from business perspective (user information update etc.).
    * | Transaction token check is mandatory for the processes such as deposit process or product purchase process etc. wherein there is a risk of duplicate execution.

|

The process flow when expected operations are performed and process flow when unexpected operations are performed using transaction token check are shown below.

 .. figure:: ./images/transaction-token-check-overview.png
   :alt: transaction token overview
   :width: 100%

| The process flow when expected operations are performed is as follows:

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Client sends the request.
   * - | (2)
     - | Server creates the transaction token (token001) and stores it on the Server.
   * - | (3)
     - | Server passes the created transaction token (token001) to the Client.
   * - | (4)
     - | Client sends the request along with the transaction token (token001).
   * - | (5)
     - | Server checks whether the transaction token (token001) stored on the Server and the transaction token (token001) submitted by the Client are same.
       | **Since the values are same, the request is considered as valid.**
   * - | (6)
     - | Server generates transaction token (token002) to be used in the next request and updates the value stored on the Server.
       | At this point, the transaction token (token001) is discarded.
   * - | (7)
     - | Server passes the updated transaction token (token002) to the Client.

| The process flow when unexpected operations are performed is as follows:
| Here, 'Back' button of the browser is taken as an example; however, this is also applicable for the direct requests from shortcuts etc.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (8)
     - | 'Back' button of the browser on Client side is clicked.
   * - | (9)
     - | Request is sent from Client side along with the transaction token (token001) of the screen which is displayed after clicking 'Back' button.
   * - | (10)
     - | Server checks whether the transaction token (token002) stored on the Server and the transaction token (token001) submitted by the Client are same.
       | **Since the values are different, the request is considered as invalid and a transaction token error is thrown.**
   * - | (11)
     - | Server sends response with an error screen to notify that a transaction token error has occurred.

|

The 3 events described below can be prevented by transaction token check.

* Invalid screen transition in case of a business process that requires fixed screen transition
* Data update due to invalid requests that do not involve valid screen transitions
* Duplicate execution of update process due to double submission

|

Invalid screen transition in case of a business process that requires fixed screen transition, can be prevented by the flow shown below.

 .. figure:: ./images/transaction-token-check-prevent-invalid-screenflow.png
   :alt: prevent invalid screen flow by transaction token check
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Buyer clicks 'Order' button on "Product Purchase" screen.
       | Since the transaction token stored on the Server and the transaction token submitted by the Client match, the process to purchase the product is executed.
       | **At this time, the value of the transaction token stored on the Server is discarded and updated to a new token value.**
   * - | (2)
     - | Server updates DB based on the purchase of the product received through request (1).
   * - | (3)
     - | Server sends response with a "Purchase Complete" screen for the product received through request (1).
   * - | (4)
     - | Buyer uses 'Back' button of the browser to go back to "Product Purchase" screen.
   * - | (5)
     - | Buyer again clicks 'Order' button on "Product Purchase" screen which is displayed using 'Back' button of the browser.
       | **Since the transaction token sent by the Client is a value which has already been discarded, a transaction token error occurs.**
   * - | (6)
     - | Server sends response with an error screen to notify that a transaction token error has occurred.

|

Data updated by an invalid request which does not involve a valid screen transition can be prevented by the flow shown below.

 .. figure:: ./images/transaction-token-check-prevent-malicious-request.png
   :alt: prevent malicious request by transaction token check
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Attacker sends a request to purchase the product directly without performing a valid screen transition.
       | **Since the request for generating a transaction token is not executed, a transaction token error occurs.**
   * - | (2)
     - | Server sends response with an error screen to notify that a transaction token error has occurred.

|

Duplicate execution of update process at the time of double submission can be prevented by the flow shown below.

 .. figure:: ./images/transaction-token-check-prevent-double-submit.png
   :alt: prevent double submit by transaction token check
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | Buyer clicks 'Order' button on "Product Purchase" screen.
       | Since the transaction token stored on the server and the transaction token submitted by the Client match, the process to purchase the product is executed.
       | **At this time, the value of the transaction token that is stored on the server is discarded and updated to a new token value.**
   * - | (2)
     - | Buyer accidentally clicks 'Order' button again before the response of (1) is returned.
       | **Since the transaction token sent by the Client is a value which has already been discarded, a transaction token error occurs** when process of (1) is executed.
   * - | (3)
     - | Server **sends response with an error screen to notify that a transaction token error has occurred** for the request (2).
   * - | (4)
     - | Server updates DB based on the purchase of the product received through request (1).
   * - | (5)
     - | Server attempts to respond with a "Purchase Complete" screen for the product received  through request (1); however since the stream for responding to the request of (1) is closed due to the transmission of the request of (2), it fails to send response with a "Purchase Complete" screen.

 .. warning::
 
    This can prevent duplicate execution of update process at the time of double submission; however this does not resolve the problem of server not being able to respond with a screen to notify the completion of process.
    Therefore, it is also recommended to deal with this problem by preventing double clicking of a button using JavaScript.

About NameSpace of transaction token
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In the transaction token check functionality provided by the common library, it is possible to set a NameSpace in the container for storing transaction token.
This enables parallel execution of update process using a tab browser or multiple windows.

Problems that occur when there is no NameSpace
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Let us see the problems that occur when there is no NameSpace.
| The figure below illustrates an example wherein two clients are arranged side by side; basically 2 windows are launched on same machine. 

 .. figure:: ./images/token-only-one.png
   :alt: token only one
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | The request is sent from Window 1, then the transaction token (token001) received as a response is stored in the browser.
       | The transaction token stored on the server is considered token001.
   * - | (2)
     - | The request is sent from Window 2, then the transaction token (token002) received as a response is stored in the browser.
       | **The transaction token stored on the server is considered token002. The transaction token (token001) generated by the process (1) is discarded at this point.**
   * - | (3)
     - | The request is sent from Window 1 along with the transaction token (token001) stored in the browser.
       | Since the transaction token stored on the server (token002) and the transaction token sent by the request (token001) do not match, the request is considered as invalid resulting in a transaction token error.

 .. warning::
 
    **The update process cannot be executed concurrently in absence of NameSpace; hence the application becomes a low-usability application.**

|

Behavior when NameSpace is specified
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
| Let us now see the behavior when NameSpace is specified.
| The problem wherein the update process could not be executed concurrently in the absence of NameSpace can now be resolved by specifying a NameSpace.
| The figure below illustrates an example wherein two clients are arranged side by side; basically 2 windows are launched on same machine. 

 .. figure:: ./images/token-namespace.png
   :alt: token namespace
   :width: 100%

| NameSpaces are shown as 111, 222 in the figure.
| ** When a NameSpace is specified, the transaction token in the NameSpace allocated to the transaction is handled independently. Hence, it does not affect the transactions of another NameSpace.**
| Here, even though the browser is explained using different Windows, the tab browser works in the same way. For generated keys and usage method, refer to \ :ref:`doubleSubmit_how_to_use_transaction_token_check`\ .

|

.. _How-to-use:

How to use
--------------------------------------------------------------------------------

Preventing double clicking of button using JavaScript
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| Double clicking of a button on Client side can be prevented using JavaScript.
| Once a button is clicked, it should not be possible to click it again till it is re-generated.

 .. todo::
 
    **TBD**
    
    The check method in JavaScript will be described in detail in subsequent versions.

Using PRG (Post-Redirect-Get) pattern
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The example of implementing PRG (Post-Redirect-Get) pattern is given below.
| The application which involves a simple screen transition such as Input Screen-> Confirmation Screen-> Completion Screen is taken as an example.

 .. figure:: ./images/staff-redirect-flow.png
   :alt: STAFF REDIRECT FLOW
   :width: 100%

| The image numbers and comment number of source code are linked.
| However, since (1)-(4) is not directly related to the PRG pattern, the explanation is omitted.

- Controller

 .. code-block:: java
    :emphasize-lines: 35,36,47-49,52-54,56

    @Controller
    @RequestMapping("prgExample")
    public class PostRedirectGetExampleController {

        @Inject
        UserService userService;

        @ModelAttribute
        public PostRedirectGetForm setUpForm() {
            PostRedirectGetForm form = new PostRedirectGetForm();
            return form;
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.GET, 
                        params = "form") // (1)
        public String createForm(
            PostRedirectGetForm postRedirectGetForm,
            BindingResult bindingResult) {
            return "prg/createForm"; // (2)
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.POST, 
                        params = "confirm") // (3)
        public String createConfirm(
            @Validated PostRedirectGetForm postRedirectGetForm,
            BindingResult bindingResult) {
            if (bindingResult.hasErrors()) {
                return "prg/createForm";
            }
            return "prg/createConfirm"; //  (4)
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.POST) // (5)
        public String create(
            @Validated PostRedirectGetForm postRedirectGetForm,
            BindingResult bindingResult,
            RedirectAttributes redirectAttributes) {
            if (bindingResult.hasErrors()) {
                return "prg/createForm";
            }

            // omitted

            String output = "result register..."; // (6)
            redirectAttributes.addFlashAttribute("output", output); // (6)
            return "redirect:/prgExample/create?complete"; // (6)
        }

        @RequestMapping(value = "create", 
                        method = RequestMethod.GET, 
                        params = "complete") // (7)
        public String createComplete() {
            return "prg/createComplete"; // (8)
        }
    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (5)
     - | A processing method to perform a process when 'Register' button (Create User button) on Confirmation screen is clicked.
       | **The request is received by POST method.**
   * - | (6)
     - | **It is redirected to URL for displaying Completion screen.**
       | In the above example, a request is sent to URL \ ``"prgExample/create?complete"`` \  by \ ``GET``\  method.
       | When data is to be delivered to redirect destination, addFlashAttribute method of \ ``RedirectAttributes``\  is called and the data to be delivered is added.
       | addAttribute method of \ ``Model``\  cannot deliver data to the redirect destination.
   * - | (7)
     - | A processing method to display Completion screen.
       | **A request is received by GET method.**
   * - | (8)
     - | View (JSP) is called to display the Completion screen and responds with Completion screen.
       | Since the extension of JSP is assigned by \ ``ViewResolver``\  defined in :file:`spring-mvc.xml`, it is omitted from the return value of the processing method.

 .. note::

    * At the time of redirecting, assign "redirect:" as the prefix of transition information to be returned by the processing method as the return value.
    * When the data is to be delivered to the process of redirect destination, call addFlashAttribute method of \ ``RedirectAttributes``\  and add the data to be delivered.

- :file:`createForm.jsp`

 .. code-block:: jsp

    <h1>Create User</h1>
    <div id="prgForm">
      <form:form 
        action="${pageContext.request.contextPath}/rpgExample/create"
        method="post" modelAttribute="postRedirectGetForm">
        <form:label path="firstName">FirstName</form:label>
        <form:input path="firstName" /><br>
        <form:label path="lastName">LastName:</form:label>
        <form:input path="lastName" /><br>
        <form:button name="confirm">Confirm Create User</form:button>
      </form:form>
    </div>

- :file:`createConfirm.jsp`

 .. code-block:: jsp
    :emphasize-lines: 5,11

    <h1>Confirm Create User</h1>
    <div id="prgForm">
      <form:form
        action="${pageContext.request.contextPath}/rpgExample/create"
        method="post"
        modelAttribute="postRedirectGetForm">
        FirstName:${f:h(postRedirectGetForm.firstName)}<br>
        <form:hidden path="firstName" />
        LastName:${f:h(postRedirectGetForm.lastName)}<br>
        <form:hidden path="lastName" />
        <form:button>Create User</form:button> <%-- (6) --%>
      </form:form>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (6)
     - | When the button to perform update process is clicked, **a request is sent by POST method.**

- :file:`createComplete.jsp`

 .. code-block:: jsp
    :emphasize-lines: 6

    <h1>Successful Create User Completion</h1>
    <div id="prgForm">
      <form:form
        action="${pageContext.request.contextPath}/rpgExample/create"
        method="get" modelAttribute="postRedirectGetForm">
        output:${f:h(output)}<br> <%-- (7) --%>
        <form:button name="backToTop">Top</form:button>
      </form:form>
    </div>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (7)
     - | When the data delivered from update process is to be referred at the redirect destination, **specify the attribute name of the data added by the addFlashAttribute method** of \ ``RedirectAttributes``\ .
       | In the above example, \ ``"output"``\  is the attribute name to refer to the delivered data.

.. _doubleSubmit_how_to_use_transaction_token_check:

Using transaction token check
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| The example of implementation using transaction token check is given below.
| Transaction token check functionality is provided by the common library and not by Spring MVC.

Transaction token check provided by common library
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Transaction token check functionality of common library provides \ ``@org.terasoluna.gfw.web.token.transaction.TransactionTokenCheck``\  annotation to perform the following tasks:

* Creation of NameSpace for transaction token
* Starting the transaction
* Token value check in the transaction
* Ending the transaction



The transaction token check can be performed declaratively by assigning \ ``@TransactionTokenCheck``\  annotation for the
Controller class and the processing methods of the Controller class.

|

Attributes of ``@TransactionTokenCheck``\  annotation
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The attributes that can be specified in ``@TransactionTokenCheck``\  annotation are explained below.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.10\linewidth}|p{0.45\linewidth}|p{0.10\linewidth}|p{0.20\linewidth}|
 .. list-table:: \ ``@TransactionTokenCheck``\ Annotation Parameter List
   :header-rows: 1
   :widths: 10 10 45 10 20

   * - Sr. No.
     - Attribute Name
     - Contents
     - default
     - Example
   * - (1)
     - value
     - | Any character string. Used as NameSpace.
     - None
     - | value = "create"
       | If there is only 1 argument, the "value =" part can be omitted.
   * - (2)
     - type
     - | **BEGIN**
       | A transaction token is created and a new transaction is started.
       | 
       | **IN**
       | Transaction token is validated.
       | When the requested token value and the token value stored on the server match, the token value of transaction token is updated.
       |
     - IN
     - | type = TransactionTokenType.BEGIN
       |
       | type = TransactionTokenType.IN
       |

 .. note::
 
    It is recommended that the value to be set in "value" attribute should be same as the config value of "value" attribute for \ ``@RequestMapping``\  annotation.

 .. note::
 
    In "type" attribute, **NONE** and **END** can be specified; however, the description is omitted as normally they are not used.

|

Format of transaction token
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Format for the transaction token used in the transaction token check of common library is as follows:

 .. figure:: ./images/transaction-token-name-pattern.png
   :alt: format of transaction token
   :width: 100%

 .. figure:: ./images/transaction-token-name-pattern-example.png
   :alt: example of transaction token
   :width: 100%

|

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.15\linewidth}|p{0.75\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 15 75

   * - Sr. No.
     - Components
     - Description
   * - | (1)
     - NameSpace
     - * NameSpace is an element for assigning a logical name to identify a series of screen transitions.
       * By setting a NameSpace, it is possible to prevent the requests belonging to different Namespaces from interfering with each other and it is also possible to increase the number of screen transitions that can operate in parallel.
       * The value specified in the "value" attribute of \ ``@TransactionTokenCheck``\  annotation is used as the value to be used for NameSpace.
       * When both "value" attribute of the "class" annotation and "value" attribute of the "method" annotation are specified, the value which concatenates both the values with \ ``"/"``\  is used as NameSpace. When the same value is specified in multiple methods, the methods belong to same NameSpace.
       * When the "value" attribute is specified only in "class" annotation, all the NameSpaces of the transaction tokens generated in that class will be the value specified in "class" annotation.
       * When the "value" attribute is specified only in "method" annotation, the NameSpace of the generated transaction tokens will be the value specified in "method" annotation. When the same value is specified in multiple methods, the methods belong to same NameSpace.
       * When both "value" attribute of "class" annotation and "value" attribute of "method" annotation are omitted, the method belong to the global token. For global token, refer to \ :ref:`doubleSubmit_appendix_global_token`\ .
   * - | (2)
     - TokenKey
     - * TokenKey is an element for identifying the transactions stored in the Namespace.
       * TokenKey is generated upon execution of a method wherein \ ``TransactionTokenType.BEGIN``\  is declared in the "type" attribute of \ ``@TransactionTokenCheck``\  annotation.
       * | A maximum limit exists for the number of multiple TokenKeys which can be concurrently stored and the default number is 10. The count of stored TokenKeys is managed for each NameSpace.
       * | When the number of TokenKeys stored for each NameSpace reaches the maximum value at the time of \ ``TransactionTokenType.BEGIN``\ , a new transaction will be stored as a valid transaction by discarding the TokenKey with the oldest date and time of execution (Least Recently Used (LRU)).
       * | When the access is made by using the discarded transaction token, a transaction token error is thrown.
   * - | (3)
     - TokenValue
     - * TokenValue is an element for storing the token value of the transaction.
       * TokenValue is generated upon execution of the method wherein \ ``TransactionTokenType.BEGIN``\  or \ ``TransactionTokenType.IN``\  is declared in the "type" attribute of \ ``@TransactionTokenCheck``\  annotation.

 .. warning::
 
    When the "value" attribute is specified only in "method" annotation and if the same value is specified in another Controller, it should be noted that it will be handled as a request for carrying out a series of screen transitions.
    "value" attribute should be specified by this method only when the screen transitions across Controllers are to be treated as the same transaction.
    
    Basically, it is recommended not to use the method wherein "value" attribute is specified only in "method" annotation.

 .. note::
 
    The method for specifying NameSpace is classified according to creation granularity of the Controller,
    
    * when "value" attributes of both "class" annotation and "method" annotation are specified
    * when "value" attribute is specified only in "class" annotation
    
    
    
    1. | When a processing method which corresponds to multiple usecases is to be implemented in Controller, "value" attributes of both "class" annotation and "method" annotation are specified.
       | For example, this pattern is used when registration, change, deletion of users is to be implemented in a single Controller.
    2. | When a processing method which corresponds to a single usecase is to be implemented in Controller, "value" attribute is specified only in "class" annotation.
       | For example, this pattern is used when a Controller is implemented for every registration, modification, deletion of users.

|

.. _LifeCycle:

Lifecycle of transaction token
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The lifecycle (Generate, Update, Discard) control of transaction token is performed in the following scenarios.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 70

   * - Sr. No.
     - Lifecycle Control
     - Description
   * - | (1)
     - | Token Generation
     - | A new token is generated and transaction is started when the processing of the method wherein \ ``TransactionTokenType.BEGIN``\  is specified in "type" attribute of \ ``@TransactionTokenCheck``\  annotation, is terminated.
   * - | (2)
     - | Token Update
     - | The token (TokenValue) is updated and transaction is continued when the processing of the method wherein \ ``TransactionTokenType.IN``\  is specified in "type" attribute of \ ``@TransactionTokenCheck``\  annotation, is terminated.
   * - | (3)
     - | Token Discard
     - | The tokens are discarded in any of the following scenarios and the transaction is terminated.
       |
       | [1]
       | When the method wherein \``TransactionTokenType.BEGIN``\  is specified in "type" attribute of \ ``@TransactionTokenCheck``\  annotation, is called, the transaction token specified in the request parameter is discarded and unnecessary transaction is terminated.
       |
       | [2]
       | If a new transaction starts when number of transaction tokens (TokenKey) that can be stored in NameSpace has reached the maximum limit, the transaction token with the oldest date and time of execution is discarded and the transaction is forcibly terminated.
       |
       | [3]
       | When exceptions such as system error occur, the transaction token specified in the request parameter is discarded and the transaction is terminated.

 .. note::
 
    A maximum limit is provided for the number of transaction tokens (TokenKey) which can be stored in a NameSpace. When the maximum limit is reached
    while generating a new transaction token, a new transaction is managed as a valid transaction,
    by discarding the transaction token which has the TokenKey with the oldest date and time of execution (Least Recently Used (LRU)).

    The maximum limit of transaction tokens that can be stored for each NameSpace is 10 by default.
    To change the maximum limit, refer to \ :ref:`doubleSubmit_how_to_extend_change_max_count`\  .

|

| The behavior when the maximum limit is reached while generating a new transaction token is explained below.
| The pre-requisites are as given below.

* A default value (10) is specified as the maximum limit for the number of transaction tokens that can be stored in the NameSpace.
* \ ``@TransactionTokenCheck("name")``\  is specified as the class annotation of Controller.
* Transaction tokens of the same NameSpace have reached the maximum limit.

 .. figure:: ./images/transaction-token-count.png
   :alt: transaction token count
   :width: 100%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | A request to start a new transaction is received when the transaction tokens of the same NameSpace have reached the maximum limit.
   * - | (2)
     - | A new transaction token is generated.
   * - | (3)
     - | The generated transaction token is added to the location where tokens are stored.
       | ** At this point, the transaction tokens that exceed the maximum limit are present in the NameSpace.**
   * - | (4)
     - | The transaction tokens exceeding the maximum limit are deleted from the NameSpace.
       | **The transaction tokens are deleted in a sequence starting with the transaction token with the oldest date and time of execution.**

|

.. _setting:

Settings for using a transaction token check
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The settings for using the transaction token check provided by the common library are shown below.

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 2-9,16,17

    <mvc:interceptors>
        <mvc:interceptor> <!-- (1) -->
            <mvc:mapping path="/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/resources/**" /> <!-- (2) -->
            <mvc:exclude-mapping path="/**/*.html" /> <!-- (2) -->
            <!-- (3) -->
            <bean
                class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>

    <bean id="requestDataValueProcessor"
        class="org.terasoluna.gfw.web.mvc.support.CompositeRequestDataValueProcessor">
        <constructor-arg>
            <util:list>
                <!-- (4) -->
                <bean class="org.terasoluna.gfw.web.token.transaction.TransactionTokenRequestDataValueProcessor" />
                <!-- omitted -->
            </util:list>
        </constructor-arg>
    </bean>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | Set \ ``HandlerInterceptor``\  to generate and check transaction tokens.
   * - | (2)
     - | Specify a request path wherein \ ``HandlerInterceptor``\  is to be applied.
       | In the above example, it is applicable to all the requests except the requests under /resources and the requests to HTML.
   * - | (3)
     - | Specify a class (\ ``TransactionTokenInterceptor``\ ) to generate and check transaction tokens using \ ``@TransactionTokenCheck``\  annotation.
   * - | (4)
     - | Set a class (\ ``TransactionTokenRequestDataValueProcessor``\ ) for automatic embedding of the transaction token to the Hidden area using  \ ``<fomr:form>``\  tag of Spring MVC.


Settings for handling transaction token errors
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When a transaction token error occurs, \``org.terasoluna.gfw.web.token.transaction.InvalidTransactionTokenException``\  is generated.

| Therefore, in order to handle transaction token errors, it is necessary to add the handling definition of \ ``InvalidTransactionTokenException``\  to the following settings.

* \ ``ExceptionCodeResolver``\  defined in :file:`applicationContext.xml`
* \ ``SystemExceptionResolver``\  defined in :file:`spring-mvc.xml`



For adding the settings, refer to the following:

* :ref:`exception-handling-how-to-use-application-configuration-common-label`
* :ref:`exception-handling-how-to-use-application-configuration-app-label`




How to use transaction token check in Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| In order to perform transaction token check, it is necessary to define the method to start the transaction and the method to carry out the checks in Controller.
| The explanation below is about the implementation of processing method required in a single usecase using a controller.

- Controller

 .. code-block:: java
    :emphasize-lines: 3,12,18,24,30,32,36

    @Controller
    @RequestMapping("transactionTokenCheckExample")
    @TransactionTokenCheck("transactionTokenCheckExample") // (1)
    public class TransactionTokenCheckExampleController {

        @RequestMapping(params = "first", method = RequestMethod.GET)
        public String first() {
            return "transactionTokenCheckExample/firstView";
        }

        @RequestMapping(params = "second", method = RequestMethod.POST)
        @TransactionTokenCheck(type = TransactionTokenType.BEGIN) // (2)
        public String second() {
            return "transactionTokenCheckExample/secondView";
        }

        @RequestMapping(params = "third", method = RequestMethod.POST)
        @TransactionTokenCheck // (3)
        public String third() {
            return "transactionTokenCheckExample/thirdView";
        }

        @RequestMapping(params = "fourth", method = RequestMethod.POST)
        @TransactionTokenCheck // (3)
        public String fourth() {
            return "transactionTokenCheckExample/fourthView";
        }

        @RequestMapping(params = "fifth", method = RequestMethod.POST)
        @TransactionTokenCheck // (3)
        public String fifth() {
            return "redirect:/transactionTokenCheckExample?complete";
        }

        @RequestMapping(params = "complete", method = RequestMethod.GET) 
        public String complete() { // (4)
            return "transactionTokenCheckExample/fifthView";
        }

    }


 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | NameSpace is specified in "value" attribute of "class" annotation.
       | In the above example, the value same as the "value" attribute of \ ``@RequestMapping``\  which is the recommended pattern of this guideline is specified.
   * - | (2)
     - | The transaction is started and a new transaction token is issued.
       | Here, since the transaction tokens are managed at Controller level, "value" attribute of "method" annotation is not specified.
   * - | (3)
     - | The transaction token is checked and transaction token value is updated.
       | If the type attribute is omitted, the behavior remains the same as when \ ``@TransactionTokenCheck(type = TransactionTokenType.IN)``\  is specified.
   * - | (4)
     - | Since it is not necessary to perform transaction token check in the request for displaying a screen which notifies the completion of the usecase, \ ``@TransactionTokenCheck``\  annotation is not specified.

 .. note::

    * When BEGIN is specified in the "type" attribute of \ ``@TransactionTokenCheck``\  annotation, transaction token is not checked since a new TokenKey is generated.
    * When IN is specified in the "type" attribute of \ ``@TransactionTokenCheck``\  annotation, it is checked whether the token value specified in the request and the token value stored on the server are the same.

.. _doubleSubmit_how_to_use_transaction_token_check_jsp:

How to use transaction token check in View (JSP)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| When transaction token is to be checked, View (JSP) should be implemented in such a way that the issued transaction token is submitted as a request parameter.
| A method wherein a transaction is automatically embedded in hidden elements by using \ ``<form:form>``\  tag is recommended as a method to submit it as a request parameter after carrying out \ :ref:`setting`\ .

- :file:`firstView.jsp`

 .. code-block:: jsp

    <h1>First</h1>
    <form:form method="post" action="transactionTokenCheckExample">
      <input type="submit" name="second" value="second" />
    </form:form>

- :file:`secondView.jsp`

 .. code-block:: jsp
    :emphasize-lines: 2

    <h1>Second</h1>
    <form:form method="post" action="transactionTokenCheckExample"><!-- (1) -->
      <input type="submit" name="third" value="third" />
    </form:form>

- :file:`thirdView.jsp`

 .. code-block:: jsp
    :emphasize-lines: 2

    <h1>Third</h1>
    <form:form method="post" action="transactionTokenCheckExample"><!-- (1) -->
      <input type="submit" name="fourth" value="fourth" />
    </form:form>

- :file:`fourthView.jsp`

 When  \ ``<form:form>``\ tag is used

 .. code-block:: jsp
    :emphasize-lines: 2

    <h1>Fourth</h1>
    <form:form method="post" action="transactionTokenCheckExample"><!-- (1) -->
      <input type="submit" name="fifth" value="fifth" />
    </form:form>

.. _fourthView:

 When  \``<form>`` tag of HTML is used

 .. code-block:: jsp
    :emphasize-lines: 3,4-6

    <h1>Fourth</h1>
    <form method="post" action="transactionTokenCheckExample">
      <t:transaction /><!-- (2) -->
      <!-- (3) -->
      <input type="hidden" name="${f:h(_csrf.parameterName)}"
                           value="${f:h(_csrf.token)}"/>
      <input type="submit" name="fifth" value="fifth" />
    </form>

- :file:`fifthView.jsp`

 .. code-block:: jsp

    <h1>Fifth</h1>
    <form:form method="get" action="transactionTokenCheckExample">
      <input type="submit" name="first" value="first" />
    </form:form>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | When \ ``<form:form>``\  tag is used in JSP and if BEGIN or IN is specified in "type" attribute of \ ``@TransactionTokenCheck``\  annotation, the Value of \ ``name="_TRANSACTION_TOKEN"``\  is automatically embedded as a hidden tag.
   * - | (2)
     - | When \ ``<form>``\  tag of HTML is used, a hidden tag same as (1) is embedded using \ ``<t:transaction />``.
   * - | (3)
     - | When \ ``<form>``\  tag of HTML is used, csrf token necessary for CSRF token check provided by Spring Security needs to be embedded as a hidden item.
       | For csrf token necessary for CSRF token check, refer to  \ :ref:`csrf_formtag-use`\ .

 .. note::
    
    If \ ``<form:form>``\  tag is used, the parameters necessary for CSRF token check are also automatically embedded. Refer to \ :ref:`csrf_formformtag-use`\  for the parameters necessary for CSRF token check.

 .. note::
    
    \ ``<t:transaction />``\  is a JSP tag library provided by the common library.
    For the "t:" used in (2), refer to  \ :ref:`view_jsp_include-label`\ .

* Example of Output HTML

 .. figure:: ./images/transaction-token-html.png
   :alt: transaction token html
   :width: 100%

Following observations can be made upon verifying the output HTML.

* | For NameSpace, the value specified in "value" attribute of the class annotation is set.
  | In the above example, \ ``"transactionTokenCheckExample"``\  (underlined in orange) is the NameSpace.
* | For TokenKey, the value that was issued at the start of the transaction is circulated and set.
  | In the above example, \ ``"c0123252d531d7baf730cd49fe0422ef"``\  (underlined in blue) is the TokenKey.
* | Value to be set for TokenValue varies depending on request.
  | In the above example, \ ``"3f610684e1cb546a13b79b9df30a7523"``\ , \ ``"da770ed81dbca9a694b232e84247a13b"``\ ,
  | \ ``"bd5a2d88ec446b27c06f6d4f486d4428"``\  (underlined in green) are TokenValues.




When multiple usecases are to be implemented in one Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

| The implementation example of the transaction token check while carrying out processing of multiple usecases in one controller is given below.
| In the example given below, (2), (3), (4) are handled as screen transitions of different usecases.

- Controller

 .. code-block:: java
    :emphasize-lines: 3,16-17,25-26,41-42,50-51,66-67,75-76

    @Controller
    @RequestMapping("transactionTokenChecFlowkExample")
    @TransactionTokenCheck("transactionTokenChecFlowkExample") // (1)
    public class TransactionTokenCheckFlowExampleController {

        @RequestMapping(value = "flowOne",
                        params = "first", 
                        method = RequestMethod.GET)
        public String flowOneFirst() {
            return "transactionTokenChecFlowkExample/flowOneFirstView";
        }

        @RequestMapping(value = "flowOne",
                        params = "second",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowOne",
                               type = TransactionTokenType.BEGIN) // (2)
        public String flowOneSecond() {
            return "transactionTokenChecFlowkExample/flowOneSecondView";
        }

        @RequestMapping(value = "flowOne",
                        params = "third",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowOne",
                               type = TransactionTokenType.IN)   // (2)
        public String flowOneThird() {
            return "transactionTokenChecFlowkExample/flowOneThirdView";
        }

        @RequestMapping(value = "flowTwo",
                       params = "first",
                        method = RequestMethod.GET)
        public String flowTwoFirst() {
            return "transactionTokenChecFlowkExample/flowTwoFirstView";
        }

        @RequestMapping(value = "flowTwo",
                        params = "second",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowTwo",
                               type = TransactionTokenType.BEGIN) // (3)
        public String flowTwoSecond() {
            return "transactionTokenChecFlowkExample/flowTwoSecondView";
        }

        @RequestMapping(value = "flowTwo",
                        params = "third",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowTwo",
                               type = TransactionTokenType.IN) // (3)
        public String flowTwoThird() {
            return "transactionTokenChecFlowkExample/flowTwoThirdView";
        }

        @RequestMapping(value = "flowThree",
                        params = "first",
                        method = RequestMethod.GET)
        public String flowThreeFirst() {
            return "transactionTokenChecFlowkExample/flowThreeFirstView";
        }

        @RequestMapping(value = "flowThree",
                        params = "second",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowThree",
                               type = TransactionTokenType.BEGIN) // (4)
        public String flowThreeSecond() {
            return "transactionTokenChecFlowkExample/flowThreeSecondView";
        }

        @RequestMapping(value = "flowThree",
                        params = "third",
                        method = RequestMethod.POST)
        @TransactionTokenCheck(value = "flowThree",
                               type = TransactionTokenType.IN) // (4)
        public String flowThreeThird() {
            return "transactionTokenChecFlowkExample/flowThreeThirdView";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | NameSpace is specified in "value" attribute of "class" annotation.
       | In the above example, the value same as the "value" attribute of \ ``@RequestMapping``\  which is a recommended pattern of this guideline, is specified.
   * - | (2)
     - | The transaction token is checked for processing of usecase with name \ ``"flowOne"``\ .
       | In the above example, the value same as the "value" attribute of \ ``@RequestMapping``\  which is a recommended pattern of this guideline, is specified.
   * - | (3)
     - | The transaction token is checked for processing of usecase with name \ ``"flowTwo"``\ .
       | In the above example, the value same as the "value" attribute of \ ``@RequestMapping``\  which is a recommended pattern of this guideline, is specified.
   * - | (4)
     - | The transaction token is checked for processing of usecase with name \ ``"flowThree"``\ .
       | In the above example, the value same as the "value" attribute of \ ``@RequestMapping``\  which is a recommended pattern of this guideline, is specified.

 .. note::
 
    Allocating a NameSpace for each usecase enables transaction token check for each usecase.


Typical example of using transaction token check
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

See the example below wherein transaction token check is applied for the usecase which carries out a simple screen transition such as "Input Screen-> Confirmation Screen-> Completion Screen".

- Controller

 .. code-block:: java
    :emphasize-lines: 3,9,16-17,27,37

    @Controller
    @RequestMapping("user")
    @TransactionTokenCheck("user") // (1)
    public class UserController {

        // omitted

        @RequestMapping(value = "create", params = "form")
        public String createForm(UserCreateForm form) { // (2)
          return "user/createForm";
        }

        @RequestMapping(value = "create", 
                      params = "confirm", 
                      method = RequestMethod.POST)
        @TransactionTokenCheck(value = "create", 
                             type = TransactionTokenType.BEGIN) // (3)
        public String createConfirm(@Validated
        UserCreateForm form, BindingResult result) {

            // omitted

            return "user/createConfirm";
        }

        @RequestMapping(value = "create", method = RequestMethod.POST)
        @TransactionTokenCheck(value = "create") // (4)
        public String create(@Validated
        UserCreateForm form, BindingResult result) {

            // omitted

            return "redirect:/user/create?complete";
        }

        @RequestMapping(value = "create", params = "complete")
        public String createComplete() { // (5)
            return "user/createComplete";
        }
      
        // omitted

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | A NameSpace called \ ``"user"``\  is set as "class" annotation.
       | In the above example, the value same as "value" attribute of \ ``@RequestMapping``\  annotation which is a recommended pattern, is specified.
   * - | (2)
     - | A processing method to display input screen.
       | **It is a screen to start a usecase; however since the process only displays data and does not involve data update, it is not necessary to start a transaction.**
       | Therefore, \ ``@TransactionTokenCheck``\  annotation is not specified in the example given above.
   * - | (3)
     - | A processing method to perform input validation and display the Confirmation screen.
       | A transaction is started at this point since a button to perform update process is placed on the Confirmation screen.
       | A View (JSP) is specified for the transition destination.
   * - | (4)
     - | A processing method to perform update.
       | **Since this method performs update, a transaction token check is performed.**
   * - | (4)
     - | A processing method to display the Completion screen.
       | **Since the method only displays a Completion screen, the transaction token check is not required.**
       | Therefore, \ ``@TransactionTokenCheck``\  annotation is not specified in the example given above.

 .. warning::

    It is necessary to specify the View (JSP) for the transition destination of processing method wherein \ ``@TransactionTokenCheck``\  annotation is defined.
    If other than View (JSP) of the redirect destination is specified as transition destination, the value of TransactionToken changes in the next process always resulting in the TransactionToken error.

Exclusion control of parallel processing while using a session
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
When a form object is stored in a session using \ ``@SessionAttribute``\  annotation,
and if multiple screen transitions of the same processing are performed in parallel, screen operations may interfere with each other and the values displayed on the screen and the values stored in the session may no longer match.

Transaction token check function can be used to prevent requests from non-conforming screens as the invalid requests.

The maximum limit of transaction tokens that can be stored for each NameSpace is set to 1.

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 6

    <mvc:interceptor>
        <mvc:mapping path="/**" />
        <!-- omitted -->
        <bean
            class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor">
            <constructor-arg value="1"/> <!-- (1) -->
        </bean>
    </mvc:interceptor>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | The number of transaction tokens stored for each NameSpace is set to "1".

 .. note::
 
    When form objects etc. are stored in session using \ ``@SessionAttribute``\  annotation, the requests from the screen displaying old data can be prevented as invalid requests
    by setting number of transaction token stored for each NameSpace to "1".

|

How to extend
--------------------------------------------------------------------------------

How to manage the lifecycle of transaction tokens using a program
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to receive \ ``org.terasoluna.gfw.web.token.transaction.TransactionTokenContext``\  as an argument for processing method of Controller and manage the lifecycle of transaction tokens programmatically by adding the settings give below.

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 3-5

    <mvc:annotation-driven>
      <mvc:argument-resolvers>
        <!-- (1) -->
        <bean
          class="org.terasoluna.gfw.web.token.transaction.TransactionTokenContextHandlerMethodArgumentResolver" />
      </mvc:argument-resolvers>
    </mvc:annotation-driven>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | In \ ``<mvc:argument-resolvers>``\  element, set the class (\ ``TransactionTokenContextHandlerMethodArgumentResolver``\ ) which passes the object (\ ``TransactionTokenContext``\ ) managing the lifecycle of transaction tokens programmatically, as an argument for methods of Controller.
       | When it is not necessary to manage the lifecycle of transaction tokens using a program, this setting is not required.

 .. note::
 
    This setting is not required as the transaction tokens that can no longer be used are automatically discarded when the tokens that can be stored in a NameSpace exceeds the maximum limit.

.. _doubleSubmit_how_to_extend_change_max_count:

How to change the maximum limit of transaction tokens
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The maximum limit of transaction tokens that can be stored on 1 NameSpace can be changed by performing settings given below.

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 8

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/resources/**" />
            <mvc:exclude-mapping path="/**/*.html" />
            <bean
                class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor" />
            <constructor-arg value="5"/> <!-- (1) -->
        </mvc:interceptor>
    </mvc:interceptors>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | The maximum limit of transaction tokens that can be stored in a NameSpace is specified as a value of constructor of \ ``TransactionTokenInterceptor``\ .
       | The default value (value that is set when the default constructor is used) is 10.
       | In the above example, the default value (10) is changed to 5.

Appendix
--------------------------------------------------------------------------------

.. _double-submit_disable-cache:

Transaction Token Check in case that the cache of browser is disabled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If the cache of web browser is disabled due to \ ``Cache-Control``\  in HTTP response header,
the message will display which tells the cache has been expired before transaction token error
in case of the illegal operation flow in "\ :ref:`double-submit_transactiontokencheck`\ "

Concretely, the following screen will display when the browser back button is clicked (8).
This is an example of Internet Explorer 11.

 .. figure:: ./images_DoubleSubmitProtection/page-expired.png
    :width: 60%

There is no problem because the double submit itself is prevented.

In \ :doc:`blank projects <../ImplementationAtEachLayer/CreateWebApplicationProject>`\  after 5.0.0.RELEASE,
it is configured so that the cache is disabled by \ :ref:`Spring Security <SpringSecurityAppendixSecHeaders>`\ .

If showing the transaction error screen is preferred instead of the screen above,
excluding \ ``<sec:cache-control />``\  is required.
However, \ ``<sec:cache-control />``\  should be configured from the point of view of security.

.. _doubleSubmit_appendix_global_token:

Global Tokens
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| If the "value" attribute of \ ``@TransactionTokenCheck``\  annotation is not specified, it is handled as global transaction token.
| \ ``"globalToken"``\  (fixed value) is used for the NameSpace of global transaction tokens.

 .. note::

    When only a single screen transition is to be allowed as an overall application, it can be implemented by setting the maximum limit of transaction tokens that can be stored for each NameSpace to 1 and using the global token.

The settings and implementation example when only a single screen transition is to be allowed as an overall application are shown below.
 
Changing the maximum limit of transaction tokens that can be stored for each NameSpace
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The maximum limit of transaction tokens that can be stored for each NameSpace is set to 1.

- :file:`spring-mvc.xml`

 .. code-block:: xml
    :emphasize-lines: 6

    <mvc:interceptor>
        <mvc:mapping path="/**" />
        <!-- omitted -->
        <bean
            class="org.terasoluna.gfw.web.token.transaction.TransactionTokenInterceptor">
            <constructor-arg value="1"/> <!-- (1) -->
        </bean>
    </mvc:interceptor>

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | The number of tokens stored for each NameSpace is set to "1".

Implementation of Controller
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

The value is not specified in "value" attribute of \ ``@TransactionTokenCheck``\  annotation, in order to make it the NameSpace for global tokens.

- Controller

 .. code-block:: java
    :emphasize-lines: 3,11,17,23

    @Controller
    @RequestMapping("globalTokenCheckExample")
    public class GlobalTokenCheckExampleController { // (1)

        @RequestMapping(params = "first", method = RequestMethod.GET)
        public String first() {
            return "globalTokenCheckExample/firstView";
        }

        @RequestMapping(params = "second", method = RequestMethod.POST)
        @TransactionTokenCheck(type = TransactionTokenType.BEGIN) // (2)
        public String second() {
            return "globalTokenCheckExample/secondView";
        }

        @RequestMapping(params = "third", method = RequestMethod.POST)
        @TransactionTokenCheck // (2)
        public String third() {
            return "globalTokenCheckExample/thirdView";
        }

        @RequestMapping(params = "fourth", method = RequestMethod.POST)
        @TransactionTokenCheck // (2)
        public String fourth() {
            return "globalTokenCheckExample/fourthView";
        }

        @RequestMapping(params = "fifth", method = RequestMethod.POST)
        public String fifth() {
            return "globalTokenCheckExample/fifthView";
        }

    }

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90

   * - Sr. No.
     - Description
   * - | (1)
     - | \ ``@TransactionTokenCheck``\  annotation is not specified as the class annotation.
   * - | (2)
     - | The "value" attribute of \ ``@TransactionTokenCheck``\  annotation to be specified as "method" annotation is not specified.

* Example of Output HTML

 | JSP used is same as the one created in \ :ref:`doubleSubmit_how_to_use_transaction_token_check_jsp`\  .
 | Other details are same except change in action from \ ``"transactionTokenCheckExample"``\ to \ ``"globalTokenCheckExample"``\ .

 .. figure:: ./images/transaction-token-global-html.png
   :alt: transaction token global html
   :width: 100%

Following observations can be made upon verifying the output HTML.

* | For NameSpace, a fixed value called \ ``"globalToken"``\  is set.
* | For TokenKey, the value that was issued while starting the transaction is circulated and set.
  | In the above example, \ ``"9d937be4adc2f5dd2032292d153f1133"``\  (underlined in blue) is the TokenKey.
* | Value to be set for TokenValue varies depending on request.
  | In the above example, \ ``"9204d7705ce7a17f16ca6cec24cfd88b"``\ , \ ``"69c809fefcad541dbd00bd1983af2148"``\ ,
  | \ ``"6b83f33b365f1270ee1c1b263f046719"``\  (underlined in green) are TokenValues.



The behavior when the maximum limit of transaction tokens for each NameSpace is set to 1 and global token is used, is explained below.

 .. figure:: ./images/transaction-token-globaltoken.png
   :alt: transaction token globaltoken
   :width: 90%

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 90


   * - Sr. No.
     - Description
   * - | (1)
     - | In window 1 process, TransactionTokenType.BEGIN is performed and global token is generated.
   * - | (2)
     - | In window 2 process, the token is updated by TransactionTokenType.BEGIN.
       | Although internally the data is reshuffled rather than getting updated, it gives the impression that the token has been updated since one transaction token can be stored on the server.
   * - | (3)
     - | Token value is checked in TransactionTokenType.IN of window 1 process.
       | \ **Transaction token generated in process 1 is submitted as request parameter; however since the specified token does not exist on the server, a transaction token error occurs.**\ 
   * - | (4)
     - | Token value is checked in TransactionTokenType.IN of window 2 process.
       | The transaction token generated in process 2 is submitted as request parameter and it is checked whether the value matches with the token value stored on the server.
       | If the value matches, the process is continued.
   * - | (5)
     - | Similar to (4) .
   * - | (6)
     - | Similar to (4) .
   * - | (7)
     - | When a screen is to be displayed using redirect, hidden tag for transaction token does not exist.

 .. note::
 
    The transaction token existing on the server is automatically deleted when a new global token is generated.


Quick Reference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


| The table below illustrates an example of a business application for managing Account and Customer. It shows the required settings for transaction tokens and business limitations.
| The usecases assumed in this business application are Create, Update, Delete relating to Account and Customer.
| By using the table below as reference, the maximum limit of tokens and NameSpace settings should be performed as per the system requirements.

 .. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|p{0.15\linewidth}|p{0.20\linewidth}|
 .. list-table::
   :header-rows: 1
   :widths: 10 20 15 20 15 20

   * - Number
     - Number of tokens stored for each NameSpace
     - NameSpace value specified in class
     - NameSpace value specified in method
     - Example of generated token
     - Business limitations
   * - | (1)
     - | 10 (Default)
     - | account
     - | Not specified
     - | account~key~value
     - | Number of concurrent executions of all Account usecases (create/update/delete) is restricted to 10.
   * - | (2)
     - | 10 (Default)
     - | account
     - | create
     - | account/create~key~value
     - | Number of concurrent executions of "create" operation of Account usecase is restricted to 10.
   * - | (3)
     - | 10 (Default)
     - | account
     - | update
     - | account/update~key~value
     - | Number of concurrent executions of "update" operation of Account usecase is restricted to 10.
   * - | (4)
     - | 10 (Default)
     - | account
     - | delete
     - | account/delete~key~value
     - | Number of concurrent executions of "delete" operation of Account usecase is restricted to 10. (By specifying  (2), (3) and (4), the number of concurrent executions of all Account usecases should be 30. Since this setting value is too high for most applications, a value smaller than default value 10 can also be specified.)
   * - | (5)
     - | 10 (Default)
     - | Unspecified
     - | create
     - | create~key~value
     - | The same Namespace called "create" is created in the application and the number of concurrent executions is restricted to 10. When the Account and Customer business processes are independent, and among them when "create" is specified in the NameSpace of TransactionToken using "create" method, the total number of concurrent executions is restricted to 10.
   * - | (6)
     - | 10 (Default)
     - | Not specified
     - | update
     - | update~key~value
     - | Same as (5)
   * - | (7)
     - | 10 (Default)
     - | Not specified
     - | delete
     - | delete~key~value
     - | Same as (5)
   * - | (8)
     - | 10 (Default)
     - | Not specified
     - | Not specified
     - | globalToken~key~value
     - | The total number of concurrent executions of all Account and Customer usecases is restricted to 10.
   * - | (9)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | Not specified
     - | account~key~value
     - | The number of concurrent executions of all Account usecases should be restricted to 1. Execution of only a single business process is possible for Account create/update/delete. This is valid when screen transition using only 1 screen is assumed.
   * - | (10)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | create
     - | account/create~key~value
     - | The number of concurrent executions of "create" operation of Account usecases should be restricted to 1. It is not possible to simultaneously execute Account "create" by opening 2 screens.
   * - | (11)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | update
     - | account/update~key~value
     - | Same as (10)
   * - | (12)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | account
     - | delete
     - | account/delete~key~value
     - | Same as (10)
   * - | (13)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | Not specified
     - | create
     - | create~key~value"
     - | A same NameSpace called "create" is created in the application and the number of concurrent executions should be restricted to 1. When the Account and Customer business processes are independent and when "create" is specified in NameSpace of TransactionToken using "create" method, "create" for Account and Customer cannot be performed concurrently
   * - | (14)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | Not specified
     - | update
     - | update~key~value
     - | Same as (13)
   * - | (15)
     - |  1 (Custom Setting in spring-mvc.xml)
     - | Not specified
     - | delete
     - | delete~key~value
     - | Same as (13)
   * - | (16)
     - | 1 (Custom Setting in spring-mvc.xml)
     - | Not specified
     - | Not specified
     - | globalToken~key~value
     - | The business processes that can be concurrently executed in the application are restricted to 1. This should be used in projects where only 1 operation is performed in 1 session.
     
.. raw:: latex

   \newpage
   
