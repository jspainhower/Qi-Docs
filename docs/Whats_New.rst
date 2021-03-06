What's New in Qi?
=================


Summary of changes:
-------------------

**20 May 2016**


* Deprecation of IQiServer C# Interface and introduction of ``IQiAdministrationService``, ``IQiMetaDataService`` and ``IQiDataService``.
* Removal of synchronous method calls
* Addition of methods and objects to assist with service and security configuration
* TCP/IP port changed from 3380 to 443

Some important changes were made to the Qi .Net Library that you should be aware of. Starting May 20th, 2016, 
the IQiServer C# interface will be deprecated and replaced with several new interfaces. The IQiServer interface 
formerly included calls to manipulate Qi objects and to read and write data. Calls that were previously 
included in the IQiServer interface were integrated into newly designed services and the IQiServer interface 
will be deprecated.

The TCP/IP port for the Qi service has also changed. You should update any references in your code from port 3380 to port 443.

The new interfaces and their descriptions are as follows:

+---------------------------+---------------------------------------------------+
| Method name               | Description                                       |
+===========================+===================================================+
| IQiAdministrationService  | Includes methods that you use to manage           |
|                           | QiNamespace objects within a tenant.              |
+---------------------------+---------------------------------------------------+
| IQiMetaDataService        | Includes methods that you use to manage QiTypes,  |
|                           | QiStreams, and QiStreamBehavior objects within    |
|                           | a namespace.                                      |
+---------------------------+---------------------------------------------------+
| IQiDataService            | Includes methods that you use to read and write   |
|                           | data to QiStreams                                 |
+---------------------------+---------------------------------------------------+

In addition to the new layout of methods shown in the previous table, all of the 
synchronous calls are also being deprecated. Before deprecating, each of the .NET 
library calls included a full complement of both synchronous and asynchronous 
overloads. With this update to the Qi libraries, only the asynchronous overloads will remain. 

If you are currently using the IQiServer, your client code will still work, but 
you will receive compiler warnings to let you know you are using a call that will 
soon become unavailable. At your earliest possible convenience, you should change 
your code to use the new services (``IQiAdministrationService``, ``IQiMetaDataService``
and ``IQiDataService``) and remove the ``IQiServer`` interface from your client code.

The libraries also include new methods and objects to assist with configuring user 
security roles within your client. QiService and QiSecurityHandler objects are 
now included in the .Net libraries.  

The QiService object is used to create service objects that implement each of the new interfaces. 

Methods included with QiService object include:

+---------------------------+---------------------------------------------------+
| Interface name            | Description                                       |
+===========================+===================================================+
| GetAdministrationService  | Returns an IQiAdministrationService object        |
+---------------------------+---------------------------------------------------+
| GetMetadataService        | Returns an IQiMetaDataService object              |
+---------------------------+---------------------------------------------------+
| GetDataService            | Returns an IQiDataService object                  |
+---------------------------+---------------------------------------------------+

The Getxxx methods in the previous table accept a URI, tenant, and namespace information 
as well as security credentials and return the service interface with the desired security access.

The ``IQiAdministrationService``, ``IQiMetaDataService`` and ``IQiDataService`` interfaces 
are passed tenant and namespace information when the methods are instantiated, 
which means that it is not necessary to provide the tenant and namespace information 
with each library call. If you are switching from the IQiServer to the new services, 
you will notice that method calls no longer include tenantId and namespaceID parameters.

Security information is provided to the QiService calls using a new ``SecurityHandler`` object. 
Constructors for this object accept security credentials that are provided by the client 
so that the appropriate role of administrator or user can be implemented for the service 
in which the created ``SecurityHandler`` is used. 

**If you are new to the Qi .NET library:**

Proceed to the documentation and samples for guidance about setting up and programming 
your Qi Clients with the Qi .NET Library.

**If you have already been using the Qi .NET library:**

Your code will continue to work, but you will need to replace your use of the 
IQiServer C# Interface as outlined above. Specifically, you will need to remove 
your instantiation of the IQiServer C# interface and replace it with code that 
instantiates ``IQiAdministrationService``, ``IQiMetaDataService`` and ``IQiDataService``. Refer to 
the documentation and samples for guidance about setting up and programming your Qi Clients 
with the Qi .NET Library

To assist you in incorporating these new C# interfaces, the Qi .NET libraries include 
a QiService class, which includes methods that can be used to quickly instantiate the 
new services. The methods are ``GetAdministrationService()``, ``GetMetadataService()`` 
and ``GetDataService()``. These calls accept URI, tenants, namespace and security 
information. A new class called ``SecurityHandler`` is now provided so you can set 
and pass security information to the Services in one easy step. 

After you have modified your code to instantiate the new interfaces, you should replace 
the specific method calls that used the old IQiServer C# Interface.

Here are some steps you might find useful when replacing these calls:

For each of the method calls that were previously made with IQiServer:

1.  Replace the use of ``IQiServer`` with one of the new 
    ``IQiAdministrationService``, ``IQiMetaDataService`` and ``IQiDataService objects.``
2.  You should be able to find the method you need by using the object's completion aids. 
    The new services use the same method names as the old IQiServer, with the exception that the 
    synchronous method overloads are no longer provided. All of the synchronous methods have an 
    equivalent asynchronous method in the new Services; they simply include ``Async`` at 
    the end of the method name. For example ``GetValue(...)`` becomes ``GetValueAsync(...)``.
3.  Notice also that you must remove the passing of a TenantId and/or NamespaceId as 
    parameters to these methods. The new methods do not require these parameters.

*Example method call change:*

A call such as this:

``var event1 = _QiServer.GetDistinctValue<TypeClass>(_tenant.Id, _testNamespaceId, streamId, startIndexString);``

becomes this: 

``var event1 = _QiDataService.GetDistinctValueAsync<TypeClass>(streamId, startIndexString).GetResult();``



