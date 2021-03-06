# Ionic Views

An application for the Fitbit Ionic can quickly become a mess. This micro-framework provides basic support for View and subview patterns in about 100 LOCs, providing the necessary structure to keep the view layer code manageable when your application grows.

## Installation

Copy `view.js` file to your project.

## API

### DOM selectors

#### `function` $( selector )

Global jQuery-style `$` selector to access SVG DOM elements returning raw elements. No wrapping is performed, the raw element or elements array is returned. Just two simple selectors are supported:

- `$( '#id-of-an-element' )` - will call `document.getElementById( 'id-of-an-element' )`
- `$( '.class-name' )` - will call `document.getElementsByClassName( 'class-name' )`

When called without arguments, returns the `document`.

```javascript
import { $ } from './view'

// Will search for #element globally
$( '#element' ).style.display = 'inline';
```

#### `function` $at( id-selector )

Create the $-function to search in the given DOM subtree.
Used to enforce DOM elements isolation for different views.

When called without arguments, returns the root element.

```javascript
import { $at } from './view'

const $ = $at( '#myscreen' );

// Make #myscreen visible
$().style.display = 'inline';

// Will search descendants of #myscreen only
$( '#element' ).style.display = 'inline';
```

#### `function` $wrap( element )

Create the $-function to search in the given DOM subtree wrapping the given element. Internally,

```javascript
const $ = $wrap( document );
const $at = selector => $wrap( $( selector ) );
```

### `pattern` Elements Group

To group SVG elements just wrap them in the class as shown below.
This pattern allows caching of the references to SVG elements and must be preferred
to ad-hoc elements lookups.

Elements group is the lightweight alternative to subviews, and should be preferred to subviews when possible.

```javascript
class Time {
  minutes = $( '#minutes' );
  seconds = $( '#seconds' );
  
  render( seconds ){
    this.minutes.text = Math.floor( seconds / 60 );
    this.seconds.text = ( seconds % 60 ).toFixed( 2 );
  }
}

// Use an element group from the View...
class Timer extends View {
  time = new Time();
  ...
  
  render(){
    ...
    this.time.render( this.seconds )
  }
}
```

### `class` View 

View is the stateful group of elements. The difference from the elements group is that views can me contained in each other and they have `onMount`/`onUnmount` lifecycle hooks. API:

- `view.el` - optional root view element. Used to show and hide the view when its mounted and unmounted.
- `view.mount()` - make the `subview.el` visible, call the `subview.onMount()` hook.
- `view.onMount()` - place to insert subviews and register events listeners.
- `view.render()` - render the view and all of its subviews.
- `view.unmount()` - hide the `subview.el`, unmount all the subviews, call the `view.onUnmount()` hook.
- `view.onUnmount()` - place to unregister events listeners.
- `view.insert( subview )` - insert and mount the subview.
- `view.remove( subview )` - remove the unmount the subview.

Example:

```javascript
import { $at } from './view'
const $ = $at( '#timer' );

class Timer extends View {
  el = $();
  
  onMount(){
    clock.granularity = "seconds";
    clock.ontick = this.onTick;
  }
  
  ticks = 0;
  onTick = () => {
    this.ticks++;
    this.render();
  }
  
  minutes = $( '#minutes' );
  seconds = $( '#seconds' );
  render(){
    const { ticks } = this;
    this.minutes.text = Math.floor( ticks / 60 );
    this.seconds.text = ( ticks % 60 ).toFixed( 2 );
  }
  
  onUnmount(){
    clock.granularity = "off";
    clock.ontick = null;
  }
}
```

### `class` Application

Application is the main view having the single `screen` subview.
It's the singleton which is globally accessible through the `Application.instance` variable.

- `MyApp.start()` - instantiate and mount the application.
- `Application.instance` - access an application instance.
- `Application.switchTo( 'screen' )` - switch to the screen which is the member of an application.
- `app.screen` - property used to retrieve and set current screen view.
- `app.render()` - render all the subviews, _if display is on_. Is called automaticaly when display goes on.

```javascript
class MyApp extends Application {
  screen1 = new Screen1View();
  screen2 = new Screen2View();
  
  onMount(){
    this.screen = new LoadingView();
  }
}

MyApp.start();

...
// To switch the screen, use:
Application.switchTo( 'screen2' );
```

## Project structure

Application may consist of several screens. The following project structure is recommended for this case:

- `app/` <- standard app folder
  - `index.js` <- `Application` subclass class, which will switch the screens
  - `view.js` <- copy this file to your project
  - `screen1.js` <- each screen is defined as `View` subclass
  - `screen2/` <- if there are many modules related to the single screen, group them to the folder
  - ...
- `resources/` <- standard resources folder
  - `index.gui` <- include screens SVG files with `<link rel="import" href="screen/index.gui" />`
  - `widgets.gui` <- include screens CSS files with `<link rel="stylesheet" href="screen/styles.css" />`
  - `screen1.gui` <- put SVG for your screens to different files
  - `screen2/` <- group SVG, CSS, and images used by a screen to a folder
