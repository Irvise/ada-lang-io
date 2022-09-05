---
sidebar_position:  171
---

# E.3  Consistency of a Distributed System

{AI05-0299-1} [This subclause defines attributes and rules associated with verifying the consistency of a distributed program.] 


#### Language Design Principles

{AI05-0248-1} The rules guarantee that remote call interface and shared passive library units are consistent among all partitions prior to the execution of a distributed program, so that the semantics of the distributed program are well defined.


#### Static Semantics

For a [prefix](./AA-4.1#S0093) P that statically denotes a program unit, the following attributes are defined: 

P'VersionYields a value of the predefined type String that identifies the version of the compilation unit that contains the declaration of the program unit.

P'Body_VersionYields a value of the predefined type String that identifies the version of the compilation unit that contains the body (but not any subunits) of the program unit. 

{8652/0084} {AI95-00104-01} The version of a compilation unit changes whenever the compilation unit changes in a semantically significant way. This document does not define the exact meaning of "semantically significant". It is unspecified whether there are other events (such as recompilation) that result in the version of a compilation unit changing. 

This paragraph was deleted.

{8652/0084} {AI95-00104-01} If P is not a library unit, and P has no completion, then P'Body_Version returns the Body_Version of the innermost program unit enclosing the declaration of P. If P is a library unit, and P has no completion, then P'Body_Version returns a value that is different from Body_Version of any version of P that has a completion. 


#### Bounded (Run-Time) Errors

In a distributed program, a library unit is consistent if the same version of its declaration is used throughout. It is a bounded error to elaborate a partition of a distributed program that contains a compilation unit that depends on a different version of the declaration of a shared passive or RCI library unit than that included in the partition to which the shared passive or RCI library unit was assigned. As a result of this error, Program_Error can be raised in one or both partitions during elaboration; in any case, the partitions become inaccessible to one another. 

Ramification: Because a version changes if anything on which it depends undergoes a version change, requiring consistency for shared passive and remote call interface library units is sufficient to ensure consistency for the declared pure and remote types library units that define the types used for the objects and parameters through which interpartition communication takes place.

Note that we do not require matching Body_Versions; it is irrelevant for shared passive and remote call interface packages, since only one copy of their body exists in a distributed program (in the absence of implicit replication), and we allow the bodies to differ for declared pure and remote types packages from partition to partition, presuming that the differences are due to required error corrections that took place during the execution of a long-running distributed program. The Body_Version attribute provides a means for performing stricter consistency checks. 


#### Wording Changes from Ada 95

{8652/0084} {AI95-00104-01} Corrigendum: Clarified the meaning of 'Version and 'Body_Version. 
