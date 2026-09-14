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

        For the "rest" method the value must carry a scheme and name a bare host, optionally with a port (`https://servername` or `https://servername:8443`). A path is refused: it belongs in the `rest element <#rest-element>`_ instead.
* **uri** *(optional=yes, default=N/A)*
        Specifies the destination as a single URL instead of separate attributes, e.g. ``https://user:password@servername/upload``. It is exclusive with ``server``, ``path``, ``user`` and ``password``: naming both is refused.
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
* **operation** *(optional=yes, default=copy)*
        "copy" or "move" the archives to the upload server. With "move" the local file is deleted only once data has actually been sent: an upload that transmits nothing - a `rest element <#rest-element>`_ dialog with no body step, or a resume ``offset`` that already covers the whole file - is reported as a failed upload and the archive is kept. An empty file is not that case, it is uploaded as one zero-length request.
* **mode** *(optional=yes, default=sync)*
        "sync" or "async": upload can be synchronous or asynchronous (asynchronous allows DFIR ORC to exit prior to BITS jobs completes). "async" is **not** supported for "filecopy" method.
* **include** *(optional=yes, default=none)*
        Specifies a comma (or semicolon) separated list of patterns, matching the file name of archives, that determine whether an output archive from ``DFIR-Orc.exe`` will be uploaded to the specified location. When missing, all archives are uploaded (if not explicitly excluded, see below). When specified, only archives whose name matches one of the patterns will be uploaded.
* **exclude** *(optional=yes, default=none)*
        Specifies a comma (or semicolon) separated list of patterns, matching the file name of archives, that determine whether an output archive should not be uploaded. When excluded, an output archive is left intact in the output directory (i.e. regardless of the ``operation`` attribute). The ``exclude`` attribute takes precedence over the ``include`` attribute, meaning an archive whose name matches both ``include`` and ``exclude`` patterns will be excluded.
* **delete_smb_share** *(optional=yes, default=none)*
        Currently only available for BITS over SMB. When set to "true" the connection to the share will be deleted at the end of jobs (with the use of net use /del). This option should only be needed when the share is served by Samba.

``include`` and ``exclude`` select **collected data**: the archives produced by the commands. The run's own files - the log, the console output, the :doc:`outline <outline>` and the :doc:`outcome <outcome>` - are uploaded whatever the filters say, since they are how a collection accounts for itself rather than something the operator chose to collect.

Several ``upload`` elements may be declared. They form a **fallback list**, tried in order until one succeeds, and each entry is used with its own ``operation`` and its own ``include``/``exclude``: a fallback that says ``operation="copy"`` keeps the local archive even when the first entry asked to move it, and one that excludes a file does not receive it.

The filters are per destination, in both directions: a file the first entry excludes is still offered to a later entry that accepts it. ``exclude`` says "this destination does not take that archive", not "this archive stays on the machine" - to keep a file from leaving at all, it must be excluded from every entry. :doc:`ToolEmbed <ToolEmbed>` warns at embedding time when the entries of a configuration do not all carry the same filters.

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

With ``method="rest"`` the archives are uploaded over HTTP or HTTPS, and the requests to issue are described by one or more `rest <#rest-element>`_ child elements. This covers a plain ``PUT`` as well as the resumable and multi-step dialogs of servers such as WebDAV or tus.io.

The following attributes apply to this method only.

**Transport security**

* **https_security** *(optional=yes, default="tls1.3, tls1.2[xp]")*
        Comma separated list of TLS versions ("tls1.0" to "tls1.3"), acting as a **floor**: the named version and every higher one the operating system supports are enabled. A version may carry an OS target between brackets - "[xp]" or "[seven]" - which replaces the baseline on that system only, since those releases cannot negotiate the newer versions. The value "os-default" (or "none") leaves the system policy untouched.

        The default therefore requires TLS 1.3 everywhere except on Windows XP/2003, where it requires TLS 1.2. A server that cannot reach the floor fails the handshake rather than being silently accepted; the log then names the versions this client required.
* **server_cert_pin** *(optional=yes, default=none)*
        One or more X.509 certificates, as PEM or base64, separated by commas or semicolons. The server is accepted only if the public key (SPKI) of the certificate it presents matches one of them. This is the secure way to use a self-signed server: it is independent of the machine's trust store and survives certificate renewal with the same key, and it needs no ``insecure``. Several values allow key rotation.
* **insecure** *(optional=yes, default=false)*
        Authorises transports that are insecure by construction: plain HTTP, TLS below 1.2, **and** disabling certificate validation altogether. Without it, a plain-HTTP REST upload is refused when the configuration is embedded. Prefer ``server_cert_pin`` for a self-signed server.
* **follow_redirects** *(optional=yes, default=false)*
        Lets WinHTTP follow a 3xx redirection. Off by default: a redirection would re-send the preset credentials to the target it names, which certificate pinning does not cover. An HTTPS to HTTP downgrade is refused even when this is enabled.

**Headers sent with every request**

* **http_header** *(optional=yes, default=none)*
        One complete header line, used verbatim and sent with every request of the dialog, e.g. ``http_header="Authorization: Bearer &lt;token&gt;"``. The value is never split on a comma or a semicolon, which are legal in a header value; to send several headers, repeat the ``header`` child element below.

A ``header`` child element of ``upload`` carries one header line each, again sent with every request:

.. code:: xml

    <upload method="rest" server="https://collect.mycorp.example">
        <header>Authorization: Bearer eyJhbGciOi...</header>
        <header>X-Collection: nightly</header>
        <rest http_method="PUT" path="/incoming/{upload.filename}" body="{upload.local_path}"/>
    </upload>

A ``header`` declared inside a `rest <#rest-element>`_ element applies to that request only. Both forms are templates, so they may carry the placeholders listed below.

**Proxy**

* **proxy** *(optional=yes, default=the machine-wide configuration)*
        Selects how the proxy is found:

        ..  csv-table::
            :header: Value, Meaning
            :align: left
            :widths: auto

            *(attribute absent)*, "follow the machine-wide WinHTTP configuration (``netsh winhttp``), which connects direct when the machine has no proxy"
            ``""`` / ``direct`` / ``none``, "no proxy and no discovery, whatever the machine is configured with"
            ``system``, the machine-wide proxy, named explicitly
            ``wpad`` / ``auto``, "WPAD auto-detection (DHCP/DNS); if nothing is discovered the upload is refused rather than silently sent direct"
            ``pac:<url>``, the PAC script at that URL
            ``<...>.pac``, "a PAC script, recognised by the ``.pac`` suffix of the URL"
            ``host:port``, an explicit proxy

        Absent and empty are deliberately different: a configuration that never mentions a proxy takes the machine as it finds it, while ``proxy=""`` states that there is no proxy. Note that WPAD is spoofable on a hostile network, and that a PAC script is downloaded and executed to choose the proxy.

        The value may come from an environment variable. If that variable is **not set** on the collected host, the reference survives expansion, no proxy can be resolved, and the upload is refused rather than attempting to reach a host by that literal name.
* **proxy_bypass** *(optional=yes, default=none)*
        WinHTTP bypass list for an explicit proxy, e.g. ``<local>;*.corp``.
* **proxy_user**, **proxy_password** *(optional=yes, default=none)*
        Credentials for an authenticating proxy. They are always explicit: DFIR ORC never offers the identity it runs under, so a collection running as SYSTEM never presents the machine account to a proxy. Because the credential is recoverable from the embedded binary, use a scoped, revocable, proxy-only account.

        They require the destination to be named: ``proxy="system"`` has to be written out loud to send them to the machine-wide proxy.
* **proxy_authscheme** *(optional=yes, default=Anonymous)*
        "Basic", "NTLM" or "Negotiate". The scheme is never learned from the proxy's ``407``. **NTLM and Negotiate need a domain-qualified user** (``DOMAIN\user`` or ``user@domain.fqdn``): WinHTTP sends an empty domain for a bare name, which a real proxy rejects with a repeating ``407``.

**Placeholders**

The ``path``, ``http_query``, ``body`` and ``offset`` attributes of a `rest <#rest-element>`_ element, and its headers, are templates. The following are substituted:

..  csv-table::
    :header: Placeholder, Description
    :align: left
    :widths: auto

    {upload.filename}, name of the file being uploaded
    {upload.filename_base64}, the same name, base64 encoded
    {upload.local_path}, full local path of the file being uploaded
    {upload.range_start}, offset of the chunk being sent
    {upload.range_end}, offset of its last byte (start + length - 1)
    {upload.range_length}, length of the chunk being sent
    {upload.total_size}, total size of the file being sent
    {upload.random:<charset>:<length>}, "a fresh random string, drawn once per file; charset is hex, num, alpha or alnum and length is 1 to 64"

Values captured from a server response (see the ``response`` element below) are substituted under the name the configuration gives them, which may not start with ``upload.``. A captured value is never re-scanned for placeholders, so a server cannot inject one.

REST example
------------

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

.. _rest-element:

``rest`` Element
================

*optional=yes, default=N/A*, `parent element: upload <#upload-element>`_

Describes one request of the dialog used by ``method="rest"``. Several ``rest`` elements may be declared and are issued in the order they appear: exactly one of them carries the body, the others are control requests (a probe, a lock, a rename, a completion notice).

Attributes
----------

* **http_method** *(optional=no, default=N/A)*
        The HTTP verb, e.g. "PUT", "POST", "PATCH", "HEAD", "MKCOL", "MOVE".
* **path** *(optional=no, default=N/A)*
        Path of the request on the server, as a template.
* **http_query** *(optional=yes, default=none)*
        Query string appended to the path, as a template, e.g. ``?start={upload.range_start}``.
* **body** *(optional=yes, default=no body)*
        The file to send as the request body. This is the step that uploads; use ``{upload.local_path}`` for the file being collected. A step without ``body`` sends no content.
* **offset** *(optional=yes, default=0)*
        Offset the body starts at, as a template, e.g. ``{remote_size}`` fed by a probe. The value is clamped to the size of the body.
* **chunk_size** *(optional=yes, default=the whole file in one request)*
        Splits the body into requests of that many bytes. Required beyond 4 GB, which a single request cannot carry.
* **accept_status** *(optional=yes, default=any 2xx)*
        Additional HTTP status codes to treat as success, comma separated, e.g. ``404`` on a probe that legitimately finds nothing.

``header`` child element
------------------------

One complete header line, used verbatim. The value is a template. Repeat the element for several headers; it is never split on a comma or a semicolon, which are legal in a header value.

``response`` child element
--------------------------

Captures a value from the server's answer to this step, for later steps to use.

* **name** *(optional=no, default=N/A)*
        Name the captured value is substituted under, written ``{name}`` in a later step. It may not start with ``upload.``, which is reserved.
* **header** *(optional=yes, default=none)*
        Name of the response header to capture.
* **dav_property** *(optional=yes, default=none)*
        Name of a WebDAV property to capture from a PROPFIND answer, e.g. ``getcontentlength``.
* **default** *(optional=yes, default=empty)*
        Value used when the server does not return the header or property.

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

        It selects archives only. Any recipient, whatever this attribute says, also receives the outline, outcome, log and console output of the collection, which are encrypted for every recipient: see :ref:`Encrypted output files <wolf_config-encrypted-output-files>`, which also explains how to decrypt them.

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
