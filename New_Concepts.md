# Concepts in XEO version 3.3

This page introduces the new concepts creating in version 3.3 of XEO, and their purpose.

The main goal behind version 3.3 was to create a new visual interface to make applications visualling more appealing but also to solve some issues with the previous version, such as:
- Navigation between viewers
- Responding to events in the visual layer
- Overriding the framework's defaults
- Customizing the look and feel of an application


## Application Container

The ApplicationContainer is a component (registered as *xvw:applicationContainer*) created with the purpose of being the container for content in an application. The main ideia is to place the component somewhere in your main page and do all navigation inside the application container. See the following snippet:

```xml
<xvw:viewer>
	<xvw:treePanel>
    	<!-- Menus -->
    </xvw:treePanel>
	<!-- Content -->
    <xvw:applicationContainer />
    <!-- More Content -->
</xvw:viewer>
```

The viewer container represents a Story (see bellow)

## Story

A Story is an ordered collection of Scenes, each scene represents a view the user sees and interacts with (materialized by a *.xvw file*). A Story represents the path a user as taken to reach a certain view.

#### Scene

A Scene is an abstraction that contains a pair of viewer and bean inside a Story. When you need to show the user something, you create a new scene and push it to the existing story.

#### SceneCrumb 

Since a story represents a series of steps the users has taken to reach a certain point, that path can be displayed with by using the concept of a breadcrumb. With that goal, the *xvw:sceneCrumb* component was created. You just need to add it to your main viewer and it will display the path the user has taken up to the current point. The SceneCrumb also allows the user to return to a previous step in his path.


## Scripts

A Script is essentially a series of steps that may return a new Scene (or special Empty Scene) to navigate to. This concept was added to encapsulate certain platform actions and to allow developers to create their own scripts. Several of the platform's built-in action were converted to scripts, including the following:
- Saving an instance
- Opening a lookup viewer
- Editing an existing instance from a list or collection attribute
- Adding a new instance from a list or collection attribute or lookup

## Bean Events

XEOMOdels have their own set of events (onBeforeSave, onAfterLoad, etc... ) but there was no counter part in the viewer layer. As such, beginning with version 4 you now have a set of events that you can register handlers to. Each bean default bean (the beans for edit/list/lookup) exposes as set of pre-defined events and you can also provide your own events. As an example, the LookupBean provides the following events:

- Before/After Confirm
- Before/After Cancel
- Before/After Add New

Events can also have an optional value and a tag associated. In the confirm example the value is the selected row from the grid. Tags are usually used to distinguish things from one another (think two different bridges in the same viewer).

Each bean that derives from the ApplicationBaseBean also gets an **init** event which is triggered after all parameters have been passed and just before the viewer is rendered to the user.

## Component References/Bindings

To make dealing with components easier, we introduced the concept of a component reference. If you create a bean with the following definition:

```java

@ComponentReference(componentClass=GridPanel.class)
public GridPanel grid;

public void action(){
	DataRecordConnector record = grid.getActiveRow();
	doSomething(record);
}

```

**Note : The "grid" variable must public.**

The component reference is handled automatically by XEO. XEO also unbinds the reference when the request ends. You can also search the reference by id.

Te viewer
```xml
	
    <!-- Content -->
    <xvw:gridPanel id='gridId'>
    	<xvw:columns>
        	<!-- Column definitions -->
        </xvw:columns>
    </xvw:gridPanel
    <!-- Content -->
	
```
And the bean
```java
	@ComponentReference(id="gridId")
	public GridPanel grid;
```


## Bootstrap

Not really a concept, but the XEO V4 visual layout is based on the popular [Bootstrap](http://www.getbootstrap.com) CSS framework. Which is one of the reasons why we get a more customizable layout.







