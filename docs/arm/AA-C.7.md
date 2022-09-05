---
sidebar_position:  150
---

# C.7  Task Information

{AI95-00266-02} {AI05-0299-1} [This subclause describes operations and attributes that can be used to obtain the identity of a task. In addition, a package that associates user-defined information with a task is defined. Finally, a package that associates termination procedures with a task or set of tasks is defined.] 


#### Wording Changes from Ada 95

{AI95-00266-02} {AI05-0299-1} The title and text here were updated to reflect the addition of task termination procedures to this subclause. 


## C.7.1  The Package Task_Identification


#### Static Semantics

The following language-defined library package exists: 

```ada
{AI95-00362-01} {AI12-0241-1} {AI12-0302-1} {AI12-0399-1} package Ada.Task_Identification
   with Preelaborate, Nonblocking, Global =&gt in out synchronized is
   type Task_Id is private
      with Preelaborable_Initialization;
   Null_Task_Id : constant Task_Id;
   function  "=" (Left, Right : Task_Id) return Boolean;

```

```ada
{8652/0070} {AI95-00101-01} {AI05-0189-1} {AI12-0241-1}    function  Image                  (T : Task_Id) return String;
   function  Current_Task     return Task_Id;
   function  Environment_Task return Task_Id;
   procedure Abort_Task             (T : in Task_Id)
      with Nonblocking =&gt False;

```

```ada
{AI05-0189-1}    function  Is_Terminated          (T : Task_Id) return Boolean;
   function  Is_Callable            (T : Task_Id) return Boolean;
   function  Activation_Is_Complete (T : Task_Id) return Boolean;
private
   ... -- not specified by the language
end Ada.Task_Identification;

```


#### Dynamic Semantics

A value of the type Task_Id identifies an existent task. The constant Null_Task_Id does not identify any task. Each object of the type Task_Id is default initialized to the value of Null_Task_Id.

The function "=" returns True if and only if Left and Right identify the same task or both have the value Null_Task_Id.

The function Image returns an implementation-defined string that identifies T. If T equals Null_Task_Id, Image returns an empty string. 

Implementation defined: The result of the Task_Identification.Image attribute.

The function Current_Task returns a value that identifies the calling task.

Ramification: {AI12-0005-1} The logical threads of control associated with the execution of a given parallel construct all execute as part of the execution of one task (see 9, "Tasks and Synchronization"). Thus, the result returned by a call to Task_Identification.Current_Task is independent of whether the call takes place during the execution of a parallel construct. 

{AI05-0189-1} The function Environment_Task returns a value that identifies the environment task.

The effect of Abort_Task is the same as the [abort_statement](./AA-9.8#S0284) for the task identified by T. [In addition, if T identifies the environment task, the entire partition is aborted, See E.1.]

The functions Is_Terminated and Is_Callable return the value of the corresponding attribute of the task identified by T. 

Ramification: {8652/0115} {AI95-00206-01} These routines can be called with an argument identifying the environment task. Is_Terminated will always be False for such a call, but Is_Callable (usually True) could be False if the environment task is waiting for the termination of dependent tasks. Thus, a dependent task can use Is_Callable to determine if the main subprogram has completed. 

{AI05-0189-1} The function Activation_Is_Complete returns True if the task identified by T has completed its activation (whether successfully or not). It returns False otherwise. If T identifies the environment task, Activation_Is_Complete returns True after the elaboration of the [library_item](./AA-10.1#S0287)s of the partition has completed.

For a [prefix](./AA-4.1#S0093) T that is of a task type [(after any implicit dereference)], the following attribute is defined: 

T'IdentityYields a value of the type Task_Id that identifies the task denoted by T.

For a [prefix](./AA-4.1#S0093) E that denotes an [entry_declaration](./AA-9.5#S0257), the following attribute is defined: 

E'Caller{AI05-0262-1} Yields a value of the type Task_Id that identifies the task whose call is now being serviced. Use of this attribute is allowed only inside an [accept_statement](./AA-9.5#S0258), or [entry_body](./AA-9.5#S0260) after the [entry_barrier](./AA-9.5#S0262), corresponding to the [entry_declaration](./AA-9.5#S0257) denoted by E. 

{AI12-0231-1} Program_Error is raised if a value of Null_Task_Id is passed as a parameter to Abort_Task, Activation_Is_Complete, Is_Terminated, and Is_Callable.

This paragraph was deleted.{AI12-0241-1} 


#### Bounded (Run-Time) Errors

{AI95-00237-01} {AI05-0004-1} It is a bounded error to call the Current_Task function from an [entry_body](./AA-9.5#S0260), interrupt handler, or finalization of a task attribute. Program_Error is raised, or an implementation-defined value of the type Task_Id is returned. 

Implementation defined: The value of Current_Task when in a protected entry, interrupt handler, or finalization of a task attribute.

Implementation Note: This value could be Null_Task_Id, or the ID of some user task, or that of an internal task created by the implementation. 

Ramification: {AI95-00237-01} An entry barrier is syntactically part of an [entry_body](./AA-9.5#S0260), so a call to Current_Task from an entry barrier is also covered by this rule. 


#### Erroneous Execution

If a value of Task_Id is passed as a parameter to any of the operations declared in this package (or any language-defined child of this package), and the corresponding task object no longer exists, the execution of the program is erroneous. 


#### Documentation Requirements

The implementation shall document the effect of calling Current_Task from an entry body or interrupt handler. 

This paragraph was deleted.

Documentation Requirement: The effect of calling Current_Task from an entry body or interrupt handler.

NOTE 1   This package is intended for use in writing user-defined task scheduling packages and constructing server tasks. Current_Task can be used in conjunction with other operations requiring a task as an argument such as Set_Priority (see D.5).

NOTE 2   The function Current_Task and the attribute Caller can return a Task_Id value that identifies the environment task.


#### Extensions to Ada 95

{AI95-00362-01} Task_Identification is now preelaborated, so it can be used in preelaborated units. 


#### Wording Changes from Ada 95

{8652/0070} {AI95-00101-01} Corrigendum: Corrected the mode of the parameter to Abort_Task to in.

{AI95-00237-01} Corrected the wording to include finalization of a task attribute in the bounded error case; we don't want to specify which task does these operations. 


#### Incompatibilities With Ada 2005

{AI05-0189-1} Functions Environment_Task and Activation_Is_Complete are added to Task_Identification. If Task_Identification is referenced in a [use_clause](./AA-8.4#S0235), and an entity E with a [defining_identifier](./AA-3.1#S0022) of Environment_Task or Activation_Is_Complete is defined in a package that is also referenced in a [use_clause](./AA-8.4#S0235), the entity E may no longer be use-visible, resulting in errors. This should be rare and is easily fixed if it does occur. 


#### Wording Changes from Ada 2012

{AI12-0231-1} Correction: Defined what happens if Null_Task_Id is passed to Activation_Is_Complete. 


## C.7.2  The Package Task_Attributes


#### Static Semantics

The following language-defined generic library package exists: 

```ada
{AI12-0241-1} {AI12-0302-1} with Ada.Task_Identification; use Ada.Task_Identification;
generic
   type Attribute is private;
   Initial_Value : in Attribute;
package Ada.Task_Attributes
   with Nonblocking, Global =&gt in out synchronized is

```

```ada
   type Attribute_Handle is access all Attribute;

```

```ada
   function Value(T : Task_Id := Current_Task)
     return Attribute;

```

```ada
   function Reference(T : Task_Id := Current_Task)
     return Attribute_Handle;

```

```ada
   procedure Set_Value(Val : in Attribute;
                       T : in Task_Id := Current_Task);
   procedure Reinitialize(T : in Task_Id := Current_Task);

```

```ada
end Ada.Task_Attributes;

```


#### Dynamic Semantics

When an instance of Task_Attributes is elaborated in a given active partition, an object of the actual type corresponding to the formal type Attribute is implicitly created for each task (of that partition) that exists and is not yet terminated. This object acts as a user-defined attribute of the task. A task created previously in the partition and not yet terminated has this attribute from that point on. Each task subsequently created in the partition will have this attribute when created. In all these cases, the initial value of the given attribute is Initial_Value.

The Value operation returns the value of the corresponding attribute of T.

The Reference operation returns an access value that designates the corresponding attribute of T.

The Set_Value operation performs any finalization on the old value of the attribute of T and assigns Val to that attribute (see 5.2 and 7.6).

The effect of the Reinitialize operation is the same as Set_Value where the Val parameter is replaced with Initial_Value. 

Implementation Note: In most cases, the attribute memory can be reclaimed at this point. 

For all the operations declared in this package, Tasking_Error is raised if the task identified by T is terminated. Program_Error is raised if the value of T is Null_Task_Id.

{AI95-00237-01} After a task has terminated, all of its attributes are finalized, unless they have been finalized earlier. When the master of an instantiation of Ada.Task_Attributes is finalized, the corresponding attribute of each task is finalized, unless it has been finalized earlier. 

Reason: This is necessary so that a task attribute does not outlive its type. For instance, that's possible if the instantiation is nested, and the attribute is on a library-level task. 

Ramification: The task owning an attribute cannot, in general, finalize that attribute. That's because the attributes are finalized after the task is terminated; moreover, a task may have attributes as soon as it is created; the task may never even have been activated. 


#### Bounded (Run-Time) Errors

{8652/0071} {AI95-00165-01} If the package Ada.Task_Attributes is instantiated with a controlled type and the controlled type has user-defined Adjust or Finalize operations that in turn access task attributes by any of the above operations, then a call of Set_Value of the instantiated package constitutes a bounded error. The call may perform as expected or may result in forever blocking the calling task and subsequently some or all tasks of the partition. 


#### Erroneous Execution

It is erroneous to dereference the access value returned by a given call on Reference after a subsequent call on Reinitialize for the same task attribute, or after the associated task terminates. 

Reason: This allows the storage to be reclaimed for the object associated with an attribute upon Reinitialize or task termination. 

If a value of Task_Id is passed as a parameter to any of the operations declared in this package and the corresponding task object no longer exists, the execution of the program is erroneous.

{8652/0071} {AI95-00165-01} {AI95-00237-01} An access to a task attribute via a value of type Attribute_Handle is erroneous if executed concurrently with another such access or a call of any of the operations declared in package Task_Attributes. An access to a task attribute is erroneous if executed concurrently with or after the finalization of the task attribute. 

Reason: There is no requirement of atomicity on accesses via a value of type Attribute_Handle. 

Ramification: A task attribute can only be accessed after finalization through a value of type Attribute_Handle. Operations in package Task_Attributes cannot be used to access a task attribute after finalization, because either the master of the instance has been or is in the process of being left (in which case the instance is out of scope and thus cannot be called), or the associated task is already terminated (in which case Tasking_Error is raised for any attempt to call a task attribute operation). 


#### Implementation Requirements

{8652/0071} {AI95-00165-01} For a given attribute of a given task, the implementation shall perform the operations declared in this package atomically with respect to any of these operations of the same attribute of the same task. The granularity of any locking mechanism necessary to achieve such atomicity is implementation defined. 

Implementation defined: Granularity of locking for Task_Attributes.

Ramification: Hence, other than by dereferencing an access value returned by Reference, an attribute of a given task can be safely read and updated concurrently by multiple tasks. 

{AI95-00237-01} After task attributes are finalized, the implementation shall reclaim any storage associated with the attributes. 


#### Documentation Requirements

The implementation shall document the limit on the number of attributes per task, if any, and the limit on the total storage for attribute values per task, if such a limit exists.

In addition, if these limits can be configured, the implementation shall document how to configure them. 

This paragraph was deleted.

Documentation Requirement: For package Task_Attributes, limits on the number and size of task attributes, and how to configure any limits.


#### Metrics

{AI95-00434-01} The implementation shall document the following metrics: A task calling the following subprograms shall execute at a sufficiently high priority as to not be preempted during the measurement period. This period shall start just before issuing the call and end just after the call completes. If the attributes of task T are accessed by the measurement tests, no other task shall access attributes of that task during the measurement period. For all measurements described here, the Attribute type shall be a scalar type whose size is equal to the size of the predefined type Integer. For each measurement, two cases shall be documented: one where the accessed attributes are of the calling task [(that is, the default value for the T parameter is used)], and the other, where T identifies another, nonterminated, task.

The following calls (to subprograms in the Task_Attributes package) shall be measured: 

a call to Value, where the return value is Initial_Value;

a call to Value, where the return value is not equal to Initial_Value;

a call to Reference, where the return value designates a value equal to Initial_Value;

a call to Reference, where the return value designates a value not equal to Initial_Value;

{AI95-00434-01} a call to Set_Value where the Val parameter is not equal to Initial_Value and the old attribute value is equal to Initial_Value;

a call to Set_Value where the Val parameter is not equal to Initial_Value and the old attribute value is not equal to Initial_Value.

Documentation Requirement: The metrics for the Task_Attributes package.


#### Implementation Permissions

{AI12-0444-1} An implementation can avoid actually creating the object corresponding to a task attribute until its value is set to something other than that of Initial_Value, or until Reference is called for the task attribute. Similarly, when the value of the attribute is to be reinitialized to that of Initial_Value, the object may instead be finalized and its storage reclaimed, to be recreated when needed later. While the object does not exist, the function Value may simply return Initial_Value, rather than implicitly creating the object. 

Discussion: The effect of this permission can only be observed if the assignment operation for the corresponding type has side effects. 

Implementation Note: {AI95-00114-01} This permission means that even though every task has every attribute, storage need only be allocated for those attributes for which function Reference has been invoked or set to a value other than that of Initial_Value. 

An implementation is allowed to place restrictions on the maximum number of attributes a task may have, the maximum size of each attribute, and the total storage size allocated for all the attributes of a task.


#### Implementation Advice

{AI95-00434-01} {AI12-0438-1} Some implementations are targeted to domains in which memory use at run time has to be completely deterministic. For such implementations, it is recommended that the storage for task attributes will be pre-allocated statically and not from the heap. This can be accomplished by either placing restrictions on the number and the size of the attributes of a task, or by using the pre-allocated storage for the first N attribute objects, and the heap for the others. In the latter case, N should be documented.

Implementation Advice: If the target domain requires deterministic memory use at run time, storage for task attributes should be pre-allocated statically and the number of attributes pre-allocated should be documented.

Discussion: We don't mention "restrictions on the size and number" (that is, limits) in the text for the Annex, because it is covered by the Documentation Requirement above, and we try not to repeat requirements in the Annex (they're enough work to meet without having to do things twice). 

{AI95-00237-01} Finalization of task attributes and reclamation of associated storage should be performed as soon as possible after task termination. 

Implementation Advice: Finalization of task attributes and reclamation of associated storage should be performed as soon as possible after task termination.

Reason: {AI95-00237-01} This is necessary because the normative wording only says that attributes are finalized "after" task termination. Without this advice, waiting until the instance is finalized would meet the requirements (it is after termination, but may be a very long time after termination). We can't say anything more specific than this, as we do not want to require the overhead of an interaction with the tasking system to be done at a specific point. 

NOTE 1   {AI12-0442-1} An attribute always exists (after instantiation), and has the initial value. An implementation can avoid using memory to store the attribute value until the first operation that changes the attribute value. The same holds true after Reinitialize.

NOTE 2   {AI12-0442-1} The result of the Reference function is always safe to use in the task body whose attribute is being accessed. However, when the result is being used by another task, the programmer will want to make sure that the task whose attribute is being accessed is not yet terminated. Failing to do so can make the program execution erroneous.


#### Wording Changes from Ada 95

{8652/0071} {AI95-00165-01} Corrigendum: Clarified that use of task attribute operations from within a task attribute operation (by an Adjust or Finalize call) is a bounded error, and that concurrent use of attribute handles is erroneous.

{AI95-00237-01} Clarified the wording so that the finalization takes place after the termination of the task or when the instance is finalized (whichever is sooner). 


## C.7.3  The Package Task_Termination


#### Static Semantics

{AI95-00266-02} The following language-defined library package exists: 

```ada
{AI12-0241-1} {AI12-0302-1} with Ada.Task_Identification;
with Ada.Exceptions;
package Ada.Task_Termination
   with Preelaborate, Nonblocking, Global =&gt in out synchronized is

```

```ada
   type Cause_Of_Termination is (Normal, Abnormal, Unhandled_Exception);

```

```ada
   type Termination_Handler is access protected procedure
     (Cause : in Cause_Of_Termination;
      T     : in Ada.Task_Identification.Task_Id;
      X     : in Ada.Exceptions.Exception_Occurrence);

```

```ada
   procedure Set_Dependents_Fallback_Handler
     (Handler: in Termination_Handler);
   function Current_Task_Fallback_Handler return Termination_Handler;

```

```ada
   procedure Set_Specific_Handler
     (T       : in Ada.Task_Identification.Task_Id;
      Handler : in Termination_Handler);
   function Specific_Handler (T : Ada.Task_Identification.Task_Id)
      return Termination_Handler;

```

```ada
end Ada.Task_Termination;

```


#### Dynamic Semantics

{AI95-00266-02} {AI05-0202-1} The type Termination_Handler identifies a protected procedure to be executed by the implementation when a task terminates. Such a protected procedure is called a handler. In all cases T identifies the task that is terminating. If the task terminates due to completing the last statement of its body, or as a result of waiting on a terminate alternative, and the finalization of the task completes normally, then Cause is set to Normal and X is set to Null_Occurrence. If the task terminates because it is being aborted, then Cause is set to Abnormal; X is set to Null_Occurrence if the finalization of the task completes normally. If the task terminates because of an exception raised by the execution of its [task_body](./AA-9.1#S0248), then Cause is set to Unhandled_Exception; X is set to the associated exception occurrence if the finalization of the task completes normally. Independent of how the task completes, if finalization of the task propagates an exception, then Cause is either Unhandled_Exception or Abnormal, and X is an exception occurrence that identifies the Program_Error exception.

{AI95-00266-02} Each task has two termination handlers, a fall-back handler and a specific handler. The specific handler applies only to the task itself, while the fall-back handler applies only to the dependent tasks of the task. A handler is said to be set if it is associated with a nonnull value of type Termination_Handler, and cleared otherwise. When a task is created, its specific handler and fall-back handler are cleared.

{AI95-00266-02} {AI05-0264-1} The procedure Set_Dependents_Fallback_Handler changes the fall-back handler for the calling task: if Handler is null, that fall-back handler is cleared; otherwise, it is set to be Handler.all. If a fall-back handler had previously been set it is replaced.

{AI95-00266-02} {AI05-0264-1} The function Current_Task_Fallback_Handler returns the fall-back handler that is currently set for the calling task, if one is set; otherwise, it returns null.

{AI95-00266-02} {AI05-0264-1} The procedure Set_Specific_Handler changes the specific handler for the task identified by T: if Handler is null, that specific handler is cleared; otherwise, it is set to be Handler.all. If a specific handler had previously been set it is replaced.

Ramification: {AI05-0005-1} This package cannot portably be used to set a handler on the program as a whole. It is possible to call Set_Specific_Handler with the environment task's ID. But any call to the handler would necessarily be a Bounded (Run-Time) Error, as the handler is called after the task's finalization has completed. In the case of the environment task, that includes any possible protected objects, and calling a protected object after it is finalized is a Bounded (Run-Time) Error (see 9.4). This might work in a particular implementation, but it cannot be depended upon. 

{AI95-00266-02} {AI05-0264-1} The function Specific_Handler returns the specific handler that is currently set for the task identified by T, if one is set; otherwise, it returns null.

{AI95-00266-02} As part of the finalization of a [task_body](./AA-9.1#S0248), after performing the actions specified in 7.6 for finalization of a master, the specific handler for the task, if one is set, is executed. If the specific handler is cleared, a search for a fall-back handler proceeds by recursively following the master relationship for the task. If a task is found whose fall-back handler is set, that handler is executed; otherwise, no handler is executed.

{AI95-00266-02} For Set_Specific_Handler or Specific_Handler, Tasking_Error is raised if the task identified by T has already terminated. Program_Error is raised if the value of T is Ada.Task_Identification.Null_Task_Id.

{AI95-00266-02} An exception propagated from a handler that is invoked as part of the termination of a task has no effect.


#### Erroneous Execution

{AI95-00266-02} For a call of Set_Specific_Handler or Specific_Handler, if the task identified by T no longer exists, the execution of the program is erroneous. 


#### Extensions to Ada 95

{AI95-00266-02} Package Task_Termination is new. 


#### Wording Changes from Ada 2005

{AI05-0202-1} Correction: Specified what is passed to the handler if the finalization of the task fails after it is completed. This was not specified at all in Ada 2005, so there is a possibility that some program depended on some other behavior of an implementation. But as this case is very unlikely (and only occurs when there is already a significant bug in the program - so should not occur in fielded systems), we're not listing this as an inconsistency. 
