# OneNote add-ins JavaScript programming overview (Preview)

Read on to learn more about OneNote add-ins, or [get started creating one](onenote-add-ins-getting-started.md).

## Overview 

OneNote introduces a JavaScript API (in preview) for OneNote Online add-ins. You can create task pane add-ins, content add-ins, and add-in commands that interact with OneNote objects and connect to web services or other web-based resources.

Add-ins consist of two basic components:

- A **web application** consisting of a webpage and any required JavaScript, CSS, or other files. These files are hosted on a web server or web hosting service, such as Microsoft Azure. In OneNote Online, the web application displays in a browser control or iframe.
	
- An **XML manifest** that specifies the URL of the add-in's webpage and any access requirements, settings, and capabilities for the add-in. This file is stored on the client. OneNote add-ins use the same [manifest](https://dev.office.com/docs/add-ins/overview/add-in-manifests) format as other Office Add-ins.

**Office Add-in = Manifest + Webpage**

![An Office add-in consists of a manifest and webpage](../../images/onenote-add-in.png)

### Using the JavaScript API

Add-ins use the runtime context of the host application to access the JavaScript API. The API has two layers: 

- A **rich API** for OneNote-specific operations, accessed through the **Application** object.
- A **common API** that's shared across Office applications, accessed through the **Document** object.

#### Accessing the rich API through the *Application* object

Use the **Application** object to access OneNote objects such as **Notebook**, **Section**, and **Page**. With rich APIs, you run batch operations on proxy objects. The basic flow goes something like this: 

1. Get the application instance from the context.

1. Create a proxy that represents the OneNote object you want to work with. You interact synchronously with proxy objects by reading and writing their properties and calling their methods. 

1. Call **load** on the proxy to fill it with the property values specified in the parameter. This call is added to the queue of commands. 

   Method calls to the API (such as `context.application.activeSection.getPages()`) are also added to the queue.
    
1. Call **context.sync** to run all queued commands in the order that they were queued. This synchronizes the state between your running script and the real objects, and by retrieving properties of loaded OneNote objects for use in your script. You can use the returned promise object for chaining additional actions.

For example: 

   ```
    function getPagesInSection() {
        OneNote.run(function (context) {
            
            // Get the id and title of the pages in the current section.
            var pages = context.application.activeSection.getPages();
            
            // Queue a command to load the id and title for each page.            
            pages.load('id,title');
            
            // Run the queued commands, and return a promise to indicate task completion.
            return context.sync()
                .then(function () {
                    
                    // Read the id and title of each page. 
                    $.each(pages.items, function(index, object) {
                        var pageId = object.id;
                        var pageTitle = object.title;
                        console.log(pageTitle + ': ' + pageId); 
                    });
                })
                .catch(function (error) {
                    app.showNotification("Error: " + error);
                    console.log("Error: " + error);
                    if (error instanceof OfficeExtension.Error) {
                        console.log("Debug info: " + JSON.stringify(error.debugInfo));
                    }
                });
        });
    }
   ```

You can find supported OneNote objects and operations in the [API reference](../../reference/onenote/onenote-add-ins-javascript-reference.md).

### Accessing the common API through the *Document* object

Use the **Document** object to access the common API, such as the [getSelectedDataAsync](https://dev.office.com/reference/add-ins/shared/document.getselecteddataasync)
and [setSelectedDataAsync](https://dev.office.com/reference/add-ins/shared/document.setselecteddataasync) methods. 

For example:  

   ```
function getSelectionFromPage() {
    Office.context.document.getSelectedDataAsync(
        Office.CoercionType.Text,
        { valueFormat: "unformatted" },
        function (asyncResult) {
            var error = asyncResult.error;
            if (asyncResult.status === Office.AsyncResultStatus.Failed) {
                console.log(error.message);
            }
            else $('#input').val(asyncResult.value);
        });
}
   ```

In general, you only use the common API to do something that isn't supported in the rich API. To learn more about using the common API, see the Office Add-ins [documentation](https://dev.office.com/docs/add-ins/overview/office-add-ins) and [reference](https://dev.office.com/reference/add-ins/javascript-api-for-office).


<a name="om-diagram"></a>
## OneNote object model diagram 

  ![OneNote object model diagram](../../images/onenote-om.png)


## Additional Resources

- [Build your first OneNote add-in (Preview)](onenote-add-ins-getting-started.md)
- [OneNote JavaScript API reference (Preview)](../../reference/onenote/onenote-add-ins-javascript-reference.md)
- [Rubric Grader sample (Preview)](https://github.com/OfficeDev/OneNote-Add-in-Rubric-Grader-Preview)
- [Office Add-ins platform overview](https://dev.office.com/docs/add-ins/overview/office-add-ins)