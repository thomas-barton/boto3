.. _guide_retries:

Retries
=======

Overview
--------

Your AWS client may see calls to AWS services fail due to unforeseen issues on the client side or may see calls fail due to rate-limiting from the AWS service you are attempting to call. In either case, these kinds of failures often don’t require special handling and the call should simply be made again, often after a brief waiting period. Boto 3 provides many features to assist in retrying client calls to AWS services when these kinds of errors or exceptions are experienced. 

Specifically, this guide provides details on:

* How to find the available retry modes and the differences between each mode
* How to configure your client to use each retry mode and other retry configurations
* How to validate if your client performs a retry attempt

Available retry modes
---------------------

Legacy retry mode
~~~~~~~~~~~~~~~~~~

Legacy mode is the default mode used by any Boto3 client you create. As its name implies, ``legacy mode`` utilizes an older (v1) retry handler that has limited functionality. 

**Legacy mode’s functionality includes:**

* Default value for maximum retry attempts of 5. This value can be overwritten through the ``max_attempts`` configuration parameter 
* Retry attempts for a limited number of error/exceptions::

   # General socket/connection errors
   ConnectionError
   ConnectionClosedError
   ReadTimeoutError
   EndpointConnectionError

   # Service-side throttling/limit errors and exceptions
   Throttling
   ThrottlingException
   ThrottledException
   RequestThrottledException
   ProvisionedThroughputExceededException

* Retry attempts on a number of HTTP status codes, including 429, 500, 502, 503, 504, and 509
* Retries include exponential backoff by a base factor of 2


.. note::
   There are some additional service-specific retry policies. For more information, please see `here <https://github.com/boto/botocore/blob/develop/botocore/data/_retry.json>`_.


Standard retry mode
~~~~~~~~~~~~~~~~~~~~

Standard mode is a retry mode that was introduced with the updated retry handler (v2). This mode is a standardization of retry logic and behavior that is consistent with other AWS SDKs. In addition to this standardization, this mode also extends the functionality of retries over that which is found in legacy mode.

**Standard mode’s functionality includes:**

* Default value for maximum retry attempts of 3. This value can be overwritten through the ``max_attempts`` parameter
* Retry attempts for an expanded list of error/exceptions::

   # Transient errors/exception
   RequestTimeout
   RequestTimeoutException
   PriorRequestNotComplete
   ConnectionError
   HTTPClientError

   # Service-side throttling/limit errors and exceptions
   Throttling
   ThrottlingException
   ThrottledException
   RequestThrottledException
   TooManyRequestsException
   ProvisionedThroughputExceededException
   TransactionInProgressException
   RequestLimitExceeded
   BandwidthLimitExceeded
   LimitExceededException
   RequestThrottled
   SlowDown
   EC2ThrottledException

* Retry attempts on non-descriptive, transient errors codes. Specifically, HTTP status codes:  500, 502, 503, 504
* Retries include exponential backoff by a base factor of 2 with a maximum backoff of 20 seconds. 

Adaptive retry mode
~~~~~~~~~~~~~~~~~~~~

Adaptive retry mode is an experimental retry mode that includes all the features of standard mode. In addition to the standard mode features, adaptive mode also introduces client-side rate limiting through the use of a token bucket and rate-limit variables that are dynamically updated with each retry attempt. This mode offers flexibility in client-side retries that adapts to the error/exception state response from an AWS service. 

With each new retry attempt, adaptive mode will modify the rate-limit variables based on the error, exception, or HTTP status code presented in the response from the AWS service. These rate-limit variables are then used to calculate a new call rate for the client. Each non-success response from an AWS service will update the rate-limit variables as retries occur until either success is reached, the token bucket is exhausted, or the configured max attempts value has been reached. 

.. note:: 
   Adaptive mode is an experimental mode and is subject to change, both in features and behavior. 


Configuring a retry mode
-------------------------

Available configuration options
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In Boto 3, there are two retry configurations available for users to customize:

* ``retry_mode`` - This tells Boto 3 which retry mode to use. As listed above, there are three retry modes available: legacy (default), standard, and adaptive. 
* ``max_attempts`` - This provides Boto 3’s retry handler with a value of max retry attempts, where the initial call counts towards the ``max_attempts`` value that you have provided. 

Defining a retry configuration in your AWS configuration file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This first method of defining your retry configuration is to update your global AWS configuration file. The default location for your AWS config file is ``~/.aws/config``. Here’s an example of an AWS config file with the retry configuration options used::

   [myConfigProfile]
   region = us-east-1
   max_attempts = 10
   retry_mode = standard

Any Boto 3 scripts or code that use your AWS config file will inherit these configurations when using your profile, unless otherwise explicitly overwritten by a ``Config`` object when instantiating your client object at runtime. If no configuration options are set, the default retry mode value is ``legacy`` and the default ``max_attempts`` value is 5. 

Defining a retry configuration in a Config object for your Boto 3 client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you do not wish to configure retry behavior globally with your AWS config file, botocore allows additional flexibility by specifying your retry configuration using a ``Config`` object that you can pass to your client at runtime. 

Additionally, if your AWS configuration file is configured with retry behavior but you’d like to override those global settings, you can use the ``Config`` object to override an individual client object at runtime. 

As you can see in the example below, the ``Config`` object takes a retries dictionary where we can supply our two configuration options, ``max_attempts`` and ``mode``, and the values we want each to be:

.. code-block:: python

   config = Config(
      retries = {
         'max_attempts': 10,
         'mode': 'standard'
      }
   )

.. note:: 
   The AWS configuration file uses ``retry_mode`` and the ``Config`` object uses ``mode`` - although named differently, they both refer to the same retry configuration whose options are legacy (default), standard, and adaptive. 

Here’s an example of instantiating a ``Config`` object and passing into an EC2 client to use at runtime:

.. code-block:: python

   import boto3
   from botocore.config import Config

   config = Config(
      retries = {
         'max_attempts': 10,
         'mode': 'standard'
      }
   )

   ec2 = boto3.client('ec2', config=config)

.. note::
   As mentioned previously, if no configuration options are set, the default mode is ``legacy`` and the default ``max_attempts`` is 5. 


Validating retry attempts
--------------------------

Checking retry attempts in your client logs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have Boto 3’s logging enabled, you can validate and check your client’s retry attempts in your client’s logs. Please note, however, that ``DEBUG`` mode will need to be enabled in your logger in order to see any retry attempts. The client log entries for retry attempts will appear differently depending on which retry mode you’ve configured.

**If legacy mode is enabled:**

Retry messages will be generated by ``botocore.retryhandler`` and you’ll see one of three messages:

* *No retry needed*
* *Retry needed, action of: <action_value>*
* *Reached the maximum number of retry attempts: <attempt_num>*


**If standard or adaptive mode is enabled:**

Retry messages will be generated by ``botocore.retries.standard`` and you’ll see one of three messages:

* *Not retrying request.*
* *Retry needed, retrying request after delay of: <delay_value>*
* *Retry needed but retry quota reached, not retrying request.*

Checking retry attempts in an AWS service response
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can check the number of retry attempts your client has made by parsing the response botocore provides when making a call to an AWS service API. Responses are handled by an underlying botocore module and formatted into a dictionary that is a part of the JSON response object. You can access the number of retry attempts your client has taken by calling the ``RetryAttempts`` key in the ``ResponseMetaData`` dictionary::

   'ResponseMetadata': {
      'RequestId': '1234567890ABCDEF',
      'HostId': 'host id data will appear here as a hash',
      'HTTPStatusCode': 400,
      'HTTPHeaders': {'header meta data key/values will appear here'},
      'RetryAttempts': 4
   }
