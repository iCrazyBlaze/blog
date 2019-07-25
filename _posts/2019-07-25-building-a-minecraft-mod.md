---
layout:     post
title:      Adventures in building a Minecraft mod
date:       2019-07-25 14:35:00
summary:    How to learn Minecraft modding
categories: coding
thumbnail: minecraft
tags:
 - minecraft
 - modding
 - code
 - java
 - forge
---

Recently, I have been hard at work building [Twitch Vs Minecraft](https://github.com/iCrazyBlaze/TwitchVsMinecraft), my first ever mod for [Minecraft Forge](https://files.minecraftforge.net). In this post, I will be showing you how to create your first Minecraft mod using the Forge API for Minecraft 1.12.2, and I will answer some of the most commonly asked modding questions with full code.

Before reading, make sure you have some basic knowledge of Java. Some Java tutorials will be linked below.

> DISCLAIMER: THIS TUTORIAL CODE WILL NOT WORK IN VERSIONS ABOVE 1.12.2, AND HAS NOT BEEN TESTED WITH ANY VERSION BELOW.

# Getting Started
First, you will need to head over to the [Minecraft Forge Website](https://files.minecraftforge.net) and download the recommended MDK version. Once you have the zip file, extract it into a folder. Next, open a terminal in that folder and type the following command:
```
gradlew setupDecompWorkspace
```
If you're using Eclipse, you will then want to type:
```
gradlew eclipse
```
to generate the correct files. If you're using [IntelliJ IDEA](https://www.jetbrains.com/idea/) (which is highly recommended), you can instead import the **build.gradle** file as a project.

You will then want to run:
```
gradlew genIntellijRuns
```
This can also be found in the **Gradle** tab on the right of the screen.

# The Example Mod
When you first set everything up, you will be given an example mod. Here's what that looks like:
```java
package com.example.examplemod;

import net.minecraft.init.Blocks;
import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.fml.common.Mod.EventHandler;
import net.minecraftforge.fml.common.event.FMLInitializationEvent;

@Mod(modid = ExampleMod.MODID, version = ExampleMod.VERSION)
public class ExampleMod
{
    public static final String MODID = "examplemod";
    public static final String VERSION = "1.0";
    
    @EventHandler
    public void init(FMLInitializationEvent event)
    {
		// some example code
        System.out.println("DIRT BLOCK >> "+Blocks.dirt.getUnlocalizedName());
    }
}
```
While you most likely want to delete this, it's a good example of how to create a main class.

# Build.gradle
This file contains details such as your mod's name, package name, author name, and version. You will need to change these values:
```gradle
version = "1.0"
group= "com.yourname.modid" // http://maven.apache.org/guides/mini/guide-naming-conventions.html
archivesBaseName = "modid"
```
Here's an example taken from Twitch Vs Minecraft:
```gradle
version = "1.3.1"
group= "com.icrazyblaze.twitchmod"
archivesBaseName = "twitchmod"
```
Remember, the **archivesBaseName** property needs to be the same as the last part of the **group** property.

# Mcmod.info
This file is no longer found in newer versions of the MDK, and is instead replaced with **mods.toml**. However, for this tutorial we are using 1.12.2.

This file also contains information for your mod, specifically the info that is displayed in the **Mods** menu in-game. [This page](https://mcforge.readthedocs.io/en/latest/gettingstarted/structuring/) tells you everything you need to know about using this file.

# Running the mod
While making your mod, you will want to be able to run it. To do this, click the **Debug icon** ![](https://www.jetbrains.com/help/img/idea/2019.2/icons.actions.startDebugger_dark.svg@2x.png) with the **runClient** configuration enabled. You can alternatively run the game from a terminal, or use the Run option rather than debugging.

While changed classes are automaitcally reloaded when debugging in Eclipse, IntelliJ IDEA doesn't have this by default.

To enable **Reloading changed classes** using a keyboard shortcut, go to **File -> settings** and search for **"keymap"** when in the keymap menu, search for **"Reload"** and find **"Reload changed classes"**, which is located under **"Run"**. I personally have this bound to <kbd>ctrl</kbd> + <kbd>alt</kbd> + <kbd>r</kbd>. While this may not always work, it helps when making quick edits to your mod.

When loading into a game, you will be logged in as a random username. You can set it up to use your Minecraft account, but this is not recommended, as it can cause security issues, and is usually not needed.

# Creating a main class - Forge events
So now we need to create our main class. As a starting point, you can use the example mod shown earlier. The **FMLInitializationEvent** happens when the mod is loading, so we will see the output somewhere around the main menu. There is also the **FMLpreInitializationEvent** event, which as the name implies, is called before the Initialization event.

The Pre-Initialization event is used for setting up the **Logger**. This is useful for outputting information to the console. The event can also be used for loading **Config Files**. Here's an example of what that looks like:
```java
    public static Logger logger;
    public static Configuration config;

    //......

    @EventHandler
    public static void Preinit(FMLPreInitializationEvent event) {
        logger = event.getModLog();
        config = new Configuration(event.getSuggestedConfigurationFile());
        ConfigManager.loadConfig();
    }
```
*ConfigManager appears later in the tutorial.*

To use the logger, we will use:
```java
Main.logger.info("Hello world!");
```
You can also make the logger output warnings and errors.

The **ServerStarting** event is used to register commands. Here's an example:
```java
    @EventHandler
    public static void serverStarting(FMLServerStartingEvent event) {
        event.registerServerCommand(new TTVCommand());
    }
```

# Creating a command
Let's create our own command that we can register from our main class just like that one. First of all, we'll create a package called **command**. This is just a standard way of keeping things neat and easy to access. We then create a class called **TutorialCommand** and extend it from **CommandBase** by adding `extends CommandBase` to the end of our `public class TutorialCommand`.

Now we want to create a list of aliases: `private final List aliases;`.
To use the list of aliases, add this code:
```java
    public TutorialCommand() {
        aliases = new ArrayList();
        aliases.add("tut");
    }
```
This piece of code adds the alias "tut" to our command. But we don't have a default command name yet. Next, we will add this:
```java
    @Override
    public String getName() {
        return "tutorial";
    }
```
This is the default command. Now, the player is able to type either "/tut" or "/tutorial" in chat to use the command. However, if the user uses the command wrong, we will want to output an error. To do this, use:
```java
    @Override
    public String getUsage(ICommandSender sender) {
        return "/tutorial <argument/otherargument> [argument value]";
    }
```
Next, we want to register the aliases we added earlier, so we add:
```java
    @Override
    public List getAliases() {
        return this.aliases;
    }
```
*(This code won't change for you - every command has this.)*

Next up is **checkPermission**. You can use this to determine who is allowed to use your command, or if it is considered a "cheat". To let everyone use your command in any gamemode, use:
```java
    @Override
    public boolean checkPermission(MinecraftServer server, ICommandSender sender) {
        return true;
    }
```
You can play around with the value it returns yourself, and see what you like best.

Now, we can add **Tab Completions**. This essentially works like AutoCorrect, where the player presses tab to finish typing an argument, or get a list of all the arguments they could use.
```java
    @Override
    public List<String> getTabCompletions(MinecraftServer server, ICommandSender sender, String[] args, BlockPos pos) {
        return CommandBase.getListOfStringsMatchingLastWord(args, autocomplete);
    }
```
This code will use a list called **autocomplete** to get its corrections, but we haven't created that yet. We can add that just underneath our `private final List aliases;` as 
```java
private final String[] autocomplete = {"argument1", "argument2"};
```
You will need to replace the arguments with your own. You can have as many of these as you like, just continue adding values to the list.

Now, for the real meat of the command: the `execute` event. This is where whatever code you want to execute when your player uses your command goes. Here's how you implement it:

```java
    @Override
    public void execute(MinecraftServer server, ICommandSender sender, String[] args) throws CommandException {

                if (sender instanceof EntityPlayer) {
                    // your code goes here!
                }

    }
```
You can check for arguments to, by using this if statement:
```java
if (args[0].equals("argument1")) {
    // the player has typed "/tutorial argument1"
} else if (args[0].equals("argument2")) {
    // the player has typed "/tutorial argument2"
}
```
You can also replace `.equals()` with `.equalsIgnoreCase()` if you don't want your command to be case-sensitive.

You can also use `sender.sendMessage()` to reply to a command, like this:
```java
if (args[0].equalsIgnoreCase("argument1")) {
    sender.sendMessage(new TextComponentString(TextFormatting.GREEN + "Argument one!"));
} else if (args[0].equalsIgnoreCase("argument2")) {
    sender.sendMessage(new TextComponentString(TextFormatting.DARK_RED + "Argument two!"));
}
```
**TextFormatting** is used here to change the colour of the text in the chat.

# Creating blocks and items
For this section, there is a really nice YouTube tutorial by Harry Talks. [Click Here](https://youtu.be/42z8_UDLmk4) to watch it.

# Creating a GUI Overlay
GUI overlays appear on the player's screen, but don't enable the mouse. Think of them as custom HUD elements. 

To get started with creating one, I will create my `TutorialOverlay` class inside a package called `gui`. Here's what the class will look like:
```java
package com.icrazyblaze.tutorialmod.gui;

import net.minecraft.client.Minecraft;
import net.minecraft.client.gui.Gui;
import net.minecraftforge.client.event.RenderGameOverlayEvent;

public class TimerGui extends Gui {

    @SubscribeEvent
    public void onRenderGui(RenderGameOverlayEvent.Post event) {

        if (event.getType() != RenderGameOverlayEvent.ElementType.TEXT)
            return;

        if (condition goes here) {

            Minecraft mc = Minecraft.getMinecraft();
            String text = "Hello world!";

            drawString(mc.fontRenderer, text, 4, 4, Integer.parseInt("AA0000", 16));
        }

    }

}
```
This code renders the text "Hello world!" in dark red in the top left corner of the screen. Replace `condition goes here` with a valid boolean to be able to enable and disable the GUI.

# Creating a GUI Screen
GUI Screens are like Overlays, but the mouse is enabled and the player can interact with the GUI. An example of this is the game's pause menu.

First, we start off by extending our class from `GuiScreen`.

Now we need to set some variables:
```java
    private static final ResourceLocation BG_TEXTURE = new ResourceLocation(Reference.MOD_ID, "textures/gui/messagebox_background.png");
    public static String message = null;
    boolean displayGUI = true;
```

We can change if the GUI pauses the game in Singleplayer mode by changing this code:
```java
    @Override
    public boolean doesGuiPauseGame() {
        return true;
    }
```

We can add buttons to our **Button List** for use later on, as seen here:
```java
    @Override
    public void initGui() {

        GuiButton btn = new GuiButton(200, width / 2 - 75, height / 2 + 55, 150, 20, I18n.format("gui.done"));
        this.buttonList.add(btn);

    }
```
This button will close our GUI.

Now we need to create our `drawScreen` event. This GUI opens a box with a message that wraps around, and has a button that closes the GUI. The variable `message` will need to be set beforehand, and I do this from whatever class actually opens the GUI, hence why it is public and static.
```java
    @Override
    public void drawScreen(int mouseX, int mouseY, float partialTicks) {

        drawDefaultBackground();

        GlStateManager.color(1, 1, 1, 1);
        mc.getTextureManager().bindTexture(BG_TEXTURE);
        int x = (width / 2) - 88;
        int y = (height / 2) - 83;

        drawTexturedModalRect(x, y, 0, 0, 256, 256);
        fontRenderer.drawString("Message Box", width / 2 - 32, height / 2 - 78, 4210752);
        fontRenderer.drawSplitString(message, x + 7, height / 2 - 60, 165, 4210752);

        if (displayGUI) {
            super.drawScreen(mouseX, mouseY, partialTicks);
        } else {
            mc.player.closeScreen();
        }

    }
```
`drawDefaultBackground` draws the transparent grey background seen in every GUI screen in Minecraft. `width` and `height` refer to the width and height of the game window, which in fullscreen for me is **1920x1080**. By dividing the width and height by 2, we get the center of the screen. This is then adjusted for the texture, because otherwise the top-left corner would be in the center, rather than the actual center of the image. `4210752` is the colour code used for GUI screen titles in Minecraft. `super.drawScreen` is used to essentially loop this funtion until `displayGUI` is false.


Now let's make our button functional:
```java
    @Override
    public void actionPerformed(GuiButton btn) {

        if (btn.id == buttonList.get(0).id) {
            displayGUI = false;
        }

    }
```
When the button is clicked, it sets a boolean called `displayGUI` to false, and in the `drawScreen` function we say that if the variable is false, we close the screen.

The background image I used for the GUI *("textures/gui/messagebox_background.png")* can be found [here](https://github.com/iCrazyBlaze/TwitchVsMinecraft/blob/master/src/main/resources/assets/twitchmod/textures/gui/messagebox_background.png). It is based on the Crafting Table GUI.

# Creating a custom crafting recipe
To create a custom crafting recipe, you want to first go to your **resources folder** (usually src/main/resources) and create a folder named **recipes**. Create a json file with whatever name you want, but make it something sensible and memorable, like your item name followed by the word "recipe". Then, head over to [this website](https://crafting.thedestruc7i0n.ca/) and copy and paste the output json into your blank json file. because we are using 1.12.2, some items, blocks and crafting methods, for example the Stonecutter and Blast Furnace are unavailable. Make sure you only use what is available in your Minecraft version.

If you want your crafting recipe to make a custom item, you can use a placeholder item. For example, I will make the output of my crafting recipe on the website TNT, and then replace where it says `minecraft:tnt` in the output json to `tutorialMod:tutorialItem`.

# Config files
Configuration files are stored in the `config` folder, as `<modname>.cfg`. To save and load a configuration file, we will create a **ConfigManager class**. I am creating under the package `util`.

Here's what it should look like:
```java
public class ConfigManager {

    public static Configuration config;

    public static void loadConfig() { // Gets called from preInit

        // Config file is made in the Main class
        config = Main.config;

        try {

            // Properties go here

        } catch (Exception e) {
            // Failed reading/writing, just continue
        }
    }

    public static void saveConfig() {

        // Config file is made in the Main class
        config = Main.config;

        try {

            // Properties go here

        } catch (Exception e) {
            // Failed reading/writing, just continue
        }
    }
}
```
Now we will replace *properties go here* with our properties.
**Properties** are how variables are stored inside the file.
```java
Property tutorialProp = config.get(Configuration.CATEGORY_GENERAL, // What category will it be saved to, can be any string
        "TUTORIAL_VARIABLE", // Property name
        "Hello world!", // Default value
        "Our string."); // Comment
```
Paste this property inside both the `loadConfig()` and `saveConfig()` functions. Underneath it in `loadConfig`, we will add:
```java
tutorialClass.tutorialVariable = tutorialProp.getString();
```
Here, our `tutorialVariable` is part of another class, called `tutorialClass`. Because our variable is a string, we use `.getString()` to return the value. You can also use `.getInt()`, `.getBoolean()` etc. depending on the data type.

Now we move on to saving to the config file. Underneath our property in `saveConfig()`, we will set our property to the value of `tutorialClass.tutorialVariable` - the reverse of what we just did in `loadConfig()`.

```java
tutorialProp.set(tutorialClass.tutorialVariable);
```
It's really that simple! We can call `saveConfig()` from other classes - this is best done using a command.

# Helpful links
Want to know more? Check out these links:

[TutorialsPoint basic Java tutorials](https://www.tutorialspoint.com/java/)

[CodeAcademy Java course](https://www.codecademy.com/learn/learn-java)

[MMD Pins (more links)](https://github.com/MinecraftModDevelopment/Modding-Resources/blob/master/dev_pins.md)

[Forge Docs](https://mcforge.readthedocs.io/)

[ShadowFacts' Tutorials](https://shadowfacts.net/tutorials/forge-modding-112/)

[McJty Modding Wiki (multiple versions)](https://wiki.mcjty.eu/modding/index.php?title=Main_Page)

[Video Tutorials (Harry Talks)](https://www.youtube.com/channel/UCUAawSqNFBEj-bxguJyJL9g/videos)

[CubiCoder's 1.12.2 modding tutorials](https://cubicoder.github.io/tutorials/1-12-2/tutorials/)

[Jabelar's tutorials (for 1.7 and above)](http://jabelarminecraft.blogspot.com/)

[Minecraft By Example (source code)](https://github.com/TheGreyGhost/MinecraftByExample/)

[Minecraft Mod Development Discord](https://discordapp.com/invite/EDbExcX)

[Minecraft Forum](https://www.minecraftforum.net/)

[Minecraft CurseForge](https://curseforge.com/minecraft)