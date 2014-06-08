***Conversion Process from Objective-C syntax to Swift***
The most important first step is to run Apple's "Convert to Modern Objective-C Syntax" refactoring, so that you're using array/dictionary literals and bracket-accesses; these will then be usable in Swift. Note also that I'm a beginner in Swift, so my apologies for any mistakes or incompleteness here.

|When you see this pattern | Replace with this |
|:----------------------------------------------------:   |:--------------------------------------: |
|**Module**                                             ||
|`@interface` *newType* : *superType* <*protocol1*, *protocol2*>   | `class` *newType* : *superType*, *protocol1*, *protocol2* |
|`@implementation` OR `@synthesize` OR `@end` | delete ||
|**Properties**                                             ||
|`property(…) TypeName * varName; 	                          ` | `var varName:TypeName                                           `|
|`property (readonly...) TypeName * varName; 	              ` | `let varName:TypeName                                           `|
|`property(…) TypeName * IBOutlet varName; 	                   `|`@IBOutlet var varName:TypeName                                           `|
|` _property`|`self.property`|
|**Compiler Directives**                                          ||
|`#define macroName value 	   `|`    	let macroName  = value`|
|'#import module.h`|Include in `...-Bridging-Header.h` <br> Delete for project modules`|
|`#if value ... #else ...#end  	   `|` #if value ... #else ...#end `|
|`#elif value`|`#elseif value`|
|`#define / #ifdef / #ifndef`| N/A|
|`#pragma mark sectionName`| `// MARK: sectionName` (not implemented yet)|
|`NSAssert(conditon,description)`|`assert(condition, description)`|
|**Types**                                            ||
|`NSString *`|`String`|
|`NSArray * arrayName = arrayValue`|`let arrayName: Array<TypeName> = arrayValue` OR `TypeName[]`|
|`NSDictionary *`|`Dictionary`|
|`NSMutableArray OR NSMutableDictionary ...`|` var arrayName...`|
|`id` | `AnyObject`|
|`TypeName *`|`TypeName`|
|`int8/16/32/64` OR `uint` OR <br>float OR double|Titlecase TypeName, e.g. UInt32 or Double|
|`NSInteger` OR `NSUInteger`|`Int` OR `UInt`|
|**Method Definitions**                                              ||
|`-(void) methodName  	   `|`    	func methodName()`|
|`-(TypeName) methodName 	   `|`    	func methodName() -> TypeName `|
|`-(IBAction) methodName 	   `|`   	@IBAction func methodName`| 
|`+(TypeName) methodName 	   `|`    	class func methodName() -> TypeName `|
|`...methodName: (TypeName1) param1 b: (TypeName2) param2  `|`   ...methodName(param: TypeName1  b param2: TypeName2)`|
|method overriden from superclass | add `override` |
|**Variables**                                              ||
|`TypeName varName = value`|`var (OR let) name = value` OR <br>`var (OR let) name: TypeName` if necessary|
|**Object Creation**                                            ||
|`TypeName * varName = [[TypeName alloc] init]    `|`   varName = TypeName()`|
|`[[TypeName alloc] initWithA:  value1 B: value2]    `|`    TypeName(a: value1, b: value2)`|
|`[TypeName TypeNameWithA: value]    `|`    TypeName(a: value)`|
|**Statements**                                            ||
|`break` in `switch` statements| optionally remove (except for empty cases),<br> but add `fallthrough` where needed|
|`if/while (expr)` | `if/while expr`, optional, but expr must be a boolean
|`for ( ... )` | `for ...`, optional

|**Method Calls**                                            ||
|`[object method]    `|`    object.method()`|
|`[object method: param1 b: param2 …]    `|`    object.method(param1, b: param2, …)`|
|**Expressions**                                            ||
|`YES   `|`    true`|
|`NO   `|`    false`|
|`(TypeName) value` to recast | `value as TypeName` OR TypeName(value)|
|`stringName.length`|`stringName.utf16` OR `stringName.countElements`|
|`stringName isEqualToString: string2Name`|`stringName == string2Name`|
|`NSString stringWithFormat@"...%@..%d",obj,int)`|`"...\(obj)...\(int)"`
|**Miscellaneous** ||
|semicolons at end of line | delete (optional) |
| @ for literals| delete|
[Manual conversion of Objective-C to Swift][section-mmd-tables-table1] 

Beyond those mostly syntactical conversions, you'll also need to do more semantic-based conversions:
o Handle all cases in `switch` statements (probably by adding a default case) 
o Add overrides for all superclass `init`s
o Change the definition for any variables that are only set once to `let` versus `var`
o Move getters/setters into `set/get` blocks in property definitions; in set, change name for the incoming value to `newValue`
o Biggest change is handling of nils and Optionals. Normal variables can no longer be assigned nil, only Optional ones. This should get rid of a vast array of program errors. On the other hand, all results from Cocoa methods are defined as Optionals with auto-unwrap. This means that the compile will NOT warn you that the returned values are handled incorrectly, leading to a whole new set of potential problems.
o For every property, either (a) assign initial value, or (b) add to init call or (c) make Optional by adding ? to type
o In initializers, you must set your own non-optional properties before calling super.init