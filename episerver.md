# EpiServer Notes

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

