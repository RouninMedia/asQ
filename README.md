# asQ
**asQ** is a *state-management* approach intended to be used by frameworkless javascript-based **Single Page Apps**.

**asQ** stores `appState` comprehensively within a single **JSON**-based `queryString` parameter.

Storing State in aSPs
=================
URL queryString:

?appstate=[
  {
    identifier: ".myList" < for querySelectorAll
    classList: {"set": [], "remove": []},
    customData: {"set": {}, "remove": []},
    attributes: {"set": {}, "remove": []},
    events: {"set": {}, "remove": {}},
    activateEvents: ["", "", ""]
  },
  {
    ...
  }
]


1) ^^^ The above initially included an entry:

invokeFunctions: ["", "", ""]

But this isn't really necessary, since any function that can be invoked by name, can be invoked inside a native event or a custom event, which can be called by activateEvents.

2) It's crucial to remember that any element state description *always* describes a transformation from initialState, so... at window.onload the initialState needs to be captured and added to each element as a custom data-attribute:
 
data-initial-state="{
    classList: {},
    customData: {},
    attributes: {},
    events: {},
    activateEvents: ["", "", ""]
  }"

3) Any change to Model:

   i) Gets appState Object from queryString
  ii) Updates appState Object
 iii) Proofreads and edits appState Object
 iv) Applies updated appState Obj to DOM
 v) Updates URL using pushState

4) Any page loaded with ?appState:

  i) Gets appState Object from queryString
 ii) Applies appState Object to DOM

5) Does this mean that a single element may undergo multiple transformations as appState is processed?

YES. And that would matter if One change happened at the start:

font-weight: bold;
background-color: blue;

and Another change happened 0.45s later:

background-color: red;

So... what needs to happen for the background-color to switch straight to red?

ie. How does Step iii) work?

Something like:
  iii.i) Adds data-appstate-update-index="[]" attribute to each element
  iii.ii) If data-appstate-update-index attribute already exists, add new index
  iii.iii) Go through document and get rid of all data-appstate-update-index attributes with a single index
 iii.iv) Create new appState Object like this:

data-appstate-update-index='["2", "6"]'
data-appstate-update-index='["3", "4"]'
data-appstate-update-index='["2", "5"]'
data-appstate-update-index='["4", "5"]'

- for each element:
   i) add :not([data-appstate-update-index="['2', '6']"]) to each identifier
  ii) add new entry at the end of appState Object: [data-appstate-update-index="['2', '6']"]
