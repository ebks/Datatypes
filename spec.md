SPEC ADT Booolean

* sorts:
- Boolean

* operations:
- true: --> Boolean
- false: --> Boolean
- not: Boolean --> Boolean
- and: Boolean x Boolean --> Boolean
- or: Boolean x Boolean --> Boolean

* axioms:
- (Axiom B1): not(not(b))=b
- (Axiom B2): not(or(b1,b2))=not(and(not(b1),not(b2)))

for all b1,b2: Boolean

END SPEC


