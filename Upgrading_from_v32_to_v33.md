# Upgrading from XEO v3.2 to v3.3

There are some important concepts that have changed from version 3.2 to 3.3 and full backward-compatability is not possible. This document **is not to explain how to convert a project in XEO V3.2 to XEO V3.3**, but instead to show you what changes in the way you do things.

## Viewer definition
XEO V3 Viewer definitions are mostly compatible with V4, with the following exceptions

- If you use the xvw:form component in a viewer, you need to **remove** it (yes delete it)
- List viewers and Main viewers require an additional parameter in the xvw:viewer element, which is **transactionLess='true'**
- The defaults beans are no longer the XEOBaseBean, XEOEditBean, XEOBaseList and XEOBaseLookupList (more on that in the beans section)
- The Main viewer definition is not compatible with the new version (more on that bellow)

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

## Beans

Since version 3.3 brings so many changes, we decided to create a new set of beans for the various types of situations.

- ApplicationBaseBean (for any viewer)
- MainBean (for Main Viewers)
- ListBean (for List viewers)
- EditBean (for Edit viewers)
- LookupBean (for Lookup viewers)

This means that you will have to change XEO Studio's default beans in the project's properties menu to have the scaffolding utility generate correct viewer definitions.