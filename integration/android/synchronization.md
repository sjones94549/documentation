# Synchronization

The Layer client provides a flexible notification system for informing applications when changes have occured on Layer objects in response to synchronization. The system is designed to be general purpose and alerts your application to the creation, update, or deletion of an object. Changes are modeled as simple dictionaries with a fixed key space.

##Event Listener
The Layer SDK leverages listeners to notifiy your application when changes occur. Your application should register as a `LayerChangeEventListener` in order to recieve change notifications.

```java
public class MyApplication extends Application implements LayerChangeEventListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        LayerClient client = LayerClient.newClient(this, "%%C-INLINE-APPID%%", "GCM ID");
        client.registerEventListener(this);
        
    }
}
```

Upon receipt of a call to the `onMainThreadChanges()` method, your application can obtain a reference to all changes by calling `getChanges()` on the event object. This will return an array value wherein each element describes a specific change to a conversation or message object.

```java
public void onEventMainThread(LayerChangeEvent event) {
	List<Change> changes = event.getChanges();
}        
```

Change notifications occur for both `Message` and `Conversation` objects. Your application can retrieve the type of object upon which a change has occured by calling getObjectType() on the change object.

```java
switch (change.getObjectType()) {
     case CONVERSATION:
     // Object is a conversation
     break;
     
     case MESSAGE:
     // Object is a message
     break;
}
break;
```

Layer Change notifications will alert your application to object creation (insert), update and deletion events. In order to acquire the specific type of change, your application can call getChangeType() on the change object.

``` java
switch (change.getChangeType()) {
	case INSERT:
	// Object was created
	break;
	
	case UPDATE:
	// Object was update
	break;
	
	case DELETE:
	// Object was deleted
	break;
}
break;
```

Your application can acquire the actual object on which an update has occured by retrieve the LYRObjectChangeObjectKey key from the change object.

```java
id changeObject = change.getObject();
```

##Synchronization Listener
The Layer SDK also provides a synchronization listener that alerts your application when a sychronization is about to begin, and when a syncrhonization has succesfully completed. Your application should register as a `LayerSyncListener` to recieve these call backs. 

```java
public class MyApplication extends Application implements LayerSyncListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        LayerClient client = LayerClient.newClient(this, "%%C-INLINE-APPID%%", "GCM ID");
        client.registerSyncListener(this);
        
    }
    
    public void onBeforeSync(LayerClient client) {
    	// LayerClient is starting synchronization
    }
        
    public void onAfterSync(LayerClient client) {
    	// LayerClient has finshed synchronization
    }
}
```
