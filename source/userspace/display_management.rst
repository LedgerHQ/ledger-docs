==================
Display Management
==================

On-screen display
=================

Screens and Flows
-----------------

Displaying information on the devices is done by defining a :code:`flow`. 
Think of a flow as a **set of screens that will be displayed** to the user.
The user will be able to navigate the flow using **left** and **right** buttons.

The macro :code:`UX_FLOW` lets you define a flow. It first takes the **name of the flow** you are defining, and then the **different steps** of your flow.

Here's an example:

.. code-block:: c

	UX_FLOW(ux_sign_transaction,		    // Name of the flow
		    &step_destination_address,	    // Reference to the first step
		    &step_display_amount,	        // Reference to second step
		    &step_display_transaction_fees,	// etc...
		    &step_display_approve_step,
		    &step_display_reject_step
		    );

"*But what are those steps? Where can I find them?*" you ask? Well, they're **yours to declare in your app**! Let's have a look at how you can declare those!

Declaring steps
---------------

A step is declared using one of the **macros provided by the SDK**. Here's a list of the **most used ones** (a more exhaustive list is available by looking at the SDK source code!):

* :code:`UX_STEP_NOCB` : Your bread and butter step. A step that simply **displays information on-screen**, without having any callback function (hence the :code:`NOCB` annotation!). This is used to display information to the user (display a public key, display the network name...).
* :code:`UX_STEP_CB` : Another widely used step declaration. You need to **associate a callback function** (hence the :code:`CB` annotation!) that will be called if the **user presses both buttons simultaneously**. This is used mainly at the end of a flow, to either APPROVE or REJECT a signature for example.
* :code:`FLOW_LOOP` : This is a special macro that **doesn't take any arguments**. Put it as the last step of your :code:`UX_FLOW` declaration, and the flow will wrap around: pressing the left button on the first screen will display to the last screen, and pressing the right button on the last screen will jump right to the first screen!

Looking back at our previous flow (the :code:`ux_display_pubkey_flow`), you can guess which steps were declared with :code:`UX_STEP_NOCB` and :code:`UX_STEP_CB`!

.. list-table:: Previous example explained
	:widths: 20 20 60
	:header-rows: 1
	
	* - Step
	  - STEP_MACRO
	  - Note
	* - :code:`ux_display_destination_address`
	  - :code:`UX_STEP_NOCB`
	  - This step will just display the destination address to the user, **we're not expecting him to confirm or reject anything**.
	* - :code:`ux_display_amount`
	  - :code:`UX_STEP_NOCB`
	  - This step will just display the amount to the user, **we're not expecting him to confirm or reject**.
	* - :code:`ux_display_transaction_fees`
	  - :code:`UX_STEP_NOCB`
	  - This step will just display the transaction fees to the user, **we're not expecting him to confirm or reject**.
	* - :code:`ux_display_approve_step`
	  - :code:`UX_STEP_CB`
	  - This step will prompt the user to CONFIRM that he wishes to sign the transaction. If the user **presses both button**, the application will **call the callback function** associated with this step (the code to sign a transaction!).
	* - :code:`ux_display_reject_step`
	  - :code:`UX_STEP_CB`
	  - This step will prompt the user to REJECT the signature of the transaction. If the user **presses both button**, the application will **call the callback function** associated with this step (the code to reject the transaction signature).

Here's how the first step is defined:

.. code-block:: c

	UX_STEP_NOCB(
		step_destination_address,
		bnnn_paging,
		{
			.title = "Address",
			.text = g_address
		}
	);

* The first argument is the **name** that we are going to give to this step.
* The second are what we call a :code:`layout` option. 
* The third argument goes hand-in-hand with the :code:`layout` option.

It looks like the :code:`layout` is where the black-magic happens. Let's have a closer look at :code:`layout` s!

Layouts
-------

Layouts are nothing more than rules that specify the format / number of lines / fonts we are going to use on the screen.
They are easy to remember once you understand that:

* :code:`b` stands for **bold**.
* :code:`n` stands for **normal**.
* :code:`p` stands for **picture**.
* :code:`paging` means that if the data doesn't fit on screen, user will be able to navigate through multiple screens to see the data (e.g a public key).
* And the number of letters used stands for the **number of lines**!

Note that **all combinations of letters are not possible**. For example :code:`paging` only exists with :code:`bnnn`. :code:`nnnn` only exists on NanoX, etc...

Now that you know all that, here's a table with the **most commonly used layouts**:

.. list-table:: Most commonly used layouts
	:widths: 5 55 40
	:header-rows: 1
	
	* - Denomination
	  - Comment
	  - Usage
	* - :code:`bn`
	  - Bold font for the first line, normal font for the second line.
	  - :code:`bn, {"BoldLine", "NormalLine"}`
	* - :code:`pb`
	  - Picture for the first line, bold font for the second line.
	  - :code:`pb, {&RefToPicture, "BoldLine"}`
	* - :code:`pnn`
	  - Picture for the first line, normal font for the second line and third line.
	  - :code:`pnn, {&RefToPicture, "NormalLine1", "NormalLine2"}`
	* - :code:`bnnn_paging`
	  - Bold first line, normal fonts for the other lines. If the data to be displayed doesn't fit on a single screen, the user will be able to navigate through different screens to see the whole text.
	  - :code:`bnnn_paging, {.title = "BoldLine", .text = "NormalLine"}`


.. |nanos_bnnn_paging| image:: images/nanos/nanos_address_merged.png
   :scale: 100%
.. |nanox_bnnn_paging| image:: images/nanox/nanox_address_merged.png
   :scale: 100%
.. |nanos_bn| image:: images/nanos/nanos_amount.png
   :scale: 100%
.. |nanox_bn| image:: images/nanox/nanox_amount.png
   :scale: 100%
.. |nanos_pb| image:: images/nanos/nanos_approve.png
   :scale: 100%
.. |nanox_pb| image:: images/nanox/nanox_approve.png
   :scale: 100%  
.. |nanos_pnn| image:: images/nanos/nanos_boilerplate.png
	:scale: 100%
.. |nanox_pnn| image:: images/nanox/nanox_boilerplate.png
   :scale: 100%  

And here's a table that compares how those layouts are displayed on a Nano S and on a Nano X!

Notice that the **Nano X can fit up to 4 lines**, whereas the **Nano S can only fit 2**!

.. list-table:: Comparing end results on NanoS and NanoX
	:widths: 10 40 40
	:header-rows: 1

	* - LAYOUT
	  - NANOS
	  - NANOX
	* - :code:`pb`
	  - |nanos_pb|
	  - |nanox_pb|
	* - :code:`bn`
	  - |nanos_bn|
	  - |nanox_bn|
	* - :code:`pnn`
	  - |nanos_pnn|
	  - |nanox_pnn|
	* - :code:`bnnn_paging`
	  - |nanos_bnnn_paging|
	  - |nanox_bnnn_paging|

You're now ready to go and fly on your owns wings! Flows, steps, and layouts are no mystery to you anymore! We've added a couple of examples just down below, because an example is worth 16x16 words...

Examples
========

Menu
----

Here's a typical flow for any app that will display its name (along with its logo), then its version, then the settings and finally a quit (along with a icon).

.. code-block:: c

	UX_STEP_NOCB(step_menu, pnn, {&C_app_logo, "App", "is ready"});
	UX_STEP_NOCB(step_version, bn, {"Version", &g_version});
	UX_STEP_CB(step_settings, pb, ui_settings_menu(), {&C_icon_settings, "Settings"})
	UX_STEP_CB(step_exit_step, pb, os_sched_exit(-1), {&C_icon_dashboard_x, "Quit"});

	UX_FLOW(ux_app_dashboard,
			&step_menu,
			&step_version,
			&step_settings,
			&step_exit_step,
			FLOW_LOOP
	);

You guessed it, pressing both buttons when on the `QUIT` screen will call :code:`os_sched_exit(-1)`, effectively quitting the app. Pressing both button while on the :code:`Settings` screen will call :code:`ui_settings_menu()`, another function that you need to define with :code:`UX_FLOW` !
We also added the :code:`FLOW_LOOP` step at the end to have the menu wrap around. Users can now indefinitely cycle through the menu, yay!

Signing a transaction
---------------------

Here's the example of a flow to sign a transaction. 
We first display "Confirm address" along with a picture, then use :code:`bnnn_paging` to 
display the address because it might not fit on a single screen.
We then display the amount and the transactions fees, and finally add two callack steps:
the first one to confirm, the second one to reject.

.. code-block:: c

	UX_STEP_NO_CB(step_review, pn, {&C_icon_eye, "Confirm Address"});
	UX_STEP_NO_CB(step_address, bnnn_paging, { .title = "Address", .text = &g_destination_address});
	UX_STEP_NO_CB(step_amount, bn, {"Amount", &g_amount});
	UX_STEP_NO_CB(step_fees, bn, {"Fees", &g_fees});
	UX_STEP_CB(step_approve, pb, sign_transaction(), {&C_icon_validate_14, "Approve"});
	UX_STEP_CB(step_reject, pb, reject_transaction(), {&C_icon_crossmark, "Reject"});

	UX_FLOW(ux_sign_transaction,
			&step_review,
			&step_address,
			&step_amount,
			&step_fees,
			&step_approve,
			&step_reject
		);

Advanced display management
----------------------------

A special :doc:`advanced display management </userspace/advanced_display_management>` section has been written where we detail more advanced UX_FLOW delcaration. Another :doc:`low_level_display_management` </userspace/low_level_display_management>` has details about the inner-workings of flows, but definitely feel free to skip it as it is in no way required to be able to write new apps!
