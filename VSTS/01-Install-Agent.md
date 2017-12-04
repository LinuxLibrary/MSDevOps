# Installing and Configuring VSTS Agent on Windows 2012 R2

- Create Agent Pool
	- To create agent pool login to your VSTS account and navigate to home
	- Click on the Administer Account(Gear Symbol) and select Agent Pool
	- Click on New Pool, give a name and Click OK
- Install Agent
	- Select the AgentPool created earlier
	- Click on Download agent
	- Create Personal Access Token (PAT)
		- Click on your user in the right most corner and select Security
		- Select Security tab
		- Click Personal Access Tokens
		- Click add and give it a name in the Description
		- Choose Selected Scopes and select (Agent Pools(read,manage))
		- Click Create Token
		- Copy and write the token to a file
	- Create a directory for the agent and unzip the contents of the package

		```
		PS C:\> mkdir agent ; cd agent
		PS C:\agent> Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win7-x64-2.124.0.zip", "$PWD")
		PS C:\agent> .\config.cmd	
	
	
		>> Connect:
	
		Enter server URL > https://msdevopstest01.visualstudio.com/
		Enter authentication type (press enter for PAT) >
		Enter personal access token > ****************************************************
		Connecting to server ...
		
		>> Register Agent:
		
		Enter agent pool (press enter for default) > MSDevOpsTest
		Enter agent name (press enter for DevTest01) > DevTest01-01
		Scanning for tool capabilities.
		Connecting to the server.
		Successfully added the agent
		Testing agent connection.
		Enter work folder (press enter for _work) >
		2017-12-04 08:26:48Z: Settings Saved.
		Enter run agent as service? (Y/N) (press enter for N) > Y
		Enter User account to use for the service (press enter for NT AUTHORITY\NETWORK SERVICE) >
		Granting file permissions to 'NT AUTHORITY\NETWORK SERVICE'.
		Service vstsagent.msdevopstest01.DevTest01-01 successfully installed
		Service vstsagent.msdevopstest01.DevTest01-01 successfully set recovery option
		Service vstsagent.msdevopstest01.DevTest01-01 successfully configured
		Service vstsagent.msdevopstest01.DevTest01-01 started successfully
		```

	- You can verify the agent running status from the Services console(servics.msc)