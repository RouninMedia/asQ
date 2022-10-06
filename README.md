# asQ (and... asH)
**asQ** is a *state-management* approach intended to be used by frameworkless javascript-based **Single Page Apps**.

**asQ** stores `appState` comprehensively within a single **JSON**-based `queryString` parameter.

______

## Storing State in aSPs

URL queryString:

```
?appstate=[
  {
    identifier: ".myList",
    id: {"set": "id", "remove": "id"},
    classList: {"set": ["class", "class"], "remove": ["class"]},
    customData: {"set": [{"key": "value"}, {"key": "value""}], "remove": ["key"]},
    attributes: {"set": [{"key": "value""}, {"key": "value""}], "remove": ["key"]},
    events: {"set": {["event", "functionName", boolean], ["event", "functionName", boolean]}, "remove": {["event", "functionName", boolean]}},
    activateEvents: ["event", "event", "event"]
  },
  {
    ...
  }
]
```


1) The object initially included a eighth entry:

```
invokeFunctions: ["functionName", "functionName", "functionName"]
```

But this isn't really necessary, since any function that can be invoked by name, can be invoked inside a native event or a custom event, which can be called by activateEvents.

2) The `identifier` value is a CSS selector and is run through a `[... document.querySelectorAll()];` statement to grab all the target elements as an array.

3) It's crucial to remember that any element state description *always* describes only the transformation from `initialState`, so... at `window.onload` the `initialState` **needs** to be captured and added to each element as a `custom-data attribute`:

```
data-initial-state="{
    id: "id",
    classList: ["class", "class", "class"],
    customData: [{"key": "value"}, {"key": "value"}],
    attributes: [{"key": "value"}, {"key": "value"}],
    events: {["event", "function", boolean], ["event", "function", boolean]},
    activateEvents: ["event", "event", "event"]
}"
```

3.5) What about if there are CSS Transitions? Nobody wants to see the transition from `initialState` endlessly repeating itself each time. So... how can this be prevented? Should there be both an `data-initial-state` and a `data-current-state`? This will need thinking about.

What about `MutationObservers`? Do they have any utility here?

4) Any change to Model:

  4.1 Gets `appState Object` from `queryString`
 
   
  4.2 Updates `appState Object`
 
 
  4.3 Proofreads and edits `appState Object`
 
 
  4.4 Applies updated `appState Object` to `DOM`
 
 
  4.5 Updates URL using `history.pushState`
  

5) Any page loaded with `?appState`:

  5.1 Gets `appState Object` from `queryString`
  
  5.2 Applies `appState Object` to `DOM`

6) Does this mean that a single element may undergo multiple transformations as the `appState Object` is processed?

YES. And that would matter if *one change* happened at the start:

`font-weight: bold;`

`background-color: blue;`

and *another change* happened 0.45s later:

`background-color: red;`

So... what needs to happen for the `background-color` to switch straight to `red`?

ie. How does Step iii) work?

Something like:

  6.1 Adds `data-appstate-update-index="[]"` attribute to each element

  6.2 If `data-appstate-update-index` attribute already exists, add new index

  6.3 Go through document and get rid of all `data-appstate-update-index` attributes with a single index
  
  6.4 Create new appState Object like this:

```
data-appstate-update-index='["2", "6"]'
data-appstate-update-index='["3", "4"]'
data-appstate-update-index='["2", "5"]'
data-appstate-update-index='["4", "5"]'
```

- for each element:
  
 i. add `:not([data-appstate-update-index="['2', '6']"])` to each identifier
  
 ii. add new entry at the end of appState Object: `[data-appstate-update-index="['2', '6']"]`


______

## Evolution of the Idea

### Part I

In early April 2021, I was working on the **Ashiva MultiPage Editor** (the foundations of which were built on the server-side PHP script from **Kubaru** and the front-end CSS layout from the **LanguageCompass**).

At a fairly early stage (even though this wasn't apparent to me, initially) I realised that I was actually using Javascript, HTML and CSS to write *views* for the **Ashiva MultiPage Editor** (e.g. a *view* for when the app had just started up, another *view* for when the user was at the text-inputting stage, a third *view* for when the user was at the page-searching stage etc.). Further, in order to communicate to both the CSS and the JS which *view* should be visible, I was already recording some of the application-state (e.g. `formAction`, `replaceActivated`) in custom-data attributes (`data-*`) at the top of the form in the `<form>` element. Once I'd realised that this was what I was doing, it occured to me that it *ought* to be possible to record *more* of the application-state (e.g. `pagesFound`) - perhaps all of it? - in the same way.

It then struck me that this HTML custom-data attribute based approach meant that instead of adding and removing elements to the DOM via javascript, a surprisingly large amount of the view-state could be handled by the **CSS** `display` property alone. (Not an approach I imagined would be particularly widespread in the JS-first SPA Dev Community.)

Being able to store all of the application-state *actually within the markup* reminded me of a suggestion I came across when I was learning `WebComponents` in Sept-Oct 2020 - that a dedicated `WebComponent` might be developed, specifically to store all application-state variables. But, as I thought about how to do this, I realised, there wasn't really anything a dedicated `WebComponent` could do that a standard element with custom-data attributes couldn't do already.

However, it also occurred to me that recording the application-state *anywhere* within the markup (either in a dedicated `WebComponent` or in custom-data attributes) meant that the application-state couldn't be shared - it would only be accessible from the SPA, while the SPA was running.

I briefly considered `localStorage` as an alternative to custom-data attributes... but that was only one step better: it *would* mean the application state could be shared between browser-tabs, but still not between browsers.

So the answer, of course, was to take advantage of the `queryString`. There was some precedent for this: in an early experiment on the **Da3shBoard** I had added a `queryString` which included state variables. But then I'd never really used them, so they were largely superfluous.

This time however, adding the following `queryString` to **Ashiva MultiPage Editor** meant that the ***entire application state*** was effectively stored in the URL:

    ?formAction=input-text&findPhrase=12345&caseSensitive=true&replaceActivated=true&replacePhrase=abcde&pagesFound=true
    
This `queryString` was effectively the equivalent of a (theoretical, but soon to be real) `appState object` like this:

```
{
  formAction: 'input-text',
  findPhrase: '12345',
  caseSensitive: 'true',
  replaceActivated: 'true',
  replacePhrase: 'abcde',
  pagesFound: 'true'
}
```

I realised that every time a user-interaction changed the state of the application I could easily keep track of everything by:

 i. updating the values in the *helper* `appState Object`
 ii. taking the updated values in the `appState Object` and invoking `history.pushState()` to update the `queryString`
 iii. replicating the updated values in the `queryString` in `custom-data` attributes in the `<form>` element like this:

```
<form class="multiPageEditorForm"
data-form-action="input-text"
data-find-phrase="12345"
data-case-sensitive="true"
data-replace-activated="true"
data-replace-phrase="abcde"
data-pages-found="true">
```

#### In Summary:
 
 > User Action => Updates `appState Object` => Translated to `queryString` (**SSOT**) => Replicated as `custom-data` attributes

____

### Part II

Using the `queryString` as the *Single Source of Truth* (*SSOT*) which maintains the entire application-state is only *half* of **asQ**.

The more difficult part is thinking how to make C-based application-state-management:

 1. scalable so that it would fit an `SPA` (or `aSP`) of any size or complexity
 2. easily applicable to any `SPA` (or `aSP`)
  
This is where **asQ** comes into its own:

 - the `state` described in the `queryString` for any given `view` only needs to declare the differences in any element from that element's own `data-initial-state`
 
______

## Further ideas...

## asH

**asH** (*appState:hashFragment*) is the ideal twin to **asQ** (*appState:queryString*).

**asH** is essentially identical to **asQ**, but utilises a `hashFragment` instead of a `queryString`.

**ie.**

The `hashFragment` on this URL:

`https://example.com/path/to/page/#["This","is",{"a":"test"}]`

can be processed via:

`JSON.parse(decodeURIComponent(location.hash.replace('#', '')))`

to give:

```
[
  "This",
  "is",
  {
    "a": "test"
  }
]
```


