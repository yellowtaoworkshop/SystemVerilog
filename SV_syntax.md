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
array[idx_D][idx_C][idx_A][idx_B]
```

See the snapshot below from the book *SystemVerilog for Verification* 

![packed array](..\snapshots\packed array.png)