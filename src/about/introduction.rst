Introduction
%%%%%%%%%%%%%%

What is this?
*************

This is an open online book containing short scholarly API code examples (i.e., "recipes") 
that demonstrate how to work with various scholarly web service APIs. It is part of the University of 
Alabama Libraries efforts to support `Research Data Services`_.

.. _Research Data Services: https://guides.lib.ua.edu/ResearchDataServices

What should I be aware of before getting started?
*************************************************

Before interacting with any scholarly APIs (or similar web service), it is very important to review the
usage policies, which generally includes information such as query limits and data reuse policies.
We have endeavored to follow all appropriate API usage policies in our examples and have linked
to the specific API policies where possible. While some APIs are openly accessible and do
not require special authentication, other scholarly APIs require that you are affiliated with a
subscribing institution, and have registered for an API key to use for authentication in API queries.
We have added instructions about any necessary authentication within the relevant code recipes.


.. important::

   In general, scholarly APIs are designed for the collection of small to medium
   sized datasets; that is, in the range of 100s or maybe a few thousand queries at most
   (various with API). If you need large bulk datasets, an API is likely not the method to use, and
   there may be bulk data downloads available from the database instead.

If you decide that your use-case is appropriate for a scholarly API (or similar web service), here are a few good general practices 
to follow when working with any web API:

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


Which Programming Languages are Covered?
****************************************

Currently, we have scholarly API code examples in Python, Bash (Unix Shell), Matlab, Mathematica, R,
and C. We hope that providing examples in several programming languages will 
provide the reader with not only a choice, but also some information about the differences between 
the programming workflows. In some cases, for example, it may be preferable to 
choose one language over the other.

For good luck, let's add ``Hello World!`` in each programming language:

.. tab-set::

   .. tab-item:: Python

      .. code-block:: python

         >>> print("Hello World!")

   .. tab-item:: Bash

      .. code-block:: shell

         $ echo "Hello World!"

   .. tab-item:: Matlab

      .. code-block:: matlab

         >> disp("Hello World!")

   .. tab-item:: Mathematica

      .. code-block:: mathematica

         In[1]:= Print["Hello World!"]

   .. tab-item:: R

      .. code-block:: r

         > print("Hello World!")

   .. tab-item:: C

      .. code-block:: c

         int main() {
            printf("Hello World!\n");
            return 0;
         }

Who is Creating the Content?
****************************

The Scholarly API Cookbook content is authored by University of Alabama 
Libraries faculty and student assistants. Specific authors are noted on each 
tutorial or document page.

.. rubric:: References

.. [#ua_work] `<https://github.com/UA-Libraries-Research-Data-Services/UALIB_Workshops>`_


