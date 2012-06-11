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

## OAuth Helper ##
Writing OAuth requests is a pain in the behind. While Windows 8 provides a [basic](http://msdn.microsoft.com) way to obtain the secret tokens from the web flow, it doesn't actually make it easy to sign the requests etc. While the [sample](http://code.msdn.microsoft.com/windowsapps/Web-Authentication-d0485122) explains it, it doesn't make it really easy when you need to do the whole OAuth 1.0 request signing shinangians.

This class attempts to solve that by providing a WinJS.Promise based API that allows chaining, errors etc to propogate. It's simple to use, and I would recommend looking deeper into the tests to use it. However, simple usage:

    var url = "http://api.twitter.com/1/statuses/update.json"  
    var request = new Codevoid.OAuth.OAuthRequest(clientInfo, url);

    request.data = [{ key: "status", value: "Test@Status %78 update: " + Date.now() }];
    request.send().done(function (resultData) {
        var result = JSON.parse(resultData);
    },

Thats all there is too it. You get the raw response back, and you can do whatever you want.