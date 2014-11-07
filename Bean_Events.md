# Bean Events

Beans now have their own set of events and they are extensible, which means you can create your own set of events.

##The init event
The init event is a special event, which is called only once when a new Scene is added to the story. The purpose of this event is to allow you to have a point of entry in the bean. When a scene is created, parameters are passed to the bean and when the scene is added to the story, its **init** method is called. Note that this works for beans that extend the **ApplicationBaseBean** class

```java
public class MyBean extends ApplicationBaseBean{

	private String customValue;
    
    public void setCustomValue(String val){
    	this.customValue = val;
    }

	public void init(){
    	//Do something with any value that was passed before
        if (this.customValue != null){
        	this.customValue = this.customValue.toUpperCase();
        }
        
    }
}
```

When creating a new scene, you could do the following.
```java
	Scene scene = new Scene("/path/to/viewer.xvw");
    MyBean bean = (MYBean) scene.getBean();
    bean.setCustomValue("Some Value");
    renderScene(scene);

```



## Default events

Every bean now has the ability to trigger events and accept event listeners to react to those events. The default beans (List,Edit and Lookup) already provide some built-in events that you can react to. 

Events may be "canceable" which means that when a handler cancels the event the remaining code will not be executed (think an onBeforeSaveEvent which you may want to prevent the save from ocurring). Typically on "*Before*" events are canceable.


Check the following list to see which events are available in each bean

#### List Bean Events
- **OnAdd** (triggered when the "Add New" button is clicked)
- **OnRowDoubleClick** (triggered when a user opens an edit viewer from the list )

#### Edit Bean Events

Events on the instance (orphan)

- **BeforeSave** (triggered when the user clicks the save button, before the instance is saved)
- **AfterSave** (triggered after the instance is saved)
- **BeforeDestroy** (triggered when the user clicks the destroy button, before the instance is destroyed)
- **AfterDestroy** (triggered after the instance is destroyed)
- **AfterSaveParent** (triggered after the save event when the instance was created from a lookup's "addNew" button)

Event on the instance (non-orphan)

- **BeforeConfirm** (triggered before the Confirm action is executed - user clicked the **Confirm** button)
- **AfterConfirm** (triggered after the Confirm action is executed)
- **BeforeCancel** (triggered before the Cancel action is executed - user clicked the **Cancel** button)
- **AfterCancel** (triggered after the Cancel action is executed)

Events on attributes of the instance

- **OnOpenLookup** (triggered when a user clicks on a lookup button)
- **OnCardIdLinkOpen** (triggered when a user clicks the cardIdLink on a lookup)
- **BeforeAddBridge** (triggered when the user clicks the "Add" button in a bridge)
- **AfterAddBridge** (triggered after the AddBridge event)
- **BeforeRemoveBridge** (triggered before the Remove instance from bridge action is executed)
- **AfterRemoveBridge**  (triggered after the Remove instance from bridge action is executed)
- **BeforeAddNewBridge** (triggered before the Add New instance to bridge action is executed)
- **AfterAddNewBridge** (triggered after the add new instance to bridge action is executed)
- **BeforeEditBridge** (triggered before the edit instance in bridge action is executed)
- **AfterEditBridge** (triggered after the edit instance in brige action is executed)


#### Lookup Bean Events

- **BeforeConfirm** (triggered before the confirmation of a selection, the user clicked the **Confirm** button)
- **AfterConfirm** (triggered after the confirmation of a selection)
- **BeforeCancel** (triggered before the cancel action - when the user clicked the **Cancel** button)
- **AfterCancel**(triggered after the cancel action)
- **BeforeAddNew** (triggered when the user clicks on the **AddNew** button of the toolbar, before the action executes)
- **AfterAddNew** (triggered after the add new action executes)

## Custom events

You can also create your own events and trigger them whenever some action occurs. Check the Handling Events section.

## Handling an Event

This section explaing how to handl BeanEvents

#### The Event

There's an event class declared in the ApplicationBaseBean which declares an event as having two properties
- Type
- Tag

The type property is the event's name (BeforeSave, AfterConfirm, etc..)
The tag property is a value associated to the event, not all events have this value. This property is usefull to distinguish between equals events of different sources (think the LookupEvent which can be triggered from different attributes in the same viewer).


#### Creating an event handler 

To create an event handler you to need to implement the following interface

```java
public interface EventHandler<T extends ApplicationBaseBean> {
	
	public void handle(BeanEvent<T> event);

	public boolean handlesEvent(Event event);
	
}
```
As an example, check the implementation for a BeforeSaveEvent:

```java
public class MyEventHandler implements EventHandler<EditBean> {

			@Override
			public void handle( BeanEvent< EditBean > event ) {
				Object instanceBoui = event.getValue();
				Demo demo = DemoFactory.get().load( Long.parseLong( instanceBoui.toString() ) );
				demo.setAge( demo.getAge() + 1);
			}

			@Override
			public boolean handlesEvent(
					netgest.bo.xwc.framework.controllers.ApplicationBaseBean.Event event ) {
				boolean isBeforeSave = EditBean.Event.BEFORE_SAVE.getType().equals( event.getType() );
				return isBeforeSave;
			}
		};
```

In an EventHandlet's handler method you receive an instance of a BeanEvent, the BeanEvent is an interface with the following definition:

```java
public interface BeanEvent<T extends ApplicationBaseBean> {
	
	public Event getType();
	
	public abstract T getSource();
	
	public boolean wasCanceled();
	
	public void cancelEvent();
	
	public Object getValue();
	
	
}
```

- The **getType** method returns the event's type (name)
- The **getSource** method returns the bean where event was triggered (the source)
- The **wasCanceled** method returns whether this event was cancelled or not
- The **cancelEvent** methods sets the event as canceled (after this the wasCanceled method will return true)
- The **getValue** method returns the value associated to the event (if was one passed) or null if no value was passed.



#### Registering an event handler

Lets for instance register the previously created class as an event handler for the before save event in the bean's init method.

```java
public class MyBean extends EditBean {

	@Override
	public void init(){
    	super.init();
    	MyEventHandler handler = new MyEventHandler();
        addListener( EditBean.Event.BEFORE_SAVE , handler );
    }

}
```






#### Creating and Triggering a custom event

You can create a new Event by extending the ApplicationBaseBean.Evet class, like the following:

```java
public static class Event extends ApplicationBaseBean.Event{

		public Event(String name) {
			super(name);
		}
        
        public Event(String name, String tag){
			super(name,tag);
		}
        
        @CanceableEvent
		public static final Event CUSTOM_EVENT = new Event("My.CustomEvent");
}        

```

Which can now be used in any action, like the following:

```java

public void myAction(){
	//SOME JAVA CODE

	BeanEvent<EditBean> customEvent = createEvent(Event.CUSTOM_EVENT, SOME_VALUE );
		fireEvent(customEvent);
		if (!customEvent.wasCanceled()){
    		//MORE CODE    
        }

	

}

```
