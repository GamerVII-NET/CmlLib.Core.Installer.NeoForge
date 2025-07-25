# CmlLib.Core.Installer.NeoForge

## Minecraft NeoForge Installer
<img src='https://raw.githubusercontent.com/CmlLib/CmlLib.Core/master/icon.png' width=128>

NeoForge Installer for [CmlLib.Core](https://github.com/CmlLib/CmlLib.Core)

> **Note:** This is an unofficial library developed by [GamerVII](https://github.com/GamerVII-NET), who is not affiliated with [CmlLib](https://github.com/CmlLib).

## Features
* Automatic installation of Vanilla Minecraft before installing NeoForge
* Automatic updating of links for NeoForge installation
* Skipping NeoForge re-installation if it's already installed
* Ability to retrieve available NeoForge versions

## Supported NeoForge Versions

NeoForge versions for Minecraft **1.20.2+** are supported.

## Install

Install the [CmlLib.Core.Installer.NeoForge NuGet package](https://www.nuget.org/packages/CmlLib.Core.Installer.NeoForge)

or download the nupkg file from [Releases](https://github.com/GamerVII-NET/CmlLib.Core.Installer.NeoForge/releases) and add references to them in your project.

## Usage Example

```csharp

using CmlLib.Core.Installer.Forge;
using CmlLib.Core;
using CmlLib.Core.Auth;
using CmlLib.Core.Installer.Forge.Versions;
using CmlLib.Core.Installers;
using CmlLib.Core.ProcessBuilder;
using SampleForgeInstaller;

// var tester = new AllInstaller();
// await tester.InstallAll();
// return;

var path = new MinecraftPath(); // use default directory
var launcher = new MinecraftLauncher(path);

// show launch progress to console
var fileProgress = new SyncProgress<InstallerProgressChangedEventArgs>(e =>
    Console.WriteLine($"[{e.EventType}][{e.ProgressedTasks}/{e.TotalTasks}] {e.Name}"));
var byteProgress = new SyncProgress<ByteProgress>(e =>
    Console.WriteLine(e.ToRatio() * 100 + "%"));
var installerOutput = new SyncProgress<string>(e =>
    Console.WriteLine(e));

//Initialize variables with the Minecraft version and the Forge version
var mcVersion = "1.21";
var forgeVersion = "21.0.24-beta";

using var client = new HttpClient();
var newVersions = new ForgeVersionLoader(client);
var versions = await newVersions.GetNeoForgeVersions(mcVersion);
var neoVersion = versions.FirstOrDefault(c => c.VersionName == forgeVersion);

if (neoVersion is null)
{
    throw new Exception("NeoForge version not found");
}

//Initialize MForge
var neoForge = new NeoForgeInstaller(launcher);

var version_name = await neoForge.Install(mcVersion, forgeVersion, new ForgeInstallOptions
{
    FileProgress = fileProgress,
    ByteProgress = byteProgress,
    InstallerOutput = installerOutput,
});
//var version_name = await forge.Install(mcVersion); // install the recommended forge version for mcVersion
//OR var version_name = forge.Install(mcVersion, forgeVersion).GetAwaiter().GetResult();

//Start Minecraft
var launchOption = new MLaunchOption
{
    MaximumRamMb = 1024,
    Session = MSession.CreateOfflineSession("TaiogStudio"),
};

var process = await launcher.CreateProcessAsync(version_name, launchOption);

// process.Start();


// print game logs
var processUtil = new ProcessWrapper(process);
processUtil.OutputReceived += (s, e) => Console.WriteLine(e);
processUtil.StartWithEvents();
await processUtil.WaitForExitTaskAsync();

Console.WriteLine();
```