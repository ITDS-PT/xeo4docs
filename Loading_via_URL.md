# Loading a viewer via URL

If you need to enable access to a viewer through a URL

## Security considerations

The biggest issue with loading viewers from URL is the possibility of people passing in parameters that you do don't expect and can cause security problems. You must be **extremelly** carefull when allowing URL parameters to be passed to a viewer.

To help with the security issues, you can use the following method to encrypt parameter values

```java
String encrypted = ParameterSecurity.encrypt("originalValue");
```

If you need to decrypt it, do the oposite:

```java
String decrypted = ParameterSecurity.decrypt("THE_ENCRYPTED_VALUE");
```


## Parameters 
You are responsible for dealing with the parameters, you can do the following in your bean to receive a value throug an url

```java
private String param;

@XUIWebParameter(name=PARAMETER,defaultValue="")
    public void setParam(String param){
	   if (StringUtils.hasValue( param )){
	  	 this.param = param;
	  }
	}

```

If your parameter was sent encrypted you would do the following:
```java
private String param;

@XUIWebParameter(name=PARAMETER,defaultValue="")
    public void setParam(String param){
	   if (StringUtils.hasValue( param )){
	  	 this.param = ParameterSecurity.decrypt(param);
	  }
	}

```

## Enabling loading via URL

You need to override the **canLoadFromUrl** method that is available in the ApplicationBaseBean class, like the following:

```java
@Override
public boolean canLoadFromUrl(){
	return true;
}

```

You also need to generate the URL for this particular viewer, the recommended way to do so is in the init method (See Events) where you pass the set of parameters you want to put in the URL through the **setUrlForNavigation** method

```java

private String param1;
private String param2;

@Override
public void init(){
	super.init();
    
    //Do something with the parameters
    if (this.param1 != null){
    	
    }
    if (this.param)
    
    //Enable the URL
    Map<String,String> parameters = new HashMap<String,String>();
    parameters.put("paramName1","paramValue1")
    parameters.put("paramName2","paramValue2")
    super.setUrlForNavigation( parameters );
}

@XUIWebParameter(name=paramName1,defaultValue="")
public void setParam1(String param1){
 	 if (StringUtils.hasValue( param1 )){
   		 this.param1 = ParameterSecurity.decrypt( param1 );
   	 }
}

@XUIWebParameter(name=paramName2,defaultValue="")
public void setParam2(String param2){
 	 if (StringUtils.hasValue( param2 )){
   		 this.param2 = ParameterSecurity.decrypt( param2 );
   	 }
}

```








