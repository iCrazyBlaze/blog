---
layout:     post
title:      Adding mod support in Unity
date:       2019-09-30 20:25:00
summary:    How to let users make mods for your game
categories: coding
thumbnail: modio
tags:
 - modding
 - code
 - unity
 - csharp
 - mod.io
---

Many, many great things have evolved from game modding. Popular games such as Counter-Strike and Rocket League were born as simple, humble modifications of someone else's game, and look where they are now. Modding is an important part of any game's ecosystem, which is why it's vital for developers to support it. But how the hell do you allow for modders to do what they do best with your game in Unity easily? Enter [Mod.io](https://mod.io/).

# What is it?
Mod.io is a modding platform and API. When you integrate it into your game, the user can download a mod from the mod.io page, or in-game. [Mod.io's Unity Asset Store package](https://apps.mod.io/unity-plugin) comes with a built-in example scene of a mod browser, similar to the one in Garry's Mod. Essentially, it's a glorified download manager.

A "mod" in our case consists of a zip file, containing an **Asset Bundle.** When we subscribe to the mod in the mod manager, mod.io will automatically unzip its contents and place them into a folder, specified in the plugin's settings.

## Let's start by opening the scene provided in `Assets/Plugins/mod.io/Mod Browser/Example Scene.`

# Asset Bundles
Once you have downloaded the asset store package and set it up correctly, it's time to actually add mod support.

First of all, Unity has something called [**Asset Bundles.**](https://docs.unity3d.com/Manual/AssetBundles-Workflow.html) An AssetBundle is sort of like a zip file: a file that contains files itself. *Asset-ception.*
This is the file that we want to read from our mod.

# Creating an Asset Bundle
To create an Asset Bundle, we will create a new C# Script inside the folder `Assets/Editor` called `CreateAssetBundles.cs`. This code was taken directly from the [Unity Docs page for Asset Bundles Workflow](https://docs.unity3d.com/Manual/AssetBundles-Workflow.html):
```csharp
using UnityEditor;
using System.IO;

public class CreateAssetBundles
{
    [MenuItem("Assets/Build AssetBundles")]
    static void BuildAllAssetBundles()
    {
        string assetBundleDirectory = "Assets/AssetBundles";
        if (!Directory.Exists(assetBundleDirectory))
        {
            Directory.CreateDirectory(assetBundleDirectory);
        }
        BuildPipeline.BuildAssetBundles(assetBundleDirectory, BuildAssetBundleOptions.None, BuildTarget.StandaloneWindows);
    }
}
```
This script will add a context menu item called **Build AssetBundles**.

To assign an asset to a Bundle, find the `AssetBundle` option at the bottom of the Inspector, and where it says "none", click the drop-down and choose "New". You will be prompted to give it a name. I'm going to call mine `modbundle`.

Once that's done, build the Asset Bundles, and you should see a new folder (unless you already had one), and inside it, your Asset Bundle.

# Converting an Asset Bundle to a Mod
Opening your project in explorer, navigate to where your bundle is (e.g. `Assets/AssetBundles/modbundle`) and click **Send to -> Compressed (Zipped) file**, or add it to a zip file however you see fit. We will be uploading this file to our Mod.io page. Find out how to do so on their website.

# Reading an Asset Bundle from a Mod
First of all, we need to create a new C# Script and add these lines at the top of the file:
```csharp
using System.Collections.Generic;
using UnityEngine;
using ModIO;
using System.IO;
```

Now we specify the name of the bundle we want to load. This is what modders will name their bundles when exporting from Unity. My Asset Bundle is called `modbundle`, so I'm going to set this to `modbundle`.
```csharp
public string bundleName = "modbundle";
```

Now, we want to create a new public function called `LoadMods()`. This is so we can call this at any time. I'm also going to call mine from the `Start()` method, for testing purposes.
```csharp
    private void Start()
    {
        LoadMods();
    }

    public void LoadMods()
    {
        // ...
    }
```
Each mod is stored as a string, which represents the path to the mod's contents. This is why our list of mods is a string list. We will initialize our list in the `LoadMods()` method we just made:
```csharp
    public void LoadMods()
    {
        List<string> mods = ModManager.GetInstalledModDirectories(true);
        // ...
    }
```
Now we will need to loop over each one of the mods to get their content individually.
```csharp
        for (int i = 0; i < mods.Count; i++)
        {
            string mod = mods[i];
            Debug.Log("Loaded mod " + Path.GetFileName(mod));
            // ...
        }
```
This code will tell us what mods we have loaded.

We're already halfway there! All we have left to do is loading the Asset Bundle and doing whatever you want to do with said Asset Bundle. To load the bundle, we will get the path by combining our mod's path with our bundle name, essentially pointing it to `"modpath/assetbundle"`.
```csharp
            string modpath = Path.Combine(mod, bundleName);
```
Here, we're using `Path.Combine` as it's cleaner than making the string manually, and works cross-platform (Linux and Mac like to use different slashes in paths to Windows).

Now we attempt to load the file, and throw an error if the file can't be read.
```csharp
            var loaded = AssetBundle.LoadFromFile(modpath);
            if (loaded == null)
            {
                Debug.LogError("Failed to load mod " + mod);
                return;
            }
            // ...
```
***And believe it or not, all of the main code is done!***

Now you have loaded your mods, you will want to access their contents. In my case, I want to load a Prefab in the Asset Bundle called `MyObject` and spawn it in the scene.
```csharp
            var prefab = loaded.LoadAsset("MyObject");
            Instantiate(prefab);
```
While this is unrealistic and very simple, it should give you a taste of the Mod.io API's possibilities.

# Games using Mod.io
- **[MORDHAU](https://store.steampowered.com/app/629760/MORDHAU/)**

- **[Totally Accurate Battle Simulator (TABS)](https://store.steampowered.com/app/508440/Totally_Accurate_Battle_Simulator/)**

# Full script (InitMods.cs)
```csharp
using System.Collections.Generic;
using UnityEngine;
using ModIO;
using System.IO;

public class InitMods : MonoBehaviour
{
    public string bundleName = "modbundle";

    // Start is called before the first frame update
    private void Start()
    {
        LoadMods();
    }

    public void LoadMods()
    {
        // Get all installed mods
        List<string> mods = ModManager.GetInstalledModDirectories(true);

        for (int i = 0; i < mods.Count; i++)
        {
            string mod = mods[i];
            Debug.Log("Loaded mod " + Path.GetFileName(mod));

            // Get path to asset bundle
            string modpath = Path.Combine(mod, bundleName);

            // Try to load mod from file
            var loaded = AssetBundle.LoadFromFile(modpath);
            if (loaded == null)
            {
                Debug.LogError("Failed to load mod " + mod);
                return;
            }


            // Below is where you add your game-specific code. Use loaded.LoadAsset to get stuff from the asset bundle.
            var prefab = loaded.LoadAsset("MyObject");
            Instantiate(prefab);

        }
    }

}
```