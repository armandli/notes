#### Special Member Function Existence in Different Conditions

|                     | Default Constructor | Copy Constructor | Copy Assignment | Move Constructor | Move Assignment | Destructor |
|---------------------|---------------------|------------------|-----------------|------------------|-----------------|------------|
| Nothing             | YES                 | YES              | YES             | YES              | YES             | YES        |
| Any Constructor     | NO                  | YES              | YES             | YES              | YES             | YES        |
| Default Constructor | OVERRIDDEN          | YES              | YES             | YES              | YES             | YES        |
| Copy Constructor    | NO                  | OVERRIDDEN       | DEPRECATED      | NO               | NO              | YES        |
| Copy Assignment     | YES                 | DEPRECATED       | OVERRIDDEN      | NO               | NO              | YES        |
| Move Constructor    | NO                  | DELETED          | DELETED         | OVERRIDDEN       | NO              | YES        |
| Move Assignment     | YES                 | DELETED          | DELETED         | NO               | OVERRIDDEN      | YES        |
| Destructor          | YES                 | DEPRECATED       | DEPRECATED      | NO               | YES             | OVERRIDDEN |

note the table above only keeps track of the condition under each independent special member function; often a class contains multiple user defined special member functions that result in the combined state of multiple rows above

where
YES means defined by default by compiler;
NO means not defined by compiler;
OVERRIDDEN means user overrides the default implementation;
DEPRECATED means compiler defined for the moment, but could be removed in the future
DELETED means compiler defines the special member function state is deleted
