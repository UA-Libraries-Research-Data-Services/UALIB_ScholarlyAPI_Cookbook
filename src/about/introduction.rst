Introduction
%%%%%%%%%%%%%%

What is this?
*************

This is an open online book containing short scholarly API code examples (i.e., "recipes") 
that demonstrate how to work with various scholarly web service APIs. It is part of the
University of Alabama Libraries efforts to support `Research Data Services`_.

.. _Research Data Services: https://guides.lib.ua.edu/ResearchDataServices

What should I be aware of before getting started?
*************************************************

.. important::

   Before interacting with any scholarly APIs (or similar web services), it is important to
   review the provider's usage policies. These typically outline information such as query
   limits, permitted use-cases, and restrictions on data reuse. In this Cookbook, we have
   endeavored to follow all relevant API usage policies and have linked to the specific policy
   pages whenever possible.

   While some APIs are accessible without special authentication, others require that you are
   affiliated with a subscribing institution and have registered for an API key. Any required
   authentication steps are noted within the individual code recipes.

   For APIs associated with subscription-licensed resources, please contact your affiliated
   institution before getting started. Library or other staff can confirm whether API access is
   included in your institution's subscriptions and provide guidance on setup and use.
   
   Remember that each database or publisher maintains its own terms for downstream use-cases
   such as text and data mining. If you have questions about these policies or appropriate use,
   contact your institution for assistance.

   In general, scholarly APIs are designed for the collection of small to medium
   sized datasets; that is, in the range of 100s or maybe a few thousand queries at most
   (various with API). If you need large bulk datasets, an API is likely not the method to use,
   and there may be bulk data downloads available from the database instead.

If you decide that your use-case is appropriate for a scholarly API (or similar web service),
here are a few good general practices to follow when working with any web API:

1. Read the API documentation and usage guidelines before starting.
2. Start with testing the behavior of the API using a single programmatic API request (i.e., not in a loop).
3. Add a 1 second delay between API requests when using a loop.
4. When using a loop to repeat API requests, start out with a small list, perhaps 3-5.
5. Cache the API returned data when testing. For example, if you are trying to parse the returned API data in a scripting workflow, save the returned data in a variable or to a file so that you do not need to repeat the API request unnecessarily for the downstream parsing or analysis.

What kind of content is included?
*********************************

The scope of this book is to provide short code examples related to the retrieval of data and information
from scholarly APIs using several different programming languages.

While there may be some introductory programming content in this book, the 
content is not meant to be a general introduction to programming. 
Instead, our aim with the Scholarly API Cookbook is to provide 
some short scripting based workflows for working with scholarly data and information APIs. 
For more general introductions to programming, we recommend searching the 
UA Libraries Scout database for programming books (e.g., `TI python`). 

.. seealso::

   UA Libraries Workshop lessons and references therein for more general 
   programming content [#ua_work]_.

What programming languages are covered?
****************************************

Currently, we have scholarly API code examples in Python and R (and a Z39.50 tutorial in Bash). 
For good luck, let's add ``Hello World!`` in each programming language:

.. tab-set::

   .. tab-item:: Python

      .. code-block:: python

         >>> print("Hello World!")

   .. tab-item:: R

      .. code-block:: r

         > print("Hello World!")

Who is creating the content?
****************************

The Scholarly API Cookbook content is authored by University of Alabama 
Libraries faculty and student assistants. Specific authors are noted on each 
tutorial or document page.

.. rubric:: References

.. [#ua_work] `<https://github.com/UA-Libraries-Research-Data-Services/UALIB_Workshops>`_


