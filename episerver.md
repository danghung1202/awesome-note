# EpiServer Notes

## How to override, decorate the default implement of the class in EpiServer

> Conditions: The class must to register as service via IOC using StructureMap or any IOC framework which support Decorate pattern

Based on my experience when working on some Epi commerce projects, depend on if the default implement has the method which can be override, there are two ways to archive it

### **The default implement have the method which can be override (has `virtual` keyword)**

In EpiServer Commerce has an interface to service for updating and retrieving `EPiServer.Commerce.Order.ILineItem.PlacedPrice` for `EPiServer.Commerce.Order.IOrderGroup`.

    public interface IPlacedPriceProcessor

The default implement 

    public class DefaultPlacedPriceProcessor : IPlacedPriceProcessor

In this class, all methods are decorated with `virtual` keyword like that 

    public virtual Money? GetPlacedPrice(EntryContentBase entry, decimal quantity, CustomerContact customerContact, IMarket market, Currency currency);

So in this way, in IOC configuration using StructureMap, you can override and decorate this class with this configuration:

```
    public void ConfigureContainer(ServiceConfigurationContext context)
        {
            context.StructureMap().Configure(ce =>
            {
                ce.For<IPlacedPriceProcessor>().Use<TrmPlacedPriceProcessor>().Singleton();
                ce.For<IAmStoreHelper>().DecorateAllWith<TrmStoreHelper>();
            });
        }
```

The new implement need to inherit from the default implement `DefaultPlacedPriceProcessor`. Following this way, we can override any method and keep other methods as default.

```
public class TrmPlacedPriceProcessor : DefaultPlacedPriceProcessor, IPlacedPriceProcessor
    {
        public override Money? GetPlacedPrice(EntryContentBase entry, decimal quantity, CustomerContact customerContact,
            MarketId marketId, Currency currency) 
            {
                //Override here
            }
    }

```

So from now on, all places are using `IPlacedPriceProcessor` to invoke the method `GetPlacedPrice` will run our override code

### **The default implement don't have the method which can be override (no `virtual` keyword)**

In Epi Commerce, there is an interface `IPriceService` with default implement class `PriceServiceDatabase`. In this class, all methods don't have any `virtual` keyword so we can not override it.

Luckily, this class is registered via IOC, we have a chance to do it.

Registered via IOC the new implement same above

```
public void ConfigureContainer(ServiceConfigurationContext context)
        {
            context.StructureMap().Configure(ce =>
            {
                ce.For<IPriceService>().Use<PriceServiceDatabase>();
                ce.For<IPriceService>().DecorateAllWith<TrmPriceService>();
            });
        }
```

The key technical step in this way: Inject `IPriceService` itself to new implement class and StructureMap will do the remain, **injecting the instance of the default implement class to using in the new implement class**

```
public class TrmPriceService : IPriceService
    {
        private readonly IPriceService _mediachasePricingService;
        public TrmPriceService(IPriceService mediachasePricingService){
            _mediachasePricingService = mediachasePricingService;
        }

        public virtual IEnumerable<IPriceValue> GetCatalogEntryPrices(CatalogKey catalogKey)
        {
            //Override here
            
            //Or decorate with a wrapper here before call the default implement
            return _mediachasePricingService.GetCatalogEntryPrices(catalogKey);
        }

        //do the same with other remaining methods
    }
```

Following this way, you can decorate the default implement of any method ex add `virtual` keyword, logging, audit..

> Thanks Ha.Bui for this useful note :)

> [More about Decorate with StructureMap](https://robertlinde.se/posts/ioc,-structuremap-and-an-async-generic-repository/)

## Create nice Admin tool

You can create EpiServer admin tool using Mvc template or `.aspx` template. In case using `.aspx` template, you should follow the standard UI

1. Using EPIServerUI control
2. Using Content Placeholder
3. Set Heading and Description
4. Using EpiServer css classes


`AdminTool.aspx`

```
<%@ Page Language="C#" AutoEventWireup="true" CodeBehind="MetalPriceImportPlugin.aspx.cs" Inherits="TRM.Web.Plugins.MetalPriceImportPlugin" %>

<%@ Register TagPrefix="EPiServerUI" Namespace="EPiServer.UI.WebControls" Assembly="EPiServer.UI" %>

<asp:content contentplaceholderid="MainRegion" runat="server">    
    <div class="epi-formArea">

        <asp:Label ID ="lblMessage"  runat="server"/>
        <div class="epi-size20 epi-paddingVertical-small">
            <div>
                <asp:Label runat="server" AssociatedControlID="fileUpload" Text="Select an csv file and upload" />
                <asp:FileUpload ID="fileUpload" runat="server"  />
            </div>
        </div>
            
        <div class="epi-buttonContainer">
            <EPiServerUI:ToolButton id="ImportButton" OnClick="ImportButton_OnClick" runat="server" SkinID="Import" text="Begin Import"  tooltip="Begin Import"  />            
        </div>
    </div>
</asp:content>
```

`AdminTool.aspx.cs`
```
[GuiPlugIn(
    DisplayName = "Import Metal Price",
    Area = PlugInArea.AdminMenu,
    Url = "~/Plugins/MetalPriceImportPlugin.aspx",
    RequiredAccess = AccessLevel.Administer)]
public partial class MetalPriceImportPlugin : WebFormsBase
{
    protected void Page_Load(object sender, EventArgs e)
    {
    }

    protected override void OnPreInit(EventArgs e)
    {
        base.OnPreInit(e);
        this.SystemMessageContainer.Heading = "This is Plugin Heading";
        this.SystemMessageContainer.Description = "This is description";
    }
}
```

**Reference** 
[How to create a nice looking admin plugin](https://world.episerver.com/blogs/Per-Nergard/Dates/2013/4/How-to-create-a-nice-looking-admin-plugin/)


## Performance issue with find Simple URL in CMS

An our customer found this issue when he investigated the performance issues on Production. Basically, his site have a lot of apis however those are quite slow in an expected way. The root cause is the router for apis were registered after "Simple URL" router of EpiServer. This is led to each api was called, the site need to resolve SimpleURL router first then api router. This is unnecessary thing.

=> So all custom routers in an Epi site should register before SimpleURL

All code for register custom routers need to make sure run firstly before the below code

```
public void Initialize(InitializationEngine context)
{
    //Register custom routers here

    //Remove "simpleaddress" router and add it to last
    Global.RoutesRegistered += Global_RoutesRegistered;
}

private void Global_RoutesRegistered(object sender, RouteRegistrationEventArgs e)
{
    var simpleAddressRouter =
        e.Routes.OfType<IContentRoute>().FirstOrDefault(r => r.Name.Equal.("simpleaddress"));
    if (simpleAddressRouter != null)
    {
        e.Routes.Remove((RouteBase) simpleAddressRouter);
        e.Routes.Add(simpleAddressRouter);
    }
}
```


**Reference**

https://vimvq1987.com/episerver-cms-performance-optimization-part-1/

https://world.episerver.com/documentation/Release-Notes/ReleaseNote/?releaseNoteId=CMS-7791

