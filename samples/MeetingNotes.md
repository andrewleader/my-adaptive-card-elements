# Data binding meeting 1/16/2019

* Sharad - Lifeline team, how to dynamically represent changing data on our backend, considering Adaptive Cards for this. Need mechanism for rendering the data and dynamically updating the data

* David - First-class designer experience, but if we build the binding into the format, we might not be able to ship that separately from the renderer. It'd be great if we can apply template before we hand anything to the actual renderer. Define a set of constructs that can be applied to any JSON payload and interpreted by a templating library... something like STJS (generic) but not use syntax that interferes with the schema of the original JSON object

```json
{
    // If source is array, repeat element as many times in array, otherwise just bind once
    "$source": "{{data.people}}",
    "title": "{{data.name}}",
    ...
}
```

versus... which only repeats the items, not the container itself

```json
{
    "type": "Container",
    "itemsSource": "{{data.people}}",
    "items": [

    ]
}
```


# Overall feedback

* David - One thing we should avoid is introducing constructs that break the format of the original data object. Implicit repeat behavior rather than explicit repeat. Given that we have `{{` for functions, a field binding would be better to have `${` or `{$` to indicate that what follows is binding. For functions, we should make it possible for hosts to introduce their own functions - extensible functions. Concerned about binding to host properties since we wouldn't be able to apply templating server-side (we would need the templating engine on the host which would raise the size of the library). Maybe go with a simple templating syntax that runs on host. Prefers always prefixing data binding with something that indicates what it's bound to (like `data.title`). Likes `when`. Thinks we should build a generic solution (not tied to Adaptive Cards). Likes Matt's transform idea. Performance should be somethignw e think about

    * Bill - Extensible functions might be a way for achieving host information
        * David - We'd still need all that functioning interpreting logic on the host in order for that to work


* Matt - Likes `$data`, `$host` to indicate binding to a specific thing. Likes implicit repeating behavior. Likes either much more than itemtemplate and itemsource. Likes `when` more than `visible`. Next topics should be template selecting and components, but also designer support. Seeing JSON alongside designer would be very valuable. Should define our exit criteria.

* Bill - Torn on `$` thing, sometimes it's a pain to type. How do we iterate over dictionaries rather than arrays, like below, especially if we're doing implicit?

```json
{
    "passengers": {
        "0": {
            "name": "Andrew"
        },
        "1": {
            "name": "Matt"
        }
    }
}
```

* Bill - Really likes functions, and also extending functions from hosts. Likes Matt's transform idea, it's kind of related to scoping data, maybe make sure there's one single way to refer to and extend data scopes. Definitely think it's important to have prototypes to see how all of these proposals work... if we have extendable functions, having those within the prototypes would be awesome to show how it works. ParseJson would be useful function.

* Julio - Keeping renderer clean and focused on small set of elements makes sense. Renderer is assembler, templating is compiler. Important to maintain our tooling (designer) compatibility. Tending to prefer shorthand syntax, it can get verbose - tooling helps with that, but shorter is more elegant since we'll do it frequently. For functions, want to make sure we're clear on where we draw the line, probably don't want to replicate all of JavaScript. Including the raw data object on the card is important for machine learning information from the card. Wants to decide components and data binding separately since these go hand-in-hand. Should have components before we go public. Should put our prototypes to test with concrete scenarios. As part of exit criteria, someone from dev div should consult with us.


# Data property brackets

```json
{
    "type": "TextBlock",
    "text": "Hello {data.name}"
},
{
    "type": "TextBlock",
    "text": "Hello {{data.name}}, it's {{DATE(2017, SHORT)}}"
},
{
    "type": "TextBlock",
    "text": "Hello ${$data.name}"
}
```

* David - Fine with either of the first two, prefers first for easier to type.
* Bill - Functions will be used less frequently than data binding. Single brackets are preferred since easy to type.
* Andrew - Prefer shorter
* Julio - Prefer shorter
* Matt - Prefer double, but fine with single. Concerns with single are consistency with our functions (and also STJS, mustache) and escape syntax.

DECISION: Short syntax

# Accessing data properties

```json
{
    "type": "TextBlock",
    "text": "Hello {data.name}, {host.name}, {root.title}"
},
{
    "type": "TextBlock",
    "text": "Hello {$data.name}, {$root.title}"
},
{
    "type": "TextBlock",
    "text": "Hello {$data.name} or {name}, {$data['complicated property']}, {$root.title}"
}
```

* David - Likes `$`, implies reserved scope, votes for bottom
* Bill - Having to type `$` seems like a pain, votes for bottom
* Matt - Votes for bottom
* 

DECISION: Bottom format


# Reserved objects

```json
{
    "$data": "The current data",
    "$root": "The very top root",
    "$index": "The current index when iterating. Otherwise, 0? -1?",
    "$host.config": "",
    "$host.runtime or $host.ambient or $env": "",
    "$elements['myId']": "Access another element in the card"
}
```

# Scoping data

**Data**

```json
{
    "departingFlight": {
        "name": "Seattle"
    }
}
```

```json
{
    "type": "Container",
    "$data": {
        "name": "Andrew",
        "flight": "{$root.departingFlight}"
    },
    "items": [
        {
            "type": "TextBlock",
            "text": "{name}"
        }
    ]
},
{
    "type": "Container",
    "$data": "{{GET('https://azure.com/subscriptions/{id}')}}",
    "items": [
        {
            "type": "TextBlock",
            "text": "{name}"
        }
    ]
},
{
    "type": "Container",
    "$data": "{departingFlight}",
    "items": [
        {
            "type": "TextBlock",
            "text": "{name}"
        }
    ]
},
{
    "type": "Container",
    "$datasource": "{departingFlight}",
    "items": [
        {
            "type": "TextBlock",
            "text": "{name}"
        }
    ]
},
{
    "type": "Container",
    "$datascope": "{departingFlight}",
    "items": [
        {
            "type": "TextBlock",
            "text": "{name}"
        }
    ]
},
```

CONSENSUS: `$data`, and inherits automatically if not specified


# Repeating items

```json
{
    "type": "AdaptiveCard",
    "$data": {
        "firstPerson": "{people[0]}",
        "secondPerson": "{people[1]}"
    },
    "body": [

    ]
}
```

```json
{
    "type": "TextBlock",
    "$data": "{people}",
    "text": "{name}"
},
{
    "type": "TextBlock",
    "$repeat": "{people}",
    "text": "{name}"
}
```

* David - Implicit
* Julio - Implicit
* Matt - Implicit
* Bill - Implicit
* Andrew - Explicit

DECISION: Implicit


# When

```json
{
    "type": "TextBlock",
    "$when": "{name == 'Andrew'}" // When false, entire object dropped
}
```

DECISION: Do it!