### Associativity [](https://booleanpy.readthedocs.io/en/latest/concepts.html#associativity)
- *x*&(*y*&*z*)=(*x*&*y*)&*z*
-
- *x*|(*y*|*z*)=(*x*|*y*)|*z*
-
### Commutativity [](https://booleanpy.readthedocs.io/en/latest/concepts.html#commutativity)
- *x*&*y*=*y*&*x*
-
- *x*|*y*=*y*|*x*
-
### Distributivity [](https://booleanpy.readthedocs.io/en/latest/concepts.html#distributivity)
- *x*&(*y*|*z*)=*x*&*y*|*x*&*z*
-
- *x*|*y*&*z*=(*x*|*y*)&(*x*|*z*)
-
### Identity [](https://booleanpy.readthedocs.io/en/latest/concepts.html#identity)
- *x*&1=*x*
-
- *x*|0=*x*
-
### Annihilator [](https://booleanpy.readthedocs.io/en/latest/concepts.html#annihilator)
- *x*&0=0
-
- *x*|1=1
-
### Idempotence [](https://booleanpy.readthedocs.io/en/latest/concepts.html#idempotence)
- *x*&*x*=*x*
-
- *x*|*x*=*x*
-
### Absorption [](https://booleanpy.readthedocs.io/en/latest/concepts.html#absorption)
- *x*&(*x*|*y*)=*x*
-
- *x*|(*x*&*y*)=*x*
-
### Negative absorption [](https://booleanpy.readthedocs.io/en/latest/concepts.html#negative-absorption)
- *x*&((∼*x*)|*y*)=*x*&*y*
-
- *x*|(∼*x*)&*y*=*x*|*y*
-
### Complementation [](https://booleanpy.readthedocs.io/en/latest/concepts.html#complementation)
- *x*&(∼*x*)=0
-
- *x*|(∼*x*)=1
-
### Double negation [](https://booleanpy.readthedocs.io/en/latest/concepts.html#double-negation)
- ∼(∼*x*)=*x*
-
### De Morgan [](https://booleanpy.readthedocs.io/en/latest/concepts.html#de-morgan)
- ∼(*x*&*y*)=(∼*x*)|(∼*y*)
-
- ∼(*x*|*y*)=(∼*x*)&(∼*y*)
-
### Elimination [](https://booleanpy.readthedocs.io/en/latest/concepts.html#elimination)
- *x*&*y*|*x*&(∼*y*)=*x*
-
- (*x*|*y*)&(*x*|(∼*y*))=*x*
- TODO
-