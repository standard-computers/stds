# Standard

A Standard is a database table construct that sets the Standard for how data represents an object. When creating a Standard, save the content with the file extension `.stds`. Each Standard requires its own file, and the Standard Name must be the same as the Standard file name.

_Starter example_

`vehicle.stds`
```
vehicle: VHL {
    private vin string:36! VN *
    protected make string MK *
    protected model string MDL *
}
```

A Standard should always start with the `standard_name: STDREF {...` . Standards will typically be referenced by the Standard reference. Standard’s _columns_, _variables_, or _properties_ are called constraints.

- `Standard_name` - Can only include lowercase letters and underscores.
- `STDREF` - Can only be uppercase letters, number (Not starting with) and underscores.

## Constraints

```
[ACCESS_TYPE] CONST_NAME [CONST_TYPE [:LENGTH[!]]] [NULL] [CONST_REF] ["DEFAULT_VALUE" | "/REGEX/"] [*]
```

- `ACCESS_TYPE` - Constraint access type (scope)
    - `private` - Only accessible in constructor, or custom getters and setters.
    - `protected` - Can be publicly read but only written to in the constructor and getters & setters.
    - `public` - public – Allows manipulation from any point or user. Public has native getters & setters.
    - `global` - Can be read from anywhere but only manipulated from within package

- `VAR_NAME` - Name of constraint. Only lowercase letters, numbers, or underscores. 
- `VAR_TYPE` - Standard variable types: `string`, `bool`, `int`, `double`, `char`, `array`, `Standard`
- `VAR_LENGTH` - Number of characters accepted as integer.
  -  The optional char `!` means the constraint value provided is equal to the `VAR_LENGTH` provided. The record will be rejected if length doesn't match.
- `VAR_REF` - Constraint reference allows you to shorthand Standards and constraints. If Standard ‘school’ is `SCH` and has a constraint ‘street’ as `SADDR`, in many cases we can access this as SCH.ADDR.
- `"DEFAULT_VALUE"` - Default value. If a value is not provided for this constraint, the value listed will be used. If you provide no value, the value saved will be `NULL`

## Extending Standards

Like other coding languages for objects, you can extend a Standard to duplicate a table for separate use. A classic example is vehicle applications and types.

```
truck: TRCK: @VHL
```

We can also extend by appending constraints.

```
truck: TRCK: @VHL {
    protected owner Standard @PER OWNR *
    protected double max_weight MXWGHT
}
```

In this example, `TRCK` will have the same constraints as `VHL` in addition to the `max_weight` constraint.

## Standards as Constraints

In other databases or languages, you are able to join properties or perform aggregations. We can do this in Standard by creating a constraint of type `standard`.

```
truck: TRCK: @VHL {
    protected owner Standard @PER OWNR *
    protected double max_weight MXWGHT
}
```

Now when accessing the `owner` constraint, you can access it like `@USR` or `OWNR`. When this record is saved, the constraint value is the record id of the parent Standard record.

```
#Get Trucks where vehicle model is 'Tesla', limit 1
foundTruck [TRCK] <VHL.MK "Tesla" 1>
print foundTruck OWNR FNAME #Prints truck owner's first name
print foundTruck owner firstname #Also prints truck owner's first name
print foundTruck USR firstname #Again
```

If your Standard references the same Standard in more than one constraint, Standard will select the first constraint that uses that Standard.

```
truck: TRCK: @VHL {
    protected owner Standard @PER OWNR *
    protected driver Standard @PER DRVR *
}

#Ini empty object
truck @TRCK

print truck PER firstname #Will only ever print the owner's firstname
print truck driver firstname #Now we print driver's firstname
```

## Arrays

Arrays are more dynamic in Standard. If you want an array of Standards as a constraint, simply use `array` type instead of Standard and specify the Standard as normal.

```
event: EVNT {
    protected invites array @PER INVTS
}

evt @EVNT

print evt INVTS.0 #Print first invite
```

## Definitions

Definitions in Standard serve as enumerations. You can combine definitions with a Standard if that definition should belong.

```
vehicle_type: VHLTP {
  def sedan "SEDAN"
  def suv "SUV"
}
```

## Querying

When querying Standards, you will almost always use the Standard Reference.
Queries performed with `(..)` require exact Standard or record match. Queries using `<...>` format are for querying Standards by constraints or definitions.

### Adding Records

```
[VHL] + ("JKBVNKD167A013982", "Ford", "Explorer")
```

### Deleting Records

**Using exact record match**
```
[VHL] - ("JKBVNKD167A013982", "Ford", "Explorer")
```

**Using query match**
```
[VHL] - <vin "JKBVNKD167A013982">
```

**Using query with limit**
```
[VHL] - <make "Ford", LIMIT 10>
```

### Finding Records

**Unlimited find**
```
[VHL] <vin "JKBVNKD167A013982">
```

**Limited Find**
```
[VHL] <make "Ford", LIMIT 10>
```

### Misc Operations

**Count records for Standard**
```
count [VHL]
```

**Listing Standards**
```
stds #Lists Standards
stds std_name #Shows one Standard
stds std_name json #Shows Standard as json
```