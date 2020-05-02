# Installation of Jenkins (for Microsoft Builds (.NETCore/NETFramework/Xamarin/NETStandard..)
Following instructions describe setup of a Jenkins configuration which is used too automate builds of Microsoft Visual Studio projects.
It's recommended to create Nuget packages instead of simple dll files as build output of your visual studio class projects and host those nuget packages in your own nuget server and uses that server in your projects. 

Furthermore this tutorial will tell you how to achieve following for each job in Jenkins:

- Display of warnings graph, test results graph, codecoverage graph, detailed HTML report of codecodeverage
- Trigger release build of project, create nuget package and push that to your own nuget server
![alt text](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Jenkins_Overview.png "Logo Title Text 1")

## Preparations
### 1. Tools required
- Jenkins Installation on Windows machine
- Visual Studio 2017+ Installation on same machine where Jenkins is running. Use installer to choose necessary features like Xamarin, NETCore....
- Nuget.exe https://www.nuget.org/downloads
- OpenCover.exe https://www.nuget.org/packages/OpenCover/
- ReportGenerator.exe https://www.nuget.org/packages/ReportGenerator/
- NugetServer like Baget (its free and open source and works) https://github.com/loic-sharma/BaGet/

Copy Nuget.exe together with Installation folders of OpenCover and Reporter Nuget Package in a new folder called ___JENKINSTOOLS___ and move that folder to C:\ root. So result structure should look something like that:

![alt text](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Jenkins_Tools.png "Logo Title Text 1")

Tip:
You can simply create a new Visual Studio project and get the Opencover, Reporter and Cobertura NugetPackages using Nuget packet managment menu. After installation go to .package folder and copy the opencover and reporter folders from there to ___JENKINSTOOLS___. Right click on folder after moving it to C:\ and set folder rights to read/write for users.

### 2. Required Jenkins Plugins
- Source Code Plugin like Git Plugin: allows use of Git as a build Source Code Management
- MSBuild Plugin: allows use of MSBuild.exe to build .NET projects
- Warnings Next Generation Plugin: collects compiler warnings of the project modules and visualizes the results
- Cobertura Plugin: allows you to capture and visualize code coverage data 
- HTML Publisher Plugin: allows you to publish HTML reports for Job builds
- Environment Injector Plugin: This plugin makes it possible to set an environment for the builds
- xUnit Plugin: This plugin makes it possible to record xUnit test reports
- change-assembly-version-plugin: The plugin will change the AssemblyVersion of all files named AssemblyInfo.cs (or other inserted) under the workspace folder.
- PowerShell plugin

#### Opional Jenkins Plugins
-	Dashboard View: Customizable dashboard that can present various views of job information
- Role-based Authorization Strategy Plugin: Enables user authorization using a Role-Based strategy
- Simple Theme Plugin: This plugin allows to customize Jenkin's appearance with custom CSS and JavaScript
- Modern Status Plugin: Change some icons

### 3. Windows service configuration
To avoid problems with the NugetPackage restore step you should set the user of Jenkins service to the local admin account instead of default systemaccount. For that go to Windows service menu, right click on Jenkins entry and set account details to admin account. 

### 4. Manage Roles & Assign Roles
I suggest that anonym users can see projects without the need to login (so they can see project statistics). Of course they should not be able to configure any settings. For that do the follwing:

Install that plugin in Jenkins:
https://wiki.jenkins.io/display/JENKINS/Role+Strategy+Plugin

After that there's a new menu entry in Jenkins main menu and you can configure like that:
Screenshots: 
![Roles.png](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Roles.png)
![Roles2.png](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Roles2.png)

### 5. Fix Javascript rights to display HTML Report
We will use the ReportGenerator.exe in our build scripts later on and it won't display the result correct without the following fix.
(The following solution is for Windows.)

In [Jenkins directory]\jenkins.xml search following lines:
<executable>java.exe</executable>
<arguments>[arguments are here]</arguments>

Replace existing argument entry in that file with following:
```shell
  <executable>%BASE%\jre\bin\java</executable>
  <arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -Dhudson.model.DirectoryBrowserSupport.CSP="sandbox; default-src 'none'; img-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" -jar "%BASE%\jenkins.war" --httpPort=8080 --webroot="%BASE%\war"</arguments>
```

Then restart the Jenkins service to pick up the change

### 6. For projects hosted on GITHUB: Github Token configuration
1. ___Jenkins -> Credentials -> System -> Global Credentials -> Add Credential -> Secret Text -> Github Personal Access Token (GPAT) (admin:repo_hook, repo) -> Save___
    
2. Go to ___Jenkins -> Global Configuration -> Add Git Server, Automanage Hooks ->  Test Connection___

### 7. MSBuild Plugin configuration
1. Jenkins Configuration -> Helper Tools configuration:
2. MSBuild Path is MSBuild.exe path to VisualStudio installation:
```shell
C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\
```
### 8. For projects hosted on GITHUB: Installation of Git
1. Download Git: https://git-scm.com/downloads
2. Jenkins Configuration -> Helper Tools configuration:
3. Change Path to Git:
```shell
C:\Program Files\Git\cmd\git.exe
```

## Job configuration
Below I will list all configuration steps needed to configure a job in Jenkins to achieve following results:

- Testresults in graph 
- CodeCoverage in graph 
- Warnings in graph
- Coverage report in HTML

### Manage SourceCode Repository: GitProject
1. Create new project
2. Source Code Management -> Git
   -> set URL Repository
   -> set Credentials from list
   
### Source Code Management: 
Set up Github or other type like SVN by youself. Should not be that hard!

### Build Triggers
Nothing special here, set up as you like

### Build Environment: 
Check: ___Delete workspace before build starts___

Check: ___Inject environment variables___

Properties Content:

```shell
JOBPROJECTNAME=MyProject
JOBTESTPROJECTNAME=MyProject.Tests
```
### 1. Build action: Change assembly

#### Configuration for NETCORE/NETSTANDARD2+ class libraries (Directory.Build.props support)
Assembly Version: 
```shell
$SVN_REVISION (or other variable you want to use as version suffix)
```
FileName: 
```shell
**/MyProject/Directory.Build.props
```
RegexPattern: 
```shell
<Version>(\d+).(\d+).(\d+).(\d+)<\/Version>
```
ReplacementPattern: 
```shell
<Version>$1.$2.%s<\/Version>
 ```
#### Configuration for WPF/pre NETSTANDARD packages (Assemblyversion (.cs/.vb) support)
Assembly Version: 
```shell
$SVN_REVISION (or other variable you want to use as version suffix)
```
FileName: 
```shell
**/MyProject/My Project/AssemblyInfo.vb
```
RegexPattern: 
```shell
Assembly(\w*)Version\("(\d+).(\d+).(\d+).(\d+)"\)
```
ReplacementPattern: 
```shell
Assembly$1Version("$2.$3.%s")
 ```
 
### 2. Build action: Execute Windows batch command 
Configure NuGet Package Restoration. For that paste a command like the following in the command text box: 
```shell
::%%%%% Restore NUGET Dependecies before MSBuild %%%%%
C:\JENKINSTOOLS\nuget.exe restore "%WORKSPACE%\%JOBPROJECTNAME%\%JOBPROJECTNAME%.sln"
```

### 3. Build action: Configure MSBuild Process using Visual Studio
MSBuild Version: Select the correct entry to msbuild.exe here
MSBuild build file: 
```shell
${WORKSPACE}\${JOBPROJECTNAME}\${JOBPROJECTNAME}.sln
```
Command Line Arguments: 
```shell
/p:Configuration=Release
```

### 4. Build action: Execute Windows batch command 

```shell
::%%%%% STATISTICS building: OpenCover, Cobertura, HTMLReport %%%%%

rmdir /q /s target
mkdir target

::%%%%% ALL PATHS %%%%%
set "opencover=C:\JENKINSTOOLS\opencover\4.7.922\tools\OpenCover.Console.exe"
set "reportgenerator=C:\JENKINSTOOLS\reportgenerator\4.0.15\tools\net47\ReportGenerator.exe"
set "dotnet=C:\Program Files\dotnet\dotnet.exe"
set "targetdir=target\Coverage"
mkdir %targetdir%
 
::%%%%% UNIT TEST REPORT, TRX %%%%%
"%dotnet%" test "%WORKSPACE%\%JOBPROJECTNAME%\%JOBTESTPROJECTNAME%\%JOBTESTPROJECTNAME%.csproj" --logger "trx;LogFileName=UnitTests.trx"

::%%%%% OPENCOVER.EXE FOR COVERAGERESULT.XML %%%%%
"%opencover%" -register:user "-target:%dotnet%" -targetargs:"test %JOBPROJECTNAME%\%JOBTESTPROJECTNAME%\%JOBTESTPROJECTNAME%.csproj" -output:%targetdir%\OpenCover.coverageresults.xml -filter:"+[*]* -[Serilog*]* -[GalaSoft*]* -[*Tests]*" -returntargetcode -coverbytest:* -mergebyhash -oldstyle

::%%%%% COVERAGERESULT.XML FOR REPORT AND COBERTURA DIAGRAMM %%%%%
"%reportGenerator%" -reports:%targetdir%\OpenCover.coverageresults.xml -reporttypes:HtmlInline;Cobertura -targetdir:%targetdir% -assemblyfilters:-xunit* -classfilters:-AutoGeneratedProgram
```

### 5. Build action: Process xUnit test result report
Type: ___MS Test Version NA___
```shell
${JOBPROJECTNAME}\${JOBTESTPROJECTNAME}\TestResults\UnitTests.trx
```

### 6. Build action: Windows Powershell 

This will create nuget package and submit to Nuget server

```shell

####### NUGET-DEPLOYMENT (BaGet Nugetserver) - NO CHANGES needed in this section!######
####### Provide nupkg-packet for Baget und save package-folder of BaGet ######

# package folder of Nugetserver Baget
$NUGETSERVERREPOSITORY = "C:\inetpub\wwwroot\BaGet\packages"

# Copy nuget folder to Workspace so that directory content can be copied to backupfolder in post build
$NUGETBACKUPSERVER = "$ENV:WORKSPACE\NUGETS\"
mkdir $NUGETBACKUPSERVER

# get created nuget package
$NUGETFILENAME = Get-ChildItem -Path "$ENV:WORKSPACE\$ENV:JOBPROJECTNAME\$ENV:JOBPROJECTNAME\bin\Release\" -Filter "*.nupkg" |Select -First 1

# Push to Nugetserver (copy to workspace first, because otherwise path could be too long?!)
$NUGETTEMP = "$ENV:WORKSPACE\NUGETTEMP\"
mkdir $NUGETTEMP
Copy-Item $ENV:WORKSPACE\$ENV:JOBPROJECTNAME\$ENV:JOBPROJECTNAME\bin\Release\$NUGETFILENAME $NUGETTEMP -Recurse -Force

dotnet nuget push -s http://localhost:5000/v3/index.json $NUGETTEMP$NUGETFILENAME

# Backup preparation for post build action
Copy-Item $NUGETSERVERREPOSITORY $NUGETBACKUPSERVER -Recurse -Force
```

### 1. Post-build Actions: Publish HTML reports
HTML directory to archive: 
```shell
target\Coverage
```
Index page[s]: 
```shell
index.htm
```

### 2. Post-build Actions: Record compiler warning and statitics results
Tool: ___MsBuild___

### 3. Post-build Actions: Publish Cobertura Coverage report
Cobertura xml report pattern: 
```shell
**/target/Coverage/*Cobertura.xml
```

### 4.Send build artifacts to windows share (optional)
  SOURCE: ___NUGETS/___
  Remove prefix: ___NUGETS/___
  Remote directory: ___NUGETS/___

# Additional tips for Jenkins setup
## Replace JENKINS label with your own text
1. Install __Simple Theme__ Plugin in Jenkins
2. __Manage Jenkins -> Configure System -> Theme -> Add -> URL of theme CSS:__
``` Shell
http://JENKINSIP/userContent/theme.css
```
2. Create a theme.css file with following content and place it in userContent folder of Jenkins:
``` CSS
@charset “utf-8”;
 
/* Custom style for CloudBees Jenkins Platform 
.logo img {
content:url("cksplus.ico");
height: 32px;
vertical-align:middle;
}
*/

.logo #jenkins-name-icon {
display: none;
}

.logo:after {
content: 'JENKINS COCOTUS';
font-weight: bold;
font-size: 20px;
margin-left: 50px;
margin-right: 12px;
color: white;
line-height: 40px;
}
```
