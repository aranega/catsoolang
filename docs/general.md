# CatsooLang Documentation (under construction...)

## What is CatsooLang?

CatsooLang is a little Domain Specific Langage (DSL) that allows you to handle models in a textual programming way into http://www.genmymodel.com. You can
handle very low level feature of your model. CatsooLang interpreter is evaluated on the client side only, there is no intensive communication with server
(there is some, we will come back on this later). Here is some details about the language:

* dynamic typed,
* adhoc polymorphic (function overloading) and type constraints/bounds,
* a little bit of functionnal programming (for collection handling, iteration...),
* looks a little bit like OCL and/or QVT,
* handles GenMyModel collaborative mode,
* still a WIP so, element deletion is not very-stable atm.

Each modification you perform on your model is propagated to other users and are stored in the cloud. In that case, your shell will communicate with the server to send the
modifications. This mode can be disable, but in that case, each modification will not be stored.

## The CatsooLang Interactive Console

The "CatsooLang Shell" is an interactive mode for "CatsooLang", this is not a UNIX-shell like but more a lua/python/ocaml/clojure (you name it) interactive console-like
which evaluates CatsooLang expressions.

## Useful Commands

Before jumping into code lines, here is some useful commands:

* `help`         --> displays help with the shortcuts you can use in the interactive shell
* `history`      --> displays all the commands you typed in the shell from the beginning of your session
* `#sandbox-on`  --> goes into safe mode (each model modification is performed locally only and are not saved)
* `#sandbox-off` --> goes back from safe mode to real-time mode (each model modification is saved)

Also, there is a special keyword you can call to get information about a function: `doc()`.
_e.g._,

```
cl> doc(objects);

1. objects : (obj:Object) -> Collection

      Returns all the objects contained in the object, including itself.
      @param obj the container object.
      @returns A collection of objects.

```

The result is the function definition the interpreter found. It gives the `objects` function signature: this function takes a model element object
(an `Object`) and returns a collection.

If there is no documentation (because not yet here) here is the output:

```
cl> doc(pow);

1. pow : (a:Real x b:Real) -> Real
* Sorry, there is no documentation or doc is empty
```

If the function does not exist:

```
cl> doc(foo);
Sorry, there is no 'foo' operation
```

Finally, if there is many signature for the same function (function overloading), the shell returns all the signature and documentation the interpreter find
for this function.

```
cl> doc(substring);

1. substring : (s:String x begin:Integer x end:Integer) -> String
* Sorry, there is no documentation or doc is empty

2. substring : (s:String x begin:Integer) -> String
* Sorry, there is no documentation or doc is empty
```

The interactive shell displays the function in the order they are register into the CatsooLang evaluation environement (we will see later what this implies).

## First Steps: Display and Accessing Model Element

The way CatsooLang display messages is using `log` keyword (this is not a function per say). In the interaction console, you can use `CTRL + i` shortcut to
quickly write `log();` and put the cursor between the `()`.

```
cl> log('foo');   // <-- strings use ' and not "
foo

cl> log(3);
3

cl> log([]);
[]

cl> log([3, 'R']);
[Integer: 3, String: R]

cl> var i := 33.3;
cl> log(i);
33.3

cl> log(__ROOT);
org.eclipse.uml2.uml.internal.impl.ModelImpl@6762 (name: model, visibility: <unset>) (URI: null) (viewpoint: <unset>)

cl> log(foo);    // <-- a non existing variable
null
```

**NOTE**

You can question variable and expression about their type using the `type()` function:
```
cl> type(3);
'Integer: 3

cl> type('R');
'String: R

cl> type(__ROOT);
'CL_EOBJECT(Model, model, 3798)

cl> type([3, __ROOT]);
'Collection: [Integer: 3, CL_EOBJECT(Model, model, 3886)]
```

As you can see, there is an already defined variable in CatsooLang evaluation environement called `__ROOT` that points to the root of your model.
We can use this variable to display a tree of our model. Here is an example on a simple model owning one class with three attributes:

```
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 31088) [packageImport]
|  +- (PackageImport, 31089) [packageImport]
|  +- Class 'UserEntity' [packagedElement]
|  |  +- Property 'ID' [ownedAttribute]
|  |  +- Property 'attribute' [ownedAttribute]
|  |  +- Property 'attribute2' [ownedAttribute]

```

This function give us a tree representation using a model element as starting point. Each line is built using the following format:
`object_type 'object_name' [feature_to_this_element_from_parent]`. So, if we look at the 4th line, we can see
that there is a `Class` named `UserEntity` contained by the `packagedElement` feature of the `Model` element named `model`.

How can we access the `UserEntity` element using CatsooLang? CatsooLang use model element name to navigate through them from the root of your model (a kind of path).
Each name must be separated by `::`, so in our example, we can access to class `UserEntity` by typing `model::UserEntity`. The beginning of a path can also be a variable
so, the expression `__ROOT::UserEntity` gives the same result.

```
cl> log(model);
org.eclipse.uml2.uml.internal.impl.ModelImpl@6762 (name: model, visibility: <unset>) (URI: null) (viewpoint: <unset>)

cl> log(model::UserEntity);
org.eclipse.uml2.uml.internal.impl.ClassImpl@7a1c (name: UserEntity, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)

cl> log(__ROOT::UserEntity);
org.eclipse.uml2.uml.internal.impl.ClassImpl@7a1c (name: UserEntity, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)

cl> var r := model;
cl> log(r::UserEntity);
org.eclipse.uml2.uml.internal.impl.ClassImpl@7a1c (name: UserEntity, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)
```

This way of accessing model element is just a quick access. You can 'manually' navigate to the desired object using model features. Here is the way we can access the same
element using features:

```
cl> log(model.packagedElement);
[CL_EOBJECT(Class, UserEntity, 32492)]  // <-- the result is a collection of model object with only one element

cl> log(model.packagedElement at(0));   // <-- we access the first element (collection index begin at 0)
org.eclipse.uml2.uml.internal.impl.ClassImpl@7a1c (name: UserEntity, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)
```

The function `at` allows you to get an element in a collection at a specific position (for a collection of n element, from 0 to n-1). Let's take a look to the function
signature:

```
cl> doc(at);

1. at : (collection:Collection x index:Integer) -> Any
* Sorry, there is no documentation or doc is empty    // <-- I know, shame on me
```

The signature tells us that this function takes a collection and an integer as parameter and return something (we will go back on types and typing system later). The important
thing here is the way the function call. There is two ways of calling a function in CatsooLang, either you give all parameters to the function in a `foo(bar1, bar2);` way
or you can use the 'sourced' call: `bar1 foor(bar2);`. The two calls are equivalent. So, our previous example could be rewritten `at(model.packagedElement, 0);`. The way you call
the function is totally up to you.

```
cl> log(at(model.packagedElement, 0));
org.eclipse.uml2.uml.internal.impl.ClassImpl@7a1c (name: UserEntity, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)

cl> log(cos(1.0));
0.5403023058681398

cl> log(1.0 cos());
0.5403023058681398

cl> log(size([1, 2, 3]));
3

cl> log([1, 2, 3] size());
3
```

## Variable definitions, Types and Typing System

CatsooLang is dynamically typed. It means, for example, that each element feature is accessed at evaluation time. If the feature does not exist, an error message is displayed but
the execution continue (no exception system for errors). If an error is displayed, the return result is `null`.

```
cl> log(__ROOT.name);
model

cl> log(__ROOT.foo);
Feature foo doest not exist for org.eclipse.emf.ecore.impl.EClassImpl@18b (name: Model) (instanceClassName: null) (abstract: false, interface: false)
null

cl> log(foo);
null

cl> log(foo := 5);
Variable foo does not exists.
null
```

You can define a variable using the `var` keyword, (_e.g._, `var i := 'R';`). A variable is typed at execution time and its type can change during its 'life'.
Each time, the variable type is deduced from the affectation expression execution (affectation operator is `:=`).

```
cl> var i := 'R';
cl> type('R');
'String: R

cl> i := 0;
cl> type(i);
'Integer: 0

cl> i := i toString();
cl> type(i);
'String: 0
```

If you want to get a listing of all the defined variable in the current evaluation context, the `_getvars()` function returns a collection
of the defined variables (we will see later how collections works):

```
cl> var i := 'R'; var a := true; var z := [];
cl> log(_getvars());
[__ROOT -> CL_EOBJECT(Model, model, 10702), i -> String: R, a -> Boolean: true, z -> Collection: []]

cl> log(_getvars() at(1) + 'est');
Rest
```


CatsooLang types are either basic types or types that depend on loaded metamodels. There is 8 basic types, here is their 'hierarchie':

```
                           +-----+
                           | Any |-------------------------------+
                           +-----+    (contains)                 |
                              ^                                  |
                              |                                  |
     +------------+-----------+-----------+---------------+      |
     |            |           |           |               |      |
+----+----+  +----+---+  +----+---+  +----+---+  +--------+---+  |
| Boolean |  | String |  | Number |  | Object |  | Collection |#-+
+---------+  +--------+  +--------+  +--------+  +------------+
                             ^
                             |
                        +----+-----+
                        |          |
                    +---+--+  +----+----+
                    | Real |  | Integer |
                    +------+  +---------+
```

* `Any` represents all the CatsooLang types,
* `Boolean` represents the bool type (constantes are `true` and `false`),
* `String` reprensents string type,
* `Number` is an abstract type that reprents either an int or a real,
* `Integer` represents integer,
* `Real` represents real,
* `Collection` represents collection of elements (can contain `Any` type of element),
* `Object` represents the higher level object for model elements (all model elements inherits from this type).

Every time a metamodel is loaded into the CatsooLang evaluation environment, each meta-object and meta-object instance will inherit from `Object`.
Actually, `Object` represents `EObject` from `Eclipse Modeling Framework` (in the example below, the output format gives the metaclass name, the
instance name (if there is any) and the instance id).

```
cl> type(uml::Class);  // <-- in UML context
'CL_EOBJECT(EClass, Class, 331)

cl> type(UserEntity); // <-- notice that the root is not needed in this path as the object UserEntity is directly contained by the root.
'CL_EOBJECT(Class, UserEntity, 3711)
```

## Model Navigation

As we previously seen, you can directly access to your model elements using paths or you can use meta-feature to navigate from an element to another.
You can use this way of accessing elements to query your model or get elements from a specific context. Here is some examples:

```
cl> log(UserEntity.ownedAttribute); // <-- Gets all the attributes of the UserEntity class
[CL_EOBJECT(Property, ID, 3596), CL_EOBJECT(Property, attribute, 3597), CL_EOBJECT(Property, attribute2, 3598)]

cl> log(UserEntity.ownedAttribute.name); // <-- Gets all the attribute names of the UserEntity class
[String: ID, String: attribute, String: attribute2]

cl> log(UserEntity.ownedAttribute.type.name); // <-- Gets all the type names of the UserEntity class attributes
[String: Integer, String: Long, String: Real]
```

As you can see, when accessing features that return collection (as `ownedAttribute`), accessing another feature iterates on each elements and produce a new
collection with the feature value (only if the collection element is an Object). In this case, it is actually syntactic sugar for a slightly more complex
expression (we will discuss them later). If the feature is a single value, you get the result as shown above, but if the feature is a collection, you will
get a collection of collections.

```
cl> log(__ROOT.packagedElement);
[CL_EOBJECT(Class, UserEntity, 5268)]  <-- this is a collection with a single element: UserEntity

cl> log(__ROOT.packagedElement.ownedAttribute);
[Collection: [CL_EOBJECT(Property, ID, 5357), CL_EOBJECT(Property, attribute, 5358), CL_EOBJECT(Property, attribute2, 5359)]] <-- a collection of collection

cl> log(__ROOT.packagedElement.ownedAttribute.ownedElement);
Obj Collection: [CL_EOBJECT(Property, ID, 5445), CL_EOBJECT(Property, attribute, 5446), CL_EOBJECT(Property, attribute2, 5447)] is not an EObject!
[]

cl> log(__ROOT.packagedElement.ownedAttribute flatten().ownedElement);
[Collection: [], Collection: [], Collection: []]
```

If you are lost in meta-features, CatsooLang defines a function that helps you finding what meta-feature you can use for a specific meta-class:

```
cl> introspect(uml::Class); // <-- function name is not great...
-= Super types =-
  [EncapsulatedClassifier, BehavioredClassifier]

-= Attributes =-
  eAnnotations : EAnnotation [0..*]
  ownedComment : Comment [0..*]
  ownedElement : Element [0..*] - READONLY
  owner : Element [0..1] - READONLY
  clientDependency : Dependency [0..*]
  name : String [0..1]
  nameExpression : StringExpression [0..1]
  namespace : Namespace [0..1] - READONLY
  qualifiedName : String [0..1] - READONLY
  visibility : VisibilityKind [0..1]
... <-- there is many other fields
```

This function displays first the super types of the meta-class (here `EncapsulatedClassifier` and `BehavioralClassifier`), then all the
meta-attribute (meta-feature) you can use on instance of this meta-class. For each meta-attribute, the feature name is displayed as well
as the feature type and its cardinality. Features annotated with `READONLY` are feature you can only read and not set.

## Comparison Operators
**TODO**

operators `=, <>, <=, <, >, >=`

```
cl> log(3 < 3);
false

cl> log(true < false);
false

cl> log(false < true);
true

cl> log('A' < 'B');
true

cl> log(model = model);
true

cl> log(model = model::UserEntity);
false

cl> log(model < model);
null

cl> log(3 < 'R');
Error> cast error between Integer: 3 and String: R
null
```

## Handling Collections (basic operation, iterations, filtering...)

In CatsooLang, collections are (almost) immutables, you cannot directly modify them. Operation you perform on them return a new result but does not modify the original collection.
Collection can contain any type of elements.

```
cl> var col := [1, 2, 3, 'A'];
cl> log(col remove('A'));  // <-- could be also: log(remove(col, 'A'));
[Integer: 1, Integer: 2, Integer: 3]

cl> log(col);
[Integer: 1, Integer: 2, Integer: 3, String: A]

cl> log(col insertAt('B', 0));
[String: B, Integer: 1, Integer: 2, Integer: 3, String: A]

cl> log(col);
[Integer: 1, Integer: 2, Integer: 3, String: A]

cl> col := col remove('A') insertAt(1, col size() - 1); // <-- function call chaining
cl> log(col);
[Integer: 1, Integer: 2, Integer: 3, Integer: 1]

cl> log(col uniq());
[Integer: 1, Integer: 2, Integer: 3]

cl> log(col);
[Integer: 1, Integer: 2, Integer: 3, Integer: 1]
```

Collections are *almost* immutable structure, you can add elements (1 or many) to a collection using the `+=` operator. This operator can only be applied on named collections (collection stored in a variable or 'many' feature).

```
cl> var a := [];
cl> a += 1;
cl> log(a);
[Integer: 1]

cl> a += [2, 1, 3];
cl> log(a);
[Integer: 1, Integer: 2, Integer: 1, Integer: 3]

cl> a := [];
cl> a += [1, [2,3]];
cl> log(a);
[Integer: 1, Collection: [Integer: 2, Integer: 3]]
```

You can put expression as collection member (between `[]`), the expressions will be evaluated to get the collection member.

```
cl> log([3+1, true or false, UserEntity.name, UserEntity]);
[Integer: 4, Boolean: true, String: UserEntity, CL_EOBJECT(Class, UserEntity, 6912)]
```

You can compare collections using `=` or `<>`. These operators will test if two collections have same size and if the content
of the first collection is/isn't in the second one. However, these operators does not test content order, only values.

```
cl> log([] = []);
true

cl> var i := []; log(i = []);
true

cl> log([1, 2, ['A']] = [2, 1, ['A']]);
true

cl> log([1, 2, [4]] = [2, 1, [5]]);
false

cl> log([1, 2, [4]] <> [2, 1, [5]]);
true
```

### Iterating on Collections

CatsooLang allows you to easily iterate on each collection elements using the `->` operator. This operator applies on each collection element
a dedicated function. In the expression `['FOO', 'Bar', 'fooBAr']->toLower();`, the function `toLower` will be applied on each element of the
`['FOO', 'Bar', 'fooBAr']` collection and will produce a new collection as result: `['foo', 'bar', 'foobar']`. Iteration operator requires that
the function are called in a "sourced" way (`foo bar()`).

```
cl> log(['FOO', 'Bar', 'fooBAr']->toLower());
[String: foo, String: bar, String: foobar]

cl> log([1, 2, 3]->toReal()->cos());
[Real: 0.5403023058681398, Real: -0.4161468365471424, Real: -0.9899924966004454]
```

In the example below, `toLower` signature is `toLower : (s:String) -> String`, it takes a `String` as parameter and returns a `String`, so it can
be applied to the whole collection whithout errors. If an element in the collection cannot match the signature, an error is displayed, but the
execution continue and the result is given whithout the error.

```
cl> log(4 toLower());
Operation 'toLower' not found in this context or for parameters [Integer: 4]
null

cl> log([4, 'FOO', 'Bar', 'fooBAr']->toLower());
Operation 'toLower' not found in this context or for parameters [Integer: 4]
[String: foo, String: bar, String: foobar]
```

As you can see in the trace, `4 toLower();` execution displays an error, but also returns `null`. In the second expression, even if an error message
had been displayed, the final result does not contains `null`. During collection construction, `null` elements are simply ignored.

To avoid many function definition to use iteration, CatsooLang defines two special functions `_iter` and `_apply` that can be used to easily apply
an expression to each element of a collection (named `self`). Difference between the two operators is quite subtle, you can try `doc()` on these
operators to have details about them.

**_iter()**

This operator takes any type of element and finally returns them. This operator is used to handle mutable element (collections in some cases and model
element), modify them and return them. We will see later how we can modify model elements and how this operator becomes handy.

```
cl> var a := [1, 2];
cl> log(a->_iter(self + 1));
[Integer: 1, Integer: 2]

cl> a:= [[1], [2]]; a->_iter(self += 1); log(a);
[Collection: [Integer: 1, Integer: 1], Collection: [Integer: 2, Integer: 1]]
```

**_apply()**

This operator takes any type of element and finally returns the result of the expression passed as parameter. This operator is used to produce new collections
without altering the initial one.

```
cl> log([1, 2, 4, 5]->_apply(self + 1));
[Integer: 2, Integer: 3, Integer: 5, Integer: 6]

cl> log(UserEntity.ownedAttribute->_apply(self.type.name)); // <-- this expression is equivalent to log(UserEntity.ownedAttribute.type.name); we've seen before
[String: Integer, String: Long, String: Real]
```

### Filtering Collections

Filtering collection is always a reccurent task. In order to ease this process, CatsooLang defines two operators: `select` and `reject` (as in OCL/QVT for example).
These operators are used to select/reject elements that satisfy an expression from a collection.
Syntax for `select` is: `_collection_->select(_id_ | _expr_);`. The same syntax is applied to `reject`. The _id_ identifier represents an element from _collection_ on
which the _expr_ is evaluated. If the _expr_ returns `true`, then the element is added to the resulting collection. Else, the element is ignored and _expr_ is evaluated
on the next _collection_ element.

```
cl> log([1, 2, 43, 5, null]->select(e | e > 3));
[Integer: 43, Integer: 5]

cl> log([4, 10, 4, 3, 45]->toReal()->pow(2.)->reject(e | e > 15));
[Real: 9]
```

### Type Filter

In order to filter collection on elements type, use the `isKindOf()` and `isTypeOf()` functions. `_input_ isKindOf(_type_)` tests if the input element has `_type_` type or
is a subclass of `_type_` whereas `_input_ isTypeOf(_type_)` tests if the _input_ type is exactly `_type_`. Here is some examples using `select`:

```
cl> log([1, 2., 'R', 'A', UserEntity]->select(e | e isKindOf(String)));
[String: R, String: A]

cl> log([1, 2., 'R', 'A', UserEntity]->select(e | e isKindOf(Number)));
[Integer: 1, Real: 2]

cl> log([1, 2., 'R', 'A', UserEntity]->select(e | e isKindOf(Classifier))); // <-- In UML, Class inherits from Classifier
[CL_EOBJECT(Class, UserEntity, 5052)]

cl> log([1, 2., 'R', 'A', UserEntity]->select(e | e isTypeOf(Number)));
[]

cl> log([1, 2., 'R', 'A', UserEntity]->select(e | e isTypeOf(Real)));
[Real: 2]

cl> log([1, 2., 'R', 'A', UserEntity]->select(e | e isTypeOf(Class)));
[CL_EOBJECT(Class, UserEntity, 5230)]
```

**Note on `isKindOf/isTypeOf` functions:**

As function parameter, you can use a variable. The function will thus test if the variable type is the same as the input element.
```
cl> var a := 0; var b := 20; log(a isKindOf(b));
true

cl> var a := 0; var b := 'R'; log(a isKindOf(b));
false

cl> var a := UserEntity::ID; var b := UserEntity::attribute; log(a isKindOf(b));
true
```

To ease collection filtering on types from a metamodel (metaclass filtering), CatsooLang defines an handy operator: `[_metaclass_]` (stolen from QVT) which is syntaxic sugar
for`->select(x | x isKindOf(_metaclass_)`. This operator returns a collection containing the element of a specific type or `[]` if no collection element has the required type.
Associated to this operator, the `![_metaclass_]` selects only one element of the collection which has the required type. If no element has the required type, `null` is returned.
Here is two examples:

```
cl> var a:= [1, 2., 'R', 'A', UserEntity];
cl> log(a[Classifier]);
[CL_EOBJECT(Class, UserEntity, 6031)]

cl> log(a[Number]);
line 2:6 mismatched input 'Number' expecting IDENTIFIER  <-- a metaclass identifier is required, this operator does not work with CatsooLang basic types.
log(a[Number]);
      ^^^^^^

cl> log(a![Class]);
org.eclipse.uml2.uml.internal.impl.ClassImpl@102a (name: UserEntity, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)
```




_to be continued_

## Creating, Modifiying Model Elements

In catsoolang, you can easily create new model elements using the `create _metaclass_` keyword.

```
cl> var a := create Class;
cl> log(a);
org.eclipse.uml2.uml.internal.impl.ClassImpl@1e15 (name: <unset>, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)

cl> type(a);
'CL_EOBJECT(Class, 7701)

cl> #load_metamodel ecore('http://www.eclipse.org/emf/2002/Ecore') // <-- we load the ecore metamodel under the 'ecore' name
!! Loading metamodel 'http://www.eclipse.org/emf/2002/Ecore' under 'ecore'

cl> log(ecore::Eclass); // <-- We can access ecore metamodel metaclasses
org.eclipse.emf.ecore.impl.EClassImpl@132 (name: EClass) (instanceClassName: null) (abstract: false, interface: false)

cl> log(create EClass); // <-- Consequently, we can now create instances of ecore metaclasses
org.eclipse.emf.ecore.impl.EClassImpl@2217 (name: null) (instanceClassName: null) (abstract: false, interface: false)
```

The two operators `:=` and `+=` are used for single value meta-attributes and multi value meta-attributes assignments:

```
cl> var a := create Class;
cl> log(a.name);
null

cl> a.name := 'MyFirstClass';
cl> log(a);
org.eclipse.uml2.uml.internal.impl.ClassImpl@227a (name: MyFirstClass, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)
cl> log(a.name);
MyFirstClass

cl> log(a.ownedAttribute);
[]

cl> a.ownedAttribute += create Property;
cl> log(a.ownedAttribute);
[CL_EOBJECT(Property, 9277)]

cl> a.ownedAttribute at(0).name := 'myproperty';
cl> log(a.ownedAttribute.name);
[String: myproperty]

cl> tree(a);
+- Class 'MyFirstClass' [-]
|  +- Property 'myproperty' [ownedAttribute]
```

In the above example, we created a model fragment (which is stored into `a`). We create a `Class`, name it `MyFirstClass` then we create a property, we add it to `MyClass` and we
name it `myproperty`. This require a lot of steps. CatsooLang defines the convenient operator`{...}` that allows you to assign many meta-attribute value at the same time. Its syntax
is ``_myexprtoaneobj_{_metaatt1_ _op_ _expr_; _metaatt2_ _op_ _expr_; ....}`. Here is some examples that exploit this:

```
cl> var a := create Class{name := 'C1'};
cl> log(a.name);
C1

cl> a{isAbstract := true; ownedAttribute += create Property{name := 'newprop'}};
cl> tree(a);
+- Class 'C1' [-]
|  +- Property 'newprop' [ownedAttribute]

cl> var b := create Class{name := 'C1'; isAbstract := true; ownedAttribute += create Property{name := 'newprop'; type := umllib::String}}; // <-- Almost the same thing, but in one line
cl> tree(b);
+- Class 'C1' [-]
|  +- Property 'newprop' [ownedAttribute]
```

__NOTE__

The `create` keyword is quite permissive and allows you to use variable as `_metaclass_` as soon as they reference a metaclass:
```
cl> var myc := Class;
cl> log(myc);
org.eclipse.emf.ecore.impl.EClassImpl@14b (name: Class) (instanceClassName: null) (abstract: false, interface: false)

cl> log(create myc);
org.eclipse.uml2.uml.internal.impl.ClassImpl@df5 (name: <unset>, visibility: <unset>) (isLeaf: false, isAbstract: false, isFinalSpecialization: false) (isActive: false)

cl> log(create myc{name := 'NewC'}.name);
NewC

```

Thanks to this feature, you can easily create new model fragments. However, up to now, these fragments are only present in CatsooLang evaluation context and they are not attached to your
model. In order to attach elements to your model, you simply have to alter the meta-attribute of your model element which must own the created fragment. Let's attach our fragment stored
in the `b` variable in our model (once the fragment is attached to your model, you can see elements in the 'project' tree view of GenMyModel).

```
cl> tree(__ROOT); // <-- before we include the fragment
+- Model 'model' [-]
|  +- (PackageImport, 10973) [packageImport]
|  +- (PackageImport, 10974) [packageImport]
|  +- Class 'UserEntity' [packagedElement]
|  |  +- Property 'ID' [ownedAttribute]
|  |  +- Property 'attribute' [ownedAttribute]
|  |  +- Property 'attribute2' [ownedAttribute]

cl> __ROOT.packagedElement += b;
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 11266) [packageImport]
|  +- (PackageImport, 11267) [packageImport]
|  +- Class 'UserEntity' [packagedElement]
|  |  +- Property 'ID' [ownedAttribute]
|  |  +- Property 'attribute' [ownedAttribute]
|  |  +- Property 'attribute2' [ownedAttribute]
|  +- Class 'C1' [packagedElement]
|  |  +- Property 'newprop' [ownedAttribute]
```

Now, let's add the `a` fragment into a `Packaged` named 'mypack':
```
cl> __ROOT.packagedElement += create Package{name := 'mypack'; packagedElement += a};
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 11753) [packageImport]
|  +- (PackageImport, 11754) [packageImport]
|  +- Class 'UserEntity' [packagedElement]
|  |  +- Property 'ID' [ownedAttribute]
|  |  +- Property 'attribute' [ownedAttribute]
|  |  +- Property 'attribute2' [ownedAttribute]
|  +- Class 'C1' [packagedElement]
|  |  +- Property 'newprop' [ownedAttribute]
|  +- Package 'mypack' [packagedElement]
|  |  +- Class 'C1' [packagedElement]
|  |  |  +- Property 'newprop' [ownedAttribute]
```

You can now drag and drop the created elements from your 'project' tree view to your diagram (we will see later how to create graphical view of these elements).
Here is some 'complex' examples that mixe collections, `_iter/_apply` and model creations. For each example, we will details how the expression is created.

Example 1: creation of a `Package` which contains 10 classes named `C1...C10`.
```
cl> var p := create Package{packagedElement += range(1,10)->_apply('C' + self)->_apply(create Class{name := self})};
cl> tree(p);
+- Package 'null' [-]
|  +- Class 'C1' [packagedElement]
|  +- Class 'C2' [packagedElement]
|  +- Class 'C3' [packagedElement]
|  +- Class 'C4' [packagedElement]
|  +- Class 'C5' [packagedElement]
|  +- Class 'C6' [packagedElement]
|  +- Class 'C7' [packagedElement]
|  +- Class 'C8' [packagedElement]
|  +- Class 'C9' [packagedElement]
|  +- Class 'C10' [packagedElement]
```
There is two main parts of this expression. The first is the 10 classes creation, the second the package creation.

* 10 classes creations:

	`range(1,10)` --> creates a collection of 10 integers.

	`range(1,10)->_apply('C' + self)` --> concatenates each integer to 'C' (in this context, `self` is an integer), the result is a collections of strings.

	`range(1,10)->_apply('C' + self)->_apply(create Class{name := self})` --> for each string (`C1`...) create a class that you name after `self` (a string in this context).

	The result of this expression is a collection of classes.
* package creation:
  `create Package{packagedElement += _previous_expr_}` --> creates a package and attach the previous 10 classes to it. As `+=` operator can either add element or add all elements
  of a list, the result of `packagedElement += ...` is a modification of the `packagedElement` collection.

Example 2: creation of 3 classes 'A', 'B', 'C' that inherits from a class named 'SuperClass' and add them to the root of your model (from an empty model).
```
cl> var sclass := create Class{name := 'SuperClass'};
cl> __ROOT.packagedElement += sclass;
cl> __ROOT.packagedElement += ['A', 'B', 'C']->_apply(create Class{name := self; superClass += sclass});
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 12645) [packageImport]
|  +- (PackageImport, 12646) [packageImport]
|  +- Class 'SuperClass' [packagedElement]
|  +- Class 'A' [packagedElement]
|  |  +- (Generalization, 12647) [generalization]
|  +- Class 'B' [packagedElement]
|  |  +- (Generalization, 12648) [generalization]
|  +- Class 'C' [packagedElement]
|  |  +- (Generalization, 12649) [generalization]

cl> // here is the same code in a more compact line (assuming an empty model)
cl> var sclass := create Class{name := 'SuperClass'};
cl> __ROOT{packagedElement += sclass; packagedElement += ['A', 'B', 'C']->_apply(create Class{name := self; superClass += sclass})};
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 3764) [packageImport]
|  +- (PackageImport, 3765) [packageImport]
|  +- Class 'SuperClass' [packagedElement]
|  +- Class 'A' [packagedElement]
|  |  +- (Generalization, 3766) [generalization]
|  +- Class 'B' [packagedElement]
|  |  +- (Generalization, 3767) [generalization]
|  +- Class 'C' [packagedElement]
|  |  +- (Generalization, 3768) [generalization]

```

Example 3: addition of a property named 'ID' with Integer type in each model class.
```
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 3764) [packageImport]
|  +- (PackageImport, 3765) [packageImport]
|  +- Class 'SuperClass' [packagedElement]
|  +- Class 'A' [packagedElement]
|  |  +- (Generalization, 3766) [generalization]
|  +- Class 'B' [packagedElement]
|  |  +- (Generalization, 3767) [generalization]
|  +- Class 'C' [packagedElement]
|  |  +- (Generalization, 3768) [generalization]

cl> __ROOT objects()[Class]->_iter(self.ownedAttribute += create Property{name := 'ID'; type := umllib::Integer});
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 3886) [packageImport]
|  +- (PackageImport, 3887) [packageImport]
|  +- Class 'SuperClass' [packagedElement]
|  |  +- Property 'ID' [ownedAttribute]
|  +- Class 'A' [packagedElement]
|  |  +- (Generalization, 3888) [generalization]
|  |  +- Property 'ID' [ownedAttribute]
|  +- Class 'B' [packagedElement]
|  |  +- (Generalization, 3889) [generalization]
|  |  +- Property 'ID' [ownedAttribute]
|  +- Class 'C' [packagedElement]
|  |  +- (Generalization, 3890) [generalization]
|  |  +- Property 'ID' [ownedAttribute]

```

Example 4: addition of a property named 'ID' with Integer type in each model class that does not inherit from another.
```
cl> __ROOT objects()[Class]->select(e | e.superClass = [])->_iter(self.ownedAttribute += create Property{name := 'ID'; type := umllib::Integer});
```

## Functions and Control Flow (if, for, ternary op.)

In CatsooLang, functions are defines using the `def` keyword. The syntax is the following:

```
ex:
def foo(p1 : T1, p2 : T2, ..., pn : Tn) : returnType
{
    _my_code_here_
}

cl> def helloWorld()
    {
        log('Hello World');
	}
!! helloWorld : () -> Void  <-- interpreter answer

cl> helloWorld();
Hello World
```

When a function is registered in the CatsooLang evaluation environement, the interpreter gives a feedback about the function name and signature. In this example,
we defined a `helloWorld` function with no parameters and no return value. Functions can accept 0 or many parameters. The `returnType` definition is optional
(only used for documentation purposes). If no `returnType` is set, `Void` is assumed by the CatsooLang interpreter (be careful, `Void` keyword does not exist).
In order to return a value, the keyword `return` is used.

```
cl> def plus1(i : Integer) : Integer { return i+1; }
!! plus1 : (i:Integer) -> Integer

cl> log(plus1(2));
3

cl> log(2 plus1());
3
```

In this example, you can clearly see the two ways of calling a function in action. In the first call, the parameter is passed as a function parameter whereas in the second
one, the parameter is used as function 'source'. In CatsooLang, function can return nothing (evaluated as `null`) or any type of value. The same function can return either
`String`, `Integer` or any type even if the return type in the signature says otherwise (remember? it is used for documentation only). The example below presents not only
the `Any` type in action, but also `if(){}else{}` control flow structure, we will see later how `if-else` works.

```
cl> def difftype(a : Any) : Any { if (a isKindOf(Number)) { return a + 1; } else { return 'Other'; }}
!! difftype : (a:Any) -> Any

cl> log(difftype(2));
3

cl> log(difftype(create Class));
Other
```

You can re-register a function in order to replace the previous one (if you made a mistake). In order to do it, your new function must have the same name and the exact same
signature. Be careful, if you want to replace a function, as CatsooLang supports ad-hoc polymorphism, the signature __MUST__ be the __EXACT SAME__.

```
cl> def fun(n : Number) : Number { return n + 3; } // <-- oups, should be n + 1
!! fun : (n:Number) -> Number

cl> def fun(n : Number) : Number { return n + 1; }
!! Replacing fun : (n:Number) -> Number
!! fun : (n:Number) -> Number

cl> def fun(i : Integer) : Number { return n + 1; }
!! fun : (i:Integer) -> Number  <-- Signature is not the same, no replacement here

cl> doc(fun);
1. fun : (n:Number) -> Number
* Sorry, there is no documentation or doc is empty

2. fun : (i:Integer) -> Number
* Sorry, there is no documentation or doc is empty
```

As you can see, the `fun` function had been registered two times with two different signatures. The `doc()` function displays the register order. The function register order
is important. When you call a function, the CatsooLang interpreter evaluate each parameters, then search in its function register the first function with the same name and
with a signature which matches the evaluated parameters. Let's take another look to the previous example.

```
cl> def fun(n : Number) : Number { return n + 3; }
!! fun : (n:Number) -> Number

cl> def fun(n : Integer) : Number { return n + 1; }
!! fun : (n:Integer) -> Number

* We have registered two functions 'fun' *

cl> type(fun(1));
'Integer: 4

cl> type(fun(1.));
'Real: 4
```

In both calls, only the first function had been called. As the first `fun` function works with `Number` and as `Integer` and `Real` 'inherit' from `Number`, then the two calls
matche the first `fun` signature. In this case, the second registered function is masked and will never be accessible. Here is what happend if we reverse the register order:

```
cl> def fun(n : Integer) : Number { return n + 1; }
!! fun : (n:Integer) -> Number

cl> def fun(n : Number) : Number { return n + 3; }
!! fun : (n:Number) -> Number

cl> type(fun(1));
'Integer: 2

cl> type(fun(1.));
'Real: 4
```
In this scenario, the two functions are accessible.

Parameters type in CatsooLang are kind of 'constraints' for signature matching and it support metamodel inheritance. In other words, you can define function that accept a 'kind
of parameter type'. Here is a common example over UML:

```
cl> def foo(c : Classifier) : String { return c.name; }

cl> log(foo(create Class{name := 'MyClass'}));
MyClass

cl> log(foo(create Interface{name := 'MyInterface'}));
MyInterface

* Here is another way of calling the defined function *

cl> log(create Class{name := 'A'} foo());
A

cl> log(create Interface{name := 'B'} foo());
B
```

Function documentation in CatsooLang is easy to drop on functions. The documentation must be in top of the function signature during its definition and must be written between
`""" """`.

```
cl> def bar() {}
!! bar : () -> Void

cl> doc(bar);

1. bar : () -> Void
* Sorry, there is no documentation or doc is empty

cl> """ *This is documentation for bar* """ def bar(){}
!! Replacing bar : () -> Void
!! bar : () -> Void

cl> doc(bar);

1. bar : () -> Void

	This is documentation for bar
```

Of course, you can enter a new line (using `SHIFT + ENTER`) and indent a little bit your function (copy/paste the following function):

```
cl> """
*My first documented CatsooLang function*

@param s a String
@returns a String
"""
def docfunc(s : String) : String
{
    return 'Yeay ' + s;
}

!! docfunc : (s:String) -> String

cl>  doc(docfunc);

1. docfunc : (s:String) -> String

  My first documented CatsooLang function

  @param s a String
  @returns a String
```
The `@param` and `@returns` documentation keywords are automatically put in italic. CatsooLang documentation also accept `** **/__ __` and `* */_ _` from markdown.

### Log Wrapper Example

This section details a little example about collection iteration, function definitions and type constraints for a "display element" wrapper. The way CatsooLang gives feedback in the
terminal is by using the `log()` functionnality. However, this is an element language and it is not a function (why? Arf, why not) so you cannot use it to iterate on elements.
```
cl> [1,2,3]->log();
line 1:10 no viable alternative at input '->log('
 [1,2,3]->log();
          ^^^^
```
In order to introduce this behavior, we will simply define a wrapper around the `log()` feature:
```
cl> def mylog(a : Any) {log(a);}
!! mylog : (a:Any) -> Void

cl> [1,2,3]->mylog();
1
2
3
```

The first parameter type is `Any`. This function can be applied to any CatsooLang type (metamodel type or basic type). Nice, but it could be more interesting if you could define exactly
what you want to print. So, we register a second `mylog` function:
```
cl> def mylog(a : Any, disp : Any) {log(disp);}
!! mylog : (a:Any x disp:Any) -> Void
```

This time, the first parameter is not directly displayed, just the second one. Actually, the two parameters working together in an iteration context are like a lambda abstraction. The
first parameter `a` is here to contrain the element type on which the iteration will be performed. It also define a `self` variable which will be bound in the `disp` lambda term. Here are
some examples:
```
cl> [1, 2, 3]->mylog(self); // <-- equivalent to [1, 2, 3]->mylog();
1
2
3

cl> [1, 2, 3]->mylog('The number is ' + self);
The number is 1
The number is 2
The number is 3

cl> [1, 2, 3]->mylog(self toReal() pow(self toReal()));
1
4
27
```

In order to push this concept a little further, take a look to the `_iter` and `_apply` functions.

## Libraries

## UML Library & UML Refactoring Library

##

## Command Examples

Rename all the classes/interfaces contained in a package/model to postfix them with something:
`_mypackage_path_ objects()[Classifier]->_iter(self.name := self.name + _mypost_)`.

You can read this expression as:
"from `_mypackage_path_`, take all the objects it contains (including itself, so a collection), filter them to keep `Classifier` only and on each element assign the `name` attribute of the
`Classifier` to its own name postfix with `_mypost`". Here is an example:

```
cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 5728) [packageImport]
|  +- (PackageImport, 5729) [packageImport]
|  +- Package 'entity' [packagedElement]           <-- One package named 'entity'
|  |  +- Class 'Entity1' [packagedElement]
|  |  |  +- (Generalization, 5723) [generalization]
|  |  +- Class 'Entity2' [packagedElement]
|  |  |  +- (Generalization, 5724) [generalization]
|  |  +- Interface 'Observable' [packagedElement]
|  |  +- Class 'Abstract' [packagedElement]
|  |  |  +- InterfaceRealization 'null' [interfaceRealization]

cl> log(entity objects()[Classifier]->_iter(self.name := self.name + 'Entity').name); // <-- log gets the modified names (here _apply would give the same result).
[String: Entity1Entity, String: Entity2Entity, String: ObservableEntity, String: AbstractEntity]

cl> tree(__ROOT);
+- Model 'model' [-]
|  +- (PackageImport, 6471) [packageImport]
|  +- (PackageImport, 6472) [packageImport]
|  +- Package 'entity' [packagedElement]
|  |  +- Class 'Entity1Entity' [packagedElement]
|  |  |  +- (Generalization, 6466) [generalization]
|  |  +- Class 'Entity2Entity' [packagedElement]
|  |  |  +- (Generalization, 6467) [generalization]
|  |  +- Interface 'ObservableEntity' [packagedElement]
|  |  +- Class 'AbstractEntity' [packagedElement]
|  |  |  +- InterfaceRealization 'null' [interfaceRealization]

```
