# Node.js Application using Socket.io
 
Socket.io provides realtime communication between between your node.js server and clients. This tutorial will walk you through hosting a socket.io based chat application on Windows Azure. For more information on Socket.io, see http://socket.io/.
 
You will learn:
 * Windows Azure specific considerations for Socket.io
 * How to create a Windows Azure worker role application
 
> **Note:** This tutorial assumes that you have installed the [Windows Azure SDK for Node.js](https://www.windowsazure.com/en-us/develop/nodejs/) and have downloaded and installed the publishing settings for your Windows Azure subscription. If you have not performed these tasks, the [Node.js Web Application](https://www.windowsazure.com/en-us/develop/nodejs/tutorials/getting-started/) tutorial will guide you through this process.
 
A screenshot of the completed application is below:
 
![Node.js Socket.IO App](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-10.png?raw=true)

## Windows Azure Considerations
 
When an application is hosted on Windows Azure, it only has access to the ports configured in the ServiceDefinition.csdef file. By default, the projects created using the **New-AzureService** cmdlet provided by the [Windows Azure SDK for Node.js](https://www.windowsazure.com/en-us/develop/nodejs/) open port 80. However when running the project in the Windows Azure emulator this port may be modified to a different port such as 81. To ensure that your application always receives traffic on correct port, you should use **process.env.port**, which will be mapped to the correct port at runtime. For example:

```
app.listen(process.env.port);
```

If your node application runs in a Web role (created using the **Add-AzureNodeWebRole** cmdlet,) you must configure Socket.io to use a transport other than WebSocket. This is because the Web role makes use of IIS7, which doesn’t currently support WebSockets. The following is an example of configuring Socket.io to use long-polling:

```
io.configure(function () {
  io.set("transports", ["xhr-polling"]);
  io.set("polling duration", 10);
}); 
```

> **Note:** This tutorial uses a Worker role, so the above Web role specific configuration is not used.
 
## Hosting the Chat Example in a Worker Role
 
The following steps will guide you through the process of creating a Windows Azure deployment project that will host the Socket.io chat example in a Worker role.
 
### Create a Project
1. On the **Start** menu, click __All Programs, Windows Azure SDK Node.js - November 2011__, right-click **Windows Azure PowerShell for Node.js**, and then select **Run As Administrator**. Opening your Windows PowerShell environment this way ensures that all of the Node command-line tools are available. Running with elevated privileges avoids extra prompts when working with the Windows Azure Emulator.
 
2. Create a new **node** directory on your C drive, and change to the c:\node directory:
 
![Create node directory](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-getting-started-6.png?raw=true)

3. Enter the following commands to create a new solution named **chatapp** and a worker role named **WorkerRole1**:

```
PS C:\node> New-AzureService chatapp
PS C:\node\chatapp> Add-AzureNodeWorkerRole
```

You will see the following response:

![Add-AzureNodeWorkerRole](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-1.png?raw=true)

### Download the Chat Example
 
For this project, we will use the [chat example](https://github.com/LearnBoost/socket.io/tree/master/examples/chat) from the Socket.io GitHub repository. Perform the following steps to download the example and add it to the project you previously created.

1. Click the **ZIP** button to download a .zip archive of the project.

![Zip Download](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-2.png?raw=true)

2. In Windows Explorer, right click the downloaded .zip file and select **Extract All**. When prompted, select a directory to extract the files to and then click Extract. The folder containing the extracted files should open.

![Extract Compressed Folders](imagess/dev-nodejs-socketio-3.png)
 
3. Navigate the folder structure until you arrive at the examples\chat folder. Copy the contents of this folder to the C:\node\chatapp\WorkerRole1 folder created earlier.

![Chat Folder](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-4.png?raw=true)

After the copy operation completes, the contents of the WorkerRole1 folder should appear as follows:

![WorkerRole1 Folder](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-5.png?raw=true)
 
4. In the C:\node\chatapp\WorkerRole1 folder, delete the server.js file, and then rename the app.js file to server.js. This removes the default server.js file created previously by the **Add-AzureNodeWorkerRole** cmdlet and replaces it with the application file from the chat example.
 
### Modify Server.js and Install Modules
 
Before testing the application in the Windows Azure emulator, we must make some minor modifications. Perform the following steps to the server.js file:

1. Open the server.js file in Notepad or other text editor.
 
2. Modifiy the require statement for socket.io by removing the ‘../../lib/’ from the beginning of the string. The modified statement should appear as:

``` 
, sio = require('socket.io'); 
```

This will ensure that the socket.io library is correctly loaded from the node_modules folder when the application is ran
 
3. To ensure the application listens on the correct port, open server.js in Notepad or your favorite editor, and then change the following line by replacing **3000** with **process.env.port**:

```
app.listen(3000, function () { 
```

4. Also remove the line that begins with **console.log** as this is not useful when running in the Windows Azure emulator or after deployment to Windows Azure.
 

After saving the changes to server.js, use the following steps to install required modules, and then test the application in the Windows Azure emulator:

1. If it is not already open, start the **Windows Azure PowerShell for Node.js** from the Start menu by expanding **All Programs, Windows Azure SDK Node.js - November 2011**, right-click **Windows Azure PowerShell for Node.js**, and then select **Run As Administrator**.
 
2. Change directories to the folder containing your application. For example, C:\node\chatapp\WorkerRole1.
 
3. To install the modules required by this application, use the following npm command:

```
PS C:\node\chatapp\WorkerRole1\npm install
```

This will install the modules listed in the package.json file. After the command completes, you should see output similar to the following:

![Install Modules](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-7.png?raw=true)
 
4. Since this example was originally a part of the Socket.io GitHub repository, and directly referenced the Socket.io library by relative path, Socket.io was not referenced in the package.json file, so we must install it by issuing the following command:

```
PS C:\node\chatapp\WorkerRole1\npm install socket.io 
```

### Test and Deploy

1. Launch the emulator by issuing the following command:

```
PS C:\node\chatapp\WorkerRole1> Start-AzureEmulator -launch
```

> **Note:** If the browser window does not open automatically, you can manually open it and browse to the address returned by the Start-AzureEmulator command.
 
2. When the browser window opens, enter a nickname and then hit enter. This will all you to post messages as a specific nickname. To test multi-user functionality, open additional browser windows using the same URL and enter different nicknames.

![Enter Nicknames](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-8.png?raw=true)
 
3. After testing the application, stop the emulator by issuing the following command:

```
PS C:\node\chatapp\WorkerRole1> Start-AzureEmulator -launch
```

4. To deploy the application to Windows Azure, use the Publish-AzureService cmdlet. For example:

```
PS C:\node\chatapp\WorkerRole1> Publish-AzureService -name chatapp -location "North Central US" -launch
```

Be sure to use a unique name, otherwise the publish process will fail. After publishing is complete, you should see the following response.

![Publish](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-9.png?raw=true)

After the deployment has completed, the browser will open and navigate to the deployed service.

![Completed App](https://github.com/microsoft-dpe/Node.js-Application-using-Socket.io/blob/master/images/dev-nodejs-socketio-10.png?raw=true)

Your application is now running on Windows Azure, and can relay chat messages between different clients using Socket.io.
