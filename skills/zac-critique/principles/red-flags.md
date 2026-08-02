# Red Flags

These are signs that code is probably more complicated than it needs to be. When you see a red flag, stop and look for a design alternative that eliminates it.

---

## 1. Shallow Module

The interface is not much simpler than the implementation. The module does not hide much — callers gain little from using it instead of doing the work themselves. A class with many methods, each doing only a few lines, is shallow. The cure is to merge related functionality into fewer, deeper modules.

## 2. Information Leakage

A design decision is reflected in multiple modules. If two classes both understand the format of a particular file, or both know the structure of a particular data type, that knowledge is leaked. Changing the format requires updating both. The cure is to consolidate the leaked knowledge into a single module.

## 3. Temporal Decomposition

Code structure follows the order in which operations happen rather than the information they use. "First read the file, then parse it, then process it" becomes three modules — but if all three need to know the file format, the knowledge is scattered. The cure is to organize around information, not time.

## 4. Overexposure

An API for a commonly used feature forces callers to learn about rarely used features. The common case should be simple. Uncommon options should be available but not required. The cure is to provide defaults and separate the simple path from the advanced one.

## 5. Pass-Through Method

A method does nothing except pass its arguments to another method, usually with the same or a similar signature. This adds a layer of interface complexity without hiding anything. The cure is to expose the lower-level class directly, redistribute functionality, or merge the classes.

## 6. Repetition

The same code (or nearly the same code) appears in multiple places. This is a sign that the right abstraction has not been found. The cure is to factor the repeated code into a shared method or module.

## 7. Special-General Mixture

A general-purpose mechanism also contains code specialized for a particular use of that mechanism. This makes the mechanism harder to understand and creates information leakage between the mechanism and the use case. The cure is to separate the general-purpose mechanism from the special-purpose code.

## 8. Conjoined Methods

You cannot understand the implementation of one method without also understanding the implementation of another. This means the two methods are not truly independent — they share hidden knowledge. The cure is to refactor so each method can be understood in isolation, or merge them if they are truly inseparable.

## 9. Comment Repeats Code

A comment says exactly what the code next to it already says, using the same words. Such comments add no information and clutter the code. The cure is to delete trivial comments and write comments that explain intent, constraints, or non-obvious consequences.

## 10. Implementation Documentation Contaminates Interface

Interface comments describe implementation details that callers should not need to know. This leaks information and makes the interface harder to understand. The cure is to describe what the method does, not how it does it.

## 11. Vague Name

A variable or method name is broad enough to refer to many different things. Names like `data`, `result`, `count`, `status`, `process`, `handle`, `manager`, `info` fail to convey specific meaning. The cure is to choose names that create a precise mental image of what the entity represents.

## 12. Hard to Pick Name

If finding a simple, clear name for a variable or method is difficult, the underlying design may not be clean. A well-designed entity has an obvious name. Naming difficulty suggests the entity does too many things, or its purpose is unclear. The cure is to reconsider the design.

## 13. Hard to Describe

If the interface comment for a method requires excessive length or complexity, the method probably does not have a clean abstraction. Long descriptions suggest the method does too many things, has too many side effects, or has a poorly defined purpose. The cure is to redesign the method's abstraction.

## 14. Nonobvious Code

The meaning and behavior of the code cannot be understood with a quick reading. The reader's initial guesses about what the code does are wrong. This creates unknown unknowns — the worst form of complexity. The cure is to restructure the code so that a reader's natural assumptions are correct.
