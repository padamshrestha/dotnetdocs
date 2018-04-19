---
title: "Managing the Data Service Context (WCF Data Services)"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework-oob"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-clr"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 15b19d09-7de7-4638-9556-6ef396cc45ec
caps.latest.revision: 6
author: "dotnet-bot"
ms.author: "dotnetcontent"
manager: "wpickett"
ms.workload: 
  - "dotnet"
---
# Managing the Data Service Context (WCF Data Services)
The <xref:System.Data.Services.Client.DataServiceContext> class encapsulates operations that are supported against a specified data service. Although [!INCLUDE[ssODataShort](../../../../includes/ssodatashort-md.md)] services are stateless, the context is not. Therefore, you can use the <xref:System.Data.Services.Client.DataServiceContext> class to maintain state on the client between interactions with the data service in order to support features such as change management. This class also manages identities and tracks changes.  
  
## Merge Options and Identity Resolution  
 When a <xref:System.Data.Services.Client.DataServiceQuery%601> is executed, the entities in the response feed are materialized into objects. For more information, see [Object Materialization](../../../../docs/framework/data/wcf/object-materialization-wcf-data-services.md). The way in which entries in a response message are materialized into objects is performed based on identity resolution and depends on the merge option under which the query was executed. When multiple queries or load requests are executed in the scope of a single <xref:System.Data.Services.Client.DataServiceContext>, the [!INCLUDE[ssAstoria](../../../../includes/ssastoria-md.md)] client only tracks a single instance of an object that has a specific key value. This key, which is used to perform identity resolution, uniquely identifies an entity.  
  
 By default, the client only materializes an entry in the response feed into an object for entities that are not already being tracked by the <xref:System.Data.Services.Client.DataServiceContext>. This means that changes to objects already in the cache are not overwritten. This behavior is controlled by specifying a <xref:System.Data.Services.Client.MergeOption> value for queries and load operations. This option is specified by setting the <xref:System.Data.Services.Client.DataServiceContext.MergeOption%2A> property on the <xref:System.Data.Services.Client.DataServiceContext>. The default merge option value is <xref:System.Data.Services.Client.MergeOption.AppendOnly>. This only materializes objects for entities that are not already being tracked, which means that existing objects are not overwritten. Another way to prevent changes to objects on the client from being overwritten by updates from the data service is to specify <xref:System.Data.Services.Client.MergeOption.PreserveChanges>. When you specify <xref:System.Data.Services.Client.MergeOption.OverwriteChanges>, values of objects on the client are replaced by the latest values from the entries in the response feed, even if changes have already been made to these objects. When a <xref:System.Data.Services.Client.MergeOption.NoTracking> merge option is used, the <xref:System.Data.Services.Client.DataServiceContext> cannot send changes made on client objects to the data service. With this option, changes are always overwritten with values from the data service.  
  
## Managing Concurrency  
 [!INCLUDE[ssODataShort](../../../../includes/ssodatashort-md.md)] supports optimistic concurrency that enables the data service to detect update conflicts. The data service provider can be configured in such a way that the data service checks for changes to entities by using a concurrency token. This token includes one or more properties of an entity type that are validated by the data service to determine whether a resource has changed. Concurrency tokens, which are included in the eTag header of requests to and responses from the data service, are managed for you by the [!INCLUDE[ssAstoria](../../../../includes/ssastoria-md.md)] client. For more information, see [Updating the Data Service](../../../../docs/framework/data/wcf/updating-the-data-service-wcf-data-services.md).  
  
 The <xref:System.Data.Services.Client.DataServiceContext> tracks changes made to objects that have been reported manually by using <xref:System.Data.Services.Client.DataServiceContext.AddObject%2A>, <xref:System.Data.Services.Client.DataServiceContext.UpdateObject%2A>, and <xref:System.Data.Services.Client.DataServiceContext.DeleteObject%2A>, or by a <xref:System.Data.Services.Client.DataServiceCollection%601>. When the <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A> method is called, the client sends changes back to the data service. <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A> can fail when data changes in the client conflict with changes in the data service. When this occurs, you must query for the entity resource again to receive the update data. To overwrite changes in the data service, execute the query using the <xref:System.Data.Services.Client.MergeOption.PreserveChanges> merge option. When you call <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A> again, the changes preserved on the client are persisted to the data service, as long as other changes have not already been made to the resource in the data service.  
  
## Saving Changes  
 Changes are tracked in the <xref:System.Data.Services.Client.DataServiceContext> instance but not sent to the server immediately. After you are finished with the required changes for a specified activity, call <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A> to submit all the changes to the data service. A <xref:System.Data.Services.Client.DataServiceResponse> object is returned after the <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A> operation is complete. The <xref:System.Data.Services.Client.DataServiceResponse> object includes a sequence of <xref:System.Data.Services.Client.OperationResponse> objects that, in turn, contain a sequence of <xref:System.Data.Services.Client.EntityDescriptor> or <xref:System.Data.Services.Client.LinkDescriptor> instances that represent the changes persisted or attempted. When an entity is created or modified in the data service, the <xref:System.Data.Services.Client.EntityDescriptor> includes a reference to the updated entity, including any server-generated property values, such as the generated `ProductID` value in the previous example. The client library automatically updates the [!INCLUDE[dnprdnshort](../../../../includes/dnprdnshort-md.md)] object to have these new values.  
  
 For successful insert and update operations, the state property of the <xref:System.Data.Services.Client.EntityDescriptor> or <xref:System.Data.Services.Client.LinkDescriptor> object associated with the operation is set to <xref:System.Data.Services.Client.EntityStates.Unchanged> and the new values are merged by using <xref:System.Data.Services.Client.MergeOption.OverwriteChanges>. When an insert, update, or delete operation fails in the data service, the entity state remains the same as it was before <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A> was called, and the <xref:System.Data.Services.Client.OperationResponse.Error%2A> property of the <xref:System.Data.Services.Client.OperationResponse> is set to an <xref:System.Data.Services.Client.DataServiceRequestException> that contains information about the error. For more information, see [Updating the Data Service](../../../../docs/framework/data/wcf/updating-the-data-service-wcf-data-services.md).  
  
### Setting the HTTP Method for Updates  
 By default, the .NET Framework client library sends updates to existing entities as MERGE requests. A MERGE request updates selected properties of the entity; however the client always includes all properties in the MERGE request, even properties that have not changed. The [!INCLUDE[ssODataShort](../../../../includes/ssodatashort-md.md)] protocol also supports sending PUT requests to update entities. In a PUT request, an existing entity is essentially replaced with a new instance of the entity with property values from the client. To use PUT requests, set the <xref:System.Data.Services.Client.SaveChangesOptions.ReplaceOnUpdate> flag on the <xref:System.Data.Services.Client.SaveChangesOptions> enumeration when calling <xref:System.Data.Services.Client.DataServiceContext.SaveChanges%2A>.  
  
> [!NOTE]
>  A PUT request will behave differently than a MERGE request when the client does not know about all properties of the entity. This might occur when projecting an entity type into a new type on the client. It might also occur when new properties have been added to the entity in the service data model and the <xref:System.Data.Services.Client.DataServiceContext.IgnoreMissingProperties%2A> property on the <xref:System.Data.Services.Client.DataServiceContext> is set to `true` to ignore such client mapping errors. In these cases, a PUT request will reset any properties that are unknown to the client to their default values.  
  
### POST Tunneling  
 By default, the client library sends create, read, update, and delete requests to an [!INCLUDE[ssODataShort](../../../../includes/ssodatashort-md.md)] service by using the corresponding HTTP methods of POST, GET, PUT/MERGE/PATCH, and DELETE. This upholds the basic principles of Representational State Transfer (REST). However, not every Web server implementation supports the full set of HTTP methods. In some cases, the supported methods might be restricted to just GET and POST. This can happen when an intermediary, like a firewall, blocks requests with certain methods. Because the GET and POST methods are most often supported, [!INCLUDE[ssODataShort](../../../../includes/ssodatashort-md.md)] prescribes a way to execute any unsupported HTTP methods by using a POST request. Known as *method tunneling* or *POST tunneling*, this enables a client to send a POST request with the actual method specified in the custom `X-HTTP-Method` header. To enable POST tunneling for requests, set the <xref:System.Data.Services.Client.DataServiceContext.UsePostTunneling%2A> property on the <xref:System.Data.Services.Client.DataServiceContext> instance to `true`.  
  
## See Also  
 [WCF Data Services Client Library](../../../../docs/framework/data/wcf/wcf-data-services-client-library.md)  
 [Updating the Data Service](../../../../docs/framework/data/wcf/updating-the-data-service-wcf-data-services.md)  
 [Asynchronous Operations](../../../../docs/framework/data/wcf/asynchronous-operations-wcf-data-services.md)  
 [Batching Operations](../../../../docs/framework/data/wcf/batching-operations-wcf-data-services.md)