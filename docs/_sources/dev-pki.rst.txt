.. _dev-pki:

EniwareNetwork Developer PKI Guide
=======================================


.. _dev-pki-dns:

1. Configure a DNS name
^^^^^^^^^^^^^^^^^^^^^^^^^


The `Transport Layer Security <https://en.wikipedia.org/wiki/Transport_Layer_Security>`_ (TLS v.1.2/August 2008 - :rfc:`5246` ; TLS v.1.3/Audust 2018 - :rfc:`8446` ) require to set up a DNS name, which should be the **name_of_your_machine**, i.e. ``eniware`` in this example. You need to added it to your system’s host file in ``/etc/hosts/`` :

.. code::
 
  127.0.0.1 localhost eniware

.. hint:: To edit the ``hosts`` file type use the simple  text editor (**nano** for example: ``sudo nano /etc/hosts``).



.. _dev-pki-service:

2. Development PKI service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When EniwareEdges are registered and then associated with EniwareNet, the last is responsible for generating a certificate for the Edge, which must be signed by the EniwareNet `certification authority <https://en.wikipedia.org/wiki/Certificate_authority>`_ (**CA**) certificate. The format of these certificates is specified by the `X.509 <https://en.wikipedia.org/wiki/X.509>`_ (`Recommendation ITU-T X.509 <https://www.itu.int/rec/T-REC-X.509/en>`_).
The bundle from ``org.eniware.central.user.pki.dev`` project provides a simple PKI service for use during development. Close any other ``central.user.pki.*`` projects except ``org.eniware.centrtal.user.pki.dev``.

Launch the `OSGI platform <https://eniware-org.github.io/eniware-dev-docs/eclipse-set-guide.html#configure-osgi-runtime>`_ with this bundle deployed. The first time this bundle is deployed it will create a new self-signed root CA certificate and store it in the ``/eniware-osgi-target/var/DeveloperCA/ca.jks`` Java keystore. The password for this keystore is randomly generated and stored in file named ``secret`` in the same directory.

The bundle also create a new webserver certificate for the **name_of_your_machine** DNS name (i.e. ``eniware`` in this example) and store this in keystore named ``central.jks``. The password for this keystore is **eniwareedge**. 

Finally the bundle will create a *trust* keystore named ``central-trust.jks``.

Stop the OSGI platform in Eclipse after the creation of all of this files and continue configuring your enviroement.
Copy the ``central.jks`` and ``central-trust.jks`` files to the ``/eniware-osgi-target/conf/tls/`` (create the ``conf`` and ``tls`` directories first). Do not move instead of copying these files because they will be created again at the next start of the OSGI platform.



.. _dev-pki-tomcat:

3. Configure Tomcat with TLS support
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Open file ``tomcat-server.xml`` from ``/eniware-osgi-target/config/``. Then add  TLS ``Connector`` element:

.. code::
 
 <Connector port="8683" protocol="HTTP/1.1" SSLEnabled="true"
    maxThreads="25" scheme="https" secure="true" sslProtocol="TLS"
    keyAlias="web" keystoreType="jks"
    keystoreFile="conf/tls/central.jks" keystorePass="eniwareedge"
    truststoreFile="conf/tls/central.jks" truststorePass="eniwareedge"
    clientAuth="want"/>

.. hint:: If you haven’t created already copy the ``tomcat-server.xml`` file from ``/eniware-osgi-target/example/config``, open it and uncoment the code above.

.. note:: The ``port`` number is important to note because it will be used later. The ``keystorePass`` and ``truststorePass`` are the same as in the key store and trust store.


In ``tomcat-server.xml`` file you must modify the ``Engine`` and ``Host`` elements to use the DNS name you created :ref:`earlier<dev-pki-dns>` and is the Command Name (`CN <https://support.dnsimple.com/articles/what-is-common-name/>`_) portion of your webserver certificate subject:

.. code::
 
 <Engine name="Catalina" defaultHost="name_of_your_machine">
   <Host name="name_of_your_machine" unpackWARs="false" autoDeploy="false" deployOnStartup="false">
 </Engine>

 
.. hint:: Using ``eniware`` as a **name_of_your_machine**, the example will look like this:
 
 .. code::
 
  <Engine name="Catalina" defaultHost="eniware">
     <Host name="eniware" unpackWARs="false" autoDeploy="false" deployOnStartup="false">
  </Engine>
 





.. _dev-pki-settiings:

4. Configure development settiings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You need configure some settings in the ``/eniware-osgi-target/configurations/services/`` directory:

 * Copy the ``org.eniware.central.in.properties`` file from ``/org.eniware.central.in.biz/example/configuration/`` project  and paste it in ``/eniware-osgi-target/configurations/services`` folder.

 * Change the file extension to ``.cfg`` an open it:
 
   .. code:: 
  
      ###############################################################################
      #org.eniware.central.in Configuration Admin properties
      ###############################################################################
      
      ###############################################################################
      #SimpleNetworkIdentityBiz.host <string>
      #SimpleNetworkIdentityBiz.port <integer>
      #SimpleNetworkIdentityBiz.forceTLS <boolean>
      #The host identity values to use.
      
      SimpleNetworkIdentityBiz.host <string>
      SimpleNetworkIdentityBiz.port <integer>
      SimpleNetworkIdentityBiz.forceTLS <boolean>
 
 * Edit this ``org.eniware.central.in.cfg`` file as it's shown below:
 
  .. code::
    
       SimpleNetworkIdentityBiz.host = eniware
       SimpleNetworkIdentityBiz.port = 8683
       SimpleNetworkIdentityBiz.forceTLS = true
	  
  .. hint::
    
    * ``SimpleNetworkIdentityBiz.host`` is the same as the :ref:`DNS name<dev-pki-dns>` **name_of_your_machine** you assigned earlier
    * ``SimpleNetworkIdentityBiz.port`` is the same :ref:`port number<dev-pki-tomcat>` that you configured the Tomcat server to use;
 
 * Open the project ``org.eniware.central.user.pki.dev`` and edit the java file ``DevedgePKIBIZ.java``
  
  .. code:: 
    
    WEBSERVER_KEYSTONE_PASSWORD = "your_password";

      
  .. todo:: Open project and edit java file org.eniware.central.user.pki.dev/src/org.eniware.central.user.pki.dev/DevedgePKIBIZ.java WEBSERVER_KEYSTONE_PASSWORD = ''your password'';

   In the same java file find : log.info("Development webserver keystore saved to {}; password is your password", and edit the password. 

 * You need to update the OSGI runtime in Eclipse so the node uses the ``central-trust.jks`` file as its *trust* store, enabling it to "trust" the development CA root certificate. Go to **Run > Run Configurations... > OSGI Framework > EniwareNetwork** (or **Name_of_your_OSGI_framework**) and select the **Arguments** tab. Add the following to the *VM arguments* fields:
 
  .. code::
   
   -Djavax.net.ssl.trustStore=${workspace_loc:eniware-osgi-target}/conf/tls/central-trust.jks



.. _dev-pki-new-edge:

5. Associate a new edge
^^^^^^^^^^^^^^^^^^^^^^^^

Start up OSGI platform in Eclipse and then use EniwareUser app ``localhost:8080/eniwareuser`` to register yourself as a new EniwareNet user. You can use the ``org.eniware.central.common.mail.mock`` bundle to have the user registration code logged to the console rather than relying on actual email to be sent. Once registered, invite a new EniwareEdge (under the **My Edges** section). Copy the invitation code, then go to the EniwareEdge setup app ``/associate``. Paste the invitation and complete the association process. Your new node should setup and get a new certificate and then be ready for general use
