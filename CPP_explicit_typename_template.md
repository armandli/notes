explicit need to use typename and template in certain context are all due to syntactic limitations within the C++ language.

some names denote types or templates. in general, whenever a name is encountered it is necessary to determine whether that name denotes one of these entities before continuing parsing the program that contains it. the process that determines this is called name lookup.

### When to explicitly give typename keyword
a name is used in a template declaration or definition and that is dependent on a template parameter is assumed not to name a type unless the applicable name lookup finds a type name or the name is qualified by the keyword typename.

when t::x is a dependent name, we prefix typename to it to signal that x is a type, not a class member. t in the context has to be a type function, i.e. something that need to take a set of template parameters, and the template parameters are not explicitly realized in the context. otherwise using typename qualification is a syntactic error.

there are also simple statements where t::x is used in a place where there can only be a type. e.g. `t::x a;` in such context, typename keyword is also not necessary.

typename parsing resolution is never needed in front of unqualified names.

### When to explicitly give template keyword
after name lookup finds that a name is a template-name, if this name is followed by a `<`, the `<` is always taken as the begining of a template-argument-list and never as a name followed by less than operator.

template keyword is required when a name may not be known during parsing as a template name and need to be parsed as if it is. this could happen in a qualified name lookup where the name after qualification is a template, but could not be known until template instantiation. e.g.

```
t::template f<int>(); // f is a hidden template type inside t, we force f to be recognized as template using tempalte f so < > are parsed as template argument list
```

template names could also occur after `->` or a `.`

```
this->template f<int>();
```

it's totally possible that in a certain context both typename and template keywords are used at the same time for syntactic disambiguation. e.g.

```
template <typename T, typename Tail>
struct UnionNode : public Tail {
  template <typename U> struct Union {
    typedef typename Tail::template inUnion<U> dummy;
  };
};
```

#### Detailed Explanation
template constructs can have different meaning depending on the template argument being passed in. The standard define if a construct is dependent or not, and separate them into 3 different groups:
1. dependent type, such as template parameter T
2. value-dependent expressions, such as non-type template parameter N
3. type-dependent expressions, such as a cast to type template parameter (T)0

most of the rules are intuitive and are built recursively.

the standard is not exactly clear regarding what is a dependent **name**. a name is a use of an identifier, operator-function-id, conversion-function-id, template-id, that denotes an entity or label. so a template including its set of template arguments, is wholely considered a name, so is the qualification.

function names are handled separately. identifier of function name is dependent no by itself, but by the type dependent argument expressions used in the call (type dependent name lookup).
