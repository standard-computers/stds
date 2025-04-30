# Standard

A standard is a database table construct that sets the standard for how data represents an object. When creating a standard, save the content with the file extension `.stds`. Each standard requires its own file and the standard name must be the same as the standard file name.

_Here’s starter example_

```
vehicle: VHL {
    protected id string:8! ID *
    private vin string:36! VN *
    protected make string MK *
    protected model string MDL *
}
```

A standard should always start with the `standard_name: STDREF {...}` . Standards will usually be referenced by the standard reference. Standard’s _columns_, _variables_, or _properties_ are called constraints.

- `standard_name` - Can only include lowercase letters and underscores.
- `STDREF` - Can only be uppercase letters, number (Not starting with) and underscores.

## Constraints

```
[ACCESS_TYPE] CONST_NAME [CONST_TYPE [:LENGTH[!]]] [NULL] [CONST_REF] ["DEFAULT_VALUE" | "/REGEX/"] [*]
```

- `ACCESS_TYPE` - Constraint access type (scope)
    - `private` - Only accessible in constructor, or custom getters & setters.
    - `protected` - Can be publicly read but only written to in the constructor and getters & setters.
    - `public` - public – Allows manipulation from any point or user. Public has native getters & setters.
    - `global` - Can be read from anywhere but only manipulated from within package

- `VAR_NAME` - Name of constraint. Can only be lowercase letters, numbers, or underscores. 
- `VAR_TYPE` - Standard variable types: `string`, `bool`, `int`, `double`, `char`, `array`, `standard`
- `VAR_LENGTH` - Number of characters accepted as integer.
  -  The optional char `!` means the constraint value provided is equal to the `VAR_LENGTH` provided. The record will be rejected if length doesn't match.
- `VAR_REF` - Constraint references allow you to shorthand standards and constraints. If standard ‘school’ is `SCH` and has a constraint ‘street’ as `SADDR`, in many cases we can access this as SCH.ADDR.
- `"DEFAULT_VALUE"` - Default value. If a value is not provided for this constraint, the value listed will be used. If you provide no value, the value saved will be `NULL`

## Extending Standards

Like other coding languages for objects, you can extend a standard to effectively duplicate a table for separate use. A classic example is vehicle applications and types.

```
truck: TRCK: @VHL
```

Using this format lets you duplicate a standard for another object. We can also extend by appending constraints.

```
truck: TRCK: @VHL {
    protected owner standard @PER OWNR *
    protected double max_weight MXWGHT
}
```

In this example, `TRCK` will have the same constraints as `VHL` in addition to the `max_weight` constraint.

## Standards as Constraints

In other databases or languages, you are able to join properties or perform aggregations. Setting a constraint as a standard is the standard solution and here’s how you could do that.

```
truck: TRCK: @VHL {
    protected owner standard @PER OWNR *
    protected double max_weight MXWGHT
}
```

Now when accessing the `owner` constraint, you can access it like `@USR` or `OWNR`.

```
#Get Trucks where vehicle model is 'Tesla', limit 1
foundTruck [TRCK] <VHL.MK "Tesla" 1>
print foundTruck OWNR FNAME #Prints truck owner's first name
print foundTruck owner firstname #Also prints truck owner's first name
print foundTruck USR firstname #Again
```

If your standard references the same standard in more than one constraint, standard will select the first constraint that uses that standard.

```
truck: TRCK: @VHL {
    protected owner standard @PER OWNR *
    protected driver standard @PER DRVR *
}

#Ini empty object
truck @TRCK

print truck PER firstname #Will only ever print the owner's firstname
print truck driver firstname #Now we print driver's firstname
```

## Arrays

Arrays are more dynamic in standard. If you want an array of standards as a constraint, simply use `array` type instead of standard and specify the standard as normal.

```
event: EVNT {
    protected invites array @PER INVTS
}

evt @EVNT

print evt INVTS.0 #Print first invite
```