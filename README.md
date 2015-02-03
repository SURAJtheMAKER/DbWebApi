# DbWebApi

### What is it?

DbWebApi is a .Net library that implement an entirely generic Web API for data-driven applications. It acts as a proxy service for web clients to call database (Oracle + SQL Server) stored procedures or functions out-of-box without any configuration or extra coding, the http response JSON or XML will have all Result Sets, Output Parameters and Return Value. If client request a CSV format (accept: text/csv), the http response will transmit the first result set as a CSV stream for almost unlimited number of rows.

### What are the benefits of DbWebApi?

- In data-driven applications area, there are a large number of scenarios without substantial logic in data access web services, however they wasted a lot of our efforts on very boring data moving coding or configurations, we've had enough of it. Since now on, most of thus repetitive works should be dumped onto DbWebApi.
- DbWebApi can coexist within your existing ASP.NET Web API, as a supplementary service to reduce new boring manual works for most common of application scenarios. DbWebApi does not attempt to replace any existing methods or cover much specific application scenarios.

## Usage

#### ApiController:
``` CSharp
using System.Net.Http;
using System.Web.Http;
using System.Collections.Generic;
using DataBooster.DbWebApi;

namespace SampleDbWebApi.Controllers
{
    public class DbWebApiController : ApiController
    {
        [HttpGet]
        [HttpPost]
        [HttpPut]
        [HttpDelete]
        [DbWebApiAuthorize]
        public HttpResponseMessage Execute(string sp, Dictionary<string, object> parameters)
        {
            return this.ExecuteDbApi(sp, parameters);
        }
    }
}
```
That's it!  
ExecuteDbApi is an extension method to ApiController provided by DbWebApi library.
``` CSharp
public static HttpResponseMessage ExecuteDbApi(this ApiController apiController,
                                               string sp, IDictionary<string, object> parameters)
// sp:         Specifies the fully qualified name of database stored procedure or function
// parameters: Specifies required input-parameters as name-value pairs
```

#### Web.config  
"DataBooster.DbWebApi.MainConnection" is the only one configuration item needs to be customized:
``` Xml
<connectionStrings>
  <add name="DataBooster.DbWebApi.MainConnection" providerName="System.Data.SqlClient" connectionString="Data Source=.\SQLEXPRESS;Initial Catalog=SAMPLEDB;Integrated Security=SSPI" />
</connectionStrings>
```

#### Client Request  
##### Url:  
As registered in your WebApiConfig Routes (e.g. http://BaseUrl/Your.StoredProcedure.FullyQualifiedName)  
-- Input Parameters:  
Only required input-parameters of the stored procedure/function need to be specified in your request body as JSON format (Content-Type: application/json). Don't put parameter prefix ('@' or ':') in the JSON body.  
For example, a SQL Server Stored Procedure:  
``` SQL
ALTER PROCEDURE dbo.prj_GetRule
    @inRuleDate  datetime,
    @inRuleId    int,
    @inWeight    float(6) = 0.1,
    @outRuleDesc varchar(256) = NULL OUTPUT
AS  ...
```
The request JSON should like:  
``` JSON
{
    inRuleDate: "2015-02-03T00:00:00Z",
    inRuleId: 108
}
```
##### Accept Response MediaType:  
1. JSON (default)  
    Specify in request heade:  
    Accept: application/json  
    or  
    Accept: text/json  
    or specify in query string: ?format=json  
       (e.g. http://BaseUrl/YourDatabase.dbo.prj_GetRule?format=json)  
    or specify in UriPathExtension which depends on your url routing.  
2. XML  
    Specify in request header:  
    Accept: application/xml  
    or  
    Accept: text/xml  
    or specify in query string: ?format=xml  
       (e.g. http://BaseUrl/YourDatabase.dbo.prj_GetRule?format=xml)  
    or specify in UriPathExtension which depends on your url routing.  
3. CSV
    Specify in request header:  
    Accept: text/csv  
    or specify in query string: ?format=csv  
       (e.g. http://BaseUrl/YourDatabase.dbo.prj_GetRule?format=csv)  
    or specify in UriPathExtension which depends on your url routing.  
    Notes: current implementation CSV response will only return the first result set if your stored procedure has multiple result sets. It's considering to allow client to specify which result set to return in future releases.  

#### Response body formats  
1. application/json, text/json  
    Sample:  
``` JSON
{
  "ResultSets":
  [
    [
      {"COL_1":"2015-02-03T00:00:00","COL_2":3.14159,"COL_3":"Hello World1","COL_4":null, "COL_5":0},
      {"COL_1":"2015-02-02T00:00:00","COL_2":3.14159,"COL_3":null,"COL_4":1234567.800099, "COL_5":1},
      {"COL_1":"2015-02-01T00:00:00","COL_2":3.14159,"COL_3":"Hello World3","COL_4":null, "COL_5":2},
      {"COL_1":"2015-01-31T00:00:00","COL_2":3.14159,"COL_3":null,"COL_4":9876541.230091, "COL_5":3}
    ],
    [
      {"COL_A":100,"COL_B":"fooA","COL_C":0},
      {"COL_A":200,"COL_B":"fooB","COL_C":null},
      {"COL_A":300,"COL_B":"fooC","COL_C":1}
    ],
    [
       {"NOTE":"Test1 for the third result set"},
       {"NOTE":"Test2 for the third result set"}
    ]
  ],
  "OutputParameters":
  {
    "outRuleDesc":"This is a test output parameter value.",
    "outSumTotal":888888.88,
    "outRC1":null
  },
  "ReturnValue":0
}
```

2. application/xml, text/xml  
    Sample:
``` XML
<StoredProcedureResponse xmlns:i="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.datacontract.org/2004/07/DbParallel.DataAccess">
  <OutputParameters>
    <outRuleDesc xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">This is a test output parameter value.</outRuleDesc>
    <outSumTotal xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">888888.88</outSumTotal>
    <outRC1 i:nil="true" xmlns="" />
  </OutputParameters>
  <ResultSets>
    <ArrayOfBindableDynamicObject>
      <BindableDynamicObject>
        <COL_1 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:dateTime" xmlns="">2015-02-03T00:00:00</COL_1>
        <COL_2 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">3.14159</COL_2>
        <COL_3 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">Hello World1</COL_3>
        <COL_4 i:nil="true" xmlns=""/>
        <COL_5 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">0</COL_5>
      </BindableDynamicObject>
      <BindableDynamicObject>
        <COL_1 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:dateTime" xmlns="">2015-02-02T00:00:00</COL_1>
        <COL_2 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">3.14159</COL_2>
        <COL_3 i:nil="true" xmlns=""/>
        <COL_4 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">1234567.800099</COL_4>
        <COL_5 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">1</COL_5>
      </BindableDynamicObject>
      <BindableDynamicObject>
        <COL_1 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:dateTime" xmlns="">2015-02-01T00:00:00</COL_1>
        <COL_2 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">3.14159</COL_2>
        <COL_3 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">Hello World3</COL_3>
        <COL_4 i:nil="true" xmlns=""/>
        <COL_5 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">2</COL_5>
      </BindableDynamicObject>
      <BindableDynamicObject>
        <COL_1 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:dateTime" xmlns="">2015-01-31T00:00:00</COL_1>
        <COL_2 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">3.14159</COL_2>
        <COL_3 i:nil="true" xmlns=""/>
        <COL_4 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:decimal" xmlns="">9876541.230091</COL_4>
        <COL_5 xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">3</COL_5>
      </BindableDynamicObject>
    </ArrayOfBindableDynamicObject>
    <ArrayOfBindableDynamicObject>
      <BindableDynamicObject>
        <COL_A xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">100</COL_A>
        <COL_B xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">fooA</COL_B>
        <COL_C xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">0</COL_C>
      </BindableDynamicObject>
      <BindableDynamicObject>
        <COL_A xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">200</COL_A>
        <COL_B xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">fooB</COL_B>
        <COL_C i:nil="true" xmlns=""/>
      </BindableDynamicObject>
      <BindableDynamicObject>
        <COL_A xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">300</COL_A>
        <COL_B xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">fooC</COL_B>
        <COL_C xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:int" xmlns="">1</COL_C>
      </BindableDynamicObject>
    </ArrayOfBindableDynamicObject>
    <ArrayOfBindableDynamicObject>
      <BindableDynamicObject>
        <NOTE xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">Test1 for the third result set</NOTE>
      </BindableDynamicObject>
      <BindableDynamicObject>
        <NOTE xmlns:d5p1="http://www.w3.org/2001/XMLSchema" i:type="d5p1:string" xmlns="">Test2 for the third result set</NOTE>
      </BindableDynamicObject>
    </ArrayOfBindableDynamicObject>
  </ResultSets>
  <ReturnValue i:nil="true" />
</StoredProcedureResponse>
```

3. text/csv  




## NuGet
There are 4 NuGet packages for 4 differenct versions of ADO.NET providers:
- [DbWebApi for SQL Server](http://www.nuget.org/packages/DataBooster.DbWebApi.SqlServer)
- [DbWebApi for Oracle (use ODP.NET Managed Driver)](http://www.nuget.org/packages/DataBooster.DbWebApi.Oracle.Managed)
- [DbWebApi for Oracle (use ODP.NET Provider)](http://www.nuget.org/packages/DataBooster.DbWebApi.Oracle.ODP)
- [DbWebApi for Oracle (use DataDirect Provider)](http://www.nuget.org/packages/DataBooster.DbWebApi.Oracle.DataDirect)

For-Oracle versions always contain the support for SQL Server. To switch from Oracle to SQL Server, simply change the providerName and connectionString of connectionStrings "DataBooster.DbWebApi.MainConnection" in your web.config.  
To switch above from one NuGet package to another NuGet Package, simply uninstall one and install another from NuGet Package Manager.

## Examples

Please refer to example ASP.NET MVC 4 Web Application - MyDbWebApi in https://github.com/DataBooster/DbWebApi/tree/master/Examples/MyDbWebApi

The example project also requires Visual Studio 2010 or above.
