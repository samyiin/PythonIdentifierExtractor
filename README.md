# PythonIdentifierExtractor
Extract identifiers from python 3 and python 2 files. All knownledge is until python 3.10. I write this tool for my masters thesis, because I have been searching around and I don't quite see such a tool in&for python programs. 

This tool depends on the ast library. We have three classes (I put all of them in one file....):

**PythonFileCleaner**: this class just make sure the file is able to be parsed by ast library. (Warning: some of the fix is inplace to the original file, I am too lazy to fix that, use carefully. )  

**ScopeTracker**: this class is the core technology of this project, it will walk through the ast tree, and extract all possibile identifiers from the file. Along with the identifiers, we also extract their nested scope number, nested indentation number, and whether they are variables in function, in class, in lambda, ... etc. You can use this class independently, it only depends on the *ast* library, and it takes a parsed ast tree as input, a list of dictionary (one for each name) as output. 

**PythonIdentifierExtractor**: This is like a "manager" class, it takes a python file path, use **PythonFileCleaner** to clean the file, and then use **ScopeTracker** to get all the identifiers in the file, and return as a pandas dataframe.

## Output fields

      {"name": name,
      "classification": classification of identifier (See Explanation-identifier classification),
      "in_function": if the identifier is in a function/method,
      "in_class": if the identifier is in a class,
      "in_lambda": if the identifier is in a lambda,
      "in_comprehension": if the identifier is in a comprehension,
      "in_match": if the identifier is in a pattern match,
      "in_for": if the identifier is in a for,
      "in_while": if the identifier is in a while,
      "in_if": if the identifier is in a if,
      "in_with": if the identifier is in a with,
      "in_try": if the identifier is in a try,
      "in_except": if the identifier is in a except,
      "in_finally": if the identifier is in a finally,
      "nested_scope_number": How many "outer scope layers" are there,
      "nested_indentation_number": How many "indentation layers" are there,
      "lineno": the line number of this variable in the file,
      "col_offset": the column number of this variable in the file,}

# Explanation

## Identifier
These things (and hopefully only these things) in python (might) introduce new identifiers:
    
    assignment - variable (could also be class/enum variable if it's in the scope of a class)
    annotated assignment - variable
    name expression - variable
    instance variable assignment (self.x) - instance variable 
    
    for/ async for loop - for_loop variable
    with/ async with statement - with_statement variable
    Exception binding - exception variable
    comprehension - comprehension variable
    Pattern Matching -  pattern_matching variable

    function/ async function declaration - function name (might be a class method name)
    function/ async function/ method parameters (Positional-only, Keyword-only, Vararg, Kwarg) - parameters 
    lambda expression - lambda parameter
    class/ enum declaration - class name
    
    import/ from import alia - variable, function name or class name 

These expressions do not introduce new identifiers:

    augmented assignment
    global declaration
    nonlocal declaration
    calling function/ passing argument to the called function
    while loop  (unless name expression)
    if statement  (unless name expression)
    try/except/finally (unless Exception binding behavior) 
    function parameter default values

## identifier classification
So here are my classification for identifeirs (it's convinent for my masters thesis), Here are a few things I ignored for the Granularity of my thesis:
1. In general if a function is inside a class, then it's a method, we ignore edge cases like nested function inside a class.
2. We also don't differentiate instance method, class method and static method---- although it's easy to implememt this, it's irrelavent to my masters thesis.
3. We don't differentiate kwargs, args and normal parameter ---- it's also an easy fix...

    variable (including class/ enum variable)
    instance variable
    for_loop variable
    with_statement variable
    exception variable
    comprehension variable
    pattern_matching variable
    
    function name 
    class name
    method name 
    function parameter
    method parameter
    lambda parameter

    import alias

## Scope
These things in python introduce new scope (identifier defined inside cannot be accessed outside):

    Builtin scope (Since we are parsing single files, we will ignore this)
    Module scope
    Class scope (including Enum)
    Funciton scope (including async, generator, decorator)
    Lambda scope
    Comprehension scope
    Pattern Matching scope 
    Exception binding behavior scope - except Ex as e

Things that does not indtroduce new scope (identifier defined inside can be accessed outside, so there could be name leak):

    for loop (including async)
    while loop
    if statement
    with statement
    try/except/finally	
There are also nested/enclosed scopes, a funciton inside a class (method), function inside function (nested function), class inside class (inner class); even rare stuff like class inside function or class inside funciton inside class...... The nested level can go very deep as long as there are enough memory. 

Also notice that for "IF, FOR, WHILE, TRY" there can be extra "else", but the else will have the same scope and indentation as them. So not really realavent to identify them in our research. 

## Indentation
These things comes with new indentation
    
    Builtin scope (Since we are parsing single files, we will ignore this)
    Module scope
    Class scope (including Enum)
    Funciton scope (including async, generator, decorator)
    Pattern Matching scope 
    for loop (including async)
    while loop
    if statement
    with statement
    try/except/finally	

These things do not come with indentation

    Lambda scope
    Comprehension scope
    


# Threats to Validity
One biggest threat is that assignments might introduce new identifier, but sometimes it's just assignment to an existing identifier. When we are collecting identifies, we might collect reapeated identifiers, which might affect the vallidity of statistics. I am too lazy to fix this, to fix this, we have to consider LEGB and see if this variable is accessible from outside, which adds complexity. 

A smaller "threat" is that I don't differentiate class method - instance method - static method. Neither do I differentiate class variable (or so called static fields) - normal variable. The fix to this is easy, when walking through funcitons, just check if "deocrator list" contains "classmethod". But it's not relavent to my thesis so I didn't implement it. 


