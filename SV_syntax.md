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

  