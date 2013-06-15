# Gson on Fire

This project implements some useful features missing from the extremely useful [Gson project](https://code.google.com/p/google-gson/)

There is no attempt to modify any existing behavior or existing class of Gson. In fact, this project depends on Gson.
The main objective is to extend Gson using TypeAdapter and TypeAdapterFactory instances to support more features.

## Features

All of the features can be accessed by the ```GsonFireBuilder```. This class will build internally a ```GsonBuilder```
with all the desired features from which a ```Gson``` instance can be obtained.

### Post Processors

Ability to alter a JsonElement after it has been generated from an object.

```java
 GsonFireBuilder builder = new GsonFireBuilder()
    .registerPostProcessor(SomeClass.class, new PostProcessor<A>() {
        @Override
        public void postDeserialize(SomeClass result, JsonElement src) {
            //Here you can add logic to change the SomeClass instance
        }

        @Override
        public void postSerialize(JsonElement result, SomeClass src) {
            //Here you can inject new fields into the result JsonElement
        }
    });
Gson gson = builder.createGson();
```

### Type Selectors

Useful when you have a base class and some sub-classes and you need to de-serialize some json into one of the sub-classes.
The sub-class type will be determined by a field on the json.

```java
GsonFireBuilder builder = new GsonFireBuilder()
    .registerTypeSelector(Base.class, new TypeSelector<Base>() {
        @Override
        public Class<? extends Base> getClassForElement(JsonElement readElement) {
            String kind = readElement.getAsJsonObject().get("kind").getAsString();
            if(kind.equals("a")){
                return A.class; //This will cause Gson to deserialize the json mapping to A
            } else if(kind.equals("b")) {
                return B.class; //This will cause Gson to deserialize the json mapping to B
            } else {
                return null; //returning null will trigger Gson's default behavior
            }
        }
    });
Gson gson = builder.createGson();
```

### Expose your metods

You can annotate methods now to be automatically evaluated an serialized

```java

public void SomeClass{

    @ExposeMethodResult("name")
    public String getName(){
        return "a";
    }

}

//Then
GsonFireBuilder builder = new GsonFireBuilder()
    .enableExposeMethodResult(); //This will make Gson to evaluate and
                                 //serialize all methods annotated with @ExposeMethodResult

```

### Date format

Dates can be serialized either to a unix epoch with or without milliseconds.

```java
GsonFireBuilder builder = new GsonFireBuilder();

//then
builder.dateSerializationPolicy(DateSerializationPolicy.unixTimeMillis);
//or
builder.dateSerializationPolicy(DateSerializationPolicy.unixTimeSeconds);

```

If no policy is specified, the default Gson behavior will be used

### Hooks

Some special annotated methods will be invoked in your java object before serializing
and after deserializing

```java

public void SomeClass{

    @PreSerialize
    public void preSerializeLogic(){
        //this method will get invoked just before
        //the class is serialized to gson
    }

    @PostDeserialize
    public void postDeserializeLogic(){
        //this method will get invoked just after
        //the class is instantiated from a json

        //NOTE: this gets invoked before the PostProcessors
    }

}

//Later

Gson builder = new GsonFireBuilder()
    .enableHooks(SomeClass.class);

Gson gson = builder.createGson();
```

## Using the Gson's GsonBuilder

You can still access the good old GsonBuilder:

```java
GsonFireBuilder fireBuilder = new GsonFireBuilder();
GsonBuilder gsonBuilder = fireBuilder.getGsonBuilder();
```

## Usage
Add to your ```pom.xml```

```xml
<repositories>
    <repository>
        <id>julman99-github</id>
        <url>https://raw.github.com/julman99/mvn-repo/master</url>
    </repository>
<repositories>
<dependencies>
    <dependency>
        <groupId>com.github.julman99</groupId>
        <artifactId>gson-fire</artifactId>
        <version>0.4</version>
    </dependency>
</dependencies>
```