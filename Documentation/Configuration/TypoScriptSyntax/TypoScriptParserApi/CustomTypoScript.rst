.. include:: /Includes.rst.txt


.. _typoscript-syntax-custom-typoscript:

=========================
Parsing Custom TypoScript
=========================

.. note::

   This example will probably seem rather quaint. However it is
   still useful to illustrate this topic.

Let's imagine that you have created an application in TYPO3 CMS, for
example a plug-in. You have defined certain parameters editable
directly in the form fields of the plug-in content element. However
you want advanced users to be able to set up more detailed parameters.
But instead of adding a host of such detailed options to the interface
- which would just clutter it all up - you rather want advanced users
to have a text area field into which they can enter configuration
codes based on a little reference you make for them.

The reference could look like this:


Root Level
==========

.. confval:: colors

   :Scope: TypoScript
   :type: COLORS

   Defining colors for various elements.


.. confval:: adminInfo

   :Scope: TypoScript
   :type: ADMINFO

   Define administrator contact information for cc-emails


.. confval:: headerImage

   :Scope: TypoScript
   :type: file-reference

   A reference to an image file relative to the website's path
   (:php:`\TYPO3\CMS\Core\Core\Environment::getPublicPath()`)

->COLORS
========

.. confval:: backgroundColor

   :Scope: TypoScript
   :type: HTML-color
   :Default: white

   The background color of ...

.. confval:: fontColor

   :Scope: TypoScript
   :type: HTML-color
   :Default: black

   The font color of text in ...

.. confval:: popUpColor

   :Scope: TypoScript
   :type: HTML-color
   :Default: #333333

   The shadow color of the pop up ...

->ADMINFO
=========

.. confval:: cc_email

   :Scope: TypoScript
   :type: string

   The email address that ...


.. confval:: cc_name

   :Scope: TypoScript
   :type: string

   The name of ...

.. confval:: cc_return_adr

   :Scope: TypoScript
   :type: string
   :Default: [servers]

   The return address of ...

.. confval:: html_emails

   :Scope: TypoScript
   :type: string
   :Default: false

   If set, emails are sent in HTML.

So these are the "objects" and "properties" you have chosen to offer
to your users of the plug-in. This reference defines *what
information makes sense* to put into the TypoScript field
(semantically), because you will program your application to use this
information as needed.


A Case Story
------------

Now let's imagine that a user inputs this TypoScript configuration in
whatever medium you have offered (e.g. a textarea field):

.. code-block:: typoscript

   colors {
     backgroundColor = red
     fontColor = blue
   }
   adminInfo {
     cc_email = email@email.com
     cc_name = Copy Name
   }
   showAll = true

   [ip("123.45.*")]

     headerImage = fileadmin/img1.jpg

   [ELSE]

     headerImage = fileadmin/img2.jpg

   [GLOBAL]

   // Wonder if this works... :-)
   wakeMeUp = 7:00

In order to parse this TypoScript we can use the following code
provided that the variable :php:`$tsString` contains the above TypoScript as
its value:

.. code-block:: php

   $TSparserObject = \TYPO3\CMS\Core\Utility\GeneralUtility::makeInstance(\TYPO3\CMS\Core\TypoScript\Parser\TypoScriptParser::class);
   $TSparserObject->parse($tsString);

   echo '<pre>';
   print_r($TSparserObject->setup);
   echo '</pre>';

As you can see, this is really as simple as creating an instance of the
:php:`TypoScriptParser` class and requesting it to parse the configuration
contained in variable :php:`$tsString`. The result is located in
:php:`$TSparserObject->setup`.

The result of this code will be this:

.. code-block:: php

   Array
   (
     [colors.] => Array
     (
       [backgroundColor] => red
       [fontColor] => blue
     )

     [adminInfo.] => Array
     (
       [cc_email] => email@email.com
       [cc_name] => Copy Name
     )

     [showAll] => true
     [headerImage] => fileadmin/img2.jpg
     [wakeMeUp] => 7:00
   )

Now your application could use this information like this, for example:

.. code-block:: php

     echo '
          <table bgcolor="' . $TSparserObject->setup['colors.']['backgroundColor'] . '">
               <tr>
                    <td>
                         <font color="' . $TSparserObject->setup['colors.']['fontColor'] . '">HELLO WORLD!</font>
                    </td>
               </tr>
          </table>
     ';

As you can see some of the TypoScript properties (or *object paths*)
which are found in the reference tables above are implemented here.
There is not much mystique about this and in fact this is how all
TypoScript is used in its respective contexts; **TypoScript contains
simply configuration values that make our underlying PHP code act
accordingly - parameters, function arguments, as you please;
TypoScript is an API to instruct an underlying system.**

This example also highlights one of the "risk" of TypoScript:
it is perfectly possible to define arbitrary properties without
triggering any error. Wrongly-named properties will just be
ignored. As such they do not cause any harm, but may be confusing
at a later stage if they are left around.
