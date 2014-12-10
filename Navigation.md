# Navigation

The navigation in XEO V3.3 is done through the Story/Scene paradigm as explained in the [New Concepts](New_Concepts) page. There are two ways to navigate to new scenes using the API
- Adding a new scene to the story directly
- Executing an existing script (there are some variants to this)

## Creating a new scene and adding it to the story

Best start with an example (assuming you the context is a bean that extends the **ApplicationBaseBean**):

```java
    Scene newScene = new Scene( "path/viewer.xvw" );
    renderScene( newScene );
```

And what if you need to pass some parameters to the viewer/bean?

```java
	Scene newScene = new Scene( "path/viewer.xvw" );
    MyBean bean = (MyBean) newScene.getBean();
    bean.setParam1("someValue");
    bean.setParam2("someOtherValue");
    renderScene( newScene );
```

## Closing the current Scene

If you want to close the current active scene you just have to do the following:

```java

	public class MyBean extends ApplicationBean{
    	public void myMethod(){
        	super.close();
        }
    }

```

## Replacing the current Scene

If you need to replace a scene with another scene, you can do the following:

```java

	public class MyBean extends ApplicationBean{
    	public void myMethod(){
        	Scene newScene = new Scene("Path/to/viewer.xvw");
        	super.replaceCurrentScene(newScene);
        }
    }

```

## Executing a script

The other option when navigation is executing a Script. Scripts are basically a series of steps that culminate in the creation of Scene. For default scripts you can execute them directly, customize their parameters or extend them, let's see examples of them (again in the context of an ApplicationBaseBean).

##### Executing a script directly
```java
OpenEditViewerScript script = new OpenEditViewerScript( SOME_BOUI );
executeScript( script );
```

#### Customizing a script

Each default script provides a built-in "builder" method which provies a fluent interface for all the parameters it can receive.

```java
OpenEditViewerScript script = OpenEditViewerScript
.builder( SOME_BOUI )
.viewer( "path/to/viewer.xvw" ).build();
executeScript( script );

```

#### Extending a script

Every step of the default scripts is a protected method that you can override to your need, like in the following example:

```java
public class MyOpenViewer extends OpenEditViewerScript{

    public MyOpenViewer( long boui ) {
        super( boui );
    }

    @Override
    protected String getViewToOpen( boObject object ) {
        return "myViewer.xvw";
    }
    
    @Override
    protected void initializeBean(ApplicationBaseBean bean, boobject object){
    	super.initializeBean(bean,object);
        object.setDisabled();
    }

}
```
And then
```java
MyOpenViewer script = new MyOpenViewer( SOME_BOUI );
executeScript( script );
```

Check the javadocs for each of the scripts to check which methods you can override.

#### Implementing a Script

The Script concept is materialized by an Interface, the *netgest.bo.xwc.framework.controllers.Script* which has a single method:

```java
public interface Script {

	/**
	 * Create the scene
	 * 
	 * @return A new scene to render
	 */
	public Scene buildScene();
	
}

```

You have two options when returning a Scene. You want to add a new scene to the story, or you don't. For instance, the SaveScript does not return a new scene (it does not navigate anywhere), as such you can return the special **Null Scene** like follows:

```java
	@Override
	public Scene buildScene(){
		//Code for the Script
		return Scene.NULL_SCENE;
	}
```

## Existing Scripts

All Scripts are in the **netgest.bo.xwc.xeo.controllers.scripts** package, you have the following scripts:

- AddNewScript
- AddNewToBridgeScript 
- AddToBridgeScript
- CardIdLinkScript
- EditBridgeScript
- LookupCreateNewScript
- OpenEditViewerScript
- OpenListViewerScript
- SaveScript

Each script corresponds to a standard action performed by the framework. Each Script also requires different parameters to be executed. We'll show a list of examples bellow:

#### Open a List Viewer (with specific BOQL query)

```java

public class MyBean extends ApplicationBaseBean {

	public void openListViewer(){
		executeScript( OpenListViewerScript.builder("Ebo_Perf").boql("select EBo_Perf where username like 'P%' ").build() );
	}

}

```
#### Open an Edit Viewer

```java

	public void openEditViewer(){
    	long boui = 12345;
        executeScript( OpenEditViewerScript.builder( boui ).viewer( "path/to/viewer" ).build() );
    }

```
#### Add a new instance to a bridge

```java

	public void addToBridge(){
    	long bridgeOwner = 1234;
		bridgeHandler bridge = XEO.load( bridgeOwner ).getBridge( "bridgeName" );
		executeScript( AddToBridgeScript.builder( bridgeOwner , bridge ).boql( "select Ebo_Perf where state = '1' " ).build() );
    }

```
