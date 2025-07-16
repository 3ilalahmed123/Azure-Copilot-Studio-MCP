# ✨ Deploy and Invoke an Azure App Service hosted MCP Server with Copilot Studio Lab✨
This lab guides users on how to deploy an ASP.NET MCP Server hosted within Azure App Service to be invoked by Copilot Studio.

## 💡 Lab Objectives

1. Scaffold and implement a minimal .NET 9 MCP (Model Context Protocol) Web SDK server locally.  
2. Deploy the server to Azure App Service with GitHub CI/CD.  
3. Configure a Custom Connector in Copilot Studio and invoke your MCP server tool. 


# ⚙️ Prerequisites
- .NET 9 SDK installed (https://dotnet.microsoft.com/en-us/download/dotnet/9.0)
- Azure Subscription (http://azure.microsoft.com/free)
- Copilot Studio (https://www.microsoft.com/en-us/microsoft-copilot/microsoft-copilot-studio)
- Github Account and Repo (https://github.com/)
- Git
- (OPTIONAL) @modelcontextprotocol/inspector, NodeJS

# 1) Scaffold and implement a minimal .NET 9 MCP (Model Context Protocol) Web SDK server locally. 💻

1. Create a Github Repo for storing your server code. This will be used later to deploy to Azure App Service.


2. Within a new folder create a new minimal Web SDK project targeting net9.0.

```
dotnet new web -o SandboxMCPServer -f net9.0

cd .\SandboxMCPServer\

git init

dotnet new gitignore
```

3. Enter the following commands to install the libraries you'll use.

```cmd
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package ModelContextProtocol --prerelease
dotnet add package ModelContextProtocol.AspNetCore --prerelease
```

4. Begin coding.
```
code .
```


5. At the top of ```Program.cs```, include:
``` 
// Add References
using ModelContextProtocol;
using ModelContextProtocol.Server;
using McpServer.Tools;
```


6. Define your tool by creating a new file called SandboxTool.cs and add the code below.

```SandboxTool.cs```
```
using ModelContextProtocol.Server;
using System.ComponentModel;

namespace McpServer.Tools;

[McpServerToolType]
public sealed class SandboxTool
{
    [McpServerTool, Description("Gets the amount of sand in a sand box")]
    public static string getSandAmount()
    {
        return "10kg";
    }
}
```

7. Go back to ```Program.cs``` after declaring the builder ```var builder = WebApplication.CreateBuilder(args);``` add the code from below. This is so we can register MCP services & CORS.

```
// Add MCP server services with HTTP transport and Sandbox tool
builder.Services.AddMcpServer()
    .WithHttpTransport()
    .WithTools<SandboxTool>()
;

// Add CORS for HTTP transport support in browsers
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});
```

9. Replace the code:
```
app.MapGet("/", () => "Hello World!");

app.Run();
```
with:

```
// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

// Enable CORS
app.UseCors();

// Map MCP endpoints at root
app.MapMcp("/mcp");

// Add a simple home page
app.MapGet("/status", () => "MCP Server on Azure App Service - Ready for use with HTTP transport");

app.Run();

```

This enables CORS, maps our MCP endpoints and creates a route for testing the web app.

10. Run the server locally with ```dotnet run```

11. Test the Server 

    Navigating to http://localhost:port/status should return ```MCP Server on Azure App Service - Ready for use with HTTP transport```

    http://localhost:port/mcp/sse should return something similar to ```event: endpoint
data: /message?sessionId=BsFNIVelagg6zPebkpStHw```

Note: You can use ```npx @modelcontextprotocol/inspector``` to test your MCP Server thoroughly.


12. Push to the github repo.

```
git add .
git commit -m "intial"
git branch -M main
git remote add origin https://github.com/[YOURNAME]/[REPONAME].git
git push -u origin main
```
If you have trouble pushing to the remote generate a token from https://github.com/settings/personal-access-tokens/new and format the remote URL like ```https://[YOURTOKEN]@github.com/[YOURNAME]/[REPONAME].git```

# 2) Deploy the server to Azure App Service with GitHub CI/CD. ☁️


1) Navigate to Azure App Service https://portal.azure.com/#browse/Microsoft.Web%2Fsites and Create a new Web App.

2) Configure the App Service Settings with the details below:


Project Details

- Resource Group: Create a new resource group for your Project
- Name: Any
- Publish: Code
- Runtime Stack: .NET 9
- Operating System: Windows
- Region: Any

Pricing Plans
- Windows Plan: Leave default or change if necesarry
- Pricing Plan: Free (F1) (Scale up if needed)

Database
- Create a database: Unchecked

Deployment
- Continuous deployment: Enable

- Configure Github Settings connecting it to where the server code is stored.

Networking
- Enable public access: On

Monitor + secure
- Enable Application Insights: No
- Enable Defender for App Service: Unchecked

3. Create the Resource and wait for the deployment.

4. After the deployment, select go to resource. Once the Last Deployment within the Properties Tab says  'Successful on [Date]', Test the website to ensure it is working.

Great, now our server is live. Next, we will need to integrate it with Copilot Studio.

# 3) Configure a Custom Connector in Copilot Studio and invoke your MCP server tool. ⛏️

1) Navigate to Copilot Studio and select Tools within the pane on the left.

2) Select ```+ New Tool``` and choose Custom Connector

3) ```+ New custom connector``` > ```Import from Github```

4) Choose the following settings and continue:
- Connector Type: Custom
- Branch: dev
- Connector: MCP-Streamable-HTTP


5) Change the host to your App Service App's Default domain.

The Swagger code should look something like this:

```
swagger: '2.0'
info:
  title: MCP Server
  description: >-
    This MCP Server will work with Streamable HTTP and is meant to work with
    Microsoft Copilot Studio
  version: 1.0.0
host: sandboxmcp-d5c9f7dyg6auece2.canadacentral-01.azurewebsites.net
basePath: /mcp
schemes:
  - https
consumes: []
produces: []
paths:
  /:
    post:
      summary: MCP Server Streamable HTTP
      x-ms-agentic-protocol: mcp-streamable-1.0
      operationId: InvokeMCP
      responses:
        '200':
          description: Success
definitions: {}
parameters: {}
responses: {}
securityDefinitions: {}
security: []
tags: []
```


6) Create the Connector.

7) Create a New Agent, skipping the configuration steps. **Ensure Generative AI Orchestration is Enabled.**

8) Within the Tools tab, Select ```+ Add a tool``` , select the ```Model Context Protocol``` pill and then your newly created connector. 

9) Create a new connection and then add it to the agent. 

10) Refresh the Test Chat Pane and ask 
``` 
How much sand is in a sandbox?
``` 

NOTE: You may need to visit the connection manager and reconnect if the Status is ```Not Connected``` and the agent can't invoke the tool.

11) Copilot Studio will invoke the MCP server answering '10kg'.

Congratulations! You have now created your own MCP server and integrated it with Copilot Studio. Feel free explore by making your own additions!


### Credits
Azure hosted Server: https://github.com/Azure-Samples/remote-mcp-webapp-dotnet

### References
MCP C# SDK: https://github.com/stephentoub/mcp

MCP: https://modelcontextprotocol.io/introduction
