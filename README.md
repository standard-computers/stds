# Standard

A standard is a database table construct that sets the standard for how data represents an object. When creating a standard, save the content with the file extension `.stds`. Each standard requires its own file, and the standard name must be the same as the file name.

_Starter example_

`vehicle.stds`
```
vehicle: VHL {
    private vin string:36! VN *
    protected make string MK *
    protected model string MDL *
}
```

This standard represents a vehicle, which is the standard name, and can be referenced by `VHL`. The vehicle has a `vin`, `make`, `model` and they represent values you might expect. VIN is a string and **must** be 36 characters long. Make and model must be strings. These variables, called constraints, can be referenced by their shorthand which are the succeeding uppercase letters.

- `[!] standard_name` - Can only include lowercase letters and underscores. Precede the standard name with an optional `!` to enforce record history keeping for this standard. 
- `STDREF` - Can only include uppercase letters, numbers (not starting with) and underscores.

A standard should always start with the `standard_name: STDREF {...` . What are frequently known as _columns_, _variables_, or _properties_, are the constraints of the object the standard represents.

## Constraints

**Format of constraints**
```
ACCESS_TYPE CONST_NAME [CONST_TYPE [:LENGTH[!]]] [NULL || STD_REF] [CONST_REF] ["DEFAULT_VALUE" | "/REGEX/"] [*]
```

- `ACCESS_TYPE` - Constraint access type (scope)
    - `private` - Only accessible in constructor, or custom getters and setters.
    - `protected` - Can be publicly read but only written to in the constructor and getters & setters.
    - `public` - public – Allows manipulation from any point or user. Public constraints have native getters and setters.
    - `global` - Can be read from anywhere but only manipulated from within package

- `CONST_NAME` - Name of constraint. Only lowercase letters, numbers, or underscores. 
- `CONST_TYPE` - Standard variable types: `string`, `bool`, `int`, `double`, `array`, `standard`. If the type is `standard`, follow the constraint name with `@STDREF` to set the standard taht the constraint should reference.
- `LENGTH` - Max number of characters accepted as integer.
  -  The optional char `!` means the constraint value provided must be equal to the `LENGTH` provided. The record will be rejected if length doesn't match.
- `CONST_REF` - Constraint reference allows you to shorthand constraints. If standard ‘school’ is `SCH` and has a constraint ‘street’ as `SADDR`, in many cases we can access this as SCH.ADDR.
- `"DEFAULT_VALUE"` - Default value wrapped in quotes. If a value is not provided for this constraint, the default value is `NULL`.

## Extending Standards

Like other coding languages for objects, you can extend a standard to duplicate a standard/table for separate use. Let's extend the vehicle standard.

```
truck: TRCK: @VHL
```

The standard `truck` now has the constraints of `vehicle`. Extend the standard further by appending constraints.

```
truck: TRCK: @VHL {
    protected owner string OWNR *
    protected max_weight double MXWGHT
}
```

In this example, `truck` will have the same constraints as `VHL` in addition to the `owner` and `max_weight` constraint.

## Standards as Constraints

In other databases or languages, you are able to join properties or perform aggregations. To do this in Standard, create a constraint of type `standard` followed by `@STDREF`. What if a truck has an owner that is a person?

**Person standard**
```
person: PER {
    protected firstname string FNAME *
    protected middlename string MNAME "UKNOWN" *
    protected lastname string LNAME *
    protected birthday string BDAY *
    private address string ADDR
    private location string LOC
}
```

```
truck: TRCK: @VHL {
    protected owner standard @PER OWNR *
    protected max_weight double MXWGHT
}
```

Now when accessing the `owner` constraint, you can access it like `owner` or `OWNR`. You can also access the constraints of the standard the parent constraint references. When a record is saved for the `truck` standard, the `owner` constraint value is the id of the record the constraint is referencing.

```
#Get Trucks where vehicle model is 'Tesla'
foundTruck [TRCK] <VHL.MK "Tesla">

print foundTruck OWNR FNAME #Prints truck owner's first name
print foundTruck owner firstname #Also prints truck owner's first name
print foundTruck USR firstname #Again
```

If your standard references the same standard in more than one constraint, Standard will select the first constraint that uses that standard.

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

Arrays are more dynamic in Standard. If you want an array of standards as a constraint, simply use `array` type instead of Standard and specify the standard as normal. All record ids passed during creation of a record for this standard must only reference records of the standard referenced in the constraint.

```
event: EVNT {
    protected invites array @PER INVTS
}
```

Access the elements of the array by prepending the constraint selector in your query with `.INDEX` where `INDEX` is an integer of the index of the desired element, starting from 0 of coarse. 

```
evet @EVNT

print evet INVTS.0 #Print first invite
```

## Definitions

Definitions in Standard serve as enumerations. You can combine definitions with a standard if that is a property of the object being represented and must be a certain value. Standard definitions are saved by themselves like standards and the header is the same as standards. The content of the definition are separate lines of definition values.

**Definition value format**
```
def DEF_VAL_NAME DEF_VAL_VAL
```

The definition value name is to reference the value. The value provided for the definition value is the actual value that is transferred to the record. Let's make a vehicle type for the vehicle standard.
```
vehicle_type: VTP {
  def sedan "SEDAN"
  def suv "SUV"
}
```

You can implement a definition by doing the following...
```
vehicle: VHL {
    private vin string:36! VN *
    protected make string MK *
    protected model string MDL *
    protected type standard @VTP TP
}
```

Below is the Standard Code implementation (`.std`). As you'll see in this standard, a record will be created and using the definition value. If a value is provided that does not exist in the definition referenced in the standard, the record will be rejected. The second record created will import the value from the definition where the definition value name is `sedan`.
```

#Using standard def value, with value check
[VHL] + ("UVOTVGHYKLPCONLSZ", "Toyota", "Camry", "SEDAN")

#Or using standard def name, pulls in that value
[VHL] + ("UVOTVGHYKLPCONLSZ", "Toyota", "Camry", sedan)
```

## Querying

As eluded, standards are referenced in Standard Code or queries as `[ standard_name || STDREF ]`. The value in brackets is case-sensitive. Any additional action indicated with arguments in your query will follow this standard selector.

### Creating Records

Create a record for the vehicle standard by succeeding the standard selector with a `+ (CONST_VAL_1, [...])`. This method of creating with parenthesis is also the constructor for a standard record.
```
[VHL] + ("JKBVNKD167A013982", "Ford", "Explorer")
```

### Deleting Records

Using exact record match
```
[VHL] - ("JKBVNKD167A013982", "Ford", "Explorer")
```

Using query match
```
[VHL] - <vin "JKBVNKD167A013982">
```

Using a query with limit
```
[VHL] - <make "Ford", LIMIT 10>
```

### Finding Records

Unlimited find
```
[VHL] <vin "JKBVNKD167A013982">
```

Limited Find
```
[VHL] <make "Ford", LIMIT 10>
```
### Exporting Records

You can export records very easily. You can choose from JSON, csv, and XML.
```
[VHL] >> "path/to/file/vehicles.csv"
```

### Importing Records

You can import records very easily. Choose from JSON, csv, and XML. The format matches the export format and a standard value check is performed. Values must match the types set by the standards and constraint types.

__Note the opposite direction of arrows__
```
[VHL] << "path/to/file/vehicles.csv"
```

### Misc Operations

Count records for standard
```
count [VHL]
```

Count records for standard with match
```
count [VHL] <make "Ford">
```

Listing Standards
```
stds #Lists Standards
stds std_name #Shows one Standard
stds std_name json #Shows Standard as json
```

Purge records for standard (delete all records)
```
purge [VHL]
```

## In-memory tables

In-memory tables provide redis level functionality that replicates and builds off of standards. In any query, prepend the standard table selector with `$`.

Here's an example :)

```
#This adds to the table on disk
[VHL] + ("JKBVNKD167A013982", "Ford", "Explorer")

#This initializes the table with the standard in-memory if not already,
#and adds the record. Does not effect table on disk.
#When the program exits, this data is deleted

$[VHL] + ("JKBVNKD167A013982", "Ford", "Explorer")

#Perform the below to load on disk into memory, add record in-memory
load $[VHL]
$[VHL] + ("JKBVNKD167A013982", "Ford", "Explorer")

#Perform changes in-memory, to on disk
sync $[VHL]
```

## Static Values

The standard system provides a quick way to fetch values that developers often code. You can use `$now` which gets the UNIX timestamp. You can use `$epoch` to get the time in milliseconds since epoch. There are many values you can fetch.

- $now - UNIX Timestamp
- $date - Today's date as YYYY-mm-dd
- $time - Current system time
- $timestamp - Seconds from EPOCH
- $epoch - Milliseconds from EPOCH
- $tomorrow - UNIX Timestamp for Tomorrow
- $year - Current Year
- $month - Current Month
- $day - Current Day
- $weekday - Name of Current Day
- $timezone - System Timezone
- $user - Current System Username
- $os - System Operating System Name
- $cwd - Current Working Directory
- $pid - Standard System Process ID
- $home - System Home Directory
- $uuid - Generate UUIDv4