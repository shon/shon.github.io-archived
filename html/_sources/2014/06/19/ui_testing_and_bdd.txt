UI Testing and BDD
==================

.. author:: default
.. categories:: tech
.. tags:: python, automation, ui-testing, testing, bdd
.. comments::


Recently I had an opportunity to automate UI testing of the web application we have developed. So I found this excellent UI testing framework `Splinter <http://splinter.cobrateam.info/>`_. It being written in Python I was instantly confortable.



.. code-block:: python

    # From Splinter's website
    from splinter import Browser

     with Browser() as browser:
         url = "http://www.google.com"
         browser.visit(url)
         browser.fill('q', 'splinter python acceptance testing')
         button = browser.find_by_name('btnG')
         button.click()
         if browser.is_text_present('splinter.cobrateam.info'):
             print "Yes, the official website was found!"
         else:
             print "No, it wasn't found... We need to improve"

Ah well sweet. It supports multiple webdrivers including remote. So it's possible to integrate it with `saucelabs <http://saucelabs.com>`_ which makes it possible to test on web browsers which are not on your dev box.

Also that you have access to live browser session in Python makes it even more pleasant.

I happened to read on BDD or (Behavior-driven development) and then soon stumbled upon `behave <https://pythonhosted.org/behave/>`_. It's good idea to use it for testing as it provides nice seperation in test cases implementation and test cases. It uses `The Gherkin language <https://pythonhosted.org/behave/philosophy.html#the-gherkin-language>`_ to describe testing scenarios.

Something like below which even non tech person in your team can write

.. code-block:: Gherkin

    Feature: SEO Test
    
        Scenario: Search Google for Splinter
            When I visit "http://google.com"
            And I fill in "q" with "Splinter Python"
            And I press "btnG"
            Then I should see "splinter.cobrateam" within 5 seconds 
    
        Scenario: Search Google for Shekhar's Blog 
            # would fail
            When I visit "http://google.com"
            And I fill in "q" with "Shekhar Tiwatne"
            And I press "btnG"
            Then I should see "shon.github.io" within 5 seconds 


Environment Setup
-----------------
To make things easier 

.. code-block:: bash

    pip install --upgrade splinter behave behaving
    mkdir -p features/steps
    touch features/steps/__init__.py
    wget https://gist.githubusercontent.com/shon/90ac6af750b575cde050/raw/e89262dd5f722dcc1d83d37c011283330c31e9fc/environment.py
    mv environment.py features

Create features/steps/everything.py with below code. It essentially imports steps implementation.

.. code-block:: python

    from behave import step
    from behaving.web.steps import *
    from behaving.personas.steps import *

Run
---
We are ready. Simply execute *behave* and see firefox window executing all tests for you. You should see one test passed and one failed as intended.

::
    behave

.. image:: images/behave.png
    :width: 1200 px
    :scale: 50 %
    :alt: Bingo!

What next
---------
Read how to implement steps (grammer) `here <https://pythonhosted.org/behave/tutorial.html>`_.
