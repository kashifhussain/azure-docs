---
title: Azure Service Bus Subscription Rule SQL Filter syntax | Microsoft Docs
description: This article provides details about SQL filter grammar. A SQL filter supports a subset of the SQL-92 standard.  
ms.topic: article
ms.date: 08/16/2023
---

# Subscription Rule SQL Filter Syntax

A *SQL filter* is one of the available filter types for Service Bus topic subscriptions. It's a text expression that leans on a subset of the SQL-92 standard. Filter expressions are used with the `sqlExpression` element of the 'sqlFilter' property of a Service Bus `Rule` in an [Azure Resource Manager template](service-bus-resource-manager-namespace-topic-with-rule.md), or the Azure CLI `az servicebus topic subscription rule create` command's [`--filter-sql-expression`](/cli/azure/servicebus/topic/subscription/rule#az-servicebus-topic-subscription-rule-create) argument, and several SDK functions that allow managing subscription rules. The allowed expressions are shown in this section.

Service Bus Premium also supports the [JMS SQL message selector syntax](https://docs.oracle.com/javaee/7/api/javax/jms/Message.html) through the JMS 2.0 API.

  
``` 
<predicate ::=  
      { NOT <predicate> }  
      | <predicate> AND <predicate>  
      | <predicate> OR <predicate>  
      | <expression> { = | <> | != | > | >= | < | <= } <expression>  
      | <property> IS [NOT] NULL  
      | <expression> [NOT] IN ( <expression> [, ...n] )  
      | <expression> [NOT] LIKE <pattern> [ESCAPE <escape_char>]  
      | EXISTS ( <property> )  
      | ( <predicate> )  
  
```  
  
```  
<expression> ::=  
      <constant>   
      | <function>  
      | <property>  
      | <expression> { + | - | * | / | % } <expression>  
      | { + | - } <expression>  
      | ( <expression> )  
  
```  
  
```  
<property> :=   
       [<scope> .] <property_name>  
  
```  
  
## Arguments  
  
-   `<scope>` is an optional string indicating the scope of the `<property_name>`. Valid values are `sys` or `user`. 
    - The `sys` value indicates system scope where `<property_name>` is any of the properties on the Service Bus message as described in [Messages, payloads, and serialization](service-bus-messages-payloads.md).
    - The `user` value indicates user scope where `<property_name>` is a key of the custom properties that you can set on the message when sending to Service  Bus.
    - The `user` scope is the default scope if `<scope>` isn't specified.  
  
## Remarks

An attempt to access a nonexistent system property is an error, while an attempt to access a nonexistent user property isn't an error. Instead, a nonexistent user property is internally evaluated as an unknown value. An unknown value is treated specially during operator evaluation.  
  
## property_name  
  
```  
<property_name> ::=  
     <identifier>  
     | <delimited_identifier>  
  
<identifier> ::=  
     <regular_identifier> | <quoted_identifier> | <delimited_identifier>  
  
```  
  
### Arguments  

 `<regular_identifier>` is a string represented by the following regular expression:  
  
```  
[[:IsLetter:]][_[:IsLetter:][:IsDigit:]]*  
```  
  
This grammar means any string that starts with a letter and is followed by one or more underscore/letter/digit.  
  
`[:IsLetter:]` means any Unicode character that is categorized as a Unicode letter. `System.Char.IsLetter(c)` returns `true` if `c` is a Unicode letter.  
  
`[:IsDigit:]` means any Unicode character that is categorized as a decimal digit. `System.Char.IsDigit(c)` returns `true` if `c` is a Unicode digit.  
  
A `<regular_identifier>` can't be a reserved keyword.  
  
`<delimited_identifier>` is any string that is enclosed with left/right square brackets ([]). A right square bracket is represented as two right square brackets. The following are examples of `<delimited_identifier>`:  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
`<quoted_identifier>` is any string that is enclosed with double quotation marks. A double quotation mark in identifier is represented as two double quotation marks. It isn't recommended to use quoted identifiers because it can easily be confused with a string constant. Use a delimited identifier if possible. Here's an example of `<quoted_identifier>`:  
  
```  
"Contoso & Northwind"  
```  
  
## pattern  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### Remarks
  
`<pattern>` must be an expression that is evaluated as a string. It's used as a pattern for the LIKE operator.      It can contain the following wildcard characters:  
  
-   `%`:  Any string of zero or more characters.  
  
-   `_`: Any single character.  
  
## escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### Remarks  

`<escape_char>` must be an expression that is evaluated as a string of length 1. It's used as an escape character for the LIKE operator.  
  
 For example, `property LIKE 'ABC\%' ESCAPE '\'` matches `ABC%` rather than a string that starts with `ABC`.  
  
## constant  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### Arguments  
  
-   `<integer_constant>` is a string of numbers that aren't enclosed in quotation marks and don't contain decimal points. The values are stored as `System.Int64` internally, and follow the same range.  
  
     Here are examples of long constants:  
  
    ```  
    1894  
    2  
    ```  
  
-   `<decimal_constant>` is a string of numbers that aren't enclosed in quotation marks, and contain a decimal point. The values are stored as `System.Double` internally, and follow the same range/precision.  
  
     In a future version, this number might be stored in a different data type to support exact number semantics, so you shouldn't rely on the fact the underlying data type is `System.Double` for `<decimal_constant>`.  
  
     The following are examples of decimal constants:  
  
    ```  
    1894.1204  
    2.0  
    ```  
  
-   `<approximate_number_constant>` is a number written in scientific notation. The values are stored as `System.Double` internally, and follow the same range/precision. The following are examples of approximate number constants:  
  
    ```  
    101.5E5  
    0.5E-2  
    ```  
  
## boolean_constant  
  
```  
<boolean_constant> :=  
      TRUE | FALSE  
```  
  
### Remarks  

Boolean constants are represented by the keywords **TRUE** or **FALSE**. The values are stored as `System.Boolean`.  
  
## string_constant  
  
```  
<string_constant>  
```  
  
### Remarks  

String constants are enclosed in single quotation marks and include any valid Unicode characters. A single quotation mark embedded in a string constant is represented as two single quotation marks.  
  
## function  
  
```  
<function> :=  
      newid() |  
      property(name) | p(name)  
```  
  
### Remarks
  
The `newid()` function returns a `System.Guid` generated by the `System.Guid.NewGuid()` method.  
  
The `property(name)` function returns the value of the property referenced by `name`. The `name` value can be any valid expression that returns a string value.  
  
## Considerations
  
Consider the following Sql Filter semantics:  
  
-   Property names are case-insensitive.  
  
-   Operators follow C# implicit conversion semantics whenever possible.  
  
-   System properties are any of the properties on the Service Bus message as described in [Messages, payloads, and serialization](service-bus-messages-payloads.md).
  
	Consider the following `IS [NOT] NULL` semantics:  
  
	-   `property IS NULL` is evaluated as `true` if either the property doesn't exist or the property's value is `null`.  
  
### Property evaluation semantics  
  
- An attempt to evaluate a nonexistent system property throws a `FilterException` exception.  
  
- A property that doesn't exist is internally evaluated as **unknown**.  
  
  Unknown evaluation in arithmetic operators:  
  
- For binary operators, if either the left or right side of operands is evaluated as **unknown**, then the result is **unknown**.  
  
- For unary operators, if an operand is evaluated as **unknown**, then the result is **unknown**.  
  
  Unknown evaluation in binary comparison operators:  
  
- If either the left or right side of operands is evaluated as **unknown**, then the result is **unknown**.  
  
  Unknown evaluation in `[NOT] LIKE`:  
  
- If any operand is evaluated as **unknown**, then the result is **unknown**.  
  
  Unknown evaluation in `[NOT] IN`:  
  
- If the left operand is evaluated as **unknown**, then the result is **unknown**.  
  
  Unknown evaluation in **AND** operator:  
  
```  
+---+---+---+---+  
|AND| T | F | U |  
+---+---+---+---+  
| T | T | F | U |  
+---+---+---+---+  
| F | F | F | F |  
+---+---+---+---+  
| U | U | F | U |  
+---+---+---+---+  
```  
  
 Unknown evaluation in **OR** operator:  
  
```  
+---+---+---+---+  
|OR | T | F | U |  
+---+---+---+---+  
| T | T | T | T |  
+---+---+---+---+  
| F | T | F | U |  
+---+---+---+---+  
| U | T | U | U |  
+---+---+---+---+  
```  
  
### Operator binding semantics
  
-   Comparison operators such as `>`, `>=`, `<`, `<=`, `!=`, and `=` follow the same semantics as the C# operator binding in data type promotions and implicit conversions.  
  
-   Arithmetic operators such as `+`, `-`, `*`, `/`, and `%` follow the same semantics as the C# operator binding in data type promotions and implicit conversions.

## Examples
For examples, see [Service Bus filter examples](service-bus-filter-examples.md).

## Next steps

- [SQLFilter class (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlfilter)
- [SQLFilter class (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlfilter)
- [SqlFilter class (Java)](/java/api/com.microsoft.azure.servicebus.rules.SqlFilter)
- [SqlRuleFilter (JavaScript)](/javascript/api/@azure/service-bus/sqlrulefilter)
- [`az servicebus topic subscription rule`](/cli/azure/servicebus/topic/subscription/rule)
- [New-AzServiceBusRule](/powershell/module/az.servicebus/new-azservicebusrule)
