# Meeting 1/22/2019

## Julio

Composing elements - reference by ID

```json
{
    "body": [
        {
            "$ref": "InventoryHeader"
        }
    ]
}
```

That references another card. Extracts elements from the body (or actions).

Alternatively could go with the type style, like `"type": "Person"`.


## Matt

Multiple views of templates/custom elements

Data loaders






## Binding to boolean and integer properties

If a binding expression evaluates to a boolean, and the binding expression consumes the entire property value, then that binding expression will become a boolean

**Data**
```json
{
    "shouldWrap": true
}
```

**Card**
```json
{
    "type": "TextBlock",
    "wrap": "{data.shouldWrap}"
}
```

**Final**
```json
{
    "type": "TextBlock",
    "wrap": true
}
```

But note if that binding value originally was actually a string, this wouldn't become a boolean

**Data**
```json
{
    "shouldWrap": "true"
}
```

**Card**
```json
{
    "type": "TextBlock",
    "wrap": "{data.shouldWrap}"
}
```

**Final**
```json
{
    "type": "TextBlock",
    "wrap": "true" // Invalid, not a boolean
}
```

DECISION: Do it! Try to word it better



## String interpolation

Sometimes, you want to inject data into a string. Since this is supported with our functions, it should also be supported by our data binding.

**Card**
```json
{
    "type": "TextBlock",
    "text": "Hello, {data.person.name}!"
}
```

DECISION: Do it!


## Combining data binding and functions

What if you have a date that you want to be displayed? Well, that DATE function of ours would be super handy! But you also want to use data binding for obtaining the date...

**Data**
```json
{
    "created": "2017-02-14T06:08:39Z"
}
```

**Card**
```json
{
    "type": "TextBlock",
    "text": "Created {{DATE({created},SHORT)}}",
    "text": "Created {created.toShortDate()}",
    "text": "Created {date(created, 'short')}  {title.toInt()} {parseInt(title)} {loadUrl('https://msn.com')}    {'https://msn.com'.load()}"
}
```

DECISION: 



## Complex JSON property names

JSON allows a property name to be literally any string. The following is valid (a property name with quotes, the quotes are escaped for serialization purposes)

**JSON**
```json
{"The \"meaning\" of life":42}
```

**Property name**
```
The "meaning" of life
```

The following is also valid JSON

**JSON**
```json
{
    "this.is.super.weird": true
}
```

Therefore, developers need a way of expressing the property names that work with these crazy formats.

In our binding language, if we interpret the `.` as an accessor to a child property, that wouldn't work with the above example.

One quick and easy solution is realizing that every JSON object is a dictionary. If our binding language supports accessing dictionaries via string keys, it can access anything!

**Card**
```json
{
    "type": "TextBlock",
    "text": "{data['this.is.super.weird']}"
}
```


## Operations and functions

We currently have one format of functions, like the date function...

```json
{
    "type": "TextBlock",
    "text": "Created {{DATE(2017-02-14T06:08:39Z,SHORT)}}"
}
```

We could use this for things like equal, substring, etc...

```json
{
    "type": "TextBlock",
    "wrap": "{{EQUAL({data.type},'Executive')}}"
}
```

However, that format is quite verbose. While it might be easier for us to implement, it's worse for authors. The following is much cleaner to author...

```json
{
    "type": "TextBlock",
    "wrap": "{data.type == 'Executive'}"
}
```

### Operators needed

#### Comparisons

* `==`
* `!=`
* `>`
* `>=`
* `<`
* `<=`

#### Math

* `+`
* `-`
* `*`
* `/`
* `%` (mod)

#### Grouping

* `()`

DECISION: Need operators





### Functions needed

We could use the existing function format for things like substring...

```json
{
    "type": "TextBlock",
    "text": "{{SUBSTR({data.day},3)}}" // "Fri"
}
```

Or we could implement something cleaner...

```json
{
    "type": "TextBlock",
    "text": "{data.day.substr(3)}",
    "text": "{substr(data.day, 3)}"
}
```

However, we'll still need static functions for things like averaging an array of numbers...

```json
{
    "type": "TextBlock",
    "text": "{avg(data.highs)}" // "Fri"
}
```

```json
{
    "day": "5",
    "dayNum": 5
}
```

```json
{
    "type": "TextBlock",
    "text": "{data.day.substr(3)}",
    "text": "{substr(data.day, 3)}",
    "text": "{Math.avg(data.highs)}", // "Fri"
    "text": "{ms-username()}",
    "text": "{str.substr(str.toString(data.dayNum), 1)}",
    "text": "{str.substr(56, 1)} {math.ceiling('5')}",
    "text": "{data.day == 3}",
    "text": "{toString(data.dayNum)}"
}
```

* Comparison operators: Explicit vs implicit
    * Gilles: Explicit
    * David: Unknown
    * Matt: Unknown
    * Andrew: Explicit

DECISION: For now go with explicit, but we should do more research since we're basing off our own personal opinions

* Function types:
    * OPTION 1: Anything can convert to a string, but not vice versa
    * OPTION 2: Explicit everywhere
    * OPTION 3: Anything can convert to anything

DECISION: Explicit for now, and support prefixes



DECISION: Need static functions. Leave instance functions for later, nice-to-have.

#### String functions

* `str.substr(string txt, int length)`
* `str.substr(string txt, int start, int length)`
* `str.len(string txt)` (returns length of string)
* `str.toUpper(string txt)`
* `str.toLower(string txt)`
* `str.toString(var variable)`

#### Math functions

* Average `math.avg(`
* Round `math.round(`


* parseJson
* http methods


## Handling fallback of functions

What if a function isn't implemented? How does card author provide fallback?


## Custom functions

Hosts can add custom functions