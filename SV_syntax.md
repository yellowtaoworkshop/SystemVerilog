# SV syntax 

## $cast usage 

- general description 

  When $cast is applied to class handles, it succeeds in only three cases:
  1) The source expression and the destination type are assignment compatible, that is, **the destination is the same type or a superclass of the source expression**.
  2) The type of the source expression is cast compatible with the destination type, that is, either:
  — the type of the source expression is a superclass of the destination type, or
  — the type of the source expression is an interface class (see 8.26)
  and the source is an object that is assignment compatible with the destination type. This type of
  assignment requires a run-time check as provided by $cast.
  3) The source expression is the literal constant null. 

- Note: 

  assignment compatible:

  the destination is the same type or a superclass of the source expression 

  cast compatible: (two condition)

  1. source (handle type) is a superclass of the destination or source is an interface class 
  2. the source is an object (that the handle pointed) that is assignment compatible with the destination type

  

## Interface Class

1.  What can the interface do?

   The interface class can make a set of classes have a same APIs, but you can implemented differently. With it, it unnecessary for related classes to share a common abstract superclass or for that superclass to contain all method definitions needed by all subclasses. And in some situations, it is impossible to do that. For example, the uvm_driver and uvm_sequence and uvm_monitor need have a same API, it it impossible to makes them to extend from one subclass which define the virtual APIs.   

2. Some rules about the interface class

   - An interface class ***shall only*** contain pure virtual methods, type declarations and the parameter declarations.  Constraint blocks, covergroups, and nested classes ***shall not be*** allowed in an interface class.

   - the interface class can extends for another interface class 

     ``` verilog
     interface class A;
     endclass
         
     interface class B extends A;
     endclass 
     ```

   - types can be parameterized in the class definition and the resolved types used in the implemented classes 

     ``` verilog
     virtual class XFifo#(type T_in = logic, type T_out = logic, int DEPTH = 1)
         extends MyQueue#(T_out) implements PutImp#(T_in), GetImp#(T_out);
     	pure virtual function T_out translate(T_in a);
     	virtual function void put(T_in a);
     		PipeQueue.push_back(translate(a));
     	endfunction
     	virtual function T_out get();
     		get = PipeQueue.pop_front();
     	endfunction
     endclass
     ```

   - An inherited ***virtual*** method can provide the implementation for a method of an implemented interface class. But the non-virtual won't.
     Here is an example:

     ```verilog
     interface class IntfClass;
     	pure virtual function bit funcBase();
     	pure virtual function bit funcExt();
     endclass
     class BaseClass;
     	virtual function bit funcBase();
     		return (1);
     	endfunction
     endclass
     class ExtClass extends BaseClass implements IntfClass;
     	virtual function bit funcExt();
     		return (0);
     	endfunction
     endclass       
             
     // part 2: show that the non-virtual will not implement the API in the interface class
     interface class IntfClass;
     pure virtual function void f();
     endclass
     class BaseClass;
     function void f();
     $display("Called BaseClass::f()");
     endfunction
     endclass
     class ExtClass extends BaseClass implements IntfClass;
     virtual function void f();
     $display("Called ExtClass::f()");
     endfunction
     endclass
     /*
     The non-virtual function f() in BaseClass does not fulfill the requirement to implement IntfClass. The
     implementation of f() in ExtClass simultaneously hides the f() of BaseClass and fulfills the
     requirement to implement IntfClass.
     /*
     ```

   - regarding the interface class handle assignment. The rule is same with superclass handle. 
   - virtual class can partial implementation the interface class 
   - interface class can not be constructed. You can define the handle, but you can''t allocate the memory for it.

3. extends VS. implements 

   Conceptually extends is a mechanism to add to or modify the behavior of a superclass while implements is a requirement to provide implementations for the pure virtual methods in an interface class. When a class is extended, all members of the class are inherited into the subclass. When an interface class is implemented, nothing is inherited.



## Virtual Class (called abstract class)

abstract class can't be create directly.  like 

``` verilog
virtual class A;
    pure vritual task tsk;
endclass 
 
A A_inst = new(); //illegal 
```

the abstract can have follwing type method:

1. pure virtual
2. virtual 
3. nonvirtual 



## External Function 

When you plan to declare a extern virtual function inside the class, and want to implement it outside the class. 

You shouldn’t keep the **extern** and **virtual* key word when implementing 

```verilog
class A;
		//Declare 
		extern virtual function void build_phase(uvm_phase phase);
endclass
        
//implementing 
//The one with Syntax error 
virtual function void A:build_phase(uvm_phase phase);
    
//The one without syntax error
function void A::build_phase(uvm_phase phase);  
      
```

## Packed Array

This section used to reminder me how to use the multiple dimensional packed array. 

```verilog
//For example, we have such multiple dimensional array
bit [A-1:0][B-1:0] array[C-1:0][D-1:0]; 

//When you reference this array, each number inside the "[]" maps which dimensionary? 
array[idx_C][idx_D][idx_A][idx_B]
```

See the snapshot below from the book *SystemVerilog for Verification* 

![packed array](..\snapshots\packed array.png)

## Assignment patterns 

*Assignment patterns* are used for assignments to describe patterns of assignments to structure fields and
array elements.
An assignment pattern specifies a correspondence between a collection of expressions and the fields and
elements in a data object or data value. An assignment pattern has no self-determined data type, but can be
used as one of the sides in an assignment-like context (see 10.8) when the other side has a self-determined
data type. An assignment pattern is built from braces, keys, and expressions and is prefixed with an
apostrophe. For example:

```verilog
var int A[N] = '{default:1};
var integer i = '{31:1, 23:1, 15:1, 8:1, default:0};
typedef struct {real r, th;} C;
var C x = '{th:PI/2.0, r:1.0};
var real y [0:1] = '{0.0, 1.1}, z [0:9] = '{default: 3.1416};
```

A positional notation without keys can also be used. For example:

```verilog
var int B[4] = '{a, b, c, d};
var C y = '{1.0, PI/2.0};
'{a, b, c, d} = B; //What's that !!!!
```

When an assignment pattern is used as the left-hand side of an assignment-like context, the positional
notation shall be required; and each member expression shall have a bit-stream data type that is assignment
compatible with and has the same number of bits as the data type of the corresponding element on the righthand
side.

An assignment pattern can be used to construct or deconstruct a structure or array by prefixing the pattern
with the name of a data type to form an assignment pattern expression. Unlike an assignment pattern, *an
assignment pattern expression* has a self-determined data type and is not restricted to being one of the sides
in an assignment-like context. When an assignment pattern expression is used in a right-hand expression, it
shall yield the value that a variable of the data type would hold if it were initialized using the assignment
pattern.

```verilog
typedef logic [1:0] [3:0] T;
shortint'({T'{1,2}, T'{3,4}}) // yields 16'sh1234
```

When an assignment pattern expression is used in a left-hand expression, the positional notation shall be
required; and each member expression shall have a bit-stream data type that is assignment compatible with
and has the same number of bits as the corresponding element in the data type of the assignment pattern
expression. If the right-hand expression has a self-determined data type, then it shall be assignment
compatible with and have the same number of bits as the data type of the assignment pattern expression.

```verilog
typedef byte U[3];
var U A = '{1, 2, 3};
var byte a, b, c;
U'{a, b, c} = A;
U'{c, a, b} = '{a+1, b+1, c+1};
```

### Array assignment patterns 

Concatenation braces are used to construct and deconstruct simple bit vectors. A similar syntax is used to
support the construction and deconstruction of arrays. The expressions shall match element for element, and
the braces shall match the array dimensions. Each expression item shall be evaluated in the context of an
assignment to the type of the corresponding element in the array. In other words, the following examples are
not required to cause size warnings:

```verilog
bit unpackedbits [1:0] = '{1,1}; // no size warning required as
// bit can be set to 1
int unpackedints [1:0] = '{1'b1, 1'b1}; // no size warning required as
// int can be set to 1'b1
```

A syntax resembling replications (see 11.4.12.1) can be used in array assignment patterns as well. Each
replication shall represent an entire single dimension. 

```verilog
unpackedbits = '{2 {y}} ; // same as '{y, y}
int n[1:2][1:3] = '{2{'{3{y}}}}; // same as '{'{y,y,y},'{y,y,y}}
```

It can sometimes be useful to set array elements to a value without having to keep track of how many
members there are. This can be done with the default keyword:

```verilog
initial unpackedints = '{default:2};
```

For arrays of structures, it is useful to specify one or more matching type keys, as described under structure
assignment patterns following

```verilog
struct {int a; time b;} abkey[1:0];
abkey = '{'{a:1, b:2ns}, '{int:5, time:$time}};
```

The matching rules are as follows:

- An index:value specifies an explicit value for a keyed element index. The value is evaluated in
  the context of an assignment to the indexed element and shall be castable to its type. It shall be an
  error to specify the same index more than once in a single array pattern expression.
- For type:value, if the element or subarray type of the array matches this type, then each element
  or subarray that has not already been set by an index key above shall be set to the value. The value
  shall be castable to the array element or subarray type. Otherwise, if the array is multidimensional,
  then there is a recursive descent into each subarray of the array using the rules in this subclause and
  the type and default keys. Otherwise, if the array is an array of structures, there is a recursive descent
  into each element of the array using the rules for structure assignment patterns and the type and
  default keys. If more than one type matches the same element, the last value shall be used.
- The default:value applies to elements or subarrays that are not matched by either index or type
  key. If the type of the element or subarray is a simple bit vector type, matches the self-determined
  type of the value, or is not an array or structure type, then the value is evaluated in the context of
  each assignment to an element or subarray by the default and shall be castable to the type of the
  element or subarray; otherwise, an error is generated. For unmatched subarrays, the type and default
  specifiers are applied recursively according to the rules in this subclause to each of its elements or
  subarrays. For unmatched structure elements, the type and default keys are applied to the element
  according to the rules for structures.

### Structure assignment patterns

A structure can be constructed and deconstructed with a structure assignment pattern built from member
expressions using braces and commas, with the members in declaration order. Replication operators can be
used to set the values for the exact number of members. Each member expression shall be evaluated in the
context of an assignment to the type of the corresponding member in the structure. It can also be built with
the names of the members.

```verilog
module mod1;
typedef struct {
int x;
int y;
} st;
st s1;
int k = 1;
initial begin
#1 s1 = '{1, 2+k}; // by position
#1 $display( s1.x, s1.y);
#1 s1 = '{x:2, y:3+k}; // by name
#1 $display( s1.x, s1.y);
#1 $finish;
end
endmodule
```

It can sometimes be useful to set structure members to a value without having to keep track of how many
members there are or what the names are. This can be done with the default keyword: (same with the array)

```verlog
initial s1 = '{default:2}; // sets x and y to 2
```

The '{member:value} or '{data_type: default_value} syntax can also be used:

```verilog
ab abkey[1:0] = '{'{a:1, b:1.0}, '{int:2, shortreal:2.0}};
```

The matching rules are as follows:

- A member:value specifies an explicit value for a named member of the structure. The named
  member shall be at the top level of the structure; a member with the same name in some level of
  substructure shall not be set. The value shall be castable to the member type and is evaluated in the
  context of an assignment to the named member; otherwise, an error is generated.
-  The type:value specifies an explicit value for each field in the structure whose type matches the
  type (see 6.22.1) and has not been set by a field name key above. If the same type key is mentioned
  more than once, the last value is used. The value is evaluated in the context of an assignment to the
  matching type.
- The default:value applies to members that are not matched by either member name or type key.
  If the member type is a simple bit vector type, matches the self-determined type of the value, or is
  not an array or structure type, then the value is evaluated in the context of each assignment to a
  member by the default and shall be castable to the member type; otherwise, an error is generated.
  For unmatched structure members, the type and default specifiers are applied recursively according
  to the rules in this subclause to each member of the substructure. For unmatched array members, the
  type and default keys are applied to the array according to the rules for arrays.
  
### "%m" Hierarchical name format
The %m format specifier does not accept an argument. Instead, it causes the display task to print the hierarchical name of the design element, subroutine, **named block**, or labeled statement that invokes the **system task** containing the format specifier. This is useful when there are many instances of the module that calls the system task. One obvious application is timing check messages in a flip-flop or latch module; the %m format specifier pinpoints the module instance responsible for generating the timing check message. 

Note: regarding the usage in the unnamed block. If there is only one unnamed block, when using "%m", it only print the hierarchy into the module.  But if there is more than one unnamed block, when using the "%m" in both the block. there will be one hierarchy with *uname* 
```verilog 
module test(); 
initial begin 
	$dispaly("block one: %m"); //It will print the "block one: test"
end

initial begin
	$dispaly("block two: %m"); //It may prin the "block two: test.$unamed"
end
endmodule 
```