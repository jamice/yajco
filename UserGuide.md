
---


_WORK IN PROGRESS_


---


**Table of contents**


# Overview #
**YAJCo is tool for simple specification of languages** and generation of language parser and language supporting tools using only annotated Java classes. Each class represents one language concept and is equivalent to abstract syntax definition. Class can contain methods representing language semantics and annotations for specification of concrete syntax.

Each YAJCo project needs to have one [@Parser](#@Parser.md) annotation for specification of root concept and other options.

In next lines you can find out a little more about our great YAJCo tool, or you can go straight to [examples](Examples.md) page and play with them and then return to this user guide. Either way, if you feel like something is not right or missing from this user guide, please [let us know](ContactUs.md).

# Abstract Syntax Definition #

Abstract syntax of the language consists of definition of language concepts and relations between them. In YAJCo language concepts correspond to classes and therefore, each for class corresponding non-terminal in the grammar is defined.

## Inheritance relation 'is-a' ##
Inheritance or implementation of the interface indicates that one concepts is a special case of its parent concepts, i.e. it can be used in all places where the parent concept is expected. YAJCo automatically uses inheritance relation between classes to construct appropriate grammar rules.

| ![https://yajco.googlecode.com/svn/wiki/images/inheritance.png](https://yajco.googlecode.com/svn/wiki/images/inheritance.png) | EBNF:<br><code>Statement::= Move | TurnLeft</code> </tbody></table>

<pre><code>public abstract class Statement {...}<br>
<br>
public class Move extends Statement {...}<br>
<br>
public class TurnLeft extends Statement {...}<br>
</code></pre>

<h2>Composition relation 'has-a' ##

A concept is composed from other concepts when description of concept instance in an input sentence includes description of its subconcepts. In the grammar it is represented by the occurrence of non-terminals of subconcepts on the right-hand side of the rule corresponding to the composed concept.

| ![https://yajco.googlecode.com/svn/wiki/images/composition.png](https://yajco.googlecode.com/svn/wiki/images/composition.png) | EBNF:<br><code>Iteration ::= Expression Statement</code> </tbody></table>

In object-oriented language this relation can be expressed using class fields that contain instances of subconcept classes. Not all fields, however, correspond to composition relation. Because of this YAJCo uses parameters of <b>class constructor</b> (or factory methods) for the specification of subconcepts. This also provides more flexibility for definition of concrete syntax as it defines also ordering of subconcepts.<br>
<br>
<pre><code>Iteration(Expression expression, Statement statement) {...}<br>
</code></pre>

In addition definition of composition on constructors allows to do some preprocessing and store subconcepts in different form.<br>
<br>
<h2>Composition multiplicity ##

A concept can contain multiple instances of subconcepts. This is automatically inferred from the use of array or one of the standard Java collection types. The multiplicity can be restricted using [@Range](UserGuide#@Range.md) annotation.

YAJCo supports multiplicity with following types:
  * Java arrays
  * `java.util.List`
  * `java.util.Set`

## Referencing (agregation) ##
A concept instance description can also contain a reference to instance of another concept that is described elsewhere in the input sentence. In this case only some identifier (marked with [@Identifier](#@Identifier.md) annotation) of referred instance is provided as a part of concept instance definition.

In the object model this relation can be expressed using aggregation. YAJCo again uses constructor parameters as a place, where the relation is described. In this case, parameter is an identifier string marked with [@References](#@References.md) annotation. YAJCo would automatically resolve the reference and inject actual object instance into field with the same type so constructor can actually ignore received identifier.

Simple example of identifiers and references is provided in [Nielsen's DESK language](Examples#Nielsen's_DESK_language.md). For more information about this feature, you can check our paper _[Declarative specification of references in DSLs](http://www.mendeley.com/download/public/27880491/6466511164/71085fd9b9fa436cefae3e71ed71111fcc8f2b06/dl.pdf)_

# Concrete Syntax Definition #

Concrete syntax (notation) for each language concept is specified in a form of annotations attached to class constructors (and factory methods). Constructors allow to describe how an object can be constructed from some input, so it is appropriate to use them also for constructing concept instances from the textual input of language sentence.

The annotations allow to specify tokens, that would be used in concepts' concrete representation. Most of described annotations expect token specification as a parameter. Named tokens are defined in [@Parser](#@Parser.md) annotation in [@#TokenDef @TokenDef] option part. If token would be used only once, it can be specified directly in annotation. If specified string does not represents existing named token, it is taken as exact character sequence to be used in concrete syntax.

## Keywords and symbols ##

The simplest form of concrete syntax annotations is a definition of keywords and symbols that must be part of concept concrete representation. These tokens can be placed before or after concept description using [@Before](#@Before.md) and [@After](#@After.md) annotations associated with concept constructor. Tokens can be also placed between description of subconcepts by using the same annotations on constructor parameters.

In a case of composition multiplicity it is possible to specify tokens that must be placed between each instance of subelement. This can be done with [@Separator](#@Separator.md) annotation.

## Alternative notations ##

It may be possible to describe instances of the same concept using different notations and even with different combinations of subconcepts. This is allowed by the way of using multiple constructors

```
JAVA:
@Before("ITERATE")
Iteration(Expression expr, @Before("TIMES") Statement statement) {...}
    
@Before("ITERATE")
Iteration(Expression expr,
    @Before("TIMES") @After("END") Statement[] statement) {...}


EBNF:
Iteration ::= <ITERATE> Expression <TIMES> Statement
Iteration ::= <ITERATE> Expression <TIMES> Statement* <END>
```

Constructors that should not become part of language definition can be marked using [@Exclude](#@Exclude.md) annotation.

## Operator definition ##
Operators represent a type of language constructs that benefits from special treatment. Otherwise they would require more complex definition of concept relations to express rules of priority and assiciativity.

In YAJCo it is possible to mark concept using [@Operator](#@Operator.md) annotation and define its priority and associativity. Annotation [@Parentheses](#@Parentheses.md) can be used to indicate the possibility to use parentheses to explicitly express priority.

## Tokens with value ##

Since language concepts can contain not only other concepts, but also primitive values, like numbers and strings, it is needed to specify their concrete notation. This is done by attaching [@Token](#@Token.md) annotation with a regular expression as a parameter to the parameter of the constructor. A string that matches the regular expression is then provided as a value of the parameter.

If you do not use `@Token` annotation YAJCo derives names of tokens from names o parameters. There are no preddefined tokens in YAJCo, so every token needs to be defined in [@Parser](#@Parser.md) annotation or with annotations inside concept classes.

Examples of usage of `@Token` annotation are in all provided [examples](Examples.md) as it is common task to mark parameters as input values.

# YAJCo annotations #
In next section we will describe each annotation usable in YAJCo language specification.

## Main annotations ##

### `@Before` and `@After` ###
Parameter:
  * `String[] value();`
    * set of actual strings or names of defined tokens
    * main parameter => does not need to be named in code

Describes placement of language tokens in concrete syntax of language. Can be used as annotation on _constructor_, _factory method_ and _parameters_ in constructors and factory methods. Specified tokens in `value` parameter can be:
  * actual textual representation of required tokens
  * names of regex tokens defined with `@TokenDef` inside `@Parser` configuration

```
Java:                          |  EBNF:
                               |
@Before("move")                |  Move ::= <move>
Move() {}                      |                       
```

```
Java:                          |  EBNF:
@Before("iterate")             |  
@After("end")                  |  Iteration ::= <iterate> Expression <times> Statement <end>
Iteration(Expression expr,     |
    @Before("times")           |
    Statement statement) {...} |
```
### `@Separator` ###
Parameter:
  * `String value();`
    * one string or name of defined tokens to be used as separator in list of elements
    * main parameter => does not need to be named in code

In case you need to specify token for separation of elements used in array or list, this is the annotation you would need.

Use only on parameters of types:
  * array - `Type[]`
  * `List`
  * `Set`

```
Java:                          |  EBNF:
@Before("define")              |  
Definition(String ident,       |  Definition ::= <define> IDENT <as> Statement (<;> Statement)*
    @Before("as")              |
    @Range(minOccurs = 1)      |
    @Separator(";")            |
    Statement[] stmts) {...}   |
```

### `@Token` ###
Parameter:
  * `String value();`
    * one name of defined token to be used as regular expression to parse value for annotated parameter
    * main parameter => does not need to be named in code

Since language concepts can contain not only other concepts, but also primitive values, like numbers and strings, it is needed to specify their concrete notation. This is done by attaching `@Token` annotation with a name of a regular expression as a parameter to the parameter of the constructor. Regular expression have to be specified inside `@TokenDef` in `@Parser` configuration annotation. A string that matches the regular expression is then provided as a value of the parameter.

In addition, if constructor has parameters of primitive Java types like int or double, YAJCo automatically convert matched strings to appropriate type.

Annotation `@Token` is not required in case that name of parameter and name of configured token is same (case-insensitive), it that case configured token is automatically maped to parameter.

`@Token` can be used on parameters of constructors and factory methods only.

```
    @Before("state")
    @After(";")
    public State(@Token("ID") String id) {
        this.label = id;
    }
-----------------------or----------------------
    @Before("state")
    @After(";")
    public State(String id) { //parameter name identical with token name, @Token annotation not needed
        this.label = id;
    }
```

### `@Range` ###
Parameters:
  * `int minOccurs() default 0;`
    * defines minimal number of occurence of elements in annotated list
  * `int maxOccurs() default -1;`
    * defines maximal number of occurence of elements in annotated list
    * value `-1` is used for _infinity_

A concept can contain multiple instances of subconcepts. This is automatically inferred from the use of array or one of the standard Java collection types. The multiplicity can be restricted using `@Range` annotation.

```
Java:                          |  EBNF:
Program(                       |
    @Range(minOccurs = 1)      |  Program ::= Statement+
    Statement[] stmts) {...}   |
```
`@Range` can be used on `array`, `List` or `Set` typed parameters of constructors and factory methods only.

### `@Operator` ###
Parameters:
  * `int priority() default 1;`
    * defines priority for operator notation, higher number means higher priority
  * `Associativity associativity() default Associativity.AUTO;`
    * defines associativity for operator notation, allowed values are `LEFT`, `RIGHT`, `NONE`, `AUTO` (actually translates to `LEFT`)

Operators represent a type of language constructs that benefits from special treatment. Otherwise they would require more complex definition of concept relations to express rules of priority and associativity.
In YAJCo it is possible to mark concept using `@Operator` annotation and
define its priority and associativity.

`@Operator` can be used on constructors.

```
    @Operator(priority = 1, associativity=Associativity.LEFT)
    public Add(Expression expression1, @Before("PLUS") Expression expression2) {
        this.expression1 = expression1;
        this.expression2 = expression2;
    }
```

For better practical understanding, see [Math expression language](Examples#Math_Expression_language.md) in examples or any other more complex example language.

### `@Parentheses` ###
Parameters:
  * `String left() default "(";`
    * defines token used for left parentheses, can be specified also as configured named token
  * `String right() default ")";`
    * defines token used for right parentheses, can be specified also as configured named token

Annotation `@Parentheses` can be used to indicate the possibility to use parentheses to explicitly express priority of operators. Usually annotations `@Operator` and `@Parentheses` are used in same language. In practical approach annotation `@Parentheses` is mostly used on abstract classes, which serve as parent for other more specific classes (language concepts).

`@Parentheses` can be used on classes.

```
@Parentheses
public abstract class Expression {

    public abstract long eval();

}
```

For better practical understanding, see [Math expression language](Examples#Math_Expression_language.md) in examples or any other more complex example language.

### `@Identifier` ###
Parameter:
  * `String unique() default "";`
    * used to define scope of uniqueness of marked identifier field, scope is defined using XPath expression

YAJCo annotation `@Identifier` serves for easy specification of referencable identifiers in language concepts (classes). Most of languages uses some sort of referencable names (identifiers). It is possible to mark field of language concept with `@Identifier` annotation. Strings inputed to such field will be automatically checked for uniqueness during parsing of sentence. Standard behavious is checking uniqueness in global scope of a language sentence, but it is possible to specify own scope for uniqueness check using `unique` parameter.

```
public class Function {
    @Identifier
    private final String name;
...
}
```

```
public class Parameter {
    @Identifier(unique = "../yajco.example.imperative.model.function.Parameter")
    private final String name;
...
}
```

Simple example of identifiers and references is provided in [Nielsen's DESK language](Examples#Nielsen's_DESK_language.md). For more information about this feature, you can check our paper _[Declarative specification of references in DSLs](http://www.mendeley.com/download/public/27880491/6466511164/71085fd9b9fa436cefae3e71ed71111fcc8f2b06/dl.pdf)_

### `@References` ###
Parameters:
  * `Class value();`
    * defines class of referenced language element
  * `String field() default "";`
    * defines name of class field in which should be injected referenced element
  * `String path() default "";`
    * defines scope for searching referenced element in, scope is defined in form of XPath expression
  * `boolean create() default false;`
    * states if YAJCo should create element if referenced identifier is not found _(actually not implemented)_

`@References` annotation is used to mark **`String`** parameters of _constructors_ or _factory methods_. Such parameters represents places for referencing existing language structures.

During parsing YAJCo would inject proper references into the fields of objects using Java reflection mechanism. Reference resolution can be restricted using XPath expression specifying nodes in constructed object tree that would be searched for referred objects. This allows to specify scoping rules and namespaces.

It is required to specify type of referencing class using `value` parameter. YAJCo automaticaly searches for field with specified type. If there are more types, it is needed to use `field` parameter to exactly specify targeted field for injection with referenced language structure. Thanks to YAJCo automatic injection it is not required to do anything with actual value of annotated parameter in constructor (as it is possible to see in examples below), value of _String_ parameter is stored internally in YAJCo during parsing phase.

```
...
private Function function;

public FunctionCall(
        @References(Function.class) String functionIdent,
        @Before("(") @After(")")
        @Separator(",") Expression[] expressions) {
    this.expressions = expressions;
}
...
```

```
...
private State source;

private State target;

@Before("trans")
@After(";")
public Transition(
       @Token("ID") String label,
       @Before(":")
       @Token("ID")
       @References(value = State.class, field = "source") String sourceLabel,
       @Before("->")
       @Token("ID")
       @References(value = State.class, field = "target") String targetLabel) {
  this.label = label;
}
...
```

Simple example of identifiers and references is provided in [Nielsen's DESK language](Examples#Nielsen's_DESK_language.md). For more information about this feature, you can check our paper _[Declarative specification of references in DSLs](http://www.mendeley.com/download/public/27880491/6466511164/71085fd9b9fa436cefae3e71ed71111fcc8f2b06/dl.pdf)_

### `@FactoryMethod` ###
Serves as marking annotation for specifying static methods, which will be included in creation of abstract syntax. Standard behaviour of YAJCo is to take all constructors as representation of concrete syntax for language concept (class). There can sometimes be problems with specification of all wanted concrete syntax notations with constructors, as they do not allow to have same signature even when annotations are different. Therefore it is sometimes required to create factory methods and mark them with our annotation.

```

@FactoryMethod
@Before("noop")
public static Statement createNoopStatement() {
...
}

@FactoryMethod
@Before("exit")
public static Statement createExitStatement() {
...
}

```

### `@Exclude` ###
Annotation `@Exclude` serves to mark constructor as excluded from abstract syntax specification. Any marked constructor will not be included in language specification.

```
...
@Exclude
public Statement() {}
...
```

Annotation `@Exclude` can be used also on class. Such class will be excluded from YAJCo language specification. It is needed when you don't want some class, which is in hierarchy of language concepts to appear in language specification.

```
@Exclude
public class NotImportant extends Statement {
...
}
```


## Config annotations ##
### `@Parser` ###
Parameters:
  * `String className() default "";`
    * defines class name for generated parser
  * `String mainNode() default "";`
    * defines name of root language concept, can be in form of full class name or in simple class name in case class is in the same package; if annotation `@Parser` is used on class, this parameter is not needed
  * `TokenDef[] tokens() default {};`
    * contains list of named lexical symbols (tokens)
  * `Skip[] skips() default {};`
    * contains list of ignored (whitespace) characters of regular expressions; if left empty, YAJCo automatically includes `\s` regular expression for whitespaces
  * `Option[] options() default {};`
    * provides way for configuration of YAJCo and modules

`@Parser` annotation is main configuration element of YAJCo tool. It is required to use this annotation once in project in order to start YAJCo tool. `@Parser` annotation can be included on class or on package (using `package-info.java` file).

Its parameters allow to specify root concept of the language, name of the generated parser class. There is also specication of lexical analyzer including denition of named tokens and regular expressions for substring that should be ignored (white-space, comments). And it is one of few input points for configurations of YAJCo and any YAJCo module.

```
@Parser(className = "yajco.exemple.sml.parser.StateMachineParser",
        mainNode = "StateMachine",
        tokens = {
                @TokenDef(name = "ID", regexp = "[a-zA-Z][a-zA-Z0-9]*")
        },
        skips = {
                @Skip("#.*\\n"), //comment
                @Skip(" "),
                @Skip("\\t"),
                @Skip("\\n"),
                @Skip("\\r")
        }) package yajco.example.sml.model;

import yajco.annotation.config.Parser;
import yajco.annotation.config.Skip;
import yajco.annotation.config.TokenDef;
```

### `@TokenDef` ###
Parameters:
  * `String name();`
    * defines name of lexical symbol (token)
  * `String regexp();`
    * defines regular expression for token

Used as part of configuration in `@Parser` annotation parameter `tokens`.

### `@Skip` ###
Parameters:
  * `String value();`
    * defines character set or regular expression to be skipped during parsing

Used as part of configuration in `@Parser` annotation parameter `skips`.

### `@Option` ###
Parameters:
  * `String name();`
    * defines name of configuration option
  * `String value();`
    * defines value of configuration option

Used as part of configuration in `@Parser` annotation parameter `options`.

## Printer annotations ##
YAJCo allows generating printer for specified language. Printer allows to print model of language sentence in form of concrete syntax. In order to specify formatting of printed text, we have introduced new annotations.

### `@NewLine` ###
Specifies place where should be put a new line in printer output. New line is put before annotated construct.

### `@Indent` ###
Parameters:
  * `int level() default 1;`
    * defines level of indentation

Specifies indentation of annotatated element. Mostly it should be used in combination with `@NewLine`. Annotated element is indented specified number of levels to right in printer output. Indentation is inherited for all subelements of indented element.

# Restrictions #
  * Do not use `long` or `float` Java primitives as types for YAJCo language specification constructors. They are not supported, yet.
  * Language concept class must be inside a package. Default package classes are not permitted.
```
OK:
  yajco.example.Concept

NOT OK:
  ConceptWithoutPackage
```
  * Each language concept class must be in the same package as main (root) language concept or in subpackages. It is not possible to have classes representing language concepts in completely different packages.
```
OK:
  yajco.example.MainConcept
  yajco.example.SecondaryConcept
  yajco.example.subpackage.SpecialConcept
  yajco.example.subpackage.subsub.VerySpecialConcept

NOT OK:
  yajco.example.MainConcept
  yajco.special.DifferentConcept
  com.google.SearchConcept
```

# IDEs how-to #
Using Maven project with YAJCo in most common IDEs can provide better language specification experience, but there are also some possible problems.

## Netbeans ##
Netbeans has problems with generated sources during compile time and sometimes cannot index them as existing sources, therefore it does not provide them for error checking. Even when build and run is working OK with Maven in Netbeans, project report fictious Netbeans' errors. It is pretty problematic, when you are trying to use generated parser in main class as it shows errors, but compile and run is OK. Netbeans is OK editor for YAJCo projects, just ignore errors concerning generated classes.

## IntelliJ IDEA ##
In IntelliJ IDEA it is required to use Maven building and running options instead of classical IDEs build and run options, as they are influencing annotated source code for YAJCo language specification in unexpected manner (mostly with Beaver parser generator, JavaCC parser generator seams OK)

`Run` configuration should not use internal `make` but rather use Maven `package` or `compile` goals before running main class with configuration. See projects provided in [examples](Examples.md) section, as they contain properly configured _run configurations_ for IntelliJ IDEA

## Eclipse ##
Eclipse is pretty without problems, as long as you are able to run Maven builds. We have not experienced any problems, but user interface of Eclipse can be pretty problematic for begginner, so we are not recommending to use it as long as you don't have previous experience with it.