Utilities
=========
This is a set of classes designed for use in Windows 8 WWAs using the WinJS frameworks.

There are a number of parts to this:

## Shared Utilities ##
These are some common utility functions that allow you to write better code across the board.

### appassert, alert ###
Because WWA's don't have an `alert` function, and instead have `Windows.UI.Popup.MessageBox`, there is a convenient replacement that looks like your mothers alert, but is async (since sync message boxes aren't possible in WWAs). Just call `alert` and all will be good.

Additionally, theres a basic assert through `appassert`, (named so as not to conflict with QUnit's assert class). This is like like a normal `assert`:
`appassert(condition, message)`

If condition is not truthy, the previously mentioned `alert` is used to show a message to the consumer. If a debugger is attached, then the debugger will immediately break at this point too.

### Signal ###
WinJS introduces a powerful async parttern through Promises. However, sometimes, creating promises to perform & share promise patterns for your own async code is hard:

    var complete, error, progress;    
    var promise = new WinJS.Promise(function(c, e, p) {    
        complete = c; error = e; progress = progress;    
    });

And thats before you start sharing them around and raising the completion etc at the correct times.

Using Codevoid.Utilities.Signal, this is a lot easier:

    var signal = new Codevoid.Utilities.Signal();
    signal.complete();
    return signal.promise;

### Codevoid.Utilities.derive ###
This is a a helper for derive a class, and having access to the original base class easily.

Example:

        var control = WinJS.Class.define(function (argA) {
            // Base constructor!
        });

        var derived = Codevoid.Utilities.derive(control, function (argA, argB) {
            this.base(argA);
            // Derived constructor!
        });

### Codevoid.Utilities.property ###
When mixing the `WinJS.Utilities.eventMixin`, enables a simple `INotifyPropertyChanged` type contract in JS. When the event changes, an event is raised with `propertyNameChanged`, where `propertyName` is the name of the property, with the old & new value available of the `detail` property.

Example:

    var object = WinJS.Class.mix(WinJS.Class.define(function () {
    }, {
        sample: Codevoid.Utilities.property("sample", null),
    }), WinJS.Utilities.eventMixin);
    
    var instance = new object();
        
    var valueChanged = false;
    instance.addEventListener("sampleChanged", function () {
        valueChanged = true;
    });

    instance.sample = Date.now();
    
### Codevoid.Utilities.addEventListeners ###
Enables better management of DOM-style event handlers for adding & removing with much simpler patters. When called, returns an object with a `cancel` method that will remove all the event handlers added.

Example:

    var cancel = Codevoid.Utilities.addEventListeners(source, {
       custom: function () {
            eventWasRaised = true;
        },
        custom2: function () {
            event2WasRaised = true;
        },
    });
    
    cancel.cancel(); // Cleaned up!
    
## UI Control Helpers ##

There are a nubmer of core challenges with WinJS, and WWAs when trying to make a maintainable, well built application:

* Avoid lots of code-driven layout.
* Enable easy clean up of controls

The utilities in `Codevoid.Utilities.DOM` try to mitigate some of these issues.

### Easy clean up of controls ###
#### disposeOfControl / disposeOfControlTree / removeChild ####
These are a family of methods that make it easy to clean up controls when they're being removed from the DOM.

These should be called to either clean up controls before removal, or if you want to be super lazy, when removing & cleanup with `removeChild`.

For this to work, each of the controls that needs to do disposal work in the DOM must:

* Implement a `dispose` method
* Be on a DOM element which has a `data-win-control` attribute


Example usage:

    Codevoid.Utilities.DOM.disposeOfControlTree(domElementTree);
    
#### Avoiding Code Driven Layout ####
Wouldn't it be nice if you could squirrel away the markup structure in another HTML file so that you can easily reuse the markup in many parts of the application. What if you could also ensure that any scripts, or CSS required by that markup was also magically included?

Turns out WinJS has this! It's just not easily usable -- `WinJS.UI.Fragments` and `WinJS.Binding.Template` give you all that you need, but they're a bit of a pain in the behind to use together easily.

#### Codevoid.Utilities.DOM.loadTemplate ####
This helper will take a fragment (e.g. a HTML file), search for a specific template in that fragment, and then hand you the `WinJS.Binding.Template` to you so you can render it into the DOM.

Example:

HTML

    <html>
        <body>
            <div data-win-control="WinJS.Binding.Template"
                 data-templateid="sampleTemplate">
                 <div>Stuff!</div>
            </div>
        </body>
    </html>
    
JavaScript:

    Codevoid.Utilities.DOM.loadTemplate("/Path/To/Templates.html",
        "sampleTemplate").then(function(control) {
        control.render(dataContext, containerDiv);
    });
                                        
More detailed examples included in the tests for this project.

#### Codevoid.Utilities.DOM.marryPartsToControl / marryEventsToControl ####
Do you ever want to load a template, and have the different sections & child controls automatically extracted so you don't have to `querySelector` all the time? What if we could attach event handlers too? These two helper methods are the answer!

Example:

HTML

    <div data-event="{ custom: handleCustom }"
         data-part="parent">
        <div data-event="{ custom2: handleCustom2 }"
             data-part="child">
        </div>
    </div>

JavaScript

    var instance = {
        handleCustom: function() {}
        handleCustom2: function() {},
    };
    
    Codevoid.Utilities.DOM.marryPartsToControl(domElement, instance);
    // Now instance has 'partent', and 'child' properties now set
    // with the elements in those properties.
    
    Codevoid.Utilities.DOM.marryEventsToControl(domElement, instance);
    // If you raise custom2 DOM event in this tree, the handler on
    // instance will be raised. Same for custom.
    
## OAuth Helper ##
Writing OAuth requests is a pain in the behind. While Windows 8 provides a [basic](http://msdn.microsoft.com/en-us/library/windows/apps/windows.security.authentication.web.aspx) way to obtain the secret tokens from the web flow, it doesn't actually make it easy to sign the requests etc. While the [sample](http://code.msdn.microsoft.com/windowsapps/Web-Authentication-d0485122) explains it, it doesn't make it really easy when you need to do the whole OAuth 1.0 request signing shenanigans.

This class attempts to solve that by providing a WinJS.Promise based API that allows chaining, errors etc to propogate. It's simple to use, and I would recommend looking deeper into the tests to use it. However, simple usage:

    var url = "http://api.twitter.com/1/statuses/update.json"  
    var request = new Codevoid.OAuth.OAuthRequest(clientInfo, url);

    request.data = [{ key: "status", value: "Test@Status %78 update: " + Date.now() }];
    request.send().done(function (resultData) {
        var result = JSON.parse(resultData);
    },

Thats all there is too it. You get the raw response back, and you can do whatever you want.
