.. _guide_configuration:

Configuration
===========

Overview
---------
Boto3 looks at various configuration locations until values are found. Boto3 adheres to the follwoing lookup order when searching through sources for configuration values:

* Creating a ``Config`` object and passing it as the config parameter when creating a client.
* Environment variables
* The ``~/.aws/config`` file.

.. note::

    Configurations are not wholly atomic. This means configuration values set in your AWS config file can be singularly overwritten by setting a specific environment variable or through the use of a ``Config`` object.

.. note::

    Credential configuration is covered in detail in the :ref:`guide_credentials` guide. 


Using the Config object
-------------------------
This option is for configuring client-specific configurations that will affect the behavior of your specific client object only. As noted earlier, there are options used here that will supersede those found in other configuration locations:

* ``region_name`` (string) - The AWS Region used in instantiating the client. If used, this will take precedence over environment variable and configuration file values, but will not overwrite a ``region_name`` value *explicitly* passed to individual service methods.
* ``signature_version`` (string) - The signature version used when signing requests. Please note that the default version is signature version 4. If using a pre-signed URL, you should specify signature version 2. 
* ``s3`` (related configurations; dictionary) - AWS S3 service specific configurations; see the `Config reference <https://botocore.amazonaws.com/v1/documentation/api/latest/reference/config.html>`_ for further details.
* ``retries`` (dictionary) - Client retry behavior configuration options that include retry mode, and maximum retry attempts. See the :ref:`guide_retries` guide for more details.  


There are additional options beyond those listed above, for more information or for a complete list of options, please see the see the `Config reference <https://botocore.amazonaws.com/v1/documentation/api/latest/reference/config.html>`_ for further details.

To set these configuration options, create a ``Config`` object with the options you desire and then pass them into your client:

.. code-block:: python

    import boto3
    from botocore.config import Config

    my_config = Config(
        region_name = 'us-west-2',
        signature_version = 'v4',
        retries = {
            'max_attempts': 10,
            'mode': 'standard'
        }
    )

    client = boto3.client('kinesis', config=my_config)

Using Environment variables 
-----------------------------

``AWS_ACCESS_KEY_ID``
    The access key for your AWS account.

``AWS_SECRET_ACCESS_KEY``
    The secret key for your AWS account.

``AWS_SESSION_TOKEN``
    The session key for your AWS account.  This is only needed when
    you are using temporary credentials.  The ``AWS_SECURITY_TOKEN``
    environment variable can also be used, but is only supported
    for backwards compatibility purposes.  ``AWS_SESSION_TOKEN`` is
    supported by multiple AWS SDKs besides python.

``AWS_DEFAULT_REGION``
    The default region to use, e.g. ``us-west-1``, ``us-west-2``, etc.

``AWS_PROFILE``
    The default profile to use, if any.  If no value is specified, boto3
    will attempt to search the shared credentials file and the config file
    for the ``default`` profile.

``AWS_CONFIG_FILE``
    The location of the config file used by boto3.  By default this
    value is ``~/.aws/config``.  You only need to set this variable if
    you want to change this location.

``AWS_SHARED_CREDENTIALS_FILE``
    The location of the shared credentials file.  By default this value
    is ``~/.aws/credentials``.  You only need to set this variable if
    you want to change this location.

``BOTO_CONFIG``
    The location of the boto2 credentials file. This is not set by default.
    You only need to set this variable if want to use credentials stored in
    boto2 format in a location other than ``/etc/boto.cfg`` or ``~/.boto``.

``AWS_CA_BUNDLE``
    The path to a custom certificate bundle to use when establishing
    SSL/TLS connections.  Boto3 includes a bundled CA bundle it will
    use by default, but you can set this environment variable to use
    a different CA bundle.

``AWS_METADATA_SERVICE_TIMEOUT``
    The number of seconds before a connection to the instance metadata
    service should time out.  When attempting to retrieve credentials
    on an EC2 instance that has been configured with an IAM role,
    a connection to the instance metadata service will time out after
    1 second by default.  If you know you are running on an EC2 instance
    with an IAM role configured, you can increase this value if needed.

``AWS_METADATA_SERVICE_NUM_ATTEMPTS``
    When attempting to retrieve credentials on an EC2 instance that has
    been configured with an IAM role, boto3 will only make one attempt
    to retrieve credentials from the instance metadata service before
    giving up.  If you know your code will be running on an EC2 instance,
    you can increase this value to make boto3 retry multiple times
    before giving up.

``AWS_DATA_PATH``
    A list of **additional** directories to check when loading botocore data.
    You typically do not need to set this value.  There's two built in search
    paths: ``<botocoreroot>/data/`` and ``~/.aws/models``. Setting this
    environment variable indicates additional directories to first check before
    falling back to the built in search paths.  Multiple entries should be
    separated with the ``os.pathsep`` character which is ``:`` on linux and
    ``;`` on windows.

``AWS_STS_REGIONAL_ENDPOINTS``
    Sets STS endpoint resolution logic. See the ``sts_regional_endpoints``
    configuration file section for more information on how to use this.

``AWS_MAX_ATTEMPTS``
    The total number of attempts made for a single request.  For more information,
    see the ``max_attempts`` configuration file section.

``AWS_RETRY_MODE``
    Specifies the types of retries the SDK will use.  For more information,
    see the ``retry_mode`` configuration file section.

Using a configuration file
-------------------

Boto3 will also search the ``~/.aws/config`` file when looking for
configuration values.  You can change the location of this file by
setting the ``AWS_CONFIG_FILE`` environment variable.

This file is an INI formatted file that contains at least one
section: ``[default]``.  You can create multiple profiles (logical
groups of configuration) by creating sections named ``[profile profile-name]``.
If your profile name has spaces, you'll need to surround this value in quotes:
``[profile "my profile name"]``.  Below are all the config variables supported
in the ``~/.aws/config`` file:

``api_versions``
    Specifies the API version to use for a particular AWS service.

    The ``api_versions`` settings are nested configuration values that require special
    formatting in the AWS configuration file. If the values are set by the
    AWS CLI or programmatically by an SDK, the formatting is handled
    automatically. If they are set by manually editing the AWS configuration
    file, the required format is shown below. Notice the indentation of each
    value.
    ::

        [default]
        region = us-east-1
        api_versions = 
            ec2 = 2015-03-01
            cloudfront = 2015-09-17

``aws_access_key_id``
    The access key to use.
``aws_secret_access_key``
    The secret access key to use.
``aws_session_token``
    The session token to use. This is typically needed only when using
    temporary credentials. Note ``aws_security_token`` is supported for
    backward compatibility.
``ca_bundle``
    The CA bundle to use. For more information, see the above description
    of the ``AWS_CA_BUNDLE`` environment variable.
``credential_process``
    Specifies an external command to run to generate or retrieve
    authentication credentials. For more information,
    see `Sourcing Credentials with an External Process`_.
``credential_source``
    To invoke an AWS service from an Amazon EC2 instance, you can use
    an IAM role attached to either an EC2 instance profile or an Amazon ECS
    container. In such a scenario, use the ``credential_source`` setting to
    specify where to find the credentials.
    
    The ``credential_source`` and ``source_profile`` settings are mutually
    exclusive.
    
    The following values are supported.

        ``Ec2InstanceMetadata``
            Use the IAM role attached to the Amazon EC2 instance profile.

        ``EcsContainer``
            Use the IAM role attached to the Amazon ECS container.

        ``Environment``
            Retrieve the credentials from environment variables.

``duration_seconds``
    The length of time in seconds of the role session. The value can range
    from 900 seconds (15 minutes) to the maximum session duration setting
    for the role. The default value is 3600 seconds (one hour).
``external_id``
    Unique identifier to pass when making ``AssumeRole`` calls.
``metadata_service_timeout``
    The number of seconds before timing out when retrieving data from the
    instance metadata service.  See the docs above on
    ``AWS_METADATA_SERVICE_TIMEOUT`` for more information.
``metadata_service_num_attempts``
    The number of attempts to make before giving up when retrieving data from
    the instance metadata service.  See the docs above on
    ``AWS_METADATA_SERVICE_NUM_ATTEMPTS`` for more information.
``mfa_serial``
    Serial number of ARN of an MFA device to use when assuming a role.
``parameter_validation``
    Disable parameter validation (default is true; parameters are
    validated by default).  This is a boolean value that can have
    a value of either ``true`` or ``false``.  Whenever you make an
    API call using a client, the parameters you provide are run through
    a set of validation checks including (but not limited to): required
    parameters provided, type checking, no unknown parameters,
    minimum length checks, etc.  You generally should leave parameter
    validation enabled.
``region``
    The default region to use, e.g. ``us-west-1``, ``us-west-2``, etc. When
    specifying a region inline during client initialization, this property
    is named ``region_name``
``role_arn``
    The ARN of the role you want to assume.
``role_session_name``
    The role name to use when assuming a role.  If this value is not
    provided, a session name will be automatically generated.
``web_identity_token_file``
    The path to a file which contains an OAuth 2.0 access token or OpenID
    Connect ID token that is provided by the identity provider. The contents of
    this file will be loaded and passed as the ``WebIdentityToken`` argument to
    the ``AssumeRoleWithWebIdentity`` operation.
``s3``
    Set S3-specific configuration data. Typically, these values do not need
    to be set.
    
    The ``s3`` settings are nested configuration values that require special
    formatting in the AWS configuration file. If the values are set by the
    AWS CLI or programmatically by an SDK, the formatting is handled
    automatically. If they are set by manually editing the AWS configuration
    file, the required format is shown below. Notice the indentation of each
    value.
    ::

        [default]
        region = us-east-1
        s3 = 
            addressing_style = path
            signature_version = s3v4

    * ``addressing_style``: The S3 addressing style. When necessary, Boto
      automatically switches the addressing style to an appropriate value.
      The following values are supported.

        ``auto``
            (Default) Attempts to use ``virtual``, but falls back to ``path`` 
            if necessary.
      
        ``path``
            Bucket name is included in the URI path.

        ``virtual``
            Bucket name is included in the hostname.

    * ``payload_signing_enabled``: Specifies whether to include an SHA-256 
      checksum with Amazon Signature Version 4 payloads. Valid settings are
      ``true`` or ``false``.

      For streaming uploads (``UploadPart`` and ``PutObject``) that use HTTPS
      and include a ``content-md5`` header, this setting is disabled by default.
    * ``signature_version``: The AWS signature version to use when signing 
      requests. When necessary, Boto automatically switches the signature
      version to an appropriate value. The following values are recognized.
    
        ``s3v4``
            (Default) Signature Version 4

        ``s3``
            (Deprecated) Signature Version 2

    * ``use_accelerate_endpoint``: Specifies whether to use the S3 Accelerate
      endpoint. The bucket must be enabled to use S3 Accelerate. Valid settings
      are ``true`` or ``false``. Default: ``false``

      Either ``use_accelerate_endpoint`` or ``use_dualstack_endpoint`` can be
      enabled, but not both.
    * ``use_dualstack_endpoint``: Specifies whether to direct all Amazon S3
      requests to the dual IPv4/IPv6 endpoint for the configured region. Valid
      settings are ``true`` or ``false``. Default: ``false``

      Either ``use_accelerate_endpoint`` or ``use_dualstack_endpoint`` can be
      enabled, but not both.
``source_profile``
    The profile name that contains credentials to use for the initial
    ``AssumeRole`` call.

    The ``credential_source`` and ``source_profile`` settings are mutually
    exclusive.

``sts_regional_endpoints``
    Sets STS endpoint resolution logic. This configuration can also be set
    using the environment variable ``AWS_STS_REGIONAL_ENDPOINTS``. By default,
    this configuration option is set to ``legacy``. Valid values are:

    * ``regional``
        Uses the STS endpoint that corresponds to the configured region. For
        example if the client is configured to use ``us-west-2``, all calls
        to STS will be make to the ``sts.us-west-2.amazonaws.com`` regional
        endpoint instead of the global ``sts.amazonaws.com`` endpoint.

    * ``legacy``
        Uses the global STS endpoint, ``sts.amazonaws.com``, for the following
        configured regions:

        * ``ap-northeast-1``
        * ``ap-south-1``
        * ``ap-southeast-1``
        * ``ap-southeast-2``
        * ``aws-global``
        * ``ca-central-1``
        * ``eu-central-1``
        * ``eu-north-1``
        * ``eu-west-1``
        * ``eu-west-2``
        * ``eu-west-3``
        * ``sa-east-1``
        * ``us-east-1``
        * ``us-east-2``
        * ``us-west-1``
        * ``us-west-2``

        All other regions will use their respective regional endpoint.

``tcp_keepalive``
    Toggles the TCP Keep-Alive socket option used when creating connections.
    By default this value is ``false``; TCP Keep-Alive will not be used
    when creating connections. To enable TCP Keep-Alive set this value to
    ``true``, enabling TCP Keep-Alive with the system default configurations.

``max_attempts``
    An integer representing the maximum number attempts that will be made for
    a single request, including the initial attempt.  For example,
    setting this value to 5 will result in a request being retried up to
    4 times.  If not provided, the number of retries will default to whatever
    is modeled, which is typically 5 total attempts in the ``legacy`` retry mode,
    and 3 in the ``standard`` and ``adaptive`` retry modes.

``retry_mode``
    A string representing the type of retries boto3 will perform.  Value values are:

        * ``legacy`` - The pre-existing retry behavior.  This is default value if
          no retry mode is provided.
        * ``standard`` - A standardized set of retry rules across the AWS SDKs.
          This includes a standard set of errors that are retried as well as
          support for retry quotas, which limit the number of unsuccessful retries
          an SDK can make.  This mode will default the maximum number of attempts
          to 3 unless a ``max_attempts`` is explicitly provided.
        * ``adaptive`` - An experimental retry mode that includes all the
          functionality of ``standard`` mode along with automatic client side
          throttling.  This is a provisional mode that may change behavior
          in the future.


.. _IAM Roles for Amazon EC2: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html
.. _Using IAM Roles: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html
.. _Sourcing Credentials with an External Process: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sourcing-external.html
