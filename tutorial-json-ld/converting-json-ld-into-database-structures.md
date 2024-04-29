# Converting JSON-LD into database structures

If the JSON-LD schema is written correctly, you can create a system to interpret the schema and create a table map of the schema. To prove this theory, we’ve created an open **Filemaker** addon that you can freely add to _any_ Filemaker database to do just that.

Let’s go through the process. We will start with a very simple schema, _Currency_, as documented at [https://grcschema.org/Currency](https://grcschema.org/Currency).

## Recording the structure

Even though obvious, the first place you’ll need to start is gathering the schema data itself. Websites such as Schema.org and GRCSchema.org have well-defined JSON-LD schemas. Below is the _Currency_ schema from GRCSchema.org.

![Currency schema](<../.gitbook/assets/0 (2).png>)

Normally, websites such as these have various options of looking at the schema, such as this one that presents **schema**, **structure**, and **visualized**. You can skip any visualization, and the structure view provides more information than you’ll need. So we’ll go with the sample schema, as shown here:

```
{
 "@context": "http://grcschema.org/",
 "@type": "Currency",
 "CoreMetaData": {
 "@type": "CoreMetaData",
 "date_created": "Date",
 "date_modified": "Date",
 "created_by": "Integer",
 "modified_by": "Integer",
 "live_status": "Boolean",
 "checksum": "Integer",
 "validated": "Boolean"
 },
 "CurrencyCodes": {
 "@type": "CurrencyCodes",
 "@set": [
 {
 "@type": "CurrencyCode",
 "id": "Integer",
 "currency_code": "String",
 "currency_fk": "Integer"
 }
 ]
 },
 "CurrencyCountries": {
 "@type": "CurrencyCountries",
 "@set": [
 {
 "@type": "CurrencyCountry",
 "name": "String",
 "id": "Integer",
 "country_fk": "Integer",
 "currency_fk": "Integer"
 }
 ]
 },
 "CurrencyNames": {
 "@type": "CurrencyNames",
 "@set": [
 {
 "@type": "CurrencyName",
 "id": "Integer",
 "currency_name": "String",
 "currency_fk": "Integer"
 }
 ]
 },
 "CurrencySymbols": {
 "@type": "CurrencySymbols",
 "@set": [
 {
 "@type": "CurrencySymbol",
 "id": "Integer",
 "currency_symbol": "Char",
 "currency_fk": "Integer"
 }
 ]
 },
 "id": "Integer"
}
```

## Creating the tables and fields to hold the data

If you are going to write an interpreter, you’ll want a minimum of three fields:

1. Schema – this will hold the schema you are going to work with.
2. Non-nested schema – this will hold a translation of the schema into a simple JSON array.
3. DB tables – this will hold a list of all of the database tables you’ll need to create.

## The methodology

We explain the methodology for each step below. The methodology uses a custom _While_ function to step through each line of the schema that repeats logic while the condition is true, then returns the result. The format of the _While_ function is shown below:

While ( \[ initialVariable ] ; condition ; \[ logic ] ; result )

The parameters for the While function are:

* **initialVariable** - variable definitions that will be available to use in the following parameters.
* **condition** - a Boolean expression evaluated before each loop iteration. While True, the loop repeats. When False, the loop stops.
* **logic** - variable definitions that are evaluated each time the loop is repeated.
* **result** - an expression that is returned when the loop stops.

### The initial variables

Below we will show all the initial variable setups we are using and explain each.

| **Initial Value**                                                  | **Explanation**                                                                                   |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| \~json = json;                                                     | This is just taking the json passed in and putting in into a local variable for readability.      |
| \~path = path ;                                                    | Take the json path string and put it into a local variable for readability.                       |
| \~extendedPath = Case ( IsEmpty ( \~path ) ; "" ; \~path & "." ) ; | Add a dot to the string for passing into next pass of recursion, unless string was not passed in. |

Now we need a few loop control variables.

| **Initial Value**                                                                        | **Explanation**                                                              |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| \~keys = JSONListKeys(\~json ; "");                                                      | This calls a standard function to get our list of JSON keys from the object. |
| \~keyCount = Case(Left(\~keys ; 1) = "?" or IsEmpty (\~keys) ; 0 ; ValueCount (\~keys)); | Get the count of the keys for looping.                                       |
| \~n = 1 ;                                                                                | This is the loop control variable.                                           |

Now we need to add a few more to initialize variables that are reset during each loop iteration.

| Initial Value                                                                                                                                                                                                                                                                                                                                                            | Explanation                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| \~thisKey = "";                                                                                                                                                                                                                                                                                                                                                          | The current key we are on.                                                                               |
| \~thisValue = "";                                                                                                                                                                                                                                                                                                                                                        | The current value associated with the current key.                                                       |
| \~isTable = "";                                                                                                                                                                                                                                                                                                                                                          | Boolean indicating whether the current key is a table name.                                              |
| \~isSet = "";                                                                                                                                                                                                                                                                                                                                                            | Boolean indicating whether the current value is a @set array.                                            |
| \~isArrayElement = ";                                                                                                                                                                                                                                                                                                                                                    | Boolean indicating whether the current value is an element of an indexed array.                          |
| \~isJSON = "";                                                                                                                                                                                                                                                                                                                                                           | Boolean indicating whether the current value is a JSON object.                                           |
| \~result = ""                                                                                                                                                                                                                                                                                                                                                            | The result which will eventually be passed out of the function.                                          |
| \~fields = "" ;                                                                                                                                                                                                                                                                                                                                                          | The fields in the current table.                                                                         |
| \~extendedPath = Case ( IsEmpty ( \~path ) ; "" ; \~path & "." ) ;                                                                                                                                                                                                                                                                                                       | The full path of the table, including parent tables, separated by periods “.”.                           |
| \~pathList = Substitute ( \~path ; "." ; ¶ ) ;                                                                                                                                                                                                                                                                                                                           | A list of all of the paths.                                                                              |
| \~lastInPath = GetValue ( \~pathList ; ValueCount ( \~pathList ) ) ;                                                                                                                                                                                                                                                                                                     | The very last line of the path list.                                                                     |
| <p>~thisTable = Case (</p><p>IsEmpty ( ~path ) and Left ( JSONGetElement ( ~json ; "@set" ) ; 1 ) ≠ "?" ; JSONGetElement ( ~json ; "@type" );</p><p>not IsEmpty ( JSONGetElement ( ~json ; "@set" ) ) and Left ( JSONGetElement ( ~json ; "@set" ) ; 1 ) ≠ "?" ; JSONGetElement ( ~json ; "@type" ) ;</p><p>GetValue ( ~pathList ; ValueCount ( ~pathList ) )<br>) ;</p> | The current table being examined during this function’s run.                                             |
| <p>~parentTable = Case (</p><p>IsEmpty ( ~lastInPath ) ; "" ;</p><p>~lastInPath = ~thisTable and ValueCount ( ~pathList ) > 1 ; GetValue ( ~pathList ; ValueCount ( ~pathList ) - 1 ) ;</p><p>~lastInPath = ~thisTable ; "" ;</p><p>~lastInPath</p><p>) ;</p>                                                                                                            | The current table’s parent.                                                                              |
| <p>~thisPath = Case (</p><p>~parentTable = ~thisTable; ~path ;</p><p>~path = ~thisTable ; ~path ;</p><p>~lastInPath = ~thisTable ; ~path ;</p><p>~extendedPath &#x26; ~thisTable</p><p>) ;</p>                                                                                                                                                                           | The current table’s path.                                                                                |
| <p>~baseJSON = JSONSetElement ( "" ;</p><p>[ "table" ; ~thisPath ; JSONString ]</p><p>)</p>                                                                                                                                                                                                                                                                              | The heading of each of the resulting arrays. This could also include parent table and other information. |

### The condition logic

The condition logic for this is pretty simple. Keep looping through the json text until you get to the last line.

\~n ≤ \~keyCount ;

### The logic used for interpretation

The logic used for the interpretation is based off of what we find in each row of the json being passed. Each of the main things to draw from the JSON are described below.

#### Handling of the loop iterations

| Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Explanation                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| <p>~thisKey = GetValue ( ~keys ; ~n ) ;</p><p>~thisValue = JSONGetElement ( ~json ; ~thisKey ) ;</p><p>~isTable = Left ( JSONGetElement ( ~thisValue ; "@set" ) ; 1 ) ≠ "?" and not IsEmpty ( JSONGetElement ( ~thisValue ; "@set" ) ) ;</p><p>~isSet = ~thisKey = "@set" ;</p><p>~isArrayElement = IsEmpty (</p><p>Substitute ( ~thisKey ;</p><p>[ 0 ; "" ] ;</p><p>[ 1 ; "" ] ;</p><p>[ 2 ; "" ] ;</p><p>[ 3 ; "" ] ;</p><p>[ 4 ; "" ] ;</p><p>[ 5 ; "" ] ;</p><p>[ 6 ; "" ] ;</p><p>[ 7 ; "" ] ;</p><p>[ 8 ; "" ] ;</p><p>[ 9 ; "" ]</p><p>)</p><p>) ;</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | These values are here for handling the various aspects of the loop iteration.                                   |
| \~isJSON = Left ( JSONFormatElements ( \~thisValue ) ; 1 ) ≠ "?" and not IsEmpty ( \~thisValue ) ;                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | If there is nothing but numerical digits, assume we’ve got a number.                                            |
| <p>~nextIteration = Case (</p><p>~isJSON ; ConvertToNonNestedJSON ( ~thisPath ; ~thisValue ) ;</p><p>//else</p><p>""</p><p>) ;</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | If we are going to have a next iteration, go ahead and get it.                                                  |
| <p>~nextIterationFields = Case (</p><p>~isJSON; ConvertToNonNestedJSON ( ~thisPath ; ~thisValue ) ;</p><p>""</p><p>) ;</p><p>~fields = Case (</p><p>not IsEmpty ( ~nextIterationFields ) ;</p><p>Let ( [</p><p>~error1 = Left (JSONFormatElements ( ~fields ) ; 1 ) ≠ "[" and not IsEmpty ( ~fields ) ;</p><p>~error2 = Left (JSONFormatElements ( ~nextIterationFields ) ; 1 ) ≠ "[" and not IsEmpty ( ~nextIterationFields );</p><p>~combinedArrays = Left ( ~fields ; Length ( ~fields ) - 1 ) &#x26; "," &#x26; Right ( ~nextIterationFields ; Length ( ~nextIterationFields ) - 1 )</p><p>];</p><p>Case (</p><p>~error1 or ~error2 ; "error" ;</p><p>IsEmpty ( ~fields ) ; ~nextIterationFields ;</p><p>IsEmpty ( ~nextIterationFields ) ; ~fields ;</p><p>~combinedarrays</p><p>)</p><p>) ;</p><p>Let ( [</p><p>#theArray = Case (</p><p>IsEmpty ( ~fields )</p><p>or</p><p>~fields = "{}" ; "[]" ;</p><p>~fields</p><p>) ;</p><p>#index = Case ( IsEmpty ( ~fields ) ; 0 ; ValueCount ( JSONListKeys ( ~fields ; "" ) ) ) ;</p><p>#error = Case ( Left ( #index ; 1 ) = "?" ; "Invalid Array" ) ;</p><p>#thisField = JSONSetElement ( ~baseJSON ;</p><p>[ "field" ; ~thisKey ; JSONString ] ;</p><p>[ "type" ; ~thisValue ; JSONString ]</p><p>)</p><p>];</p><p>Case (</p><p>~thisKey = "@type" or ~thisKey = "@context" ; ~fields ;</p><p>JSONSetElement ( #theArray ; #index ; #thisField ; JSONObject )</p><p>)</p><p>)</p><p>);</p> | Get any fields passed back by the next iteration. The result is passed back as json with lines and fields keys. |

#### Interpreting @type as a table name

The second element of the JSON-LD schema should be the @type keyword. The @type keyword as the second element in the schema represents the primary table name that should be created. It is linked to the schema by a project ID.

Therefore, there is no special code necessary to determine the primary table name.

For every field in the primary table, this information will be stored in the field **parent\_table**.

#### Interpreting "@set": \[ at the top level as an array

If, at the primary level an @set keyword is found, that denotes what follows will be returned as an array of information with each of the keys denoting the individual fields within the array.

While _Currency_ doesn’t have this pattern, the JSON for returning the stub-list of _all_ _Currency_ records does (see [https://grcschema.org/Currencies](https://grcschema.org/Currencies)).

{

"@context": "http://grcschema.org/",

"@type": "Currencies",

"@set": \[

{

"country\_fk": "Integer",

"id": "Integer",

"name": "String"

}

]

}

The _properties_ are then used to create the field names (**field\_name**) in our Fields table. The **field\_type** is derived from the element’s _type_.

Your system should then go through each of the element’s keys and create records in your **Fields** table, as shown below:

| **parent\_table** | **sub\_table** | **field\_name** | **field\_type** |
| ----------------- | -------------- | --------------- | --------------- |
| Currencies        |                | country\_fk     | Integer         |
| Currencies        |                | id              | id              |
| Currencies        |                | name            | String          |

#### Interpreting "propertynamegoeshere" : { as an object

Within our example of _Currency_, you can see that the 4th line of the schema text is:

"CoreMetaData" : {

This pattern represents a grouped _object_. In many databases (ours included) the fields of the grouped object are treated as any other field and the is recorded _as an object_ but not turned in to a field in-and-of itself.

In layouts, however, grouped objects are represented as a _field set_, as shown below:

![Object as a field set](../.gitbook/assets/1.png)

#### Interpreting "propertynamegoeshere": { "@set": \[ as a subtable

In the example below we can see that within _Currency_, right after _CoreMetaData_ we have a subtable of _CurrencyCodes_ that follow this pattern.

"CurrencyCodes": {

"@set": \[

{

"id": "Integer",

"currency\_code": "String",

"currency\_fk": "Integer"

}

]

},

That’s our cue that each currency can be assigned multiple currency codes. And hence, a subtable should be created. Another hint, by the way, is that the array shows both an **id** property and a **currency\_fk** property so that the two tables can be linked together.

In filling out our interpretation database, CurrencyCodes becomes subtable and also tells us that each of the fields below it belong to _it_.

| **parent\_table** | **sub\_table** | **field\_name** | **field\_type** |
| ----------------- | -------------- | --------------- | --------------- |
| Currency          | CurrencyCodes  | id              | Integer         |
| Currency          | CurrencyCodes  | currency\_code  | String          |
| Currency          | CurrencyCodes  | currency\_fk    | Integer         |

This allows us to create a table structure that links the Currency table to its subtable CurrencyCodes, as shown below:

![CurrencyCodes subtable](<../.gitbook/assets/2 (2).png>)

### The result is a simple JSON array of fields, tables, and field types

What results is an array of each of the fields with its table, object identifier (optional), and the field type, as shown below:

```
{
 "field" : "checksum",
 "object" : "CoreMetaData",
 "table" : "Currency",
 "type" : "Integer"
},
```

Once you have converted the JSON-LD into this simple array, you can easily create scripts to re-intrepret the array into SQL table commands, like the one shown below:

```
CREATE TABLE "Currency" (
"checksum" int,
"created_by" int,
"date_created" date,
"date_modified" date,
"live_status" int,
"modified_by" int,
"validated" int,
"id" int
)
```

## Sample files

We have some sample files to help you through this.

ConvertToNonNested.txt – this is the custom function described above.

SchemaToTableConverter.fmp12 – this is a Filemaker database that is completely open and takes you through each of the steps we’ve described above.
