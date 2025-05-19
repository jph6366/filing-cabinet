# Game Development in Godot


## How C# Works in Godot, for eggheads 🐤🐥🥚🐣

### Reading

- [GodotEngine - C#/.NET](https://docs.godotengine.org/en/stable/tutorials/scripting/c_sharp/index.html)
- [Chickensoft - How CSharp Works in Godot](https://chickensoft.games/docs/how_csharp_works)
### Overview

Godot supports C# as an option for a scripting language, alongside GDScript.

- **Godot needs to download a .NET version of the editor from [Godot Website](https://godotengine.org/download/)**
  - Godot bundles the parts of .NET needed to run already compiled games.
    - Godot does not bundle the tools required to build and compile games, such as MSBuild and C# compiler, which need to be installed separately
- You ought to setup an external IDE, vscode works because microsoft, but I'm working on my doomemacs config.

- **To configure a project, you need a _tasks.json_ and _launch.json_ file in the _.vscode_ folder with the necessary cofiguration.**

**launch.json**
``` json
{
	"version": "0.2.0",
	"configurations": [
	// For these launch configurations to work, you need to setup a GODOT
    // environment variable. On mac or linux, this can be done by adding
    // the following to your .zshrc, .bashrc, or .bash_profile file:
    //macOS: export GODOT="/Applications/Godot.app/Contents/MacOS/Godot"
	//linux: export GODOT="/home/user/.config/godotenv/godot/bin/godot"
		{
			"name": "🕹 Debug Game",
			"type": "coreclr",
			"request": "launch",
			"preLaunchTask": "build-without-tests",
			"program": "${env:GODOT}",
			"args": [],
			"cwd": "${workspaceFolder}",
			"stopAtEntry": false,
			"console": "integratedTerminal"
		}
	// # For other environment configuration in my .bashrc
	// # .NET SDK Configuration
	// export DOTNET_ROOT="/usr/share/dotnet"
	// export DOTNET_CLI_TELEMETRY_OPTOUT=1 # Disable analytics
	// export DOTNET_ROLL_FORWARD_TO_PRERELEASE=1
	//
	// # Add the .NET SDK to the system paths so we can use the `dotnet` tool.
	// export PATH="$DOTNET_ROOT:$PATH"
	// export PATH="$DOTNET_ROOT/sdk:$PATH"
	// export PATH="$HOME/.dotnet/tools:$PATH"
	//
	// # Run this if you ever run into errors while doing a `dotnet restore`
	// alias nugetclean="dotnet nuget locals --clear all"
	]

}
```
**tasks.json**  both json taken from [ChickenSoft.GodotGame](https://github.com/chickensoft-games/GodotGame) 
- A C# Game template w/ debug launch configs, testing (locally and CI/CD), code coverage, dependency update checks, and spell check.

``` json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build",
      "command": "dotnet",
      "type": "process",
      "args": [
        "build",
      ],
      "problemMatcher": "$msCompile",
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      }
    },
    {
      "label": "build-without-tests",
      "command": "dotnet",
      "type": "process",
      "options": {
        "env": {
          "SKIP_TESTS": "1"
        }
      },
      "args": [
        "build",
      ],
      "problemMatcher": "$msCompile",
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      }
    },
    {
      "label": "coverage",
      "group": "test",
      "command": "${workspaceFolder}/coverage.sh",
      "type": "shell",
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": true
      },
    },
    {
      "label": "build-solutions",
      "group": "test",
      "command": "dotnet restore; ${env:GODOT} --headless --build-solutions --quit || exit 0",
      "type": "shell",
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "presentation": {
        "echo": true,
        "reveal": "silent",
        "focus": false,
        "panel": "shared",
        "showReuseMessage": false,
        "clear": false
      }
    },
  ]
}
```

#### First steps in adding C# to Godot

- You first create a Node in Godot and attach a C# script to it
  - Godot edits _project.godot_ file to include a reference to the GodotSharp module, as well as a .NET solution file and C# project file.
  - the **GD** static class which is part of the Godot namespace
    - see the class reference pages for [@GDScript](https://docs.godotengine.org/en/stable/classes/class_%40gdscript.html#class-gdscript) and [@GlobalScope](https://docs.godotengine.org/en/stable/classes/class_%40globalscope.html#class-globalscope)
- The C# API uses PascalCase instead of snake_case in GDScript.

``` csharp
using Godot;

public partial class YourCustomClass : Node
{
    // Member variables here, example:
    private int _a = 2;
    private string _b = "textvar";

    public override void _Ready()
    {
        // Called every time the node is added to the scene.
        // Initialization here.
        GD.Print("Hello from C# to Godot! ('v') ");
    }

    public override void _Process(double delta)
    {
        // Called every frame. Delta is time since the last frame.
        // Update game logic here.
    }
}
```

- Now these scripts will be compiled whenever the _dotnet build_ is run
- When referenced from a scene or resource, the code file should be considered a script asset rather than just a loose C# file sitting somewhere in your project
  - C# script assets (scripts added to a Godot scene or resource)
  - C# code files (plain C# code files not referenced directly by the engine)
  - 

#### How is Godot Engine interoperable with C#.NET?

- Godot is a native application written largely in C++
- To run C# code, Godot leverages Microsoft's nethost and hostfxr libraries which allow native applications to load and execute .NET assemblies
  - [Future Improvements to this section - Write a custom .NET host to control the .NET runtime from your native code](https://learn.microsoft.com/en-us/dotnet/core/tutorials/netcore-hosting)
- Godot uses custom [C# source generators](https://github.com/godotengine/godot/tree/master/modules/mono/editor/Godot.NET.Sdk/Godot.SourceGenerators) to generate the interop bindings between C# code and the engine's native code
- For practice, Write a Godot node script class
  - You can actually see into workings the _.csproj_ file, which instructs .NET to emit the generated code to a dir.
  - The source generators are implicitly included with _Godot.NET.Sdk_, special nuget packages which declare MSBuild targets and properties, package refs, etc. 


#### Packages, Assembly and External Code

- Using the .NET solution C# project file, you can reference external packages and assemblies in your project the same way you would in a vanilla C# project.
- **NuGet** is Microsoft's package manager for .NET.
  - The **NuGet client** tools provide the ability to produce and consume packages, and their gallery contains thousands of packages for C#.
    - A **NuGet package** is a single ZIP file with the _.nupkg_ extension that contains compiled code (DLLs), other files related to that code, and a descriptive manifest that includes information like the package's version number.
    - A compatible package means that it contains assemblies built for at least one target .NET framework thats compatible with the consuming project's target framework.
      - Meaning developers can create packages for specific frameworks or conform to a .NET standard, which all .NET and .NET Core projects can consume.

_heres some pictures, infer something from them_
  ![](https://learn.microsoft.com/en-us/nuget/media/nuget-roles.png)
  ![](https://learn.microsoft.com/en-us/nuget/media/dependency-graph.png)
  ![](https://learn.microsoft.com/en-us/nuget/media/nuget-restore.png)

#### External Code

   @TODO REWRITE FOR IMPROVEMENTS

 - Plain C# packages which target `netstandard` or a compatible target framework can be easily added to your project with `dotnet add package <package-name>` or by editing the `.csproj` file directly and running `dotnet restore`.
``` csharp
  <ItemGroup>
    <PackageReference Include="Chickensoft.Serialization" Version="2.0.0" />
    <!-- ... -->
```
 - External nuget packages which use the Godot API and have the same (or compatible) `GodotSharp` version as your game's `Godot.NET.Sdk` can be added and referenced from your game's project in the same way.
 - You can also include source generator packages by including the additional tags needed for .NET to understand them as source generators. This allows you to benefit from powerful metaprogramming tools, like the ones Chickensoft provides.
 ``` csharp
 <PackageReference Include="Chickensoft.Introspection.Generator" Version="2.0.0" PrivateAssets="all" OutputItemType="analyzer" />
 ```

## Game Design Documentation

A game design document may be made of text, images, diagrams, concept art, or any applicable media to better illustrate design decisions. Some design documents may include functional prototypes or a chosen game engine for some sections of the game.
- Introduction
  The game combines the fast-paced excitement of sports with arcade-style mechanics, blending elements of air hockey and a competitive frisbee throw.
- Background (optional)
- Description
  _Windjammers_ is a head-to-head arcade sports game where players compete in fast-paced matches of disc-throwing. Two players stand on opposite ends of a court, aiming to score points by getting the disc past their opponent into a goal zone. With quick reflexes, strategic throws, and unique character abilities, the game balances skill, tactics, and chaos for an exhilarating experience.
- Key features
	- character roster
	- character abilties/timing
	- arena(s)
	- arena environment
	- scoreboard & clock
	- competitive mode
	- casual mode
	- local multiplayer
	- ~~online multiplayer~~
- Genre
	- arcade sports
- Platform(s)
	1. Steam
	2. Nintendo Switch
	3. rest of em
	4. the web
#### The majority of video games should require an inclusion or variation of the following sections:
- Story 
	- story mode
	- roster-driven story
	- in-game chemistry
- Characters
	- region-based characters
	- playstyle class typing
	- archetypal characters (all-around, heavy, light, etc.)
- Level/environment design
	- arena-based
	- arena themes
	- implied depth
	- interactables
- Gameplay
	- Core
		- movement
		- skills
		- disc
		- player(s)
		- animations
	- Seconday
		- character selection
		- bots
		- modes
		- scoreboard
		- maps
		- character rule sets / conditions
		- stat tracking
- Art
	- visual style
		- 8bit/pixel art
	- character design
		- red and blue
	- environment
		- basic court
	- user interface
		- start menu
		- in-game menu
		- in-game overlay
		- character selection
		- settings
- Sound and Music
	- character-driven sounds
	- theme music
	- arena music
- Game Controls
	- keyboard
	- controller
- Accessibility
	- disability
	- languages
- Monetization
	- one-time buy
	- in-game appearance items
 . .