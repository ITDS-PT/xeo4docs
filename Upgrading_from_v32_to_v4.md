# Upgrading from XEO v3.2 to v4

There are some important concepts that have changed from version 3.2 to 4 and full backward-compatability is not possible. This document **is not to explain how to convert a project in XEO V3.2 to XEO V4**, but instead to show you what changes in the way you do things.

## Viewer definition
XEO V3 Viewer definitions are mostly compatible with V4, with the following exceptions

- If you use the xvw:form component in a viewer, you need to **remove** it (yes delete it)
- List viewers and Main viewers require an additional parameter in the xvw:viewer element, which is **transactionLess='true'**
- The defaults beans are no longer the XEOBaseBean, XEOEditBean, XEOBaseList and XEOBaseLookupList (more on that in the beans section)
- The Main viewer definition is not compatible with the new version (more on that bellow)
- The faces-config.xml and web.xml files require a small change

### Main Viewer

In XEO V4 the Main viewer definition is different from the previous version. In V4 you use a template (or create oyour own) and then replace the declared "areas" in the template with your content. In XEO you can declare a template as a regular viewer ( usually without any bean associated ) with special "areas" marked as replaceable (using the **xvw:insert** directive). See the following exemple of a template (**named MainTemplate.xvw**)

```xml
<xvw:root xmlns:xeo="http://www.netgest.net/xeo/xeo" xmlns:xvw="http://www.netgest.net/xeo/xvw">
    <xvw:viewer transactionLess='true'>
        <xvw:form id='mainForm'>
        	<header id="header">
				 <xvw:insert src='header.xvw' name='header'/>
			</header>
		<aside id="left-panel">
			<!-- User info -->
			<div class="login-info">
				<xvw:insert source='profile.xvw' name='profile' />
			</div>
            <!-- TREE PANEL -->
			<xvw:insert name='treePanel'>
			</xvw:insert>	
			<span class="minifyme" data-action="minifyMenu"> <i class="fa fa-arrow-circle-left hit"></i> </span>
		</aside>
		
		<div id="main" role="main">

			<!-- RIBBON -->
			<div id="ribbon">
				<!-- breadcrumb -->
				<xvw:sceneCrumb id='breadcrumb'></xvw:sceneCrumb>
			</div>
			<!-- END RIBBON -->

			<!-- MAIN CONTENT -->
			<div id="content" class='scroll-content'>
				<xvw:errorMessages></xvw:errorMessages>
				<xvw:applicationContainer id='appContainer'></xvw:applicationContainer> 
			</div>
			<!-- END MAIN CONTENT -->

		</div>
		
		</xvw:form>
    </xvw:viewer>
</xvw:root>

```
And use a Main Viewer definition using the xvw:composition directive and replacing the content of the "treePanel" and "profile" areas with your content, like the following:

```xml
<xvw:root xmlns:xeo="http://www.netgest.net/xeo/xeo" xmlns:xvw="http://www.netgest.net/xeo/xvw">
	<xvw:viewer beanClass="path.to.the.MainBean" beanId="viewBean"	transactionLess='true'>
		<xvw:composition template='MainTemplate.xvw'>
        
				<xvw:define name='profile'>
					<xvw:template template="profile.ftl" username='#{viewBean.loggedUser}' profiles='#{viewBean.profiles}'></xvw:template>
				</xvw:define>
			<xvw:define name='treePanel'>
				<xvw:treePanel>

					<xvw:menu text="Some action" 
						icon="path/to/icon.gif" serverAction="#{viewBean.createObject}"
						value='{viewerName : "path/to/viewer/edit.xvw" , objectName : "SOME_OBJECT"}'></xvw:menu>
                        <!-- More menus -->
                        
				</xvw:treePanel>
				<!-- Content to include, can be html or components -->
			</xvw:define>
		</xvw:composition>
	</xvw:viewer>
</xvw:root>

```
This allows to reuse the template across the project (for multiple main viewers) only replacing what you need.

### Sample List Viewer

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xvw:root xmlns:xeo="http://www.netgest.net/xeo/xeo" xmlns:xvw="http://www.netgest.net/xeo/xvw">
    <xvw:viewer
        beanClass="netgest.bo.xwc.xeo.controllers.beans.ListBean"
        beanId="viewBean" transactionLess='true' >
        <xeo:formList>
            <xeo:list>
                <xvw:columns>
                    <!-- Columns specific to the model -->
                </xvw:columns>
            </xeo:list>
        </xeo:formList>
    </xvw:viewer>
</xvw:root>

```

####Important Note:
The template/main viewer should have a xvw:form component envolving the entire content (it's required for things such as asking the user if he wants to loose content when changing scenes)

## Beans (to be used in scenes inside the applicationContainer)

Since version 4
brings so many changes, we decided to create a new set of beans for the various types of situations.

- netgest.bo.xwc.framework.controllers.ApplicationBaseBean (for any viewer)
- netgest.bo.xwc.xeo.controllers.MainBean (for Main Viewers)
- netgest.bo.xwc.xeo.controllers.beans.ListBean (for List viewers)
- netgest.bo.xwc.xeo.controllers.beans.EditBean (for Edit viewers)
- netgest.bo.xwc.xeo.controllers.beans.LookupBean (for Lookup viewers)

This means that you will have to change XEO Studio's default beans in the project's properties menu to have the scaffolding utility generate correct viewer definitions.

## FacesConfig.xml

For the new version you need to make a slight change in the faces-config.xml, previous versions of XEO will have the following:

```xml
	<lifecycle>
    <phase-listener>netgest.bo.xwc.framework.jsf.XUIPhaseListener</phase-listener>
  </lifecycle>
```

You need to change it to the following (notice the additional phase-listener)

```xml
<lifecycle>
    <phase-listener>netgest.bo.xwc.framework.jsf.XUIPhaseListener</phase-listener>
    <phase-listener>netgest.bo.xwc.framework.jsf.ComponentReferenceBindingPhaseListener</phase-listener>
  </lifecycle>

```
##Web.xml / boconfig.xml

You also need to change the **renderKit**, which can be done in the web.xml like the following (you will need the to set the renderKit to **XEOSMART**:

```xml
<servlet>
		<servlet-name>XWC Servlet</servlet-name>
		<servlet-class>netgest.bo.xwc.framework.http.XUIServlet</servlet-class>
		<!-- Other parameters -->
		<init-param>
			<param-name>renderKit</param-name>
			<param-value>XEOSMART<param-value>
		</init-param>
		<!-- Other parameters -->
	</servlet>

```

This can also be done in the boconfig.xml file (which makes the renderKit available to all web contexts in the application)

```xml
<!-- remaining boConfig -->
<renderKits default="XEOSMART">
</renderKits>
<!-- remaining boconfig -->
```

## Eclipse

Until the XEO Studio plugin is updated to support the new version you have to manually set somethings, namely the templates for the Scaffolding tool.

For each project you want to use with XEO 4 you may want to:
- Right Click the Project in Eclipse (Project Explorer)
- Find the "XEO Studio" options section
- Select the "Default Beans" option and replace the beans with the ones described in the previous section.

EditBean     -> netgest.bo.xwc.xeo.controllers.beans.EditBean
ListBean     -> netgest.bo.xwc.xeo.controllers.beans.ListBean
LookupBean   -> netgest.bo.xwc.xeo.controllers.beans.LookupBean
MainBean     -> netgest.bo.xwc.xeo.controllers.MainBean
Main with Regions Bean -> netgest.bo.xwc.xeo.controllers.MainBean
BaseBean     -> netgest.bo.xwc.framework.controllers.ApplicationBaseBean

You also want to add the "transactionLess='true'" to the List and Main viewer templates.
