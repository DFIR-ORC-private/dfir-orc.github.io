=================================
DFIR ORC Local Configuration File
=================================

DFIR ORC can be locally configured to specify a limited set of configuration elements. Typically, those elements are the client's specific configuration options (like the upload method, priority, temporary folder, etc.). The local configuration can be specified using:


    - The ``/local=<LocalConfigFile>`` command-line option
    - A file in the same directory as DFIR ORC, with the same base name and .xml extension, e.g.:

        - ``<SomeDirectory>\\DFIR-Orc.exe``
        - ``<SomeDirectory>\\DFIR-Orc.xml``

.. _anchor-root:

The index of this sections consists in the following XML skeleton file, which features all the elements that can appear
in a real configuration file.
It is not a usable configuration, in the sense that it does not contain any attribute key or value, and can exhibit incompatible elements.
Its point is to be exhaustive from the point of view of existing usable elements.

| <`dfir-orc <#dfir-orc-element>`_ *attributes="..."*>
|      <`temporary <#temporary-element>`_> *value* <`/temporary <#temporary-element>`_>
|      <`output <#output-element>`_> *value* <`/output <#output-element>`_>
|      <`upload <#upload-element>`_ *attributes="..."*>
|          <`header <#headers-sent-with-every-request>`__> *value* </header>
|          <`rest <#rest-element>`_ *attributes="..."*>
|              <`header <#header-child-element>`__> *value* </header>
|              <`response <#response-child-element>`_ *attributes="..."* />
|          </`rest <#rest-element>`_>
|      <`/upload <#upload-element>`_>
|      <`recipient <#recipient-element>`_ *attributes="..."*> *value* <`/recipient <#recipient-element>`_>
|      <`key <#key-element>`_> *value*  <`/key <#key-element>`_>
|      <`enable_key <#enable-key-and-disable-key-elements>`_> *value* <`/enable_key <#enable-key-and-disable-key-elements>`_>
|      <`disable_key <#enable-key-and-disable-key-elements>`_> *value* <`/disable_key <#enable-key-and-disable-key-elements>`_>
|      <`log <#log-element>`_ *attributes="..."> *value* <`/log <#log-element>`_>
| <`/dfir-orc <#dfir-orc-element>`_>

.. _orc_local_config-dfir-orc-element:

``dfir-orc`` Element
====================

*optional=no, default=N/A*

Root element

Attributes
----------

* **priority** *(optional=yes, default=normal)*
        Configures Windows process (and thread) priority class. Available values for this attribute are: Low, Normal & High.
* **powerstate** *(optional=yes, default=unmodified power state)*
        Configures DFIR ORC's main thread power state to optionally prevent the system from going to sleep when DFIR ORC is running. Allowed value is a comma separated list of

        * SystemRequired
        * Displayrequired
        * UserPresent
        * AwayMode.

        When only looking to prevent sleep, recommended value for this option is SystemRequired,AwayMode.
        More information on power states: https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-setthreadexecutionstate

`Back to Root <#anchor-root>`_

.. _orc_local_config-temporary-element:

``temporary`` Element
=====================

*optional=yes, default=%temp%*, `parent element: dfir-orc <#dfir-orc-element>`_

This element configures the location of temporary files created by the tool. The inner text of this element contains the name of the folder. Environment variables will be substituted.

Attributes
----------

None

Example
-------

.. code:: xml

  <temporary>%Temp%\WorkingTemp</temporary>

`Back to Root <#anchor-root>`_

.. _orc_local_config-output-element:

``output`` Element
==================

*optional=no, default='.'*, `parent element: dfir-orc <#dfir-orc-element>`_

This element configures the folder where the various archives will be created. A local drive or a remote SMB share can be specified (in the latter, the upload syntax should be privileged to reduce network congestion). Environment variables will be substituted.

Attributes
----------

None

Example
-------

.. code:: xml

  <output>%Temp%</output>


`Back to Root <#anchor-root>`_


.. _orc_local_config-upload-element:

``upload`` Element
==================

*optional=yes, default=no upload*, `parent element: dfir-orc <#dfir-orc-element>`_

The upload element is used to configure an optional upload operation when an archive is created.

Attributes
----------

* **job** *(optional=yes, default=none)*
        Describes the upload operation.
* **method** *(optional=no, default=N/A)*
        Describes the method to upload the files: "filecopy" (uses SMB), "BITS", or "rest" (HTTP/HTTPS, see `REST upload`_).
* **server** *(optional=no, default=N/A)*
        Specifies the server name (e.g. `file://servername` or `http://servername`, or `https://servername`) when using BITS or SMB.

        For "rest", a scheme and a host, optionally with a port, e.g. ``https://servername:8443``. The path is given in the `rest element <#rest-element>`_.
* **uri** *(optional=yes, default=N/A)*
        The destination as a single URL, e.g. ``https://user:password@servername/upload``, instead of ``server``, ``path``, ``user`` and ``password``.
* **path** *(optional=no, default= / or \\ depending on the method)*
        Specifies the file share or folder for the upload
* **user** *(optional=yes, default=the current user (executing DFIR ORC))*
        Specifies the user name to be used to connect to the remote server.
* **password** *(optional=yes, default=N/A)*
        Specifies the password to use (for the user defined above)
* **authscheme** *(optional=yes, default=Negotiate (if a user name is specified, anonymous otherwise))*
        Specifies the authentication scheme for the connection. Possible scheme values are:

        * Anonymous
        * Basic
        * NTLM
        * Kerberos
        * Negotiate
        * AWSv4 (``method="rest"`` only, see `AWS SigV4`_)
* **operation** *(optional=yes, default=copy)*
        "copy" or "move" the archives to the upload server. With "move", the local file is deleted once it has been sent.
* **mode** *(optional=yes, default=sync)*
        "sync" or "async": upload can be synchronous or asynchronous (asynchronous allows DFIR ORC to exit prior to BITS jobs completes). "async" is **not** supported for "filecopy" method.
* **include** *(optional=yes, default=none)*
        Specifies a comma (or semicolon) separated list of patterns, matching the file name of archives, that determine whether an output archive from ``DFIR-Orc.exe`` will be uploaded to the specified location. When missing, all archives are uploaded (if not explicitly excluded, see below). When specified, only archives whose name matches one of the patterns will be uploaded.
* **exclude** *(optional=yes, default=none)*
        Specifies a comma (or semicolon) separated list of patterns, matching the file name of archives, that determine whether an output archive should not be uploaded. When excluded, an output archive is left intact in the output directory (i.e. regardless of the ``operation`` attribute). The ``exclude`` attribute takes precedence over the ``include`` attribute, meaning an archive whose name matches both ``include`` and ``exclude`` patterns will be excluded.
* **delete_smb_share** *(optional=yes, default=none)*
        Currently only available for BITS over SMB. When set to "true" the connection to the share will be deleted at the end of jobs (with the use of net use /del). This option should only be needed when the share is served by Samba.

``include`` and ``exclude`` select **collected data**: the archives produced by the commands. The run's own files - the log, the console output, the :doc:`outline <outline>` and the :doc:`outcome <outcome>` - are not affected by these filters.

Several ``upload`` elements may be declared. They form a **fallback list**, tried in order until one succeeds, and each entry is used with its own ``operation`` and its own ``include``/``exclude``: a fallback that says ``operation="copy"`` keeps the local archive even when the first entry asked to move it, and one that excludes a file does not receive it.

The filters are per destination, in both directions: a file the first entry excludes is still offered to a later entry that accepts it. ``exclude`` says "this destination does not take that archive", not "this archive stays on the machine" - to keep a file from leaving at all, it must be excluded from every entry.

Example
-------

.. code:: xml

    <upload job="DFIR-ORC" method="BITS"
      server="http://MyBits.MyOrg.com"
      path="upload"
      user="MyORG\BITSUploadClient" password="P@ssw0rd!"
      operation="move"
      include="DFIR-ORC_*_Hives.7z" />

.. _upload-rest:

REST upload
-----------

With ``method="rest"`` the archives are uploaded over HTTP or HTTPS. The requests are described by `rest <#rest-element>`_ child elements: a single ``PUT``, or the dialog of a server such as WebDAV, Nextcloud or S3 (see `REST server dialogs`_).

.. warning::

    The REST method is new and has only been tested in a laboratory. Before relying on it, run a complete collection against your own server.

The following attributes apply to this method only.

**Transport security**

* **https_security** *(optional=yes, default="tls1.3, tls1.2[xp]")*
        Minimum TLS version, as a comma separated list of "tls1.0" to "tls1.3": the version given and every higher one are allowed. A version followed by "[xp]" or "[seven]" applies to that system only. "os-default" keeps the system settings, still requiring TLS 1.2 or higher; "os-max" uses only the highest version the system supports.

        The default requires TLS 1.3, or TLS 1.2 on Windows XP and 2003.
* **server_cert_pin** *(optional=yes, default=none)*
        One or more certificates, PEM or base64, separated by commas. The server is accepted only if the public key of its certificate matches one of them. This is the way to use a self-signed server. Give several certificates to allow a key change.
* **insecure** *(optional=yes, default=false)*
        Allows plain HTTP and TLS below 1.2, and disables certificate validation. For a laboratory only; prefer ``server_cert_pin``.
* **follow_redirects** *(optional=yes, default=false)*
        Follows HTTP redirections. Off by default, since a redirection would send the credentials to another server.

**Authentication**

The scheme is given by ``authscheme``:

* **Basic** - with ``user`` and ``password``.
* **NTLM, Kerberos, Negotiate** - with ``user`` and ``password``. The account DFIR ORC runs under is never used.
* **AWSv4** - see `AWS SigV4`_.
* **A token**, e.g. Bearer - written as a header, see below.

Use an account dedicated to uploads, that can be revoked: the credentials embedded in the collector can be read by whoever obtains it.

.. _upload-rest-awsv4:

AWS SigV4
~~~~~~~~~

``authscheme="AWSv4"`` signs each request as an S3 server expects: AWS S3, or a compatible server such as MinIO, RustFS or SeaweedFS.

..  csv-table::
    :header: Attribute, Required, Meaning
    :align: left
    :widths: auto

    ``access_key``, yes, the access key
    ``secret_key``, yes, the secret key
    ``session_token``, no, "the session token of a temporary credential"
    ``aws_region``, yes, "the region, e.g. ``eu-west-3``"
    ``aws_service``, "no (default ``s3``)", the service name
    ``aws_clock``, "no (default ``local``)", "``server`` uses the server's time when the local clock is wrong"

* The keys are used as written: environment variables are not expanded in them. A temporary credential, with ``session_token``, avoids embedding a permanent secret.
* Write the ``path`` as plain text, e.g. a space as a space, not encoded as ``%20``.
* Redirections are not followed.
* Not available on Windows XP and 2003.

.. code:: xml

    <upload job="DFIR-ORC" method="rest"
      server="https://s3.mycorp.example"
      operation="move"
      authscheme="AWSv4"
      access_key="AKIA..." secret_key="..." aws_region="eu-west-3">
        <rest http_method="PUT" path="/orc-bucket/incoming/{upload.filename}" body="{upload.local_path}"/>
    </upload>

**Headers sent with every request**

* **http_header** *(optional=yes, default=none)*
        One header line, sent with every request, e.g. ``http_header="Authorization: Bearer &lt;token&gt;"``.

To send several headers, use ``header`` child elements of ``upload``, one line each:

.. code:: xml

    <upload method="rest" server="https://collect.mycorp.example">
        <header>Authorization: Bearer eyJhbGciOi...</header>
        <header>X-Collection: nightly</header>
        <rest http_method="PUT" path="/incoming/{upload.filename}" body="{upload.local_path}"/>
    </upload>

A ``header`` inside a `rest <#rest-element>`_ element is sent with that request only. Headers may use the placeholders below.

**Proxy**

* **proxy** *(optional=yes, default=the machine proxy settings)*
        How the proxy is found:

        ..  csv-table::
            :header: Value, Meaning
            :align: left
            :widths: auto

            *(attribute absent)*, "the machine proxy settings (``netsh winhttp``)"
            ``""`` / ``direct`` / ``none``, no proxy
            ``system``, the machine proxy settings
            ``wpad`` / ``auto``, automatic discovery (WPAD)
            ``pac:<url>`` or ``<url>.pac``, a PAC script
            ``host:port``, this proxy
* **proxy_bypass** *(optional=yes, default=none)*
        Hosts reached without the proxy, e.g. ``<local>;*.corp``.
* **proxy_user**, **proxy_password** *(optional=yes, default=none)*
        Credentials for the proxy. ``proxy`` must be set, e.g. to "system". Use an account dedicated to the proxy.
* **proxy_authscheme** *(optional=yes, default=Anonymous)*
        "Basic", "NTLM" or "Negotiate". For NTLM and Negotiate, give a domain user, e.g. ``DOMAIN\user``.

**Placeholders**

The ``path``, ``http_query``, ``body`` and ``offset`` attributes of a `rest <#rest-element>`_ element, and its headers, may use these placeholders:

..  csv-table::
    :header: Placeholder, Description
    :align: left
    :widths: auto

    {upload.filename}, name of the file
    {upload.filename_base64}, "name of the file, base64 encoded"
    {upload.local_path}, local path of the file
    {upload.range_start}, offset of the chunk being sent
    {upload.range_end}, offset of the last byte of the chunk
    {upload.range_length}, length of the chunk
    {upload.total_size}, size of the file
    {upload.random:<charset>:<length>}, "a random string, the same for the whole file; charset is hex, num, alpha or alnum, length is 1 to 64"

A value captured with a `response <#response-child-element>`_ element is written ``{name}``.

REST example
------------

.. warning::

    Understand each parameter before using a configuration: a wrong setting can make the upload insecure, for instance by sending the collection in clear text or to another server.

.. code:: xml

    <upload job="DFIR-ORC" method="rest"
      server="https://collect.mycorp.example"
      operation="move"
      proxy="system" proxy_authscheme="NTLM" proxy_user="MYCORP\orc-proxy" proxy_password="P@ssw0rd!"
      server_cert_pin="MIIDLDCCAhSgAwIBAgIU...">
        <rest http_method="PUT" path="/incoming/{upload.filename}" body="{upload.local_path}"
              chunk_size="1048576">
            <header>Content-Range: bytes {upload.range_start}-{upload.range_end}/{upload.total_size}</header>
        </rest>
    </upload>

A resumable upload asks the server what it already holds, then starts from there:

.. code:: xml

    <upload method="rest" server="https://collect.mycorp.example" operation="move">
        <rest http_method="HEAD" path="/incoming/{upload.filename}" accept_status="404">
            <response header="content-length" name="remote_size" default="0"/>
        </rest>
        <rest http_method="PUT" path="/incoming/{upload.filename}" body="{upload.local_path}"
              offset="{remote_size}" chunk_size="1048576">
            <header>Content-Range: bytes {upload.range_start}-{upload.range_end}/{upload.total_size}</header>
        </rest>
    </upload>

More examples, for each tested server, are in the ``tests/RestAgent`` directory of the source tree.

.. _rest-element:

``rest`` Element
================

*optional=yes, default=N/A*, `parent element: upload <#upload-element>`_

Describes one request sent by ``method="rest"``. Requests are sent in the order they are declared; one of them sends the file.

Attributes
----------

* **http_method** *(optional=no, default=N/A)*
        The HTTP verb, e.g. "PUT", "POST", "PATCH", "HEAD", "MKCOL", "MOVE".
* **path** *(optional=no, default=N/A)*
        Path of the request on the server, as a template.
* **http_query** *(optional=yes, default=none)*
        Query string appended to the path, as a template, e.g. ``?start={upload.range_start}``.
* **body** *(optional=yes, default=no body)*
        The file to send, normally ``{upload.local_path}``.
* **offset** *(optional=yes, default=0)*
        Where to start in the file, e.g. ``{remote_size}`` captured by a previous step, to resume an upload.
* **chunk_size** *(optional=yes, default=the whole file in one request)*
        Sends the file in chunks of that many bytes. Required beyond 4 GB. Each chunk must be identified by ``{upload.range_start}`` in the path, the query or a header.
* **accept_status** *(optional=yes, default=any 2xx)*
        Additional HTTP status codes to treat as success, comma separated, e.g. ``404`` on a probe that legitimately finds nothing.

``header`` child element
------------------------

One header line, sent with this request only. Repeat the element for several headers.

``response`` child element
--------------------------

Captures a value from the server's answer to this step, for later steps to use.

* **name** *(optional=no, default=N/A)*
        Name of the captured value, written ``{name}`` in a later step. It cannot start with ``upload.``.
* **header** *(optional=yes, default=none)*
        Name of the response header to capture.
* **dav_property** *(optional=yes, default=none)*
        Name of a WebDAV property read from a PROPFIND answer, e.g. ``getcontentlength``. Not available in the standard build.
* **default** *(optional=yes, default=empty)*
        Value used when the server does not return the header or property.

`Back to Root <#anchor-root>`_

.. _rest-dialogs:

REST server dialogs
===================

The `rest <#rest-element>`_ elements describe requests, not a protocol: another server can be used by writing its own sequence of requests. These dialogs have been tested:

..  csv-table::
    :header: Dialog, Steps, Where it has been exercised
    :align: left
    :widths: auto

    "Single ``PUT``", one body step, "the reference sink, Nextcloud 35, MinIO, RustFS, SeaweedFS"
    "Chunked", "one body step with ``chunk_size``, the offset in the path, the query or a ``Content-Range`` header", the reference sink
    "Resume", "a ``HEAD`` probe capturing ``content-length`` into ``{remote_size}``, then a body step with ``offset=""{remote_size}""``", the reference sink
    "WebDAV", "``MKCOL`` (with ``accept_status=""405""``), the body step under a temporary name, then ``MOVE`` to the final name", "the reference sink, Nextcloud 35"
    "Nextcloud chunked upload v2", "``MKCOL`` the collection, ``MKCOL`` an upload directory named by ``{upload.random:...}``, one chunk ``PUT`` per ``{upload.range_start}``, then ``MOVE``", Nextcloud 35
    "S3", "one ``PUT`` per object, anonymous or signed (`AWS SigV4`_)", "MinIO, RustFS, SeaweedFS"

* A ``PUT`` replaces the file: resuming only works with a server that appends. With Nextcloud or S3, an interrupted upload starts again from the beginning.
* Chunks are named by their offset, and the server must sort them as numbers. Nextcloud 35 does.
* With S3, a file is sent in a single request, which limits it to 5 GB.

The test configurations for these servers are in ``tests/RestAgent`` of the source tree.

`Back to Root <#anchor-root>`_


.. _orc_local_config-recipient-element:

``recipient`` Element
=====================

*optional=yes, default=N/A*, `parent element: dfir-orc <#dfir-orc-element>`_

The recipient element is used to create the list of recipients able to open the enveloped CMS archives. It basically consists of a list of encoded certificates. This element is used to add a recipient's certificate to the list of possible recipients for individual archives. This element implies encryption of the archives specified in its compulsory archive attribute.

Attributes
----------

* **name** *(optional=no, default=N/A)*
        Name of the recipient
* **archive** *(optional=no, default=Does not encrypt any archive)*
        Comma separated list of archive keyword specs to match against archive names. Specifies one or more archives encrypted in a CMS PKCS#7 message (cf http://tools.ietf.org/html/rfc2315 )

Example
-------

.. code:: xml

  <recipient name='certfr' archive='*' >
    -----BEGIN CERTIFICATE-----
    MIIC7TCCAdmgAwIBAgIQR5AF92Ti8qtEwuT3PMVrJzAJBgUrDgMCHQUAMBIxEDAO
    BgNVBAMTB0NFUlQtRlIwHhcNMDQxMjMxMjIwMDAwWhcNMTQxMjMxMjIwMDAwWjAS
    MRAwDgYDVQQDEwdDRVJULUZSMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKC
    AQEAiufyRATXw5Kc/DUcEr/5nNygcbluyS5gkUd1pGaUqKHMSMEVOBzYqcvq3cMw
    4shAL3TSgYdoOJaLG4ErvyRU87fWYRcwiHzGdFg89E3pBEWnyV3j3fR0fVB5t3MD
    jbooTGI/qQGl1l3MZ+bOiHkYcIG50R5343VT5vjRLmPv16iopGczLXKkNFxN480f
    BnCF8HcJesFiMIDUI+d9OWpLJNDSCerouMr75HVD47+gBKKgH2PrxWozk2L6R9gQ
    l8/6xzM4VKiNt4BTGfChG8AnO8sJzPETjJaDXrIGaYVLxU4OxFh/a9x61dlM/5A/
    TASXpLhXrsi+ib3YLLl+pNh+aQIDAQABo0cwRTBDBgNVHQEEPDA6gBD47GaJKs91
    qsThQIQ7f8Y5oRQwEjEQMA4GA1UEAxMHQ0VSVC1GUoIQR5AF92Ti8qtEwuT3PMVr
    JzAJBgUrDgMCHQUAA4IBAQBgvEE7qyLVV+Y5B0sR5VuPmfeqakOxBxLmb8VoTNKn
    /7ai1XwtJeWD1vumKx5Q29GiUfVhvBgn0zhjM5syVDFCqEcp+eu6l2XbN8uvllCY
    daTOT/9UylLxu1L/epiWiYtqRZOO/9i1fyqrkguIww7EjXXT3ybL5U/BakEC2Yg5
    6vUoxbo2EbA1UoMWurRxYNYxyFfHpvBYXFf4uDaAFIVMtEgH5VkKyM3Kj2hi/PJH
    /a30ndTWVSY/82hoRGCa+SkevR5VbDsxTqHtEHys4K+ETVTNXp29HwG+1YG7BTTc
    4VdFRqUm7e3o6VUArFar8I01oHiHzqKJiu1Omm2Fkmc1
    -----END CERTIFICATE-----
  </recipient>


`Back to Root <#anchor-root>`_

.. _orc_local_config-key-element:

``key`` Element
===============

*optional=yes, default=N/A*, `parent element: dfir-orc <#dfir-orc-element>`_

The key element allows to select only specific commands to be executed or archives to be generated. All non-matching keywords or archives are not executed or generated. This element is exclusive with ``enable_key`` and ``disable_key``.

Attributes
----------

None

Example
-------


.. code:: xml

  <dfir-orc>
    <key>ORC_Quick</ key>
    <key>GetRam_winpmem1,Flashback</key>
  </dfir-orc>


`Back to Root <#anchor-root>`_

.. _orc_local_config-enable-key-and-disable-key-elements:

``enable_key`` and ``disable_key`` Elements
===========================================

*optional=yes, default=N/A*, `parent element: dfir-orc <#dfir-orc-element>`_

The ``enable_key`` element will enable an optional archive or command (cf. `archive element <wolf_config.html#the-archive-element>`_ , `command element <wolf_config.html#command-element>`_).
The ``disable_key`` element will disable an archive generation or command execution. Elements ``enable_key`` and ``disable_key`` can be combined and repeated. All ``enable_key`` elements take effect before the ``disable_key`` elements. Keywords are case insensitive. The data in the element can be a comma separated list of keywords.

Attributes
----------

None

Example
-------


.. code:: xml

  <dfir-orc>
      <disable_key>DFIR-ORC_Detail</disable_key>
      <enable_key>GetRam_winpmem1</enable_key>
  </dfir-orc>

`Back to Root <#anchor-root>`_


.. _orc_local_config-log-element:

``log`` Element
===============

*optional=yes, default=N/A*, `parent element: dfir-orc <#dfir-orc-element>`_

The log element can be used to create an optional log file of DFIR ORC execution. This file will be uploaded if an <upload/> element is specified in a :doc:`DFIR ORC local configuration file <orc_local_config>`.

The log message are passing through "sinks" like 'console' or 'file'. To configure log output a sink must be specified.

``Console`` sink element, ``/log:console,...`` Option
------------------------------------------------------

*optional=yes, default=N/A*, `parent element: log`

``level`` Attribute, ``/log:console,level=<Level>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=critical*, `parent element: console`

Log level is one of 'trace', 'debug', 'info' 'error', 'warning', 'critical'.

``backtrace`` Attribute, ``/log:console,backtrace=<Level>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=off*, `parent element: console`

Specify a log level which will trigger a log backtrace which will contain logs up to level 'debug'.

Value is one of 'trace', 'debug', 'info' 'error', 'warning', 'critical', off.

``File`` sink element, ``/log:file,...`` Option
------------------------------------------------

*optional=yes, default=N/A*, `parent element: log`

The logging can be written to the file at the end of the tool execution.
This implies that tool progress cannot be followed from log file using "tail

``level`` Attribute, ``/log:file,level=<Level>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=info*, `parent element: file`

Log level is one of 'trace', 'debug', 'info' 'error', 'warning', 'critical'.

``backtrace`` Attribute, ``/log:file,backtrace=<Level>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=error*, `parent element: file`

Specify a log level which will trigger a log backtrace which will contain logs up to level 'debug'.

Value is one of 'trace', 'debug', 'info' 'error', 'warning', 'critical', off.


``output`` Element, ``/log:file,output=Path>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=N/A*, `parent element: file`

Path to the log file. Patterns are supported as with archive element (cf `archive element <#the-archive-element>`_).

``Syslog`` sink element, ``/log:syslog,...`` Option
----------------------------------------------------

*optional=yes, default=N/A*, `parent element: log`

Redirect high level logs to a syslog server.

**Currently 'syslog' use is restricted to WolfLauncher**.

``level`` Attribute, ``/log:syslog,level=<Level>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=info*, `parent element: syslog`

Log level is one of 'trace', 'debug', 'info' 'error', 'warning', 'critical'.

``backtrace`` Attribute, ``/log:syslog,backtrace=<Level>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=off*, `parent element: syslog`

Specify a log level which will trigger a log backtrace which will contain logs up to level 'debug'.

Value is one of 'trace', 'debug', 'info' 'error', 'warning', 'critical', off.

``host`` Attribute, ``/log:syslog,host=<ip4_or_ip6>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=no, default=N/A*, `parent element: syslog`

Address of the syslog server

``port`` Attribute, ``/log:syslog,port=<port>,...`` Option
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*optional=yes, default=514*, `parent element: syslog`

Port of the syslog server.

Example
--------

.. code:: xml

    <log>
        <console level="critical" backtrace="off"></console>
        <file level="error" backtrace="error">
            <output disposition="truncate">ORC_{SystemType}_{FullComputerName}_{TimeStamp}.dev.log</output>
        </file>
        <syslog>
            <host>127.0.0.1</host>
            <port>514</port>
        </syslog>
    </log>

.. code:: bat

    dfir-orc.exe \
        /log:console,level=critical,backtrace=off \
        /log:file,level=debug,backtrace=error,output="dfir-orc.log" \
        /log:syslog,host=127.0.0.1,port=514 ...

`Back to Root <#anchor-root>`_
