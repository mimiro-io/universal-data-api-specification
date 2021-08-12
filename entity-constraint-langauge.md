# Entity Constraint Language

The Entity Constraint Language is a vocabulary for expressing constraints over instances of the Entity Data Model [1]. As well as the vocabulary there are validation semantics connected with each constraint class. 

# Bootstrapping

To bootstrap the constraint classes and the evaluation semantics the following base formalisms are defined.

## The Type Relationship

An Entity can have relationships to other entities. These relationships are typed. To indicate that one entity is an instance of a class the `rdf:type` relationship MUST be used. 

The following example shows an entity connected to its EntityType via the `rdf:type`reference type.

```json
{
    "id" : "http://example.org/people/bob",
    "refs" : {
        "rdf:type" : "http://example.org/schema/Person"
    }        
}
```

## Entity Class

To indicate that an entity is an Entity Class then it MUST have an `rdf:type` reference whose value is `uda:EntityClass` .

```json
{
    "id" : "http://example.org/schema/Person",
    "refs" : {
        "rdf:type" : "uda:EntityClass"
    }        
}
```

## SubClass Relationship

To convey that an EntityClass is a subclass of another entity class the relationship type `uda:subClassOf` must be used.

The following example shows how to express that the `Person` class is a subclass of the `Thing` class.

```json
{
    "id" : "http://example.org/schema/Thing",
    "refs" : {
        "rdf:type" : "uda:EntityClass"
    }        
}
,
{
    "id" : "http://example.org/schema/Person",
    "refs" : {
        "rdf:type" : "uda:EntityClass",
        "uda:subClassOf" : "http://example.org/schema/Thing"
    }        
}
```

## SubClass Relationship Entailment

The `subClassOf` relationship described above is transitive in nature. Such that if:

```
a rdf:type B 

amd

B uda:subClassOf C

it follows (or is entailed) that:

a rdf:type C is also true.

```

This means that constraints defined as applying to any super class also apply to any instance of subclasses.

# Validation Semantics

Validation of a entity model instance is formally defined as:

```
Given:

i := entity model instance
c := constraint model instance
v := validation function
r := validation result

such that

v(i,c) => r
```

Such that given a constraint model instance and an entity model instance applying the validation rules associated with each constraint type yields a validation result. The validation result reports violations in the entity model instance.   

## Validation Semantics Helper Functions

To support expressing validation semantics a small number of functions with clear semantics are introduced. 

### Hop

The `hop` function is used to traverse the entity graph. It takes the URI of a starting entity, the URI of the reference type to follow and an optional parameter to indicate if the traversal should be inverse. The result of the `hop` function returns a list of Entities.

### Properties

The properties function takes an entity and the URI of a property type and returns an array of values corresponding to the values of that property residing on the entity.

The following example would return the list of values assigned to the `schema:name` property of the entity `e`. If the entity only has a single value for the specified property then that value is returned as the only value in the array.

If there are no properties that match the property type provided then the function returns an array of length 0.

```javascript
let namepropvalues = properties(e, "schema:name")
```

### Length

The `len` function takes an array as a parameter and returns the number of elements in the array.

```javascript
let arraylength = len(property_value_array)
```

### Property DataType

The property data type function returns the data type from a property value. The values returned are as defined by the URIs for XML built in primitive datatypes (https://www.w3.org/TR/2004/REC-xmlschema-2-20041028/).

```javascript
let datatype = property_datatype(value)
```

Based on XML primitive datatypes the result will be a URI of one of the following:

- http://www.w3.org/2001/XMLSchema#int
- http://www.w3.org/2001/XMLSchema#duration
- http://www.w3.org/2001/XMLSchema#dateTime
- http://www.w3.org/2001/XMLSchema#time
- http://www.w3.org/2001/XMLSchema#date
- http://www.w3.org/2001/XMLSchema#boolean
- http://www.w3.org/2001/XMLSchema#base64Binary
- http://www.w3.org/2001/XMLSchema#hexBinary
- http://www.w3.org/2001/XMLSchema#float
- http://www.w3.org/2001/XMLSchema#double
- http://www.w3.org/2001/XMLSchema#decimal
- http://www.w3.org/2001/XMLSchema#anyURI
- http://www.w3.org/2001/XMLSchema#string
- http://www.w3.org/2001/XMLSchema#gYearMonth
- http://www.w3.org/2001/XMLSchema#gYear
- http://www.w3.org/2001/XMLSchema#gMonthDay
- http://www.w3.org/2001/XMLSchema#gDay
- http://www.w3.org/2001/XMLSchema#gMonth

# Constraint Data Model and Validation Semantics

Each constraint is defined in terms of a data model and an associated validation rule. The validation rule is generic and parameterised. The parameters to the validation rule are retreived from the instance of the constraint data model at run time. Note that the formalism for expressing the constraint is to convey unambiguously the intent of the constraint and not as an implementation specification. 

# Constraint Classes

## IsAbstract Constraint Class

Instances of the `IsAbstract` constraint class are used to indicate that the specified `EntityClass` MUST not have any instanes.

The data model for this constraint defines:
- the entity class that must have no instances.

The entity model representation of this constraint is as follows and indicates placeholders that are used in the formal semantics when validating.

```json
{
    "id" : "schema:constraint-0",
    "refs" : {
        "rdf:type" : "constraint:IsAbstractConstraint",
        "constraint:applies-to-class" : "$A"
    }
}

```

The formal semantics for this constraint class are defined in terms of the evaluation functions and bind the variable from the constraint instance as a parameter.  

``` javascript
// get the instances of the Entity Class indentified in the constraint instance
let instancesOfClass = hop($A, "rdf:type", true)
let instanceCount = len(instancesOfClass)

assert_true(instanceCount, 0)
```

## Property Constaint

Instances of the `Property Constraint` class are used to ensure that an instance of an Entity Class contains the correct number of the specified properties. e.g. that a person must have a name and a date of birth.

The data model for this constraint defines:
- the entity class whose instances must meet the constraint
- the property class being constrained
- the min cardinality of properties of the property class that are allowed on an instance
- the max cardinality of properties of the property class that are allowed on an instance

```json
{
    "id" : "schema:constraint-1",
    "refs" : {
        "rdf:type" : "constraint:PropertyConstraint",
        "constraint:applies-to-class" : "$CLASS",
        "constraint:property-class" : "$PROPERTY_CLASS"
    },
    "props" : {
        "constraint:minCard" : "$MIN_CARD",
        "constraint:maxCard" : "$MAX_CARD"       
    }
}
```

The formal semantics for this constraint class are defined in terms of the evaluation functions and bind the variable from the constraint instance as parameters.  

``` javascript

// get the instances of the Entity Class indentified in the constraint instance
let instancesOfClass = hop($CLASS, "rdf:type", true)

// for each instance of the specified class get the properties of the specified property class and assert the cardinailty meets the specified values. 
for entityInstance in instancesOfClass {
    let propertiesOfSpecifiedType = properties(entityInstance, $PROPERTY_CLASS)
    let propertiesLength = len(propertiesLength)

    assert_true(propertiesLength >= $MIN_CARD)
    assert_true(propertiesLength <= $MAX_CARD)
}
```

## Property Value Type Constraint

Instances of the `Property Value TYpe Constraint` class are used to ensure that an the value of a given property class on a given entity class must be of a given type. e.g. date of birth on person must be of type DateTime.

The data model for this constraint defines:
- the entity class whose instances must meet the constraint
- the property class being constrained
- the required data type of value of the constrained property

```json
{
    "id" : "schema:constraint-1",
    "refs" : {
        "rdf:type" : "constraint:PropertyValueConstraint",
        "constraint:applies-to-class" : "$CLASS",
        "constraint:property-class" : "$PROPERTY_CLASS",
        "constraint:datatype" : "$DATA_TYPE"
    }
}
```

The formal semantics for this constraint class are defined in terms of the evaluation functions and bind the variable from the constraint instance as parameters.  

``` javascript

// get the instances of the Entity Class indentified in the constraint instance
let instancesOfClass = hop($CLASS, "rdf:type", true)

// for each instance of the specified class get the properties of the specified property class and assert all values are of the correct type.
for entityInstance in instancesOfClass {
    let propertiesOfSpecifiedType = properties(entityInstance, $PROPERTY_CLASS)
    let propertiesLength = len(propertiesOfSpecifiedType)

    for value in propertiesOfSpecifiedType {
        let dt = property_datatype(value)
        assert_equal(dt, $DATA_TYPE)
    }
}
```

TODO: add violation data structures as a result of failed results.

## Reference Constraint

Instances of the `Reference Constraint` class are used to ensure that an instance of an Entity Class contains the correct number of the specified references of a given type. e.g. that a person must have a reference of type `mother` and  constraint the type of the related entity to be `Person`.

The data model for this constraint defines:
- the entity class whose instances must meet the constraint
- the reference class being constrained
- the required class of the referenced
- the min cardinality of properties of the property class that are allowed on an instance
- the max cardinality of properties of the property class that are allowed on an instance

```json
{
    "id" : "schema:constraint-1",
    "refs" : {
        "rdf:type" : "constraint:PropertyConstraint",
        "constraint:applies-to-class" : "$CLASS",
        "constraint:reference-class" : "$REFERENCE_CLASS",
        "constraint:referenced-entity-class" : "$REFERENCED_ENTITY_CLASS"
    },
    "props" : {
        "constraint:minCard" : "$MIN_CARD",
        "constraint:maxCard" : "$MAX_CARD"       
    }
}
```

The formal semantics for this constraint class are defined in terms of the evaluation functions and bind the variable from the constraint instance as parameters.  

``` javascript

// get the instances of the Entity Class indentified in the constraint instance
let instancesOfClass = hop($CLASS, "rdf:type", true)

// for each instance of the specified class get the properties of the specified property class and assert the cardinailty meets the specified values. 
for entityInstance in instancesOfClass {
    let references = references(entityInstance, $PROPERTY_CLASS)
    let referencesLength = len(references)

    assert_true(propertiesLength >= $MIN_CARD)
    assert_true(propertiesLength <= $MAX_CARD)

    // check associated entity has type reference to correct class
    for refvalue in references {
        // get the related entity
        relatedentity = get_entity(refvalue)

        // get all the classes for the related entity (this is a list of entities)
        classes = get_classes(relatedentity)

        // get the constraining class that all referenced entities must be an instance of 
        referenced_class = get_entity($REFERENCED_ENTITY_CLASS)

        // check if the list of classes contains the required constraint class
        let isofclass = contains(classes, referenced_class)

        assert(isofclass)
    }
}
```

## Reference Restriction Constraint

The reference restriction constraint is used to restrict allowed the allowed classes of related entities. e.g. an org unit can be part of another org unit, but a restriction could say that a group can only be part of a department, and a department can only be part of a company, and a company can only be part of a conglomorate. This allows the reuse of the reference type but refines the constraints on subclass. 

To do: do we need this?

```javascript
```

## One Of Reference Constraint

The one of reference constraint constrains the allowed set of entities that can be referenced by instances of a given class with a given property class. e.g. It is used as mechanism to restrict the set of possible instances to a subset of the complete range of entities of a given class.


## Query Constraint

The entity query constraint takes a EQL (Entity Query Language) expression, evaluates it and assume that any entities returned are in violation of the constraint. 


## Application Constraint

An application constraint is an application specific non-standaised constraint that is executed by the validation engine and returns instances of type constraint viloation.


# Examples

This section is informative and illustrates how to use the constraint classes for common data modelling problems.

# Compliance

TODO: Take a set of small tests cases with input in terms of the model instance and the constraint model instance and define the expected outputs in terms of violation data structures.

# References

1. https://open.mimiro.io/specifications/uda/latest.html


