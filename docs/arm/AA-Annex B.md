---
sidebar_position:  137
---

# Annex B Interface to Other Languages

:::warning
We're still working on the Reference manual output.  Internal links are broken,
as are a bunch of other things.
See the [tracking issue](https://github.com/ada-lang-io/ada-lang-io/issues/20)
:::
This Annex describes features for writing mixed-language programs. General interface support is presented first; then specific support for C, COBOL, and Fortran is defined, in terms of language interface packages for each of these languages. 

Ramification: This Annex is not a "Specialized Needs" annex. Every implementation must support all nonoptional features defined here (mainly the package Interfaces). 


#### Language Design Principles

Ada should have strong support for mixed-language programming. 


#### Implementation Requirements

{AI05-0229-1} {AI05-0262-1} {AI05-0299-1} Support for interfacing to any foreign language is optional. However, an implementation shall not provide any optional aspect, attribute, library unit, or pragma having the same name as an aspect, attribute, library unit, or pragma (respectively) specified in the subclauses of this Annex unless the provided construct is either as specified in those subclauses or is more limited in capability than that required by those subclauses. A program that attempts to use an unsupported capability of this Annex shall either be identified by the implementation before run time or shall raise an exception at run time. 

Discussion: The intent is that the same rules apply for the optional parts of language interfacing as apply for Specialized Needs Annexes. See  for a discussion of the purpose of these rules. 


#### Extensions to Ada 83

Much of the functionality in this Annex is new to Ada 95. 


#### Wording Changes from Ada 83

This Annex contains what used to be RM83-13.8. 


#### Wording Changes from Ada 2005

{AI05-0262-1} Moved the clarification that interfacing to foreign languages is optional and has the same restrictions as a Specialized Needs Annex here. 
