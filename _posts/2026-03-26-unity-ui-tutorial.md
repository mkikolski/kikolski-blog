---
layout: post
title: "Introduction to UI in Unity"
subtitle: "This one is actually a tutorial - trust me"
date: 2026-03-26 23:00:00 +0200
tags: [gamedev, ui, unity, coding]
excerpt: "This is a tutorial I've done for students of medical informatics for the VR Game Development course..."
---

This is a tutorial I've done for students of medical informatics for the VR Game Development course. Even though the course name states it's VR programming, this tutorial is made for regular PC games. I'll try to introduce you to some basics of UI design in general and a process I've been using with some success for the projects I've been developing. So, here it is:

## UI Examples

When the first video games were created, the UI was very limited, as the games weren't exactly sophisticated. The player was mostly interested in the score, high score and the number of lives (or credits on arcade machines) left. The good example of that is the original Pac-Man ([image source](https://en.wikipedia.org/wiki/Pac-Man)):

![Pac-Man UI](/assets/images/12.png)

As the games started getting more complex, the UI evolved too. The progress brought more and more data to display, such as health, money, and the current time. The example of that is the original Grand Theft Auto ([image source](https://www.gry-online.pl/gry/grand-theft-auto/zf15b7)):

![GTA UI](/assets/images/13.png)

After some time, the UI started evolving in terms of aesthetics too. The developers realized that the UI can be a part of the game world and can contribute to the immersion. The example of that is the original Dead Space, which is set in space and the UI is designed to look like it's projected onto the character's suit ([image source](https://www.gameuidatabase.com/index.php?&set=1&sort=2&tag=80&series=101&scrn=100)):

![Dead Space UI](/assets/images/14.png)

This approach in design is called diegetic UI and its main goal is to immerse the player in the game world. The opposite of diegetic UI is non-diegetic UI, which is not a part of the game world and is only visible to the player. Does the game need to use the diegetic UI? Definitely not. The diegetic UI isn't always the best choice - it is harder to make it work both technically and artistically. You usually need a really good idea to make it work and not be confusing for the player. After all, the main goal of the UI is to present information to the player in a clear and concise way and even perfect diegetic UI won't make a bad game good (but a bad UI can make a good game unplayable).

I'll show you some other examples of UI design in games to have you inspired for your own projects.

### Deus Ex (2000) ([image source](https://film.org.pl/a/deus-ex-filmowe-gry-komputerowe-115538))

![Deus Ex UI](/assets/images/15.png)

One of my favourite games of all time and perfectly good when it comes to UI design. All the information is presented to the player without cluttering the screen, the shortcuts are clearly visible and the overall design is consistent with the game's atmosphere.

### Don't Starve Together (2014) ([image source](https://hot.game/en/game/Dont-Starve-Together))

![Don't Starve Together UI](/assets/images/16.png)

The UI fits the atmosphere perfectly and might seem lacking a bit of information at first glance, but that's where its beauty lies. It embraces the game's idea of learning to play the game by yourself and discovering things on your own - it deprives the player of exact numbers and lets them infer the state of the game from the visuals.

### Lethal Company (2023) ([image source](https://store.steampowered.com/app/1966720/Lethal_Company?l=polish&t=1701734400))

![Lethal Company UI](/assets/images/17.png)

Modern example of diegetic UI at its best.

## Creating UI in Unity

To create UI in Unity, we need to use the **UI** system. It's a separate system from the 3D objects and it's used to create the user interface of the game. The UI system is based on **Canvas** objects. A Canvas is a container for UI elements. It's a 2D plane that is projected onto the screen. I've created a simple scene in Unity that you'll be using during this lab. The scene is specially prepared to bring the game a certain vibe that you'll have to match with your UI design. To explain the mechanics of UI in Unity, I'll use the example of a health bar, which is a common UI element in games.

The basic scene already contains a Canvas which is a place for all the UI elements. I've placed a small reticle in the center of the screen that will help to see where the player is aiming.

![Basic scene with reticle](/assets/images/18.png)

To start I've added `GameObject > UI (Canvas) > Image` to the scene. This has created the main container for our UI elements with a default image of a white square inside. I named my Canvas `UIPlayerCamera`, changed its UI Scale Mode to `Scale With Screen Size`, set Screen Match Mode to `Expand` and set the reference resolution to 1920x1080, as it's the most common resolution for PC games. The Image got renamed to `ReticleCenter`. I decided to leave the reticle as a square but I changed its size and color. Additionally I duplicated the reticle image and used it to create a frame around the center reticle for improved contrast. Please note that the elements on the top of the hierarchy are displayed as the bottom layer - that's why the `ReticleCenter` is below the `ReticleOutline`.

First step I took for designing my health bar was taking a screenshot of the game and importing it to GIMP. This step is important for real UI design as it lets you plan the whole layout easily. Moreover, if you're aiming for some sort of more complex shapes in your UI, you can create them in GIMP and then import them to Unity as images which can be used as UI elements. After a while of tinkering, I came up with this design:

![Health bar design](/assets/images/UI-1.png)

*Spoiler: I didn't end up implementing the ECG animation in the final version of the UI. It was a nice idea but it was too much work to implement it properly for the sake of this tutorial.*

To host our UI elements responsible for the health bar, in the main canvas I've created a new empty object called `HPContainer`.

![HPContainer](/assets/images/UI-2.png)

You may notice that the Empty object already received a **Rect Transform** component. This is the equivalent of the **Transform** component for UI elements. It defines the position, size and rotation of the UI element in the 2D space of the Canvas. I used this component to set the anchor and positioning of the `HPContainer` to the center of the full stretch with alignment to the middle (bottom-rightmost option in the anchor dropdown menu).

Now back to the design from GIMP. I've exported two images - `UI-Frame.png` and `UI-BG.png` that will help me build the health bar base. If you wish to follow my steps, you should export them as white images with transparent background. That will allow us to color them later in Unity. As you can notice, the `UI-Frame.png` has a slight glow to it which was achieved by using the Drop Shadow effect in GIMP with white shadow color. When importing the images to Unity, make sure to set their **Texture Type** to **Sprite (2D and UI)**. Also make sure to set **Sprite Mode** to **Single** and Alpha Source to **Input Texture Alpha**.

![Import settings](/assets/images/UI-4.png)

You will see the small arrows appearing on the left of the imported assets after applying the new import settings. Clicking on them will reveal the generated sprites. 

In our `HPContainer` object we will add two Image objects. The first one will be responsible for the glowing frame of the health bar panel and the second one for the background. I named them `Frame` and `Background` respectively. If you exported the sprites from GIMP in 1920x1080 resolution, you can set them up as shown below:

![Frame and Background setup](/assets/images/UI-5.png)

You can also use the color selector to finally add the white components some color. After setting it up for both of the images I needed to add the health bar itself. I added a new Image object alongside `Frame` and `Background` and named it `BarBase`.

![BarBase setup](/assets/images/UI-6.png)

But wait, this doesn't look like a health bar yet. I've decided to anchor and move it to the centre-bottom of the screen and then I played around with the size until it looked right.

![BarBase setup](/assets/images/UI-7.png)
![BarBase setup](/assets/images/UI-8.png)

This way of manipulating the UI elements is a way to go if you're aiming for simpler shapes in your UI - no need to export images from GIMP, just scaling and moving the basic rectangles and circles will do the trick. You can also try searching for some free UI assets on Unity Asset Store for your further projects.

Generally there are two ways of making a progress bar (or any other dynamic bar for that matter) in Unity. The first one is to use the Image component with the **Filled** fill type. You won't be dealing with two separate images but rather one image that will be filled with color. Unfortunately, I'm stubborn and when creating the tutorial, I've made up the second way - using two images, one for the background and one for the foreground. To proceed my way, you'll need to add a new Image as a child of `BarBase` and name it `BarFill`. The `BarFill` image should be anchored to the left-center of the `BarBase` and its width will be controlled by the HP of the player.

![BarFill setup](/assets/images/UI-9.png)

Continuing with the design of the health bar, we now need to add the text elements. Find yourself a font you like and import it to the project. You can download fonts from sites like [Google Fonts](https://fonts.google.com/). After importing the font, you'll need to create Font Asset. You can access this option in `Window > TextMeshPro > Font Asset Creator`. You'll need to select your imported font:

![Font Asset Creator](/assets/images/UI-10.png)

After creating the font asset, we can actually add the text elements to the UI. On the way you might get prompted to import the TextMeshPro Essentials. Do it, it's the current way of rendering the text in Unity. I've added a single element as a child of `HPContainer` and named it `HPText`. I've set its anchor to the center-bottom of the `HPContainer` and played around with the size and position until it looked right. Then I selected the font asset I've just created and set the text to "100 HP".

![HPText setup](/assets/images/UI-11.png)

Further steps included setting the color and for my case making sure that the text was centered.

![HPText setup](/assets/images/UI-12.png)

Now, we've created the static UI elements. The next step is to make them dynamic. Beside dynamic health bar and the text, I wanted to implement the idea taken from Left 4 Dead and probably many other games - the bar will be colored differently depending on the amount of HP the player has. We will start with attaching a MonoBehavior script to the `HPContainer` object. I named it `HealthUI`. The HP is updated in the `PlayerController` script, so we'll need to get a reference to it in the `HealthUI` script. To update certain elements of the UI, we'll need to get references to them as well. After setting up the fields I got:

![HealthUI setup](/assets/images/UI-14.png)

The color fields are the addition I mentioned earlier - I've set them up according to my idea and added `SerializeField` attribute to them so they're customizable from the Inspector. I've created one method that will handle all the operations as they're quite easy and I didn't want to clutter the code with multiple methods.

![HealthUI setup](/assets/images/UI-15.png)

We obtain health values from the `PlayerController` script and use them to update the health bar and the text. The health bar is updated by changing the width of the `BarFill` image. The text is updated by changing the text of the `HPText` image. The only tricky part here was the line 38 and it's all due to my design choice of using two images for the health bar instead of one. I needed to dive into the Unity documentation to figure out how to do it properly and came up with the method `SetSizeWithCurrentAnchors`. It requires us to specify the axis along which we want to scale the image (in this case horizontal) and the size we want to scale it to. I used the percentage of maximum health as the scaling factor and hardcoded the full size of the HP Bar. Of course, the size of the bar could have been read from the object properties and I strongly encourage you to do it in your projects.

The thresholds in the color setting part of the script could have also been exposed to the Inspector but for demonstration purposes they're hardcoded. 

The last thing we need to do is to set the correct references in the inspector:

![HealthUI setup](/assets/images/UI-13.png)

And that's it! Now we have a working health bar that updates dynamically and changes color depending on the health of the player.

## Want to try this on your own?

Here's the part that contains the assignments for the students. For those who'd like to test this on their own, here's the repo link: [GitHub - AbandonedSpaceStation](https://github.com/mkikolski/AbandonedSpaceStation) which was built using [a great pack of assets by Sickhead Games](https://assetstore.unity.com/packages/3d/environments/sci-fi/sci-fi-construction-kit-modular-159280).

But this is not all. The game has interaction system implemented for some of the objects in the scene. The code is in the project and you should take a look at it to understand how it works, because this will be a thing you'll need to implement in your assignment.

## Required UI Elements

1. Health Bar
2. Interaction Prompt - each of the interactions should display a prompt if player is looking at the object from certain distance. Checking the distance is already implemented and it prints to the debug log. You'll need to implement the UI part.
3. Interaction Timer - if we detect that player is holding the interaction button, we should display a timer showing the progress of the interaction. It can be a circle or a bar.
4. Interaction notifications - when the interaction is completed, we should display a notification saying that the interaction was completed. The interaction system already has them implemented and printed to the debug log.
5. Death Screen - when the player dies, we should display a death screen with the option to restart the game.

### Version control

For this assignment you'll need to clone the repository and import the project to your Unity Hub. In the `Projects` tab of Unity Hub, click on `Add` dropdown and then `Add project from disk` and select the folder where you cloned the repository. Unity will automatically fetch the required packages and you'll be able to work on the project. After loading the project, you'll need to go into `Scenes` folder and open the `SampleScene`. There also might be a need to regenerate the scene lighting data by going to `Window > Rendering > Lighting` and clicking on `Generate Lighting` button.

Further into this course, you'll be asked to create your own repositories and submit your projects as a link to your repository. The process of creating them is quite simple as long as your assets are not too large - if they are, you'll need to use Git LFS (Large File Storage). This project didn't need this, but it's good to be aware of it.

When you want to add a repository to the project, you can navigate to the root folder of the project in command line and run `git init` command. The crucial step here is to use well designed `.gitignore` file - I've used example from [GitHub](https://github.com/github/gitignore/blob/main/Unity.gitignore). Then you can add the remote repository using `git remote add origin <repository_url>` command. Finally, you can push the project to the repository using `git push -u origin main` command.
