# Dot Net Framework



## Entity Framework (EF) Migration

### Auto migration

Just using the code 

`Database.SetInitializer(new MigrateDatabaseToLatestVersion<MyDBContext, MyDBConfiguration>());`

```csharp
    public class MyDBContext : DbContext
    {
        public MyDBContext() : base("Connection String Name")
        {
            Database.SetInitializer(new CreateDatabaseIfNotExists<MyDBContext>());
            Database.SetInitializer(new MigrateDatabaseToLatestVersion<MyDBContext, MyDBConfiguration>());
        }

    }

    public class MyDBConfiguration : DbMigrationsConfiguration<MyDBContext>
    {
        public MyDBConfiguration()
        {
            AutomaticMigrationsEnabled = true;
            AutomaticMigrationDataLossAllowed = true;
            MigrationsDirectory = @"Migrations\MyDBContext";
        }
    }
```

### Manually migration (Migration Command)

General flow migration command

```mermaid
    graph LR
    A(Enable-Migration)-->B(Add-Migration)
    B-->C(Update-Database)
```

| Command          | Description                                                                                     | Parameters                                                                                                                                                                 |
| ---------------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Enable-Migration | Without param -> generate `Migrates` folder and `Configuration.cs`                              | `-ProjectName NameOfProject`<br>`-MigrationsDirectory "Migrations\ContextA"` <br> `-ContextTypeName MyProject.Models.DBContextA`                                           |
| Add-Migration    | Detect the difference between Model and Database <br>to generate the script or migrate cs files | `-ProjectName NameOfProject`<br>`-Name MigrateFileName` <br>`-ConfigurationTypeName ConfigA` <br> Note: Without parameter will use `Configuration.cs` in `Migrates` folder |
| Update-Database  | Run update database based on migrate files                                                      | `-ProjectName NameOfProject`<br>`-ConfigurationTypeName ConfigurationB` <br> Rollback to a migrate file:  <br>  `-TargetMigration:MigrateFileName`                         |


**Remarks**

>To see the examples, type: get-help Enable-Migrations -examples.

>For more information, type: get-help Enable-Migrations -detailed.

>For technical information, type: get-help Enable-Migrations -full.

**Reference**

[How do I enable EF migrations for multiple contexts to separate databases?](https://stackoverflow.com/questions/13469881/how-do-i-enable-ef-migrations-for-multiple-contexts-to-separate-databases)

## Entity Framework 6: Managing DbContext in right way

[MANAGING DBCONTEXT THE RIGHT WAY WITH ENTITY FRAMEWORK 6: AN IN-DEPTH GUIDE](https://mehdi.me/ambient-dbcontext-in-ef6/)

After take a while to read this article, there are some points I wanna summary to note:

### MANAGING THE DBCONTEXT INSTANCE LIFETIME INDEPENDENTLY OF THE BUSINESS TRANSACTION LIFETIME.

> Make decouple the lifetime of the DbContext instance from the business transaction lifetime.

A `DbContext` instance can however span across multiple (sequential) business transactions. Once a business transaction has completed and has called the `DbContext.SaveChanges()` method to persist all the changes it made, it's entirely possible to just re-use the same DbContext instance for the next business transaction.

I.e. the lifetime of a DbContext instance is not necessarily bound to the lifetime of a single business transaction.





## Add WebAPI into the existing WebMVC project

Step 1: Adding the necessary WebAPI packages

Step 2: Configuring the routing

```csharp
public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            config.MapHttpAttributeRoutes();
 
            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{action}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
        }
    }
```
Step 3: Register api routing
In the `Application_Start` method of the file `Global.asax.cs` file, we will add a call to  `GlobalConfiguration.Configure`; be careful to place it before  the call to `RouteConfig.RegisterRoutes`(RouteTable.Routes):

```csharp
public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            GlobalConfiguration.Configure(WebApiConfig.Register);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
        }
    }
```

Example:

```csharp
namespace WebApplication1.Controllers.webapi
{
    [RoutePrefix("api/Students")]
    public class WepApiController : ApiController
    {
        [HttpGet]
        [Route("GetStudents")]
        public List GetStudents()
        {
            List studentList = new List();
 
            string[] students = { "John", "Henry", "Jane", "Martha" };
 
            foreach (string student in students)
            {
                Student currentStudent = new Student
                {
                    Name = student,
                    Email = student + "@academy.com"
                };
                studentList.Add(currentStudent);
            }
            return studentList;
        }
    }
}
```

**http://localhost/api/Students/GetStudents**

**Reference**

[Adding Web API Support to an Existing ASP.NET MVC Project](https://developerslogblog.wordpress.com/2016/12/30/adding-web-api-support-to-an-existing-asp-net-mvc-project/)


## Practices for creating Api

> Should apply for creating service

* Should have base api controller for handle exception in one place

```csharp
public abstract class BaseApiController : ApiController
    {
        private readonly ILog _logger = LogManager.GetLogger(typeof(BaseApiController));

        protected T HandleExceptions<T>(Func<T> func) where T : BaseApiResponse, new()
        {
            return HandleExceptions(func, null);
        }

        protected T HandleExceptions<T>(Func<T> func, string errorMessage) where T : BaseApiResponse, new()
        {
            try
            {
                var response = func();

                var customResponse = response.ResponseResult;
                response.ResponseResult = new TrmResponse()
                {
                    StatusCode = customResponse != null && customResponse.StatusCode != 0 ? customResponse.StatusCode : HttpStatusCode.OK,
                    ResponseMessage = customResponse?.ResponseMessage ?? "OK"
                };
                return response;
            }
            catch (Exception ex)
            {
                if (string.IsNullOrEmpty(errorMessage))
                    errorMessage = $"An error has occurred when getting {typeof(T).Name}";

                _logger.Error(errorMessage + ex.StackTrace);
                var t = new T()
                {
                    ResponseResult = new TrmResponse()
                    {
                        StatusCode = HttpStatusCode.InternalServerError,
                        ResponseMessage = errorMessage,
                        Exception = ex
                    }
                };
                return t;
            }
        }
    }

```
Example using `HandleException`

```csharp
    [HttpGet]
    [AcceptVerbs("GET")]
    [ResponseType(typeof(ExportTransactionResponse))]
    public virtual ExportTransactionResponse GetExportTransactions(
        int transactionType = -1,
        string integrationStatus = null,
        string contactId = null,
        int numberItems = 0)
        {
            return HandleExceptions(() => new ExportTransactionResponse()
            {
                ExportTransactions = _exportTransactionsHelper.GetExportTransactions(
                    transactionType,
                    integrationStatus, contactId,
                    numberItems).ToList()
            });
        }
```
> Should have base response class for consistency

## .Net Web API - How security to call api



Reference:

[Securing ASP.NET Web API](https://code.tutsplus.com/tutorials/securing-aspnet-web-api--cms-26012)



## Authentication and authorization in Asp.Net

The evolution of provider

```mermaid
    graph TD
    0(Asp.net 1.0)-->A(Membership Provider <br> Asp.Net 2.0 release in 2005)
    A-->B(Simple Membership <br> Asp.net 4.0/Mvc 4 around 2008)
    B-->C(Universal Provider <br> Asp.net 4.0/4.5)
    C-->D(Asp.net Identity <br> Asp.net 4.5/Mvc 5)
```

Asp.net Identity step by step:

Source: https://www.tektutorialshub.com/asp-net/asp-net-introduction-to-asp-net-identity/

https://bitoftech.net/2015/01/21/asp-net-identity-2-with-asp-net-web-api-2-accounts-management/

Why?

Membership Provider:
* Must use the MS SQL Database
* Can not custom the User table

Simple Membership Provider: tries to address these issues by offering a flexible model for authenticating the users.

The inheritance hierarchy of SimpleMembership

![The inheritance hierarchy of SimpleMembership](images/SimpleMembership.png)

Reference:

[Using SimpleMembership in ASP.NET MVC 4](https://www.codeguru.com/csharp/.net/net_asp/mvc/using-simplemembership-in-asp.net-mvc-4.htm)


**What is OAuth?**

OAuth is about authorization and not authentication. Authorization is asking for permission to do stuff. 
Authentication is about proving you are the correct person because you know things. 

OAuth doesn’t pass authentication data between consumers and service providers – but instead acts as an authorization token of sorts.


**OAuth implementation**

**OAuth 1 vs OAuth 2**

OAuth 2.0 is a complete redesign from OAuth 1.0, and the two are not compatible

OAuth 2.0 is faster and easier to implement. OAuth 1.0 used complicated cryptographic requirements, only supported three flows, and did not scale.

OAuth 2.0, on the other hand, has six flows for different types of applications and requirements, and enables signed secrets over HTTPS. OAuth tokens no longer need to be encrypted on the endpoints in 2.0 since they are encrypted in transit.

https://www.varonis.com/blog/what-is-oauth/

**OAuth vs. OpenID**

OpenID is about authentication (ie. proving who you are), 

OAuth is about authorisation (ie. to grant access to functionality/data/etc.. without having to deal with the original authentication).

> OAuth is not an OpenID extension. Why OAuth is not an OpenID extension?

The answer is simple, OAuth attempts to provide a standard way for developers to offer their services via an API without forcing their users to expose their passwords (and other credentials)

If OAuth depended on OpenID, only OpenID services would be able to use it, and while OpenID is great, there are many applications where it is not suitable or desired.

> However you can OAuth and OpenID together

http://cakebaker.42dh.com/2008/04/01/openid-versus-oauth-from-the-users-perspective/

http://techastute.blogspot.com/2012/05/openid-authentication-oauth.html

[What's the difference between OpenID and OAuth?](https://stackoverflow.com/questions/1087031/whats-the-difference-between-openid-and-oauth)

**SAML vs. OAuth**

SAML (Security Assertion Markup Language) is an alternative federated authentication standard that many enterprises use for Single-Sign On (SSO). 
SAML enables enterprises to monitor who has access to corporate resources.

OAuth doesn’t share password data but instead uses authorization tokens to prove an identity between consumers and service providers. 
OAuth is an authorization protocol that allows you to approve one application interacting with another on your behalf without giving away your password.

| SAML                                                                                                                                                                                                             | OAuth                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SAML uses XML to pass messages                                                                                                                                                                                   | OAuth uses JSON                                                                                                                                                                               |
| SAML is geared towards enterprise security                                                                                                                                                                       | OAuth provides a simpler mobile experience                                                                                                                                                    |
| SAML, on the other hand, drops a session cookie in a browser that allows a user to access certain web pages – great for short-lived work days, but not so great when have to log into your thermostat every day. | OAuth uses API calls extensively, which is why mobile applications, modern web applications, game consoles, and Internet of Things (IoT) devices find OAuth a better experience for the user. |

Reference

[What is OAuth? Definition and How it Works](https://www.varonis.com/blog/what-is-oauth/)

**SAML vs. OpenID**

They are two different protocols of authentication and they differ at the technical level.

SAML2 supports single sign-out - but OpenID does not

SAML 2 is based on XML while OpenID is not.

SAML2 has different bindings while the only binding OpenID has is HTTP

Same in primary use case but has difference scope

SAML - SSO for enterprise apps

OpenID - SSO for consumer apps

![](images/saml_openid_oauth.png)

https://www.resilient-networks.com/concept-week-saml-oauth2-openid-connect/

![](images/saml_openIdc_oauth2.png)

https://spin.atomicobject.com/2016/05/30/openid-oauth-saml/

https://stackoverflow.com/questions/7699200/what-is-the-difference-between-openid-and-saml

**OAuth vs JWT**

OAuth 2.0 defines a protocol, i.e. specifies how tokens are transferred, JWT defines a token format.

> Firstly, we have to differentiate JWT and OAuth. Basically, JWT is a token format. OAuth is an authorization protocol that can use JWT as a token.

OAuth uses server-side and client-side storage. If you want to do real logout you must go with OAuth2. Authentication with JWT token can not logout actually. Because you don't have an Authentication Server that keeps track of tokens.

If you want to provide an API to 3rd party clients, you must use OAuth2 also.

OAuth2 is very flexible. JWT implementation is very easy and does not take long to implement.

If your application needs this sort of flexibility, you should go with OAuth2. But if you don't need this use-case scenario, implementing OAuth2 is a waste of time.


https://stackoverflow.com/questions/39909419/what-are-the-main-differences-between-jwt-and-oauth-authentication

**Can OAuth can use to authentication?**

https://oauth.net/articles/authentication/

A standard for user authentication using OAuth: OpenID Connect

## MVC Only Validation for fields which be declared in html form

```csharp
    public class ValidateOnlyIncomingValuesAttribute : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext filterContext)
        {
            var modelState = filterContext.Controller.ViewData.ModelState;
            var valueProvider = filterContext.Controller.ValueProvider;

            var keysWithNoIncomingValue = modelState.Keys.Where(x => !valueProvider.ContainsPrefix(x));
            foreach (var key in keysWithNoIncomingValue)
                modelState[key].Errors.Clear();
        }
    }
    
    //Usage
    [HttpPost]
    [ValidateOnlyIncomingValues]
    public ActionResult Index(PageViewModel viewModel)
    {
        if (!ModelState.IsValid)
        {
            // ReSharper disable once Mvc.ViewNotResolved
            return View(viewModel);
        }
    }

```
