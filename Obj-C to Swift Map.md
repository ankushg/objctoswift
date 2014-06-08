***How to Port an Objective-C module in a project to Swift***

0)	Download a copy of the project [here] (https://developer.apple.com/library/ios/samplecode/iPhoneCoreDataRecipes/iPhoneCoreDataRecipes.zip) and open Recipes.xcodeproj in Xcode version 6.

1)	Choose `File>New File…>iOS Source>Swift File> IngredientDetailViewController` (Folder: Classes, Group: Recipe View Controllers)

2)	Reply Yes to “Would you like to configure an Objective-C bridging header?”

3)	Copy the first three lines below from `Recipes_Prefix.pch`  and the next three from `IngredientDetailViewController.m` into `Recipes-Bridging-Header.h`. If you do further files, obviously don't duplicate lines, and remove any files that you've converted to Swift. I haven't found any where that documents the need for the Cocoa lines, given that they're imported in the swift file, but 
````
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
#import <CoreData/CoreData.h>
#import "Recipe.h"
#import "Ingredient.h"
#import "EditingTableViewCell.h"
````

4) Copy/paste the text from both the `IngredientDetailViewController.h` file and the `IngredientDetailViewController.m` files into `IngredientDetailViewController.swift`. 

5) Delete both `IngredientDetailViewController.h` and `.m` files from project.

6) Do a global Find-and-Replace from `#import "IngredientDetailViewController.h"` to `#import "Recipes-Swift.h"` (Only one conversion in this case, and again for further files, don't duplicate this line in your Objective-C modules.)

7) Check the Project>Targets>Recipes>Build Settings `Runpath Search Paths`. If it shows `$(inherited)`, remove this line or you'll get an error on launch about "no image found"

8) Convert Objective-C syntax in `IngredientDetailViewController.swift` to Swift. See below for some of the substitutions required, or way below for my converted version.

9) You may need to update the IB links. Do a Find>Find in Files on `IngredientDetailViewController` and select the one in Interface Builder. Open the Identity Inspector in the right-hand column. Select `IngredientDetailViewController` in the Class field, type `xxx` or something and tab.

10) Build and Run. Note that after going into a recipe, you must tap Edit and then the info button of an ingredient to activate `IngredientDetailViewController`

12) Congrats on building your first mixed Swift/Objective-C program!

***Conversion Process from Objective-C syntax to Swift***
The most important first step is to run Apple's "Convert to Modern Objective-C Syntax" refactoring, so that you're using array/dictionary literals and bracket-accesses; these will then be usable in Swift. Note also that I'm a beginner in Swift, so my apologies for any mistakes or incompleteness here.

***Manual conversion of Objective-C to Swift***
|When you see this pattern | Replace with this |
|:----------------------------------------------------:   |:--------------------------------------: |
|**Module**                                             ||
|`@interface *newType* : *superType* <*protocol1*, *protocol2*>`   | `class *newType* : *superType*, *protocol1*, *protocol2*` |
|`@implementation` OR `@synthesize` OR `@end` | \<delete\> ||
|**Properties**                                             ||
|`property(…) TypeName * varName; 	                          ` | `var varName:TypeName                                           `|
|`property (readonly...) TypeName * varName; 	              ` | `let varName:TypeName                                           `|
|`property(…) TypeName * IBOutlet varName; 	                   `|`@IBOutlet var varName:TypeName                                           `|
|` _property`|`self.property`|
|**Compiler Directives**                                          ||
|`#define macroName value 	   `|`    	let macroName  = value`|
|`#import module.h`|Include in `...-Bridging-Header.h` <br> Delete for project modules`|
|`#define / #ifdef / #ifndef`| N/A|
|`#if value ... #else ...#end  	   `|` #if value ... #else ...#end `|
|`#elif value`|`#elseif value`|
|`#pragma mark sectionName`| `// MARK: sectionName` (not implemented yet)|
|`NSAssert(conditon,description)`|`assert(condition, description)`|
|**Types**                                            ||
|`NSString *`|`String`|
|`NSArray * arrayName = arrayValue`|`let arrayName: Array<TypeName> = arrayValue` OR <br>`let arrayName: TypeName[] = arrayValue`|
|`NSDictionary *`|`Dictionary`|
|`NSMutableArray OR NSMutableDictionary ...`|` var arrayName...`|
|`id` | `AnyObject`|
|`TypeName *`|`TypeName`|
|c types, e.g. `uint32` OR `float` |Titlecase , e.g. `UInt32` or `Float`|
|`NSInteger` OR `NSUInteger`|`Int` OR `UInt`|
|**Method Definitions**                                              ||
|`-(void) methodName  	   `|`    	func methodName()`|
|`-(TypeName) methodName 	   `|`    	func methodName() -> TypeName `|
|`-(IBAction) methodName 	   `|`   	@IBAction func methodName`| 
|`+(TypeName) methodName 	   `|`    	class func methodName() -> TypeName `|
|`...methodName: (Type1) param1 b: (Type2) param2  `|`   ...methodName(param: Type1  b param2: Typ2)`|
|method overriden from superclass | add `override` |
|**Variables**                                              ||
|`TypeName varName = value`|`var (OR let) name = value` OR <br>`var (OR let) name: TypeName` if necessary|
|**Object Creation**                                            ||
|`TypeName * varName = [[TypeName alloc] init]    `|`   varName = TypeName()`|
|`[[TypeName alloc] initWithA:  value1 B: value2]    `|`    TypeName(a: value1, b: value2)`|
|`[TypeName TypeNameWithA: value]    `|`    TypeName(a: value)`|
|**Statements**                                            ||
|`break` in `switch` statements| not necessary, except for empty cases,<br> but add `fallthrough` where needed|
|`if/while (expr)` | `if/while expr`, parentheses optional,<br> but expr must now be a boolean
|`for ( ... )` | `for ...`, optional|
|**Method Calls**                                            ||
|`[object method]    `|`    object.method()`|
|`[object method: param1 b: param2 …]    `|`    object.method(param1, b: param2, …)`|
|**Expressions**                                            ||
|`YES   `|`    true`|
|`NO   `|`    false`|
|`(TypeName) value` to recast | `value as TypeName` OR `TypeName(value)`|
|`stringName.length`|`stringName.utf16` OR `stringName.countElements`|
|`stringName isEqualToString: string2Name`|`stringName == string2Name`|
|`NSString stringWithFormat@"...%@..%d",obj,int)`|`"...\(obj)...\(int)"`
|**Miscellaneous** ||
|semicolons at end of line | delete (optional) |
| @ for literals| delete|

Beyond those mostly syntactical conversions, you'll also need to do more semantic-based conversions:

* Handle all cases in `switch` statements (probably by adding a default case) 
o Add overrides for all superclass `init`s
* Change the definition for any variables that are only set once to `let` versus `var`
* Move getters/setters into `set/get` blocks in property definitions; in set, change name for the incoming value to `newValue`
* Biggest change is handling of nils and Optionals. Normal variables can no longer be assigned nil, only Optional ones. This should get rid of a vast array of program errors. On the other hand, all results from Cocoa methods are defined as Optionals with auto-unwrap. This means that the compile will NOT warn you that the returned values are handled incorrectly, leading to a whole new set of potential problems.
* For every property, either (a) assign initial value, or (b) add to init call or (c) make Optional by adding ? to type
* In initializers, you must set your own non-optional properties before calling super.init


You can see why I hope (and expect) Apple will provide a refactor; also almost all of these conversions would be relatively easy to do, if one has the clang compiler's abstract syntax tree.

Here's my cut at this particular module:

````

`````
