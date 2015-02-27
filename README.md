# TypeScript Type Generator
TypeScript type generator is a simple library that generates TypeScript interfaces, classes and enums based on .NET types. It does support generics, inheritance, and also it could handle string/number based dictionaries as maps.

The binaries are available through the nuget feed - [TypeScript.NET](http://www.nuget.org/packages/TypeScript.NET/ "TypeScript.NET").

## Usage scenario

The main usage scenario is to mirror to TypeScript world data objects defined on the .NET server side, this is why the generator does not handle any methods/properties with a body. Although it supports auto-properties and sets default values for primitive types such as number, boolean, string and enums.

For example, based on the following C# type declaration:
```csharp
  public class DerivedClass<T1, T2> : BaseClass, IGenericInterface<int, T1>, 	ISimpleInterface 
  {
    public IDictionary<int, T1> Map { get; set; }
    
    public ICollection<T2> GenericData { get; set; }
    
    public string Code { get; set; }
    
    public DerivedClass()
    {
      this.Code = "The Code";
    }
  }
```
the generator produces the following TypeScript definition:

```typescript
    export class DerivedClass<T1, T2> extends BaseClass implements IGenericInterface<number, T1>, ISimpleInterface {
	    Map: Common.IMapOfNumber<T1>;
	    GenericData: T2[];
	    Code: string;
        
	    constructor() {
		    super();
		    this.Map = {};
		    this.GenericData = [];
		    this.Code = "The Code";
	    }
    }
```


## Usage

There are currently two aprroaches to generate TypeScript files:
- Imperatively, using *TypeScriptGenerator* class 
- Using MsBuild task

In both cases you need to pass to generator entity information about the source types to be used, destination location for TypeScript code and overrides for default configuration.

### Generate from code

In order to generate TypeScript code from your application, you need to reference [TypeScript.NET](http://www.nuget.org/packages/TypeScript.NET/ "TypeScript.NET")  nuget package.

You could generate TypeScript definition based on specific types:

```csharp

    var generator = new TypeScriptGenerator();
    
    var typeScriptCode = generator.GenerateFromTypes(
    	typeof(ISimpleInterface),
    	typeof(MyDataModel),
    	typeof(MyEnum));
``` 

...based on all types from specific assembly:

```csharp

    var generator = new TypeScriptGenerator();
    
    var typeScriptCode = generator.GenerateFromAssembly(
    	Assembly.FromFile("MyAssembly.dll"));
``` 

...or based on all types from specific assembly which are decorated by a special attribute:

```csharp

    var generator = new TypeScriptGenerator();
    
    var typeScriptCode = generator.GenerateFromAssembly<GenerateTypeScriptAttribute>(
    	Assembly.FromFile("MyAssembly.dll"));
``` 

And if you would want to override some generation behavior you need to pass configuration object into TypeScriptGenerator constructor. Refer to Configuration section if you want to know which parts of code generation are configurable.

### Generate using MsBuild task

You might want to use this version if you would want to produce new versions of TypeScript definitions during each build automatically. To do that you could use the dedicated MsBuild task that could be registered as an additional build target. 

### Configuration

This is the full list of parameters you could configure:

* **UseAssociativeArray** - Whatever use or not special logic to create an associated array interface for IDictionary<string, T>/IDictionary<*number type*, T> types.
* **StringMapInterfaceName** - Fullname of TypeScript interface for string based generic associate array.
* **NumberMapInterfaceName** - Fullname of TypeScript interface for number based generic associate array.
