---
title: Actor Types
weight: 1
geekdocCollapseSection: true
---


StepMania (and it's many forks) use what's known as the [Actor model](https://en.wikipedia.org/wiki/Actor_model). OutFox inherits this behaviour. Everything in the engine is an Actor at its' very core.

Because everything is an actor, there are many types of actors, which are described below. Not everything listed is able to be created in Lua. Those that aren't will be noted as such. However, all of these can be found in the Lua environment.

## A note on LoadActor()

This is a helper function that can be used to dynamically load an actor based on the file type. However, this should not be used as it is more resource intensive and can be easily avoided when the filetype is known. For loading external lua files, `loadfile(path)()` can be used, though the full path is required.

<!--{{<toc>}}-->

<center>
{{<columns>}}
{{< button size="large" relref="actormultivertex/" >}}ActorMultiVertex{{< /button >}}
{{< button size="large" relref="sprite/" >}}Sprite{{< /button >}}
{{< button size="large" relref="bitmaptext/" >}}BitmapText{{< /button >}}
{{</columns>}}
</center>

## ActorFrame

An Actor that can contain one or more Actors. Think of it as a box that can hold as many objects as it can.

Example:
```lua
Def.ActorFrame{
    -- This sprite is now included inside of the ActorFrame.
    -- Any changes from the ActorFrame will affect the sprite, such as position, rotation,
    -- zoom and such.
    Def.Sprite{}
}
```

ActorFrames can hold other actors. The ``Def.`` format is set up like any other lua table, allowing for creating actors in batches. Because of this, there are multiple ways to build an ActorFrame.

### Inline building

Actors can be explicitly placed into an ActorFrame like this:
```lua
Def.ActorFrame{
	Def.Actor{
		--Commands and stuff go here.
	},
	Def.Sprite{
		--More commands and stuff go here.
	}
}
```

### Concatenation

Because lua tables can be concatenated (added) to each other, so can ActorFrames.

This can allow for programatically creating Actors in batches as needed.

However, if one does not plan on creating actors programmaticly, then a simple `return Def.ActorFrame{...` is all that's needed. Storing it into a local variable that gets returned will waste resources.

There are two ways to add onto an actorframe.

<!-- TODO: There's probably better ways to show this. -->
Method 1 (indexing):
```lua
local t = Def.ActorFrame{
	Def.Sprite{
		--commands & stuff
	},
	Def.BitmapText{
		--who even knows
	},
}

t[#t+1] = Def.Actor{
	--It's the third actor
}
return t
```

Method 2 (lua concatenation):
```lua
return Def.ActorFrame{
	Def.Sprite{
		--commands & stuff
	},
	Def.BitmapText{
		--who even knows
	},
} .. Def.Actor{
	--It's the third actor
}
```

## ActorFrameTexture

It's like an ActorFrame, but instead of displaying what's inside onto the screen, it places the contents of the ActorFrame onto a Texture that a Sprite (or any other actor that can load textures) can read from.

Popularized by its' use in NotITG, although the behavior is different. NotITG's version places the screen itself onto a texture instead of what's inside the ActorFrame and requires basezoom to be used on sprites to make the image fit inside the game window.

Example:
```lua
--[[
    This example will create an ActorFrameTexture that uses a sample BitmapText, and project it
    to the center to the screen in half of the available size of the viewport window.
]]

-- This ActorFrame will contain everything.
local t = Def.ActorFrame{}

-- First, create the ActorFrameTexture.
local AFT = Def.ActorFrameTexture{
	InitCommand = function(self)
		-- Start by adding a name to the texture. This is optional, but best to provide one,
		-- in case you need to work with several ActorFrameTextures in the same screen.
		self:SetTextureName( "MyAFT" )

		-- Now we need to give the ActorFrameTexture a resolution, which will be
		-- how much is available to render.
		-- As this example will be half the size of the available screen, we'll give the size
		-- half of the screen.
		self:SetWidth( SCREEN_WIDTH*.5 ):SetHeight( SCREEN_HEIGHT*.5 )

		-- By default this option is disabled, but you can enable it to allow transparency
		-- in your ActorFrameTexture.
		-- In this case, it will disabled to show the available render area (shown as black)
		self:EnableAlphaBuffer( false )

		-- We are done with the AFT, so now we have to create it.
		self:Create()
	end,

	-- After the initializer command, any actors included inside will be projected.
	-- The actors here act exactly the same as a ActorFrame, so they will start on the top-left
	-- corner of the available draw area given (in this case, half of the screen.)
	Def.BitmapText{
		Font = "Common Normal",
		Text = "Testing String!",
		BeginCommand = function(self)
			self:zoom(2):spin()
			-- In order to see the texture, let's center it.
			-- Calculations in actors inside of the AFT will need conversion as dimensions are
			-- different, so for example:

			-- Since we're doing half of the screen for dimensions, to center it, we need to half
			-- that size as well.

			-- 1280 / 2 = 640 (Screen Size)
			-- 640 / 2 = 320 (Actor Position)
			-- 0 - 640 = 0 .. 1 (Size Area, using Screen Size)
			-- (320/640) = 0.5 (Point of Actor Position using above calculation)

			self:xy( SCREEN_WIDTH*.25, SCREEN_HEIGHT*.25 )
		end
	}
}

t[#t+1] = AFT

-- Now that we've created the ActorFrameTexture, it will be sent into a global texture, that we
-- can then grab into a sprite, and it can be done in two ways. These will be shown inside the
-- OnCommand in the sprite.

-- Method 1: Load sprite directly into sprite.
t[#t+1] = Def.Sprite{
	-- Sprite name given inside the SetTextureName command in your AFT.
	Texture = "MyAFT",
	-- Let's center the projection so we can see it.
	OnCommand=function(self)
		self:xy( SCREEN_CENTER_X, SCREEN_CENTER_Y )
	end
}

-- Method 2: Load sprite on demand on runtime.
t[#t+1] = Def.Sprite{
	-- Let's center the projection so we can see it.
	OnCommand=function(self)
		self:xy( SCREEN_CENTER_X, SCREEN_CENTER_Y )

		-- Sprite name given inside the SetTextureName command in your AFT.
		self:Load( "MyAFT" )
	end
}

return t
```

## ActorMultiTexture

A quad that allows for layering of multiple textures. Every layer's TextureMode can be manipulated using SetTextureMode (Note that it's 0-indexed).

Can be manipulated like how a Sprite would. once textures are added. By default, every layer is multiplicative.

 <!-- Seems complicated to use, and usage in the wild appears to be almost non-existant. Also can cause HUGE video leaks when using EffectMode. Why???-->
<!--TODO: Anyone know how to even use this? Please help!-->

## ActorProxy

Creates a visual clone of any Actor given as a target. Anything done to the Target Actor shows up in the Proxy, but anything done to the Proxy will not show up on the Target.

```lua
local t = Def.ActorFrame{}

-- Let's say you want to clone this object, it's a bitmaptext actor that contains some words.
-- An example using an object assigned to a local variable.
local mytext
t[#t+1] = Def.BitmapText{
	Font="Common Normal",
	Text="Hello There!"
	InitCommand=function(self)
		-- We assign this BitmapText to become mytext, so any actor who uses mytext can control this BitmapText.
		-- Note the InitCommand, this is neccesary so the object can be assigned at construction time, leaving space for it to be properly
		-- generated.
		mytext = self
	end
}

t[#t+1] = Def.ActorProxy{
	BeginCommand=function(self)
		-- With mytext generated, using a later command in the pipeline, like [On/Begin]Command, we call SetTarget to get mytext,
		-- and clone the actor that's in it.
		self:SetTarget( mytext )
	end
}

-- You can also grab from nested ActorFrames, and even outside of the scope.
t[#t+1] = Def.ActorFrame{
	Def.BitmapText{
		Font="Common Normal",
		Text="Hello There!",
		Name="MyText",
	},

	Def.ActorProxy{
		BeginCommand=function(self)
			-- For information about the GetParent and GetChild functions, check "Anatomy of a Lua Actor: Obtaining Childs and ActorFrame Levels".
			self:SetTarget( self:GetParent():GetChild("MyText") )
		end
	}
}

return t -- Finish by returning the ActorFrame.
```

## ActorScroller

An actor that allows for creating scrolling displays. If BitmapTexts are used, they have to be encased in an ActorFrame.

<!--TODO: Do all actors have to be wrapped in an ActorFrame?-->

## Banner

A song or course banner, meant for use with custom music wheels or any other place a song's banner would show up.

## BGAnimation

A background animation that can be used during gameplay Can either be loaded from a BGAnimation.ini file (Only if Quirks Mode is enabled), or constructed like an ActorFrame would.

## [BitmapText](bitmaptext)

Used to display text on screen.

## BPMDisplay

Shows the BPMs of the song currently selected. What shows depends on what the simfile sets for the display BPM.

Usually used as part of ScreenSelectMusic.

## ComboGraph

Shows the combos that were held throughout the song, with combo breaks separating the blocks, and the largest combo being colored.

```lua
Def.ComboGraph{
	InitCommand=function(self)
		-- Load the MetricsGroup that will create the basis for the graph.
		self:Load("ComboGraph")
		local playerStageStats = STATSMAN:GetCurStageStats():GetPlayerStageStats(player)
		local stageStats = STATSMAN:GetCurStageStats()
		-- Data is graph from the current stage stats and the player's stats.
		self:Set(stageStats, playerStageStats)
	end
}
```

## ControllerStateDisplay

Appears to show the status of controllers/pads.
<!-- TODO: Is there anything out there that uses this? Any examples at all? -->

## CourseContentsList

A list of the songs for a given course. Can be given a limit for how many songs to show and how many to show at a time.

## DeviceList

Displays a list of all input devices. Often, Keyboard and Mouse will be listed.

Functions like most other BitmapText actors.

## DifficultyIcon

A Sprite-type actor that shows a different icon for each difficulty.

If there number of frames are double the amount of difficulties available in the engine, the player number will offset the icon shown.

## DynamicActorScroller

Inherits from ActorScroller, uses only one child Actor to draw all items.

## FadingBanner

A song or course banner that can fade between the banner for different songs or courses.

Often seen in ScreenSelectMusic.

## GradeDisplay

Shows the grade a player has gotten for a given song's chart.

## GraphDisplay

Displays a graph containing data points for a player's life throughout the stage.

Settings for a GraphDisplay can only be defined through metrics.
<!--TODO: Maybe we could make a different load function to allow for one to be set up through lua and not metrics???-->

```lua
Def.GraphDisplay{
	InitCommand=function(self)
		-- Load the MetricsGroup that will create the basis for the graph.
		self:Load("GraphDisplay")
		local playerStageStats = STATSMAN:GetCurStageStats():GetPlayerStageStats(player)
		local stageStats = STATSMAN:GetCurStageStats()
		-- Data is graph from the current stage stats and the player's stats.
		self:Set(stageStats, playerStageStats)
	end
}
```

## GrooveRadar

A recreation of the five-point "Groove Radar" from DDR. Can take arbitrary values as well as the song's radar values.
<!--TODO: Do we mention what the radar values are?-->

## HelpDisplay

A BitmapText-type actor that allows for cycling automatically through tips.

```lua
Def.HelpDisplay {
	File="Common Normal",
	InitCommand=function(self)
		local s = "Hello" .. "::" .. "This is a new line" .. "::" .. "And another one!"
		-- We'll set the next to toggle between each line (Separated by double colons) by 0.5 seconds.
		self:SetSecsBetweenSwitches(0.5)
		-- And tell the HelpDisplay to show that text.
		:SetTipsColonSeparated(s)
	end
}
```

## HoldJudgment

Often found in Player, though there is no way to grab the ones that are in Player.

The judgment that shows up on a column when dropping or clearing a hold & roll.

There is an extra function to allow tracking the hold judgments from a MultiPlayer.

## InputList

Displays a list of inputs as they occur. Can show unmapped and mapped inputs.

Functions like most other BitmapText actors.

## LogDisplay

This is an actor type created from _fallback's scripts.

Displays lua script errors as they happen. Most themers shouldn't need to touch this.

## LifeMeter

Cannot be created from lua, but can be grabbed from ScreenGameplay.

Often used in gameplay screens. This shows how much life a Player currently has.

## LifeMeterBattery

Cannot be created from lua.

Often used in gameplay screens. This shows how many more mistakes a player is allowed before failing.

## MemoryCardDisplay

Shows the current state of a player's inserted memory card. Made of images, with one for each state.

## MenuTimer

Cannot be created from lua, but is a part of every screen that inherits from ScreenWithMenuElements.

A Timer that counts down and proceeds to the next screen when it reaches 0.

## MeterDisplay

Shows the current progress of an operation. It appears to only show the progress as 50%.
<!--TODO: Is there a way to set the percentage?-->

## Model

Allows one to display MilkShape3D ASCII models, having separate attributes for materials, bones and meshes.

```lua
Def.Model{
	Meshes="MyModel.txt",
	Materials="MyModel.txt",
	Bones="MyModel.txt",
	OnCommand=function(self)
		self:Center()

		-- When a model begins its animation, it will loop indefinitely. To stop that, use the loop command to
		-- set the flag to false.
		self:loop(false)
	end
}
```

Meshes are the composition and structure of the Model. This data represents the vertices that make the Model take shape.

Materials are the textures that the model will use. These can be any of the image formats listed in the Supported File Extensions page. They can also be .ini files that define animated textures on a Def.Sprite.

Bones make the model come to life. They can be defined within the primary model file, or, in the case of dancing characters, be controlled via a separate file that only contains the bones.

In the above example, all three attributes used the same filepath; all the necessary data was contained within a single file. It is possible to configure the MilkShape 3D software to output distinct files for meshes, materials, and bones, and set each Def.Model attribute accordingly, but that is outside the scope of this lesson.

***NOTE: All three attributes must be provided within Def.Model as paths to resources that can be loaded or the game will crash.***

With OutFox, you can store several types of animations on the same model and call them at any moment.
```lua
-- Let's load a animation called WarmUp, located on a file called "Warm1.txt".
self:LoadBones( "WarmUp", "Warm1.txt" )

-- Now the model has an animation called "WarmUp", and can be called at any time with playanimation.
-- The second value is optional, and sets the rate of the animation's speed.
self:playanimation( "WarmUp", 1 )
```

## ModIconRow

Shows icons for the currently set modifiers of a player.

Not all modifiers have an icon, and the settings can only be set through metrics.

## MusicWheel

Cannot be created from lua, but can be grabbed from the TopScreen.

The wheel used to select songs in ScreenSelectMusic or courses in ScreenSelectCourse.

## NoteColumnRenderer

Cannot be created from lua, but can be grabbed from NoteField.

This is a dedicated actor for a column in the notefield, and can be treated like any other actor.

## NoteField

Cannot be created from lua, but resides in Player.

Contains the receptors and notes as seen in gameplay.

## OptionRow

Cannot be created from lua.

Often seen in options screens, this actor allows for picking and choosing various options for a given setting.

## PaneDisplay

Shows the number of steps, jumps, holds, rolls, mines, hands, lifts, fakes, the machine profile's highscore & name and the current profile's high score for a given chart.

Settings must be defined through Metrics.

## PercentageDisplay

Shows a player's current percentage (Actual DancePoints divided by Possible DancePoints).

## Player

Cannot be created from lua, but is part of ScreenGameplay

The entirety of the playfield. Holds the judgment, hold judgments, combo and the NoteField.

## Quad

An arbitrary rectangle. Acts like a Sprite with a blank texture.

```lua
-- Generate a 64 x 64 rectangle on the center of the screen, and color it Yellow.
Def.Quad{
	OnCommand=function(self)
		self:zoomto( 64,64 ):diffuse( Color.Yellow )
		:xy( SCREEN_CENTER_X, SCREEN_CENTER_Y )
	end
}
```

## RollingNumbers

A BitmapText with a number that transitions to the target number.

Requires metrics for it to be loaded before anything else is done with it.

```lua
Def.RollingNumbers{
	-- Load the font that the numbers will be rendered with.
	File = THEME:GetPathF("Common", "Normal"),
	InitCommand=function(self)
		-- Loads the MetricsGroup to use it's settings on.
		self:Load("RollingNumbers")

		-- To update the value, use the targetnumber function.
		self:targetnumber(300)
	end
}
```
## ScoreDisplayAliveTime

A BitmapText-type actor that shows how long a player has survived through the stage.

## ScoreDisplayCalories

A RollingNumbers-type actor that shows how many calories a player has burned through the stage.

## Screen

Can't be defined in lua, but can be defined in a theme's metrics.

A screen the theme can go to. There are screens for gameplay, selecting music, pre-gameplay, etc. Each screen has a background, underlay, overlay and decorations that can be used.

<!--TODO: Do I note down the other ScreenTypes listed in luadocs?-->

## SongBPMDisplay

Shows the current BPM during gameplay. Does not appear to take split timing into account.

## SongMeterDisplay

A MeterDisplay that shows the current position of the song being played.


```lua
Def.SongMeterDisplay{
	InitCommand=function(self)
		-- This generates a 300 x [the height of your Stream texture] that will define the current progress of whatever song is currently being played.
		-- The actor will automatically update progress for the Tip and the Stream.
		self:SetStreamWidth( 300 )
	end,
	-- Both the Stream and Tip are AutoActors, so they can be any actor type.
	Stream=Def.Sprite{ Texture="MyStreamBar" },
	Tip=Def.Sprite{ Texture="MyTip" }
}
```

## Sound

Used to play sound files outside of the common theme sound effects and the simfile's song itself.

Removes the need to use `SOUND:PlayOnce()`, as it allows for pre-loading the sound file at the start instead of loading (possibly many times) and playing the sound mid-screen.

```lua
Def.Sound{
	-- Load the audio called MySound, which is a ogg file in this example.
	File="MySound.ogg",
	-- Lets the audio pane from side to side. Useful for audios that need to play on a specific player side.
	SupportPan=true,
	-- Allows the audio to change rate and pitch.
	SupportRateChanging=false,
	-- this assigns the audio to be an Action, which is a flag for sounds that allows it to be muted by the player,
	-- with the use of the Mute Actions key (Default to "Pause").
	IsAction=true,
	OnCommand=function(self)
		-- When creating the actor, sound will not play automatically, so you need to use the play command
		-- to perform such action.
		self:play()

		-- If the audio has the "(loop)" flag set on its filename, it will loop infinetly. So to top it, use the
		-- appropiate command.
		self:stop()

		-- If the sound needs to be paused on a particular frame, and not to reset, use the pause command.
		self:pause( true ) -- Use false to resume it.

		-- To control actions like volume, you need to access the ActorSound's RageSound, by using the get function.
		local MyRageSound = self:get()

		MyRageSound:volume(0.5) -- Changes volume (0 to 1).
		MyRageSound:pitch( 1.2 ) -- Requires SupportRateChanging to work.
		MyRageSound:speed( 1.4 ) -- Requires SupportRateChanging to work.
	end
}
```

## StepsDisplay

Displays the data for a given chart. Can show difficulty number, description, credit, if it's autogen and steps type.

Currently, all setings are done through metrics.

## StepsDisplayList

Shows the list of steps available for a given song.

The name given determines what metrics group to load from.

```lua
Def.StepsDisplayList {
	Name="StepsDisplayList",
	-- These define the cursor the players will be controlling. The rest of the elements are defined
	-- by the metrics.
	CursorP1=Def.Actor{},
	CursorP2=Def.Actor{},
	CursorP1Frame=Def.Actor{},
	CursorP2Frame=Def.Actor{}
}
```

## TextBanner

A "Banner" that contains the song name, artist and subtitle.

Usually seen in the CourseContentsList of ScreenSelectMusic or the ScrollerItem in ScreenHighScores.

## WheelBase

Cannot be created from lua.

A base class for wheels. Currently, the MusicWheel and RoomWheel inherit from this.

## WheelItemBase

Cannot be created from lua.

A base class for items residing in wheels. MusicWheelItems inherit from this.

## WorkoutGraph

A graph showing the calories burned over time during a workout.
