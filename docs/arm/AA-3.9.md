---
sidebar_position:  25
---

# 3.9  Tagged Types and Type Extensions

[ Tagged types and type extensions support object-oriented programming, based on inheritance with extension and run-time polymorphism via dispatching operations. ]


#### Language Design Principles

{AI95-00251-01} The intended implementation model is for the static portion of a tag to be represented as a pointer to a statically allocated and link-time initialized type descriptor. The type descriptor contains the address of the code for each primitive operation of the type. It probably also contains other information, such as might make membership tests convenient and efficient. Tags for nested type extensions must also have a dynamic part that identifies the particular elaboration of the type.

The primitive operations of a tagged type are known at its first freezing point; the type descriptor is laid out at that point. It contains linker symbols for each primitive operation; the linker fills in the actual addresses.

{AI95-00251-01} Primitive operations of type extensions that are declared at a level deeper than the level of the ultimate ancestor from which they are derived can be represented by wrappers that use the dynamic part of the tag to call the actual primitive operation. The dynamic part would generally be some way to represent the static link or display necessary for making a nested call. One implementation strategy would be to store that information in the extension part of such nested type extensions, and use the dynamic part of the tag to point at it. (That way, the "dynamic" part of the tag could be static, at the cost of indirect access.)

{AI95-00251-01} If the tagged type is descended from any interface types, it also will need to include "subtags" (one for each interface) that describe the mapping of the primitive operations of the interface to the primitives of the type. These subtags could directly reference the primitive operations (for faster performance), or simply provide the tag "slot" numbers for the primitive operations (for easier derivation). In either case, the subtags would be used for calls that dispatch through a class-wide type of the interface.

Other implementation models are possible.

The rules ensure that "dangling dispatching" is impossible; that is, when a dispatching call is made, there is always a body to execute. This is different from some other object-oriented languages, such as Smalltalk, where it is possible to get a run-time error from a missing method.

{AI95-00251-01} Dispatching calls should be efficient, and should have a bounded worst-case execution time. This is important in a language intended for real-time applications. In the intended implementation model, a dispatching call involves calling indirect through the appropriate slot in the dispatch table. No complicated "method lookup" is involved although a call which is dispatching on an interface may require a lookup of the appropriate interface subtag.

The programmer should have the choice at each call site of a dispatching operation whether to do a dispatching call or a statically determined call (i.e. whether the body executed should be determined at run time or at compile time).

The same body should be executed for a call where the tag is statically determined to be T'Tag as for a dispatching call where the tag is found at run time to be T'Tag. This allows one to test a given tagged type with statically determined calls, with some confidence that run-time dispatching will produce the same behavior.

All views of a type should share the same type descriptor and the same tag.

The visibility rules determine what is legal at compile time; they have nothing to do with what bodies can be executed at run time. Thus, it is possible to dispatch to a subprogram whose declaration is not visible at the call site. In fact, this is one of the primary facts that gives object-oriented programming its power. The subprogram that ends up being dispatched to by a given call might even be designed long after the call site has been coded and compiled.

Given that Ada has overloading, determining whether a given subprogram overrides another is based both on the names and the type profiles of the operations.

{AI95-00401-01} When a type extension is declared, if there is any place within its immediate scope where a certain subprogram of the parent or progenitor is visible, then a matching subprogram should override. If there is no such place, then a matching subprogram should be totally unrelated, and occupy a different slot in the type descriptor. This is important to preserve the privacy of private parts; when an operation declared in a private part is inherited, the inherited version can be overridden only in that private part, in the package body, and in any children of the package.

If an implementation shares code for instances of generic bodies, it should be allowed to share type descriptors of tagged types declared in the generic body, so long as they are not extensions of types declared in the specification of the generic unit. 


#### Static Semantics

{AI95-00345-01} A record type or private type that has the reserved word tagged in its declaration is called a tagged type. In addition, an interface type is a tagged type, as is a task or protected type derived from an interface (see 3.9.4). [When deriving from a tagged type, as for any derived type, additional primitive subprograms may be defined, and inherited primitive subprograms may be overridden.] The derived type is called an extension of its ancestor types, or simply a type extension.

{AI95-00345-01} Every type extension is also a tagged type, and is a record extension or a private extension of some other tagged type, or a noninterface synchronized tagged type (see 3.9.4). A record extension is defined by a [derived_type_definition](./AA-3.4#S0035) with a [record_extension_part](./AA-3.9#S0075) (see 3.9.1)[, which may include the definition of additional components]. A private extension, which is a partial view of a record extension or of a synchronized tagged type, can be declared in the visible part of a package (see 7.3) or in a generic formal part (see 12.5.1). 

Glossary entry: The objects of a tagged type have a run-time type tag, which indicates the specific type with which the object was originally created. An operand of a class-wide tagged type can be used in a dispatching call; the tag indicates which subprogram body to invoke. Nondispatching calls, in which the subprogram body to invoke is determined at compile time, are also allowed. Tagged types may be extended with additional components.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[tagged type], Def=[a type whose objects each have a run-time type tag, which indicates the specific type for which the object was originally created], Note1=[Tagged types can be extended with additional components.] 

Ramification: {AI95-00218-03} If a tagged type is declared other than in a [package_specification](./AA-7.1#S0230), it is impossible to add new primitive subprograms for that type, although it can inherit primitive subprograms, and those can be overridden. If the user incorrectly thinks a certain subprogram is primitive when it is not, and tries to call it with a dispatching call, an error message will be given at the call site. Similarly, by using an [overriding_indicator](./AA-8.3#S0234) (see 6.1), the user can declare that a subprogram is intended to be overriding, and get an error message when they made a mistake. The use of [overriding_indicator](./AA-8.3#S0234)s is highly recommended in new code that does not need to be compatible with Ada 95.

An object of a tagged type has an associated (run-time) tag that identifies the specific tagged type used to create the object originally. [ The tag of an operand of a class-wide tagged type T'Class controls which subprogram body is to be executed when a primitive subprogram of type T is applied to the operand (see 3.9.2); using a tag to control which body to execute is called dispatching.] 

{AI95-00344-01} The tag of a specific tagged type identifies the [full_type_declaration](./AA-3.2#S0024) of the type, and for a type extension, is sufficient to uniquely identify the type among all descendants of the same ancestor. If a declaration for a tagged type occurs within a [generic_package_declaration](./AA-12.1#S0312), then the corresponding type declarations in distinct instances of the generic package are associated with distinct tags. For a tagged type that is local to a generic package body and with all of its ancestors (if any) also local to the generic body, the language does not specify whether repeated instantiations of the generic body result in distinct tags. 

This paragraph was deleted.{AI95-00344-01} 

Implementation Note: {AI95-00344-01} In most cases, a tag need only identify a particular tagged type declaration, and can therefore be a simple link-time-known address. However, for tag checks (see 3.9.2) it is essential that each descendant (that currently exists) of a given type have a unique tag. Hence, for types declared in shared generic bodies where an ancestor comes from outside the generic, or for types declared at a deeper level than an ancestor, the tag needs to be augmented with some kind of dynamic descriptor (which may be a static link, global display, instance descriptor pointer, or combination). This implies that type Tag may need to be two words, the second of which is normally null, but in these identified special cases needs to include a static link or equivalent. Within an object of one of these types with a two-word tag, the two parts of the tag would typically be separated, one part as the first word of the object, the second placed in the first extension part that corresponds to a type declared more nested than its parent or declared in a shared generic body when the parent is declared outside. Alternatively, by using an extra level of indirection, the type Tag could remain a single-word.

{AI95-00344-01} For types that are not type extensions (even for ones declared in nested scopes), we do not require that repeated elaborations of the same [full_type_declaration](./AA-3.2#S0024) correspond to distinct tags. This was done so that Ada 2005 implementations of tagged types could maintain representation compatibility with Ada 95 implementations. Only type extensions that were not allowed in Ada 95 require additional information with the tag. 

To be honest: {AI95-00344-01} The wording "is sufficient to uniquely identify the type among all descendants of the same ancestor" only applies to types that currently exist. It is not necessary to distinguish between descendants that currently exist, and descendants of the same type that no longer exist. For instance, the address of the stack frame of the subprogram that created the tag is sufficient to meet the requirements of this rule, even though it is possible, after the subprogram returns, that a later call of the subprogram could have the same stack frame and thus have an identical tag. 

The following language-defined library package exists: 

```ada
{AI95-00362-01} {AI12-0241-1} {AI12-0302-1} {AI12-0399-1} package Ada.Tags 
    with Preelaborate, Nonblocking, Global =&gt in out synchronized is
    type Tag is private
       with Preelaborable_Initialization;

```

```ada
{AI95-00260-02}     No_Tag : constant Tag;

```

```ada
{AI95-00400-01}     function Expanded_Name(T : Tag) return String;
    function Wide_Expanded_Name(T : Tag) return Wide_String;
    function Wide_Wide_Expanded_Name(T : Tag) return Wide_Wide_String;
    function External_Tag(T : Tag) return String;
    function Internal_Tag(External : String) return Tag;

```

```ada
{AI95-00344-01}     function Descendant_Tag(External : String; Ancestor : Tag) return Tag;
    function Is_Descendant_At_Same_Level(Descendant, Ancestor : Tag)
        return Boolean;

```

```ada
{AI95-00260-02}     function Parent_Tag (T : Tag) return Tag;

```

```ada
{AI95-00405-01}     type Tag_Array is array (Positive range &lt&gt) of Tag;

```

```ada
{AI95-00405-01}     function Interface_Ancestor_Tags (T : Tag) return Tag_Array;

```

```ada
{AI05-0173-1}     function Is_Abstract (T : Tag) return Boolean;

```

```ada
    Tag_Error : exception;

```

```ada
private
   ... -- not specified by the language
end Ada.Tags;

```

Reason: Tag is a nonlimited, definite subtype, because it needs the equality operators, so that tag checking makes sense. Also, equality, assignment, and object declaration are all useful capabilities for this subtype.

For an object X and a type T, "X'Tag = T'Tag" is not needed, because a membership test can be used. However, comparing the tags of two objects cannot be done via membership. This is one reason to allow equality for type Tag. 

{AI95-00260-02} No_Tag is the default initial value of type Tag. 

Reason: {AI95-00260-02} This is similar to the requirement that all access values be initialized to null. 

{AI95-00400-01} The function Wide_Wide_Expanded_Name returns the full expanded name of the first subtype of the specific type identified by the tag, in upper case, starting with a root library unit. The result is implementation defined if the type is declared within an unnamed [block_statement](./AA-5.6#S0191). 

To be honest: This name, as well as each [prefix](./AA-4.1#S0093) of it, does not denote a [renaming_declaration](./AA-8.5#S0238). 

Implementation defined: The result of Tags.Wide_Wide_Expanded_Name for types declared within an unnamed [block_statement](./AA-5.6#S0191).

{AI95-00400-01} The function Expanded_Name (respectively, Wide_Expanded_Name) returns the same sequence of graphic characters as that defined for Wide_Wide_Expanded_Name, if all the graphic characters are defined in Character (respectively, Wide_Character); otherwise, the sequence of characters is implementation defined, but no shorter than that returned by Wide_Wide_Expanded_Name for the same value of the argument. 

Implementation defined: The sequence of characters of the value returned by Tags.Expanded_Name (respectively, Tags.Wide_Expanded_Name) when some of the graphic characters of Tags.Wide_Wide_Expanded_Name are not defined in Character (respectively, Wide_Character).

The function External_Tag returns a string to be used in an external representation for the given tag. The call External_Tag(S'Tag) is equivalent to the [attribute_reference](./AA-4.1#S0100) S'External_Tag (see 13.3). 

Reason: It might seem redundant to provide both the function External_Tag and the attribute External_Tag. The function is needed because the attribute can't be applied to values of type Tag. The attribute is needed so that it can be specified via an [attribute_definition_clause](./AA-13.3#S0349). 

{AI95-00417-01} The string returned by the functions Expanded_Name, Wide_Expanded_Name, Wide_Wide_Expanded_Name, and External_Tag has lower bound 1.

{AI95-00279-01} The function Internal_Tag returns a tag that corresponds to the given external tag, or raises Tag_Error if the given string is not the external tag for any specific type of the partition. Tag_Error is also raised if the specific type identified is a library-level type whose tag has not yet been created (see 13.14).

Reason: {AI95-00279-01} {AI05-0005-1} The check for uncreated library-level types prevents a reference to the type before execution reaches the freezing point of the type. This is important so that T'Class'Input or an instance of Tags.Generic_Dispatching_Constructor do not try to create an object of a type that hasn't been frozen (which might not have yet elaborated its constraints). We don't require this behavior for non-library-level types as the tag can be created multiple times and possibly multiple copies can exist at the same time, making the check complex. 

{AI95-00344-01} {AI05-0113-1} The function Descendant_Tag returns the (internal) tag for the type that corresponds to the given external tag and is both a descendant of the type identified by the Ancestor tag and has the same accessibility level as the identified ancestor. Tag_Error is raised if External is not the external tag for such a type. Tag_Error is also raised if the specific type identified is a library-level type whose tag has not yet been created, or if the given external tag identifies more than one type that has the appropriate Ancestor and accessibility level.

Reason: Descendant_Tag is used by T'Class'Input to identify the type identified by an external tag. Because there can be multiple elaborations of a given type declaration, Internal_Tag does not have enough information to choose a unique such type. Descendant_Tag does not return the tag for types declared at deeper accessibility levels than the ancestor because there could be ambiguity in the presence of recursion or multiple tasks. Descendant_Tag can be used in constructing a user-defined replacement for T'Class'Input.

{AI05-0113-1} Rules for specifying external tags will usually prevent an external tag from identifying more than one type. However, an external tag can identify multiple types if a generic body contains a derivation of a tagged type declared outside of the generic, and there are multiple instances at the same accessibility level as the type. (The Reference Manual allows default external tags to not be unique in this case.) 

{AI95-00344-01} The function Is_Descendant_At_Same_Level returns True if the Descendant tag identifies a type that is both a descendant of the type identified by Ancestor and at the same accessibility level. If not, it returns False.

Reason: Is_Descendant_At_Same_Level (or something similar to it) is used by T'Class'Output to determine whether the item being written is at the same accessibility level as T. It may be used to determine prior to using T'Class'Output whether Tag_Error will be raised, and also can be used in constructing a user-defined replacement for T'Class'Output. 

{AI05-0115-1} For the purposes of the dynamic semantics of functions Descendant_Tag and Is_Descendant_At_Same_Level, a tagged type T2 is a descendant of a type T1 if it is the same as T1, or if its parent type or one of its progenitor types is a descendant of type T1 by this rule[, even if at the point of the declaration of T2, one of the derivations in the chain is not visible].

Discussion: In other contexts, "descendant" is dependent on visibility, and the particular view a derived type has of its parent type. See 7.3.1. 

{AI95-00260-02} {AI12-0056-1} The function Parent_Tag returns the tag of the parent type of the type whose tag is T. If the type does not have a parent type (that is, it was not defined by a [derived_type_definition](./AA-3.4#S0035)), then No_Tag is returned.

Ramification: {AI12-0005-1} The parent type is always the parent of the full type; a private extension appears to define a parent type, but it does not (only the various forms of derivation do that). As this is a run-time operation, ignoring privacy is OK. 

{AI95-00405-01} The function Interface_Ancestor_Tags returns an array containing the tag of each interface ancestor type of the type whose tag is T, other than T itself. The lower bound of the returned array is 1, and the order of the returned tags is unspecified. Each tag appears in the result exactly once.[ If the type whose tag is T has no interface ancestors, a null array is returned.]

Ramification: The result of Interface_Ancestor_Tags includes the tag of the parent type, if the parent is an interface.

Indirect interface ancestors are included in the result of Interface_Ancestor_Tags. That's because where an interface appears in the derivation tree has no effect on the semantics of the type; the only interesting property is whether the type has an interface as an ancestor. 

{AI05-0173-1} The function Is_Abstract returns True if the type whose tag is T is abstract, and False otherwise.

For every subtype S of a tagged type T (specific or class-wide), the following attributes are defined: 

S'ClassS'Class denotes a subtype of the class-wide type (called T'Class in this document) for the class rooted at T (or if S already denotes a class-wide subtype, then S'Class is the same as S).

S'Class is unconstrained. However, if S is constrained, then the values of S'Class are only those that when converted to the type T belong to S. 

Ramification: This attribute is defined for both specific and class-wide subtypes. The definition is such that S'Class'Class is the same as S'Class.

Note that if S is constrained, S'Class is only partially constrained, since there might be additional discriminants added in descendants of T which are not constrained. 

Reason: {AI95-00326-01} The Class attribute is not defined for untagged subtypes (except for incomplete types and private types whose full view is tagged - see J.11 and 7.3.1) so as to preclude implicit conversion in the absence of run-time type information. If it were defined for untagged subtypes, it would correspond to the concept of universal types provided for the predefined numeric classes. 

S'TagS'Tag denotes the tag of the type T (or if T is class-wide, the tag of the root type of the corresponding class). The value of this attribute is of type Tag. 

Reason: S'Class'Tag equals S'Tag, to avoid generic contract model problems when S'Class is the actual type associated with a generic formal derived type.

Given a [prefix](./AA-4.1#S0093) X that is of a class-wide tagged type [(after any implicit dereference)], the following attribute is defined: 

X'TagX'Tag denotes the tag of X. The value of this attribute is of type Tag. 

Reason: X'Tag is not defined if X is of a specific type. This is primarily to avoid confusion that might result about whether the Tag attribute should reflect the tag of the type of X, or the tag of X. No such confusion is possible if X is of a class-wide type. 

{AI95-00260-02} {AI95-00441-01} The following language-defined generic function exists:

```ada
{AI05-0229-1} {AI12-0241-1} {AI12-0302-1} generic
    type T (&lt&gt) is abstract tagged limited private;
    type Parameters (&lt&gt) is limited private;
    with function Constructor (Params : not null access Parameters)
        return T is abstract;
function Ada.Tags.Generic_Dispatching_Constructor
   (The_Tag : Tag;
    Params  : not null access Parameters) return T'Class
   with Preelaborate, Convention =&gt Intrinsic,
        Nonblocking, Global =&gt in out synchronized;

```

{AI95-00260-02} Tags.Generic_Dispatching_Constructor provides a mechanism to create an object of an appropriate type from just a tag value. The function Constructor is expected to create the object given a reference to an object of type Parameters.

Discussion: This specification is designed to make it easy to create dispatching constructors for streams; in particular, this can be used to construct overridings for T'Class'Input.

{AI12-0005-1} Note that almost any tagged type can be used in an instance of Generic_Dispatching_Constructor. Using a tagged incomplete view or a tagged partial view before the completion of the type in such an instance would be illegal; all other tagged types can be used in an instance of Generic_Dispatching_Constructor. 


#### Dynamic Semantics

The tag associated with an object of a tagged type is determined as follows: 

The tag of a stand-alone object, a component, or an [aggregate](./AA-4.3#S0106) of a specific tagged type T identifies T. 

Discussion: The tag of a formal parameter of type T is not necessarily the tag of T, if, for example, the actual was a type conversion. 

{AI12-0437-1} The tag of an object created by an allocator for an access type with a specific designated tagged type T identifies T. 

Discussion: The tag of an object designated by a value of such an access type might not be T, if, for example, the access value is the result of a type conversion.

The tag of an object of a class-wide tagged type is that of its initialization expression. 

Ramification: The tag of an object (even a class-wide one) cannot be changed after it is initialized, since a "class-wide" [assignment_statement](./AA-5.2#S0173) raises Constraint_Error if the tags don't match, and a "specific" [assignment_statement](./AA-5.2#S0173) does not affect the tag. 

The tag of the result returned by a function whose result type is a specific tagged type T identifies T. 

Implementation Note: {AI95-00318-02} For a limited tagged type, the return object is "built in place" in the ultimate result object with the appropriate tag. For a nonlimited type, a new anonymous object with the appropriate tag is created as part of the function return. See 6.5, "Return Statements". 

{AI95-00318-02} The tag of the result returned by a function with a class-wide result type is that of the return object. 

The tag is preserved by type conversion and by parameter passing. The tag of a value is the tag of the associated object (see 6.2).

{AI95-00260-02} {AI95-00344-01} {AI95-00405-01} {AI05-0092-1} {AI05-0262-1} Tag_Error is raised by a call of Descendant_Tag, Expanded_Name, External_Tag, Interface_Ancestor_Tags, Is_Abstract, Is_Descendant_At_Same_Level, Parent_Tag, Wide_Expanded_Name, or Wide_Wide_Expanded_Name if any tag passed is No_Tag.

{AI95-00260-02} An instance of Tags.Generic_Dispatching_Constructor raises Tag_Error if The_Tag does not represent a concrete descendant of T or if the innermost master (see 7.6.1) of this descendant is not also a master of the instance. Otherwise, it dispatches to the primitive function denoted by the formal Constructor for the type identified by The_Tag, passing Params, and returns the result. Any exception raised by the function is propagated.

Ramification: The tag check checks both that The_Tag is in T'Class, and that it is not abstract. These checks are similar to the ones required by streams for T'Class'Input (see 13.13.2). In addition, there is a check that the tag identifies a type declared on the current dynamic call chain, and not a more nested type or a type declared by another task. This check is not necessary for streams, because the stream attributes are declared at the same dynamic level as the type used. 


#### Erroneous Execution

{AI95-00260-02} If an internal tag provided to an instance of Tags.Generic_Dispatching_Constructor or to any subprogram declared in package Tags identifies either a type that is not library-level and whose tag has not been created (see 13.14), or a type that does not exist in the partition at the time of the call, then execution is erroneous.

Ramification: One reason that a type might not exist in the partition is that the tag refers to a type whose declaration was elaborated as part of an execution of a [subprogram_body](./AA-6.3#S0216) which has been left (see 7.6.1).

We exclude tags of library-level types from the current execution of the partition, because misuse of such tags should always be detected. T'Tag freezes the type (and thus creates the tag), and Internal_Tag and Descendant_Tag cannot return the tag of a library-level type that has not been created. All ancestors of a tagged type must be frozen no later than the (full) declaration of a type that uses them, so Parent_Tag and Interface_Ancestor_Tags cannot return a tag that has not been created. Finally, library-level types never cease to exist while the partition is executing. Thus, if the tag comes from a library-level type, there cannot be erroneous execution (the use of Descendant_Tag rather than Internal_Tag can help ensure that the tag is of a library-level type). This is also similar to the rules for T'Class'Input (see 13.13.2). 

Discussion: {AI95-00344-01} Ada 95 allowed Tag_Error in this case, or expected the functions to work. This worked because most implementations used tags constructed at link-time, and each elaboration of the same [type_declaration](./AA-3.2#S0023) produced the same tag. However, Ada 2005 requires at least part of the tags to be dynamically constructed for a type derived from a type at a shallower level. For dynamically constructed tags, detecting the error can be expensive and unreliable. To see this, consider a program containing two tasks. Task A creates a nested tagged type, passes the tag to task B (which saves it), and then terminates. The nested tag (if dynamic) probably will need to refer in some way to the stack frame for task A. If task B later tries to use the tag created by task A, the tag's reference to the stack frame of A probably is a dangling pointer. Avoiding this would require some sort of protected tag manager, which would be a bottleneck in a program's performance. Moreover, we'd still have a race condition; if task A terminated after the tag check, but before the tag was used, we'd still have a problem. That means that all of these operations would have to be serialized. That could be a significant performance drain, whether or not nested tagged types are ever used. Therefore, we allow execution to become erroneous as we do for other dangling pointers. If the implementation can detect the error, we recommend that Tag_Error be raised. 


#### Implementation Permissions

{AI95-00260-02} {AI95-00279-01} The implementation of Internal_Tag and Descendant_Tag may raise Tag_Error if no specific type corresponding to the string External passed as a parameter exists in the partition at the time the function is called, or if there is no such type whose innermost master is a master of the point of the function call. 

Reason: {AI95-00260-02} {AI95-00279-01} {AI95-00344-01} Locking would be required to ensure that the mapping of strings to tags never returned tags of types which no longer exist, because types can cease to exist (because they belong to another task, as described above) during the execution of these operations. Moreover, even if these functions did use locking, that would not prevent the type from ceasing to exist at the instant that the function returned. Thus, we do not require the overhead of locking; hence the word "may" in this rule. 


#### Implementation Advice

{AI95-00260-02} {AI05-0113-1} Internal_Tag should return the tag of a type, if one exists, whose innermost master is a master of the point of the function call. 

Implementation Advice: Tags.Internal_Tag should return the tag of a type, if one exists, whose innermost master is a master of the point of the function call.

Reason: {AI95-00260-02} {AI95-00344-01} It's not helpful if Internal_Tag returns the tag of some type in another task when one is available in the task that made the call. We don't require this behavior (because it requires the same implementation techniques we decided not to insist on previously), but encourage it. 

Discussion: {AI05-0113-1} There is no Advice for the result of Internal_Tag if no such type exists. In most cases, the Implementation Permission can be used to raise Tag_Error, but some other tag can be returned as well. 

NOTE 1   {AI12-0442-1} A type declared with the reserved word tagged is normally declared in a [package_specification](./AA-7.1#S0230), so that new primitive subprograms can be declared for it.

NOTE 2   Once an object has been created, its tag never changes.

NOTE 3   Class-wide types are defined to have unknown discriminants (see 3.7). This means that objects of a class-wide type have to be explicitly initialized (whether created by an [object_declaration](./AA-3.3#S0032) or an [allocator](./AA-4.8#S0164)), and that [aggregate](./AA-4.3#S0106)s have to be explicitly qualified with a specific type when their expected type is class-wide.

NOTE 4   {AI95-00260-02} {AI95-00326-01} The capability provided by Tags.Generic_Dispatching_Constructor is sometimes known as a factory. 


#### Examples

Examples of tagged record types: 

```ada
type Point is tagged
  record
    X, Y : Real := 0.0;
  end record;

```

```ada
type Expression is tagged null record;
  -- Components will be added by each extension

```


#### Extensions to Ada 83

Tagged types are a new concept. 


#### Inconsistencies With Ada 95

{AI95-00279-01} Amendment Correction: Added wording specifying that Internal_Tag must raise Tag_Error if the tag of a library-level type has not yet been created. Ada 95 gave an Implementation Permission to do this; we require it to avoid erroneous execution when streaming in an object of a library-level type that has not yet been elaborated. This is technically inconsistent; a program that used Internal_Tag outside of streaming and used a compiler that didn't take advantage of the Implementation Permission would not have raised Tag_Error, and may have returned a useful tag. (If the tag was used in streaming, the program would have been erroneous.) Since such a program would not have been portable to a compiler that did take advantage of the Implementation Permission, this is not a significant inconsistency.

{AI95-00417-01} We now define the lower bound of the string returned from [[Wide_]Wide_]Expanded_Name and External_Name. This makes working with the returned string easier, and is consistent with many other string-returning functions in Ada. This is technically an inconsistency; if a program depended on some other lower bound for the string returned from one of these functions, it could fail when compiled with Ada 2005. Such code is not portable even between Ada 95 implementations, so it should be very rare. 


#### Incompatibilities With Ada 95

{AI95-00260-02} {AI95-00344-01} {AI95-00400-01} {AI95-00405-01} {AI05-0005-1} Constant No_Tag, and functions Parent_Tag, Interface_Ancestor_Tags, Descendant_Tag, Is_Descendant_At_Same_Level, Wide_Expanded_Name, and Wide_Wide_Expanded_Name are added to Ada.Tags. If Ada.Tags is referenced in a [use_clause](./AA-8.4#S0235), and an entity E with the same [defining_identifier](./AA-3.1#S0022) as a new entity in Ada.Tags is defined in a package that is also referenced in a [use_clause](./AA-8.4#S0235), the entity E may no longer be use-visible, resulting in errors. This should be rare and is easily fixed if it does occur. 


#### Extensions to Ada 95

{AI95-00362-01} Ada.Tags is now defined to be preelaborated.

{AI95-00260-02} Generic function Tags.Generic_Dispatching_Constructor is new. 


#### Wording Changes from Ada 95

{AI95-00318-02} We talk about return objects rather than return expressions, as functions can return using an [extended_return_statement](./AA-6.5#S0225).

{AI95-00344-01} Added wording to define that tags for all descendants of a tagged type must be distinct. This is needed to ensure that more nested type extensions will work properly. The wording does not require implementation changes for types that were allowed in Ada 95. 


#### Inconsistencies With Ada 2005

{AI05-0113-1} Correction: Added wording specifying that Dependent_Tag must raise Tag_Error if there is more than one type which matches the requirements. If an implementation had returned a random tag of the matching types, a program may have worked properly. However, such a program would not be portable (another implementation may return a different tag) and the conditions that would cause the problem are unlikely (most likely, a tagged type extension declared in a generic body with multiple instances in the same scope). 


#### Incompatibilities With Ada 2005

{AI05-0173-1} Function Is_Abstract is added to Ada.Tags. If Ada.Tags is referenced in a [use_clause](./AA-8.4#S0235), and an entity E with the [defining_identifier](./AA-3.1#S0022) Is_Abstract is defined in a package that is also referenced in a [use_clause](./AA-8.4#S0235), the entity E may no longer be use-visible, resulting in errors. This should be rare and is easily fixed if it does occur. 


#### Wording Changes from Ada 2005

{AI05-0115-1} Correction: We explicitly define the meaning of "descendant" at runtime, so that it does not depend on visibility as does the usual meaning. 


## 3.9.1  Type Extensions

{AI95-00345-01} [ Every type extension is a tagged type, and is a record extension or a private extension of some other tagged type, or a noninterface synchronized tagged type.] 


#### Language Design Principles

We want to make sure that we can extend a generic formal tagged type, without knowing its discriminants.

We don't want to allow components in an extension aggregate to depend on discriminants inherited from the parent value, since such dependence requires staticness in aggregates, at least for variants. 


#### Syntax

record_extension_part<a id="S0075"></a> ::= with [record_definition](./AA-3.8#S0067)


#### Legality Rules

{AI95-00344-01} {AI95-00345-01} {AI95-00419-01} The parent type of a record extension shall not be a class-wide type nor shall it be a synchronized tagged type (see 3.9.4). If the parent type or any progenitor is nonlimited, then each of the components of the [record_extension_part](./AA-3.9#S0075) shall be nonlimited. In addition to the places where Legality Rules normally apply (see 12.3), these rules apply also in the private part of an instance of a generic unit. 

Reason: If the parent is a limited formal type, then the actual might be nonlimited.

{AI95-00344-01} Ada 95 required the record extensions to be the same level as the parent type. Now we use accessibility checks on class-wide [allocator](./AA-4.8#S0164)s and return statements to prevent objects from living longer than their type.

{AI95-00345-01} Synchronized tagged types cannot be extended. We have this limitation so that all of the data of a task or protected type is defined within the type. Data defined outside of the type wouldn't be subject to the mutual exclusion properties of a protected type, and couldn't be used by a task, and thus doesn't seem to be worth the potential impact on implementations. 

{AI95-00344-01} Within the body of a generic unit, or the body of any of its descendant library units, a tagged type shall not be declared as a descendant of a formal type declared within the formal part of the generic unit.

Reason: This paragraph ensures that a dispatching call will never attempt to execute an inaccessible subprogram body.

{AI95-00344-01} The convoluted wording ("formal type declared within the formal part") is necessary to include tagged types that are formal parameters of formal packages of the generic unit, as well as formal tagged and tagged formal derived types of the generic unit.

{AI95-00344-01} This rule is necessary in order to preserve the contract model.

{AI05-0005-1} {AI95-00344-01} If an ancestor is a formal of the generic unit , we have a problem because it might have an unknown number of subprograms that require overriding, as in the following example: 

```ada
package P is
    type T is tagged null record;
    function F return T; -- Inherited versions will require overriding.
end P;

```

```ada
generic
    type TT is tagged private;
package Gp is
    type NT is abstract new TT with null record;
    procedure Q(X : in NT) is abstract;
end Gp;

```

```ada
package body Gp is
    type NT2 is new NT with null record; -- Illegal!
    procedure Q(X : in NT2) is begin null; end Q;
    -- Is this legal or not? Can't decide because
    -- we don't know whether TT had any functions that require
    -- overriding on extension.
end Gp;

```

```ada
package I is new Gp(TT =&gt P.T);

```

I.NT is an abstract type with two abstract subprograms: F (inherited as abstract) and Q (explicitly declared as abstract). But the generic body doesn't know about F, so we don't know that it needs to be overridden to make a nonabstract extension of NT. Hence, we have to disallow this case.

Similarly, since the actual type for a formal tagged limited private type can be a nonlimited type, we would have a problem if a type extension of a limited private formal type could be declared in a generic body. Such an extension could have a task component, for example, and an object of that type could be passed to a dispatching operation of a nonlimited ancestor type. That operation could try to copy the object with the task component. That would be bad. So we disallow this as well.

If TT were declared as abstract, then we could have the same problem with abstract procedures.

We considered disallowing all tagged types in a generic body, for simplicity. We decided not to go that far, in order to avoid unnecessary restrictions.

We also considered trying make the accessibility level part of the contract; i.e. invent some way of saying (in the [generic_declaration](./AA-12.1#S0310)) "all instances of this generic unit will have the same accessibility level as the [generic_declaration](./AA-12.1#S0310)". Unfortunately, that doesn't solve the part of the problem having to do with abstract types.

This paragraph was deleted.

Ramification: {AI95-00344} This rule applies to types with ancestors (directly or indirectly) of formal interface types (see 12.5.5), formal tagged private types (see 12.5.1), and formal derived private types whose ancestor type is tagged (see 12.5.1). 


#### Static Semantics

{AI95-00391-01} A record extension is a null extension if its declaration has no [known_discriminant_part](./AA-3.7#S0061) and its [record_extension_part](./AA-3.9#S0075) includes no [component_declaration](./AA-3.8#S0070)s.

{AI12-0191-1} In the case where the (compile-time) view of an object X is of a tagged type T1 or T1'Class and the (run-time) tag of X is T2'Tag, only the components (if any) of X that are components of T1 (or that are discriminants which correspond to a discriminant of T1) are said to be components of the nominal type of X. Similarly, only parts (respectively, subcomponents) of T1 are parts (respectively, subcomponents) of the nominal type of X.

Ramification: {AI12-0191-1} For example, if T2 is an undiscriminated extension of T1 which declares a component named Comp, then X.Comp is not a component of the nominal type of X. 

Discussion: {AI12-0191-1} For example, there is a Dynamic Semantics rule that finalization of an object includes finalization of its components (see 7.6.1). In the following case: 

```ada
type T1 is tagged null record;
type T2 is new T1 with record
   Comp : Some_Controlled_Type;
end record;
function Func return T1'Class is (T2'(others =&gt &lt&gt));
X : T1'Class := Func;

```

the rule that "every component of the object is finalized" (as opposed to something like "every component of the nominal type of the object is finalized") means that the finalization of X will include finalization of X.Comp. For another example, see the rule about accessibility checking of access discriminants of parts of function results in 6.5. In contrast, the rules in 7.3.2 explicitly state that type invariant checks are only performed for parts which are of the type-invariant bearing type and which are parts of the nominal type of the object (as opposed to for all parts, whether part of the nominal type or not, which are of the invariant-bearing type). Similarly, the rule in 13.13.2 governing which components of a composite value are read and written by the default implementations of Read and Write for a composite type states that only the components of the object which are components of the nominal type of the object are read or written.


#### Dynamic Semantics

The elaboration of a [record_extension_part](./AA-3.9#S0075) consists of the elaboration of the [record_definition](./AA-3.8#S0067). 

NOTE 1   The term "type extension" refers to a type as a whole. The term "extension part" refers to the piece of text that defines the additional components (if any) the type extension has relative to its specified ancestor type. 

Discussion: We considered other terminology, such as "extended type". However, the terms "private extended type" and "record extended type" did not convey the proper meaning. Hence, we have chosen to uniformly use the term "extension" as the type resulting from extending a type, with "private extension" being one produced by privately extending the type, and "record extension" being one produced by extending the type with an additional record-like set of components. Note also that the term "type extension" refers to the result of extending a type in the language Oberon as well (though there the term "extended type" is also used, interchangeably, perhaps because Oberon doesn't have the concept of a "private extension"). 

NOTE 2   {AI95-00344-01} When an extension is declared immediately within a body, primitive subprograms are inherited and are overridable, but new primitive subprograms cannot be added.

NOTE 3   A [name](./AA-4.1#S0091) that denotes a component (including a discriminant) of the parent type is not allowed within the [record_extension_part](./AA-3.9#S0075). Similarly, a [name](./AA-4.1#S0091) that denotes a component defined within the [record_extension_part](./AA-3.9#S0075) is not allowed within the [record_extension_part](./AA-3.9#S0075). It is permissible to use a [name](./AA-4.1#S0091) that denotes a discriminant of the record extension, providing there is a new [known_discriminant_part](./AA-3.7#S0061) in the enclosing type declaration. (The full rule is given in 3.8.) 

Reason: The restriction against depending on discriminants of the parent is to simplify the definition of extension aggregates. The restriction against using parent components in other ways is methodological; it presumably simplifies implementation as well. 

NOTE 4   Each visible component of a record extension has to have a unique name, whether the component is (visibly) inherited from the parent type or declared in the [record_extension_part](./AA-3.9#S0075) (see 8.3). 


#### Examples

Examples of record extensions (of types defined above in 3.9): 

```ada
type Painted_Point is new Point with
  record
    Paint : Color := White;
  end record;
    -- Components X and Y are inherited

```

```ada
Origin : constant Painted_Point := (X | Y =&gt 0.0, Paint =&gt Black);

```

```ada
type Literal is new Expression with
  record                 -- a leaf in an Expression tree
    Value : Real;
  end record;

```

```ada
{AI12-0404-1} type Expr_Ptr is access all Expression'Class;
                               -- see 3.9

```

```ada
type Binary_Operation is new Expression with
  record                 -- an internal node in an Expression tree
    Left, Right : Expr_Ptr;
  end record;

```

```ada
type Addition is new Binary_Operation with null record;
type Subtraction is new Binary_Operation with null record;
  -- No additional components needed for these extensions

```

```ada
Tree : Expr_Ptr :=         -- A tree representation of "5.0 + (13.07.0)"
   new Addition'(
      Left  =&gt new Literal'(Value =&gt 5.0),
      Right =&gt new Subtraction'(
         Left  =&gt new Literal'(Value =&gt 13.0),
         Right =&gt new Literal'(Value =&gt 7.0)));

```


#### Extensions to Ada 83

Type extension is a new concept. 


#### Extensions to Ada 95

{AI95-00344-01} Type extensions now can be declared in more nested scopes than their parent types. Additional accessibility checks on [allocator](./AA-4.8#S0164)s and return statements prevent objects from outliving their type. 


#### Wording Changes from Ada 95

{AI95-00345-01} Added wording to prevent extending synchronized tagged types.

{AI95-00391-01} Defined null extension for use elsewhere. 


#### Wording Changes from Ada 2012

{AI12-0191-1} Defined the term "components of the nominal type" to remove a confusion as to how components are described in Dynamic Semantics rules. 


## 3.9.2  Dispatching Operations of Tagged Types

{AI95-00260-02} {AI95-00335-01} {AI12-0342-1} {AI12-0419-1} The primitive subprograms of a tagged type, the subprograms declared by [formal_abstract_subprogram_declaration](./AA-12.6#S0337)s, the Put_Image attribute (see 4.10) of a specific tagged type, and the stream attributes of a specific tagged type that are available (see 13.13.2) at the end of the declaration list where the type is declared are called dispatching operations. [A dispatching operation can be called using a statically determined controlling tag, in which case the body to be executed is determined at compile time. Alternatively, the controlling tag can be dynamically determined, in which case the call dispatches to a body that is determined at run time;] such a call is termed a dispatching call. [As explained below, the properties of the operands and the context of a particular call on a dispatching operation determine how the controlling tag is determined, and hence whether or not the call is a dispatching call. Run-time polymorphism is achieved when a dispatching operation is called by a dispatching call.] 

Reason: {AI95-00335-01} For the stream attributes of a type declared immediately within a [package_specification](./AA-7.1#S0230) that has a partial view, the declaration list to consider is the visible part of the package. Stream attributes that are not available in the same declaration list are not dispatching as there is no guarantee that descendants of the type have available attributes (there is such a guarantee for visibly available attributes). If we allowed dispatching for any available attribute, then for attributes defined in the private part we could end up executing a nonexistent body. 


#### Language Design Principles

The controlling tag determination rules are analogous to the overload resolution rules, except they deal with run-time type identification (tags) rather than compile-time type resolution. As with overload resolution, controlling tag determination may depend on operands or result context. 


#### Static Semantics

{AI95-00260-02} {AI95-00416-01} {AI05-0076-1} A call on a dispatching operation is a call whose [name](./AA-4.1#S0091) or [prefix](./AA-4.1#S0093) denotes the declaration of a dispatching operation. A controlling operand in a call on a dispatching operation of a tagged type T is one whose corresponding formal parameter is of type T or is of an anonymous access type with designated type T; the corresponding formal parameter is called a controlling formal parameter. If the controlling formal parameter is an access parameter, the controlling operand is the object designated by the actual parameter, rather than the actual parameter itself. If the call is to a (primitive) function with result type T (a function with a controlling result), then the call has a controlling result - the context of the call can control the dispatching. Similarly, if the call is to a function with an access result type designating T (a function with a controlling access result), then the call has a controlling access result, and the context can similarly control dispatching. 

Ramification: This definition implies that a call through the dereference of an access-to-subprogram value is never considered a call on a dispatching operation. Note also that if the [prefix](./AA-4.1#S0093) denotes a [renaming_declaration](./AA-8.5#S0238), the place where the renaming occurs determines whether it is primitive; the thing being renamed is irrelevant. 

{AI12-0236-1} A [name](./AA-4.1#S0091) or expression of a tagged type is either statically tagged, dynamically tagged, or tag indeterminate, according to whether, when used as a controlling operand, the tag that controls dispatching is determined statically by the operand's (specific) type, dynamically by its tag at run time, or from context. A [qualified_expression](./AA-4.7#S0163) or parenthesized expression is statically, dynamically, or indeterminately tagged according to its operand. A [conditional_expression](./AA-4.5#S0148) is statically, dynamically, or indeterminately tagged according to rules given in 4.5.7. A [declare_expression](./AA-4.5#S0156) is statically, dynamically, or indeterminately tagged according to its body_[expression](./AA-4.4#S0132). For other kinds of [name](./AA-4.1#S0091)s and expressions, this is determined as follows: 

{AI95-00416-01} The [name](./AA-4.1#S0091) or expression is statically tagged if it is of a specific tagged type and, if it is a call with a controlling result or controlling access result, it has at least one statically tagged controlling operand; 

Discussion: It is illegal to have both statically tagged and dynamically tagged controlling operands in the same call -- see below. 

{AI95-00416-01} The [name](./AA-4.1#S0091) or expression is dynamically tagged if it is of a class-wide type, or it is a call with a controlling result or controlling access result and at least one dynamically tagged controlling operand;

{AI95-00416-01} The [name](./AA-4.1#S0091) or expression is tag indeterminate if it is a call with a controlling result or controlling access result, all of whose controlling operands (if any) are tag indeterminate. 

{8652/0010} {AI95-00127-01} [A [type_conversion](./AA-4.6#S0162) is statically or dynamically tagged according to whether the type determined by the [subtype_mark](./AA-3.2#S0028) is specific or class-wide, respectively.] For an object that is designated by an expression whose expected type is an anonymous access-to-specific tagged type, the object is dynamically tagged if the expression, ignoring enclosing parentheses, is of the form X'Access, where X is of a class-wide type, or is of the form new T'(...), where T denotes a class-wide subtype. Otherwise, the object is statically or dynamically tagged according to whether the designated type of the type of the expression is specific or class-wide, respectively. 

Ramification: A [type_conversion](./AA-4.6#S0162) is never tag indeterminate, even if its operand is. A designated object is never tag indeterminate.

{8652/0010} {AI95-00127-01} Allocators and access attributes of class-wide types can be used as the controlling parameters of dispatching calls. 


#### Legality Rules

A call on a dispatching operation shall not have both dynamically tagged and statically tagged controlling operands. 

Reason: This restriction is intended to minimize confusion between whether the dynamically tagged operands are implicitly converted to, or tag checked against the specific type of the statically tagged operand(s). 

{8652/0010} {AI95-00127-01} If the expected type for an expression or [name](./AA-4.1#S0091) is some specific tagged type, then the expression or [name](./AA-4.1#S0091) shall not be dynamically tagged unless it is a controlling operand in a call on a dispatching operation. Similarly, if the expected type for an expression is an anonymous access-to-specific tagged type, then the object designated by the expression shall not be dynamically tagged unless it is a controlling operand in a call on a dispatching operation. 

Reason: This prevents implicit "truncation" of a dynamically-tagged value to the specific type of the target object/formal. An explicit conversion is required to request this truncation. 

Ramification: {AI95-00252-01} {AI12-0039-1} This rule applies to all expressions or [name](./AA-4.1#S0091)s with a specific expected type, not just those that are actual parameters to a dispatching call. This rule does not apply to a membership test whose tested_[simple_expression](./AA-4.4#S0138) is class-wide, since any type that covers the tested type is explicitly allowed. See 4.5.2. This rule also doesn't apply to a [selected_component](./AA-4.1#S0098) whose [selector_name](./AA-4.1#S0099) is a subprogram, since the rules explicitly say that the prefix may be class-wide (see 4.1.3). 

{8652/0011} {AI95-00117-01} {AI95-00430-01} In the declaration of a dispatching operation of a tagged type, everywhere a subtype of the tagged type appears as a subtype of the profile (see 6.1), it shall statically match the first subtype of the tagged type. If the dispatching operation overrides an inherited subprogram, it shall be subtype conformant with the inherited subprogram. The convention of an inherited dispatching operation is the convention of the corresponding primitive operation of the parent or progenitor type. The default convention of a dispatching operation that overrides an inherited primitive operation is the convention of the inherited operation; if the operation overrides multiple inherited operations, then they shall all have the same convention. An explicitly declared dispatching operation shall not be of convention Intrinsic. 

Reason: These rules ensure that constraint checks can be performed by the caller in a dispatching call, and parameter passing conventions match up properly. A special rule on aggregates prevents values of a tagged type from being created that are outside of its first subtype. 

{AI95-00416-01} The [default_expression](./AA-3.7#S0063) for a controlling formal parameter of a dispatching operation shall be tag indeterminate. 

Reason: {AI95-00416-01} This rule ensures that the [default_expression](./AA-3.7#S0063) always produces the "correct" tag when called with or without dispatching, or when inherited by a descendant. If it were statically tagged, the default would be useless for a dispatching call; if it were dynamically tagged, the default would be useless for a nondispatching call.

{AI95-00404-01} If a dispatching operation is defined by a [subprogram_renaming_declaration](./AA-8.5#S0242) or the instantiation of a generic subprogram, any access parameter of the renamed subprogram or the generic subprogram that corresponds to a controlling access parameter of the dispatching operation, shall have a subtype that excludes null.

A given subprogram shall not be a dispatching operation of two or more distinct tagged types. 

Reason: This restriction minimizes confusion since multiple dispatching is not provided. The normal solution is to replace all but one of the tagged types with their class-wide types. 

Ramification: {8652/0098} {AI95-00183-01} This restriction applies even if the partial view (see 7.3) of one or both of the types is untagged. This follows from the definition of dispatching operation: the operation is a dispatching operation anywhere the full views of the (tagged) types are visible. 

The explicit declaration of a primitive subprogram of a tagged type shall occur before the type is frozen (see 13.14). [For example, new dispatching operations cannot be added after objects or values of the type exist, nor after deriving a record extension from it, nor after a body.]

Reason: {AI95-00344-01} This rule is needed because (1) we don't want people dispatching to things that haven't been declared yet, and (2) we want to allow the static part of tagged type descriptors to be static (allocated statically, and initialized to link-time-known symbols). Suppose T2 inherits primitive P from T1, and then overrides P. Suppose P is called before the declaration of the overriding P. What should it dispatch to? If the answer is the new P, we've violated the first principle above. If the answer is the old P, we've violated the second principle. (A call to the new one necessarily raises Program_Error, but that's beside the point.)

Note that a call upon a dispatching operation of type T will freeze T.

We considered applying this rule to all derived types, for uniformity. However, that would be upward incompatible, so we rejected the idea. As in Ada 83, for an untagged type, the above call upon P will call the old P (which is arguably confusing). 

Implementation Note: {AI95-00326-01} Because of this rule, the type descriptor can be created (presumably containing linker symbols pointing at the not-yet-compiled bodies) at the first freezing point of the type. It also prevents, for a (nonincomplete) tagged type declared in a [package_specification](./AA-7.1#S0230), overriding in the body or by a child subprogram. 

Ramification: {AI95-00251-01} A consequence is that for a tagged type declaration in a [declarative_part](./AA-3.11#S0086), only the last (overriding) primitive subprogram can be declared by a [subprogram_body](./AA-6.3#S0216). (Other overridings must be provided by [subprogram_declaration](./AA-6.1#S0195)s.) 

To be honest: {AI05-0222-1} This rule applies only to "original" declarations and not to the completion of a primitive subprogram, even though a completion is technically an explicit declaration, and it may declare a primitive subprogram. 


#### Dynamic Semantics

For the execution of a call on a dispatching operation of a type T, the controlling tag value determines which subprogram body is executed. The controlling tag value is defined as follows: 

If one or more controlling operands are statically tagged, then the controlling tag value is statically determined to be the tag of T.

If one or more controlling operands are dynamically tagged, then the controlling tag value is not statically determined, but is rather determined by the tags of the controlling operands. If there is more than one dynamically tagged controlling operand, a check is made that they all have the same tag. If this check fails, Constraint_Error is raised unless the call is a [function_call](./AA-6.4#S0218) whose [name](./AA-4.1#S0091) denotes the declaration of an equality operator (predefined or user defined) that returns Boolean, in which case the result of the call is defined to indicate inequality, and no [subprogram_body](./AA-6.3#S0216) is executed. This check is performed prior to evaluating any tag-indeterminate controlling operands. 

Reason: Tag mismatch is considered an error (except for "=" and "/=") since the corresponding primitive subprograms in each specific type expect all controlling operands to be of the same type. For tag mismatch with an equality operator, rather than raising an exception, "=" returns False and "/=" returns True. No equality operator is actually invoked, since there is no common tag value to control the dispatch. Equality is a special case to be consistent with the existing Ada 83 principle that equality comparisons, even between objects with different constraints, never raise Constraint_Error. 

{AI95-00196-01} If all of the controlling operands (if any) are tag-indeterminate, then: 

{AI95-00239-01} {AI95-00416-01} If the call has a controlling result or controlling access result and is itself, or designates, a (possibly parenthesized or qualified) controlling operand of an enclosing call on a dispatching operation of a descendant of type T, then its controlling tag value is determined by the controlling tag value of this enclosing call;

Discussion: {AI95-00239-01} For code that a user can write explicitly, the only contexts that can control dispatching of a function with a controlling result of type T are those that involve controlling operands of the same type T: if the two types differ there is an illegality and the dynamic semantics are irrelevant.

In the case of an inherited subprogram however, if a default expression is a function call, it may be of type T while the parameter is of a type derived from T. To cover this case, we talk about "a descendant of T" above. This is safe, because if the type of the parameter is descended from the type of the function result, it is guaranteed to inherit or override the function, and this ensures that there will be an appropriate body to dispatch to. Note that abstract functions are not an issue here because the call to the function is a dispatching call, so it is guaranteed to always land on a concrete body. 

{AI95-00196-01} {AI95-00416-01} If the call has a controlling result or controlling access result and (possibly parenthesized, qualified, or dereferenced) is the expression of an [assignment_statement](./AA-5.2#S0173) whose target is of a class-wide type, then its controlling tag value is determined by the target;

Otherwise, the controlling tag value is statically determined to be the tag of type T. 

Ramification: This includes the cases of a tag-indeterminate procedure call, and a tag-indeterminate [function_call](./AA-6.4#S0218) that is used to initialize a class-wide formal parameter or class-wide object. 

{AI95-00345-01} {AI05-0126-1} For the execution of a call on a dispatching operation, the action performed is determined by the properties of the corresponding dispatching operation of the specific type identified by the controlling tag value:

{AI05-0126-1} if the corresponding operation is explicitly declared for this type, [even if the declaration occurs in a private part], then the action comprises an invocation of the explicit body for the operation;

{AI95-00345-01} {AI05-0126-1} if the corresponding operation is implicitly declared for this type and is implemented by an entry or protected subprogram (see 9.1 and 9.4), then the action comprises a call on this entry or protected subprogram, with the target object being given by the first actual parameter of the call, and the actual parameters of the entry or protected subprogram being given by the remaining actual parameters of the call, if any;

{AI05-0197-1} if the corresponding operation is a predefined operator then the action comprises an invocation of that operator;

{AI95-00345-01} {AI05-0126-1} {AI05-0197-1} {AI05-0250-1} {AI05-0254-1} otherwise, the action is the same as the action for the corresponding operation of the parent type or progenitor type from which the operation was inherited except that additional invariant checks (see 7.3.2) and class-wide postcondition checks (see 6.1.1) may apply. If there is more than one such corresponding operation, the action is that for the operation that is not a null procedure, if any; otherwise, the action is that of an arbitrary one of the operations. 

This paragraph was deleted.{AI05-0126-1} 

Ramification: {AI05-0005-1} {AI05-0126-1} "Corresponding dispatching operation" refers to the inheritance relationship between subprograms. Primitive operations are always inherited for a type T, but they might not be declared if the primitive operation is never visible within the immediate scope of the type T. If no corresponding operation is declared, the last bullet is used and the corresponding operation of the parent type is executed (an explicit body that happens to have the same name and profile is not called in that case).

{AI05-0005-1} {AI05-0126-1} We have to talk about progenitors in the last bullet in case the corresponding operation is a null procedure inherited from an interface. In that case, the parent type might not even have the operation in question.

{AI05-0197-1} For the last bullet, if there are multiple corresponding operations for the parent and progenitors, all but one of them have to be a null procedure. (If the progenitors declared abstract routines, there would have to be an explicit overriding of the operation, and then the first bullet would apply.) We call the nonnull routine if one exists.

{AI05-0126-1} Any explicit declaration for an inherited corresponding operation has to be an overriding routine. These rules mean that a dispatching call executes the overriding routine (if any) for the specific type. 

Reason: {AI05-0005-1} The wording of the above rules is intended to ensure that the same body is executed for a given tag, whether that tag is determined statically or dynamically. For a type declared in a package, it doesn't matter whether a given subprogram is overridden in the visible part or the private part, and it doesn't matter whether the call is inside or outside the package. For example: 

```ada
package P1 is
    type T1 is tagged null record;
    procedure Op_A(Arg : in T1);
    procedure Op_B(Arg : in T1);
end P1;

```

```ada
with P1; use P1;
package P2 is
    type T2 is new T1 with null record;
    procedure Op_A(Param : in T2);
private
    procedure Op_B(Param : in T2);
end P2;

```

```ada
with P1; with P2;
procedure Main is
    X : P2.T2;
    Y : P1.T1'Class := X;
begin
    P2.Op_A(Param =&gt X); -- Nondispatching call to a dispatching operation.
    P1.Op_A(Arg =&gt Y); -- Dispatching call.
    P2.Op_B(Arg =&gt X); -- Nondispatching call to a dispatching operation.
    P1.Op_B(Arg =&gt Y); -- Dispatching call.
end Main;

```

The two calls to Op_A both execute the body of Op_A that has to occur in the body of package P2. Similarly, the two calls to Op_B both execute the body of Op_B that has to occur in the body of package P2, even though Op_B is overridden in the private part of P2. Note, however, that the formal parameter names are different for P2.Op_A versus P2.Op_B. The overriding declaration for P2.Op_B is not visible in Main, so the name in the call actually denotes the implicit declaration of Op_B inherited from T1.

If a call occurs in the program text before an overriding, which can happen only if the call is part of a default expression, the overriding will still take effect for that call.

Implementation Note: Even when a tag is not statically determined, a compiler might still be able to figure it out and thereby avoid the overhead of run-time dispatching. 

NOTE 1   The body to be executed for a call on a dispatching operation is determined by the tag; it does not matter whether that tag is determined statically or dynamically, and it does not matter whether the subprogram's declaration is visible at the place of the call.

NOTE 2   {AI95-00260-02} This subclause covers calls on dispatching subprograms of a tagged type. Rules for tagged type membership tests are described in 4.5.2. Controlling tag determination for an [assignment_statement](./AA-5.2#S0173) is described in 5.2.

NOTE 3   A dispatching call can dispatch to a body whose declaration is not visible at the place of the call.

NOTE 4   A call through an access-to-subprogram value is never a dispatching call, even if the access value designates a dispatching operation. Similarly a call whose [prefix](./AA-4.1#S0093) denotes a [subprogram_renaming_declaration](./AA-8.5#S0242) cannot be a dispatching call unless the renaming itself is the declaration of a primitive subprogram. 


#### Extensions to Ada 83

The concept of dispatching operations is new. 


#### Incompatibilities With Ada 95

{AI95-00404-01} If a dispatching operation is defined by a [subprogram_renaming_declaration](./AA-8.5#S0242), and it has a controlling access parameter, Ada 2005 requires the subtype of the parameter to exclude null. The same applies to instantiations. This is required so that all calls to the subprogram operate the same way (controlling access parameters have to exclude null so that dispatching calls will work). Since Ada 95 didn't have the notion of access subtypes that exclude null, and all access parameters excluded null, it had no such rules. These rules will require the addition of an explicit not null on nondispatching operations that are later renamed to be dispatching, or on a generic that is used to define a dispatching operation. 


#### Extensions to Ada 95

{AI95-00416-01} Functions that have an access result type can be dispatching in the same way as a function that returns a tagged object directly. 


#### Wording Changes from Ada 95

{8652/0010} {AI95-00127-01} {AI05-0299-1} Corrigendum: Allocators and access attributes of objects of class-wide types can be used as the controlling parameter in a dispatching calls. This was an oversight in the definition of Ada 95. (See 3.10.2 and 4.8).

{8652/0011} {AI95-00117-01} {AI95-00430-01} Corrigendum: Corrected the conventions of dispatching operations. This is extended in Ada 2005 to cover operations inherited from progenitors, and to ensure that the conventions of all inherited operations are the same.

{AI95-00196-01} Clarified the wording to ensure that functions with no controlling operands are tag-indeterminate, and to describe that the controlling tag can come from the target of an [assignment_statement](./AA-5.2#S0173).

{AI95-00239-01} Fixed the wording to cover default expressions inherited by derived subprograms. A literal reading of the old wording would have implied that operations would be called with objects of the wrong type.

{AI95-00260-02} An abstract formal subprogram is a dispatching operation, even though it is not a primitive operation. See 12.6, "Formal Subprograms".

{AI95-00345-01} Dispatching calls include operations implemented by entries and protected operations, so we have to update the wording to reflect that.

{AI95-00335-01} A stream attribute of a tagged type is usually a dispatching operation, even though it is not a primitive operation. If they weren't dispatching, T'Class'Input and T'Class'Output wouldn't work. 


#### Wording Changes from Ada 2005

{AI05-0076-1} Correction: Defined "function with a controlling result", as it is used in 3.9.3.

{AI05-0126-1} {AI05-0197-1} Correction: Corrected holes in the definition of dynamic dispatching: the behavior for operations that are never declared and/or inherited from a progenitor were not specified. 


#### Wording Changes from Ada 2012

{AI12-0236-1} Added a special rule for taggedness of [declare_expression](./AA-4.5#S0156)s, and added a pointer to the existing special rules for [conditional_expression](./AA-4.5#S0148)s, as the list of exceptions to the usual rules appear to be exhaustive.

{AI12-0419-1} Added wording to clarify that the Put_Image attribute is a dispatching operation. Put_Image follows the model of stream-oriented attributes, and thus need to be mentioned in the same place. 


## 3.9.3  Abstract Types and Subprograms

{AI95-00345-01} [ An abstract type is a tagged type intended for use as an ancestor of other types, but which is not allowed to have objects of its own. An abstract subprogram is a subprogram that has no body, but is intended to be overridden at some point when inherited. Because objects of an abstract type cannot be created, a dispatching call to an abstract subprogram always dispatches to some overriding body.] 

Glossary entry: An abstract type is a tagged type intended for use as an ancestor of other types, but which is not allowed to have objects of its own.

Version=[5],Kind=(Added),Group=[T],Term=[abstract type], Def=[a tagged type intended for use as an ancestor of other types, but which is not allowed to have objects of its own] 


#### Language Design Principles

{AI05-0299-1} An abstract subprogram has no body, so the rules in this subclause are designed to ensure (at compile time) that the body will never be invoked. We do so primarily by disallowing the creation of values of the abstract type. Therefore, since type conversion and parameter passing don't change the tag, we know we will never get a class-wide value with a tag identifying an abstract type. This means that we only have to disallow nondispatching calls on abstract subprograms (dispatching calls will never reach them). 


#### Syntax

{AI95-00218-03} {AI95-00348-01} {AI05-0183-1} abstract_subprogram_declaration<a id="S0076"></a> ::= 
    [[overriding_indicator](./AA-8.3#S0234)]
    [subprogram_specification](./AA-6.1#S0196) is abstract
        [[aspect_specification](./AA-13.1#S0346)];


#### Static Semantics

{AI95-00345-01} Interface types (see 3.9.4) are abstract types. In addition, a tagged type that has the reserved word abstract in its declaration is an abstract type. The class-wide type (see 3.4.1) rooted at an abstract type is not itself an abstract type. 


#### Legality Rules

{AI95-00345-01} Only a tagged type shall have the reserved word abstract in its declaration. 

Ramification: Untagged types are never abstract, even though they can have primitive abstract subprograms. Such subprograms cannot be called, unless they also happen to be dispatching operations of some tagged type, and then only via a dispatching call.

Class-wide types are never abstract. If T is abstract, then it is illegal to declare a stand-alone object of type T, but it is OK to declare a stand-alone object of type T'Class; the latter will get a tag from its initial value, and this tag will necessarily be different from T'Tag. 

{AI95-00260-02} {AI95-00348-01} A subprogram declared by an [abstract_subprogram_declaration](./AA-3.9#S0076) or a [formal_abstract_subprogram_declaration](./AA-12.6#S0337) (see 12.6) is an abstract subprogram. If it is a primitive subprogram of a tagged type, then the tagged type shall be abstract. 

Ramification: Note that for a private type, this applies to both views. The following is illegal: 

```ada
package P is
    type T is abstract tagged private;
    function Foo (X : T) return Boolean is abstract; -- Illegal!
private
    type T is tagged null record; -- Illegal!
    X : T;
    Y : Boolean := Foo (T'Class (X));
end P;

```

The full view of T is not abstract, but has an abstract operation Foo, which is illegal. The two lines marked "-- Illegal!" are illegal when taken together. 

Reason: {AI95-00310-01} We considered disallowing untagged types from having abstract primitive subprograms. However, we rejected that plan, because it introduced some silly anomalies, and because such subprograms are harmless. For example: 

```ada
package P is
   type Field_Size is range 0..100;
   type T is abstract tagged null record;
   procedure Print(X : in T; F : in Field_Size := 0) is abstract;
  . . .
package Q is
   type My_Field_Size is new Field_Size;
   -- implicit declaration of Print(X : T; F : My_Field_Size := 0) is abstract;
end Q;

```

It seemed silly to make the derivative of My_Field_Size illegal, just because there was an implicitly declared abstract subprogram that was not primitive on some tagged type. Other rules could be formulated to solve this problem, but the current ones seem like the simplest.

{AI95-00310-01} In Ada 2005, abstract primitive subprograms of an untagged type may be used to "undefine" an operation. 

Ramification: {AI95-00260-02} Note that the second sentence does not apply to abstract formal subprograms, as they are never primitive operations of a type. 

{AI95-00251-01} {AI95-00334-01} {AI95-00391-01} {AI05-0097-1} {AI05-0198-1} If a type has an implicitly declared primitive subprogram that is inherited or is a predefined operator, and the corresponding primitive subprogram of the parent or ancestor type is abstract or is a function with a controlling access result, or if a type other than a nonabstract null extension inherits a function with a controlling result, then: 

Ramification: {AI05-0068-1} These rules apply to each view of the type individually. That is necessary to preserve privacy. For instance, in the following example: 

```ada
package P is
   type I is interface;
   procedure Op (X : I) is abstract;
end P;

```

```ada
with P;
package Q is
   type T is abstract new P.I with private;
   -- Op inherited here.
private
   type T is abstract new P.I with null record;
   procedure Op (X : T) is null;
end Q;

```

```ada
with Q;
package R is
   type T2 is new Q.T with null record;
   -- Illegal. Op inherited here, but requires overriding.
end R;

```

If this did not depend on the view, this would be legal. But in that case, the fact that Op is overridden in the private part would be visible; package R would have to be illegal if no overriding was in the private part.

Note that this means that whether an inherited subprogram is abstract or concrete depends on where it inherited. In the case of Q, Q.Op in the visible part is abstract, while Q.Op in the private part is concrete. That is, R is illegal since it is an unrelated unit (and thus it cannot see the private part), but if R had been a private child of Q, it would have been legal. 

{AI95-00251-01} {AI95-00334-01} If the type is abstract or untagged, the implicitly declared subprogram is abstract. 

Ramification: Note that it is possible to override a concrete subprogram with an abstract one. 

{AI95-00391-01} {AI12-0080-1} {AI12-0444-1} Otherwise, the subprogram shall be overridden with a nonabstract subprogram or, in the case of a private extension inheriting a nonabstract function with a controlling result, have a full type that is a null extension[; for a type declared in the visible part of a package, the overriding may be either in the visible or the private part]. Such a subprogram is said to require overriding. However, if the type is a generic formal type, the subprogram is allowed to be inherited as is, without being overridden for the formal type itself; [a nonabstract version will necessarily be provided by the actual type.] 

Reason: {AI95-00228-01} {AI95-00391-01} A function that returns the parent type requires overriding for a type extension (or becomes abstract for an abstract type) because conversion from a parent type to a type extension is not defined, and function return semantics is defined in terms of conversion (other than for a null extension; see below). (Note that parameters of mode in out or out do not have this problem, because the tag of the actual is not changed.)

Note that the overriding required above can be in the private part, which allows the following: 

```ada
package Pack1 is
    type Ancestor is abstract ...;
    procedure Do_Something(X : in Ancestor) is abstract;
end Pack1;

```

```ada
with Pack1; use Pack1;
package Pack2 is
    type T1 is new Ancestor with record ...;
        -- A concrete type.
    procedure Do_Something(X : in T1); -- Have to override.
end Pack2;

```

```ada
with Pack1; use Pack1;
with Pack2; use Pack2;
package Pack3 is
    type T2 is new Ancestor with private;
        -- A concrete type.
private
    type T2 is new T1 with -- Parent different from ancestor.
      record ... end record;
    -- Here, we inherit Pack2.Do_Something.
end Pack3;
    

```

{AI95-00228-01} T2 inherits an abstract Do_Something, but T2 is not abstract, so Do_Something has to be overridden. However, it is OK to override it in the private part. In this case, we override it by inheriting a concrete version from a different type. Nondispatching calls to Pack3.Do_Something are allowed both inside and outside package Pack3, as the client "knows" that the subprogram was necessarily overridden somewhere.

{AI95-00391-01} For a null extension, the result of a function with a controlling result is defined in terms of an [extension_aggregate](./AA-4.3#S0111) with a null record extension part (see 3.4). This means that these restrictions on functions with a controlling result do not have to apply to null extensions.

{AI95-00391-01} However, functions with controlling access results still require overriding. Changing the tag in place might clobber a preexisting object, and allocating new memory would possibly change the pool of the object, leading to storage leaks. Moreover, copying the object isn't possible for limited types. We don't need to restrict functions that have an access return type of an untagged type, as derived types with primitive subprograms have to have the same representation as their parent type. 

{AI12-0413-1} A call on an abstract subprogram shall be a dispatching call; nondispatching calls to an abstract subprogram are not allowed. In addition to the places where Legality Rules normally apply (see 12.3), these rules also apply in the private part of an instance of a generic unit. 

Ramification: {AI95-00310-01} {AI12-0413-1} If an abstract subprogram is not a dispatching operation of some tagged type, then it cannot be called at all. In Ada 2005, such subprograms are not even considered by name resolution (see 6.4). However, this rule is still needed for cases that can arise in the instance of a generic specification where Name Resolution Rules are not reapplied, but Legality Rules are, when the equality operator for an untagged record type is abstract, while the operator for the formal type is not abstract (see 12.5). 

{AI12-0189-1} {AI12-0292-1} {AI12-0320-1} If the [name](./AA-4.1#S0091) or [prefix](./AA-4.1#S0093) given in an [iterator_procedure_call](./AA-5.5#S0187) (see 5.5.3) denotes an abstract subprogram, the subprogram shall be a dispatching subprogram.

{AI05-0073-1} {AI05-0203-1} {AI12-0437-1} The type of an [aggregate](./AA-4.3#S0106), of an object created by an [object_declaration](./AA-3.3#S0032) or an [allocator](./AA-4.8#S0164), or of a generic formal object of mode in, shall not be abstract. The type of the target of an assignment operation (see 5.2) shall not be abstract. The type of a component shall not be abstract. If the result type of a function is abstract, then the function shall be abstract. If a function has an access result type designating an abstract type, then the function shall be abstract. The type denoted by a [return_subtype_indication](./AA-6.5#S0226) (see 6.5) shall not be abstract. A generic function shall not have an abstract result type or an access result type designating an abstract type. 

Reason: This ensures that values of an abstract type cannot be created, which ensures that a dispatching call to an abstract subprogram will not try to execute the nonexistent body.

Generic formal objects of mode in are like constants; therefore they should be forbidden for abstract types. Generic formal objects of mode in out are like renamings; therefore, abstract types are OK for them, though probably not terribly useful.

{AI05-0073-1} Generic functions returning a formal abstract type are illegal because any instance would have to be instantiated with a nonabstract type in order to avoid violating the function rule (generic functions cannot be declared abstract). But that would be an implied contract; it would be better for the contract to be explicit by the formal type not being declared abstract. Moreover, the implied contract does not add any capability. 

If a partial view is not abstract, the corresponding full view shall not be abstract. If a generic formal type is abstract, then for each primitive subprogram of the formal that is not abstract, the corresponding primitive subprogram of the actual shall not be abstract. 

Discussion: By contrast, we allow the actual type to be nonabstract even if the formal type is declared abstract. Hence, the most general formal tagged type possible is "type T(&lt&gt) is abstract tagged limited private;".

For an abstract private extension declared in the visible part of a package, it is only possible for the full type to be nonabstract if the private extension has no abstract dispatching operations. 

To be honest: {AI95-00294-01} In the sentence about primitive subprograms above, there is some ambiguity as to what is meant by "corresponding" in the case where an inherited operation is overridden.  This is best explained by an example, where the implicit declarations are shown as comments: 

```ada
package P1 is
   type T1 is abstract tagged null record;
   procedure P (X : T1); -- (1)
end P1;

```

```ada
package P2 is
   type T2 is abstract new P1.T1 with null record;
   -- procedure P (X : T2); -- (2)
   procedure P (X : T2) is abstract; -- (3)
end P2;

```

```ada
generic
   type D is abstract new P1.T1 with private;
   -- procedure P (X : D); -- (4)
procedure G (X : D);

```

```ada
procedure I is new G (P2.T2); -- Illegal.

```

Type T2 inherits a nonabstract procedure P (2) from the primitive procedure P (1) of T1. P (2) is overridden by the explicitly declared abstract procedure P (3). Type D inherits a nonabstract procedure P (4) from P (1). In instantiation I, the operation corresponding to P (4) is the one which is not overridden, that is, P (3): the overridden operation P (2) does not "reemerge". Therefore, the instantiation is illegal. 

{AI05-0073-1} For an abstract type declared in a visible part, an abstract primitive subprogram shall not be declared in the private part, unless it is overriding an abstract subprogram implicitly declared in the visible part. For a tagged type declared in a visible part, a primitive function with a controlling result or a controlling access result shall not be declared in the private part, unless it is overriding a function implicitly declared in the visible part. 

Reason: The "visible part" could be that of a package or a generic package. This rule is needed because a nonabstract type extension declared outside the package would not know about any abstract primitive subprograms or primitive functions with controlling results declared in the private part, and wouldn't know that they need to be overridden with nonabstract subprograms. The rule applies to a tagged record type or record extension declared in a visible part, just as to a tagged private type or private extension. The rule applies to explicitly and implicitly declared abstract subprograms: 

```ada
package Pack is
    type T is abstract new T1 with private;
private
    type T is abstract new T2 with record ... end record;
    ...
end Pack;

```

The above example would be illegal if T1 has a nonabstract primitive procedure P, but T2 overrides P with an abstract one; the private part should override P with a nonabstract version. On the other hand, if the P were abstract for both T1 and T2, the example would be legal as is. 

{AI95-00260-02} A generic actual subprogram shall not be an abstract subprogram unless the generic formal subprogram is declared by a [formal_abstract_subprogram_declaration](./AA-12.6#S0337). The [prefix](./AA-4.1#S0093) of an [attribute_reference](./AA-4.1#S0100) for the Access, Unchecked_Access, or Address attributes shall not denote an abstract subprogram. 

Ramification: An [abstract_subprogram_declaration](./AA-3.9#S0076) is not syntactically a [subprogram_declaration](./AA-6.1#S0195). Nonetheless, an abstract subprogram is a subprogram, and an [abstract_subprogram_declaration](./AA-3.9#S0076) is a declaration of a subprogram.

{AI95-00260-02} The part about generic actual subprograms includes those given by default. Of course, an abstract formal subprogram's actual subprogram can be abstract. 


#### Dynamic Semantics

{AI95-00348-01} The elaboration of an [abstract_subprogram_declaration](./AA-3.9#S0076) has no effect. 

NOTE 1   Abstractness is not inherited; to declare an abstract type, the reserved word abstract has to be used in the declaration of the type extension. 

Ramification: A derived type can be abstract even if its parent is not. Similarly, an inherited concrete subprogram can be overridden with an abstract subprogram. 

NOTE 2   A class-wide type is never abstract. Even if a class is rooted at an abstract type, the class-wide type for the class is not abstract, and an object of the class-wide type can be created; the tag of such an object will identify some nonabstract type in the class. 


#### Examples

Example of an abstract type representing a set of natural numbers: 

```ada
package Sets is
    subtype Element_Type is Natural;
    type Set is abstract tagged null record;
    function Empty return Set is abstract;
    function Union(Left, Right : Set) return Set is abstract;
    function Intersection(Left, Right : Set) return Set is abstract;
    function Unit_Set(Element : Element_Type) return Set is abstract;
    procedure Take(Element : out Element_Type;
                   From : in out Set) is abstract;
end Sets;

```

NOTE 3   {AI12-0442-1} Notes on the example: Given the above abstract type, one can derive various (nonabstract) extensions of the type, representing alternative implementations of a set. One possibility is to use a bit vector, but impose an upper bound on the largest element representable, while another possible implementation is a hash table, trading off space for flexibility. 

Discussion: One way to export a type from a package with some components visible and some components private is as follows: 

```ada
package P is
    type Public_Part is abstract tagged
        record
            ...
        end record;
    type T is new Public_Part with private;
    ...
private
    type T is new Public_Part with
        record
            ...
        end record;
end P;

```

The fact that Public_Part is abstract tells clients they have to create objects of type T instead of Public_Part. Note that the public part has to come first; it would be illegal to declare a private type Private_Part, and then a record extension T of it, unless T were in the private part after the full declaration of Private_Part, but then clients of the package would not have visibility to T. 


#### Extensions to Ada 95

{AI95-00391-01} It is not necessary to override functions with a controlling result for a null extension. This makes it easier to derive a tagged type to complete a private type. 


#### Wording Changes from Ada 95

{AI95-00251-01} {AI95-00345-01} Updated the wording to reflect the addition of interface types (see 3.9.4).

{AI95-00260-02} Updated the wording to reflect the addition of abstract formal subprograms (see 12.6).

{AI95-00334-01} The wording of shall-be-overridden was clarified so that it clearly applies to abstract predefined equality.

{AI95-00348-01} Moved the syntax and elaboration rule for [abstract_subprogram_declaration](./AA-3.9#S0076) here, so the syntax and most of the semantics are together (which is consistent with null procedures).

{AI95-00391-01} We define the term require overriding to make other wording easier to understand. 


#### Incompatibilities With Ada 2005

{AI05-0073-1} Correction: Added rules to eliminate holes with controlling access results and generic functions that return abstract types. While these changes are technically incompatible, it is unlikely that they could be used in a program without violating some other rule of the use of abstract types.

{AI05-0097-1} Correction: Corrected a minor glitch having to do with abstract null extensions. The Ada 2005 rule allowed such extensions to inherit concrete operations in some rare cases. It is unlikely that these cases exist in user code. 


#### Extensions to Ada 2005

{AI05-0183-1} An optional [aspect_specification](./AA-13.1#S0346) can be used in an [abstract_subprogram_declaration](./AA-3.9#S0076). This is described in 13.1.1. 


#### Wording Changes from Ada 2005

{AI05-0198-1} Correction: Clarified that the predefined operator corresponding to an inherited abstract operator is also abstract. The Ada 2005 rules caused the predefined operator and the inherited operator to override each other, which is weird. But the effect is the same either way (the operator is not considered for resolution).

{AI05-0203-1} Correction: Added wording to disallow abstract return objects. These were illegal in Ada 2005 by other rules; the extension to support class-wide type better opened a hole which has now been plugged. 


#### Incompatibilities With Ada 2012

{AI12-0413-1} Correction: Clarified that a recheck is needed in the case of an actual that is a record type with an abstract equality. This is an incompatibility as the generic boilerplate was previously omitted, meaning that such a recheck should not have been performed in the private part of an instance. Usually, this would just change an elaboration time raise of Program_Error into an error (a good thing, as the instance will never be useful), but could break a working instance if the equality usage is in a default expression that appears in the private part of the generic unit and it is never used in a call. In that case, Ada 2022 will reject the instance while it would have worked in Ada 2012. As a practical matter, it's more likely that a compiler already does the recheck in the entire instance spec, or does not do it at all; thus for many implementations there will be no practical incompatibility. 


## 3.9.4  Interface Types

{AI95-00251-01} {AI95-00345-01} [An interface type is an abstract tagged type that provides a restricted form of multiple inheritance. A tagged type, task type, or protected type may have one or more interface types as ancestors.] 

Glossary entry: An interface type is a form of abstract tagged type which has no components or concrete operations except possibly null procedures. Interface types are used for composing other interfaces and tagged types and thereby provide multiple inheritance. Only an interface type can be used as a progenitor of another type.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[interface type], Def=[an abstract tagged type that has no components or concrete operations except possibly null procedures], Note1=[Interface types are used for composing other interfaces and tagged types and thereby provide multiple inheritance. Only an interface type can be used as a progenitor of another type.] 


#### Language Design Principles

{AI95-00251-01} {AI95-00345-01} The rules are designed so that an interface can be used as either a parent type or a progenitor type without changing the meaning. That's important so that the order that interfaces are specified in a [derived_type_definition](./AA-3.4#S0035) is not significant. In particular, we want: 

```ada
type Con1 is new Int1 and Int2 with null record;
type Con2 is new Int2 and Int1 with null record;

```

to mean exactly the same thing. 


#### Syntax

{AI95-00251-01} {AI95-00345-01} interface_type_definition<a id="S0077"></a> ::= 
    [limited | task | protected | synchronized] interface [and [interface_list](./AA-3.9#S0078)]

{AI95-00251-01} {AI95-00419-01} interface_list<a id="S0078"></a> ::= interface_[subtype_mark](./AA-3.2#S0028) {and interface_[subtype_mark](./AA-3.2#S0028)}


#### Static Semantics

{AI95-00251-01} An interface type (also called an interface) is a specific abstract tagged type that is defined by an [interface_type_definition](./AA-3.9#S0077).

{AI95-00345-01} An interface with the reserved word limited, task, protected, or synchronized in its definition is termed, respectively, a limited interface, a task interface, a protected interface, or a synchronized interface. In addition, all task and protected interfaces are synchronized interfaces, and all synchronized interfaces are limited interfaces. 

Glossary entry: A synchronized entity is one that will work safely with multiple tasks at one time. A synchronized interface can be an ancestor of a task or a protected type. Such a task or protected type is called a synchronized tagged type.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[synchronized entity], Def=[an entity that can be safely operated on by multiple tasks concurrently], Note1=[A synchronized interface can be an ancestor of a task or a protected type. Such a task or protected type is called a synchronized tagged type.]

{AI95-00345-01} {AI95-00443-01} [A task or protected type derived from an interface is a tagged type.] Such a tagged type is called a synchronized tagged type, as are synchronized interfaces and private extensions whose declaration includes the reserved word synchronized.

Proof: The full definition of tagged types given in 3.9 includes task and protected types derived from interfaces. 

Ramification: The class-wide type associated with a tagged task type (including a task interface type) is a task type, because "task" is one of the language-defined classes of types (see 3.2). However, the class-wide type associated with an interface is not an interface type, as "interface" is not one of the language-defined classes (as it is not closed under derivation). In this sense, "interface" is similar to "abstract". The class-wide type associated with an interface is a concrete (nonabstract) indefinite tagged composite type.

"Private extension" includes generic formal private extensions, as explained in 12.5.1. 

{AI95-00345-01} A task interface is an [abstract] task type. A protected interface is an [abstract] protected type. 

Proof: The "abstract" follows from the definition of an interface type. 

Reason: This ensures that task operations (like abort and the Terminated attribute) can be applied to a task interface type and the associated class-wide type. While there are no protected type operations, we apply the same rule to protected interfaces for consistency. 

{AI95-00251-01} [An interface type has no components.] 

Proof: This follows from the syntax and the fact that discriminants are not allowed for interface types. 

{AI95-00419-01} An interface_[subtype_mark](./AA-3.2#S0028) in an [interface_list](./AA-3.9#S0078) names a progenitor subtype; its type is the progenitor type. An interface type inherits user-defined primitive subprograms from each progenitor type in the same way that a derived type inherits user-defined primitive subprograms from its progenitor types (see 3.4). 

Glossary entry: A progenitor of a derived type is one of the types given in the definition of the derived type other than the first. A progenitor is always an interface type. Interfaces, tasks, and protected types may also have progenitors.

Version=[5],Kind=(AddedNormal),Group=[T],Term=[progenitor of a derived type], Def=[one of the types given in the definition of the derived type other than the first], Note1=[A progenitor is always an interface type. Interfaces, tasks, and protected types can also have progenitors.] 


#### Legality Rules

{AI95-00251-01} All user-defined primitive subprograms of an interface type shall be abstract subprograms or null procedures.

{AI95-00251-01} The type of a subtype named in an [interface_list](./AA-3.9#S0078) shall be an interface type.

{AI95-00251-01} {AI95-00345-01} A type derived from a nonlimited interface shall be nonlimited.

{AI95-00345-01} An interface derived from a task interface shall include the reserved word task in its definition; any other type derived from a task interface shall be a private extension or a task type declared by a task declaration (see 9.1).

{AI95-00345-01} An interface derived from a protected interface shall include the reserved word protected in its definition; any other type derived from a protected interface shall be a private extension or a protected type declared by a protected declaration (see 9.4).

{AI95-00345-01} An interface derived from a synchronized interface shall include one of the reserved words task, protected, or synchronized in its definition; any other type derived from a synchronized interface shall be a private extension, a task type declared by a task declaration, or a protected type declared by a protected declaration.

Reason: We require that an interface descendant of a task, protected, or synchronized interface repeat the explicit kind of interface it will be, rather than simply inheriting it, so that a reader is always aware of whether the interface provides synchronization and whether it may be implemented only by a task or protected type. The only place where inheritance of the kind of interface might be useful would be in a generic if you didn't know the kind of the actual interface. However, the value of that is low because you cannot implement an interface properly if you don't know whether it is a task, protected, or synchronized interface. Hence, we require the kind of the actual interface to match the kind of the formal interface (see 12.5.5). 

{AI95-00345-01} No type shall be derived from both a task interface and a protected interface.

Reason: This prevents a single private extension from inheriting from both a task and a protected interface. For a private type, there can be no legal completion. For a generic formal derived type, there can be no possible matching type (so no instantiation could be legal). This rule provides early detection of the errors. 

{AI95-00251-01} In addition to the places where Legality Rules normally apply (see 12.3), these rules apply also in the private part of an instance of a generic unit. 

Ramification: {AI05-0299-1} This paragraph is intended to apply to all of the Legality Rules in this subclause. We cannot allow interface types which do not obey these rules, anywhere. Luckily, deriving from a formal type (which might be an interface) is not allowed for any tagged types in a generic body. So checking in the private part of a generic covers all of the cases. 


#### Dynamic Semantics

{AI95-00251-01} {AI05-0070-1} The elaboration of an [interface_type_definition](./AA-3.9#S0077) creates the interface type and its first subtype. 

Discussion: There is no other effect. An [interface_list](./AA-3.9#S0078) is made up of [subtype_mark](./AA-3.2#S0028)s, which do not need to be elaborated, so the [interface_list](./AA-3.9#S0078) does not either. This is consistent with the handling of [discriminant_part](./AA-3.7#S0059)s. 

NOTE 1   {AI95-00411-01} {AI12-0440-1} Nonlimited interface types have predefined nonabstract equality operators. These can be overridden with user-defined abstract equality operators. Such operators will then require an explicit overriding for any nonabstract descendant of the interface. 


#### Examples

{AI95-00433-01} Example of a limited interface and a synchronized interface extending it:

```ada
type Queue is limited interface;
procedure Append(Q : in out Queue; Person : in Person_Name) is abstract;
procedure Remove_First(Q      : in out Queue;
                       Person :    out Person_Name) is abstract;
function Cur_Count(Q : in Queue) return Natural is abstract;
function Max_Count(Q : in Queue) return Natural is abstract;
-- See 3.10.1 for Person_Name.

```

```ada
{AI05-0004-1} Queue_Error : exception;
-- Append raises Queue_Error if Cur_Count(Q) = Max_Count(Q)
-- Remove_First raises Queue_Error if Cur_Count(Q) = 0

```

```ada
type Synchronized_Queue is synchronized interface and Queue; -- see 9.11
procedure Append_Wait(Q      : in out Synchronized_Queue;
                      Person : in     Person_Name) is abstract;
procedure Remove_First_Wait(Q      : in out Synchronized_Queue;
                            Person :    out Person_Name) is abstract;

```

```ada
...

```

```ada
procedure Transfer(From   : in out Queue'Class;
                   To     : in out Queue'Class;
                   Number : in     Natural := 1) is
   Person : Person_Name;
begin
   for I in 1..Number loop
      Remove_First(From, Person);
      Append(To, Person);
   end loop;
end Transfer;

```

{AI12-0442-1} This defines a Queue interface defining a queue of people. (A similar design is possible to define any kind of queue simply by replacing Person_Name by an appropriate type.) The Queue interface has four dispatching operations, Append, Remove_First, Cur_Count, and Max_Count. The body of a class-wide operation, Transfer is also shown. Every nonabstract extension of Queue will provide implementations for at least its four dispatching operations, as they are abstract. Any object of a type derived from Queue can be passed to Transfer as either the From or the To operand. The two operands can be of different types in a given call.

{AI12-0440-1} The Synchronized_Queue interface inherits the four dispatching operations from Queue and adds two additional dispatching operations, which wait if necessary rather than raising the Queue_Error exception. This synchronized interface can only be implemented by a task or protected type, and as such ensures safe concurrent access.

{AI95-00433-01} Example use of the interface:

```ada
{AI05-0004-1} type Fast_Food_Queue is new Queue with record ...;
procedure Append(Q : in out Fast_Food_Queue; Person : in Person_Name);
procedure Remove_First(Q : in out Fast_Food_Queue; 
                       Person : out Person_Name);
function Cur_Count(Q : in Fast_Food_Queue) return Natural;
function Max_Count(Q : in Fast_Food_Queue) return Natural;

```

```ada
...

```

```ada
Cashier, Counter : Fast_Food_Queue;

```

```ada
{AI12-0312-1} ...
-- Add Casey (see 3.10.1) to the cashier's queue:
Append (Cashier, Casey);
-- After payment, move Casey to the sandwich counter queue:
Transfer (Cashier, Counter);
...

```

{AI12-0442-1} An interface such as Queue can be used directly as the parent of a new type (as shown here), or can be used as a progenitor when a type is derived. In either case, the primitive operations of the interface are inherited. For Queue, the implementation of the four inherited routines will necessarily be provided. Inside the call of Transfer, calls will dispatch to the implementations of Append and Remove_First for type Fast_Food_Queue.

{AI95-00433-01} Example of a task interface:

```ada
type Serial_Device is task interface;  -- see 9.1
procedure Read (Dev : in Serial_Device; C : out Character) is abstract;
procedure Write(Dev : in Serial_Device; C : in  Character) is abstract;

```

The Serial_Device interface has two dispatching operations which are intended to be implemented by task entries (see 9.1).


#### Extensions to Ada 95

{AI95-00251-01} {AI95-00345-01} Interface types are new. They provide multiple inheritance of interfaces, similar to the facility provided in Java and other recent language designs. 


#### Wording Changes from Ada 2005

{AI05-0070-1} Correction: Corrected the definition of elaboration for an [interface_type_definition](./AA-3.9#S0077) to match that of other type definitions. 
