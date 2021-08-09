# Binding-of-Isaac-Item-ID
Examines a picture containing an item in the Binding of Isaac, compares it against existing images, and returns the name of the item and its effects.

# Identifying an Item 

## Problem
If you play the Binding of Isaac, you'll most likely have faced the challenge of trying to figure out what an item does. Maybe it's a new item. Maybe it's one you've seen before but you can't quite remember its effects. If you're playing on Steam, you can download a mod that helps you out. If you're playing on console, you're out of luck and need to instead sift through the hundreds of items listed on the Wiki which is time consuming and not very fun. 

## Solution
* Build an image classifier that can take in a picture of the game (including some item that we want to ID)
* Classify the unknown item and return its effects

## Process
* Gather images of items
* Compute item features
* Compare a new image to the item images and find the most similar item

# Gathering Images

## Web Scraping

A treasure trove of information exists on the [Binding of Isaac Wiki](https://bindingofisaacrebirth.fandom.com) including a [list of all of the items](https://bindingofisaacrebirth.fandom.com/wiki/Items), their images, and their descriptions. We can use the ImageExtraction file to scrape all of the images and item effects and store them locally.

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/IsaacImg1.png)
![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/IsaacImg2.png)
![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/IsaacImg3.png)
![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/IsaacImg4.png)


# Feature Extraction

The cv2 library contains some useful tools that can help us compute information about the images. We can use ORB which is [a fast robust local feature detector](https://en.wikipedia.org/wiki/Oriented_FAST_and_rotated_BRIEF) to extract information about an image and then store it locally for each of the images we've downloaded. When we stumble upon an unknown item, we can take a picture of the game, extract the features from the new image, and then compare these features with the features we saved and attempt to find a match. We can use the ImageFeatureExtraction file to compute these features.

# Image Comparison

If we want to compare a picture of an item we've discovered by playing the game to the images we've downloaded, all we need to do is compute the features of the new image and compare them to the features of the images we computed in the last step. Helpfully, we are able to see the matched features between the item pictures and the image we want to ID. Let's look at two different items and the top 10 feature matches to see what's happening.
Note: The lines between the two images are depicting the matching image features. The small, colored circles are matching features but we are not displaying the match (the lines between the two images are removed).

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/IsaacImageComp1.png)
![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/IsaacImageComp2.png)

The first item is the correct one, the Halo of Flies, while the second is an incorrect item, Capricorn. So, how do we select the correct item? As humans, our brains tell us which is the most similar item. Unfortunately computers don't have that same intuition. To select an item programmatically we need some sort of metric. We could use a metric like the total number of matching features. We would just count up all of those little colorful circles for each image and then choose the image with the highest number. However, as we can see from the second image, there are quite a few incorrect matches. Using a metric like the total number of feature matches may give us bad predictions if there are more incorrect matches than correct matches. For example, there may be 100 incorrect matches in the second image and only 95 correct matches in the first image. 

Looking at the first picture, it seems that the top 10 matches cluster closely together around the item in the test image. The matches in the second, incorrect image are spread out over the entire image. Based on this, we could use the variance of pixel distances for the top 10 matches to predict which is the correct item. That is to say, when the top matching features are close together in the test image, then we have a higher chance of the item being correct. If the features we find are scattered throughout the image then the likelihood of a match goes down.

Once this metric is applied we can return the top 5 matching items and descriptions and see how well our metric works.

# Results

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/Distant_Admiration.png)

Item: Distant_Admiration

Effects:
* Spawns a red attack fly that circles around Isaac, and deals 5 damage per tick or 75 damage per second on contact.
* The fly circles farther out than regular orbitals, but closer than Forever Alone.
* This item belongs to the Beelzebub set. Collecting three items from this set will transform Isaac into a giant humanoid fly.

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/Halo_of_Flies_sprite.png)

Item: Halo_of_Flies

Effects:
* Spawns two Pretty Flies that block enemy shots.
    * Pretty flies kill some fly enemies upon contact.
    * The orbital limit of three flies cannot be surpassed with this item, reducing its effectiveness if Isaac has two or more orbitals.
* This item belongs to the Beelzebub set. Collecting three items from this set will transform Isaac into a giant humanoid fly.

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/Vengeful_Spirit.png)

Item: Vengeful_Spirit

Effects:
* Spawns a red wisp when Isaac takes damage that fires spectral tears and deal contact damage to enemies (but does not block projectiles). These red wisps last for the entire floor, and Isaac can have up to 6 red wisps at a time.
    * Wisps deal 3 + 0.1 * (current_floor - 1) damage per tear, and fire tears once per second.
    * Wisps deal contact damage equal to 0.5x Isaacs's damage per tick (15 ticks per second = 7.5x Isaac's damage).

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/Skatole_sprite.png)

Item: Skatole

Effects:
* This item causes many Fly enemies to alter their behavior toward Isaac.
    * The following flies transform into a neutral normal Fly: Attack Fly, Dart Fly, Eternal Fly, Ring Fly, Army Fly, Large Attack Fly, and Swarm Fly. 
    * The following flies move slower but will still deal contact damage: Boom Fly, Red Boom Fly, and Drowned Boom Fly.
    * The following flies move slower and deal no contact damage: Moter, Level 2 Fly, Pooter, Super Pooter, Bulb, Mama Fly, Bloodfly, Ultra Famine Fly, and Ultra Pestilence Fly.
        * Pooters will not fire at Isaac, Moters will not split into Attack Flies, and Bulbs will not drain active item charges if touched.
    * The following flies move slower and deal no contact damage, but can still damage Isaac through other means: Full Fly, Boom Fly, Red Boom Fly, Drowned Boom Fly, Sucker, Soul Sucker, Spit, Ink, Willo, Level 2 Willo, Dragon Fly, Dragon Fly X, Bone Fly, Sick Boom Fly, Tainted Boom Fly, and Tainted Sucker.
        * Bone Flies will still throw bones at Isaac, and Willos and Level 2 Willos will still shoot.
    * The following flies are completely unaffected, retaining their normal attacks and speed: Large Attack Fly, Swarm Fly, Hush Fly, Sucker, Soul Sucker, and Spit.
* This item belongs to the Beelzebub set. Collecting three items from this set will transform Isaac into a giant humanoid fly.
* This item belongs to the Oh Crap set. Collecting three items from this set will transform Isaac into a walking pile of poop.

![](https://github.com/Vinnie-Singleton/Binding-of-Isaac-Item-ID/blob/main/pics/Forever_Alone_sprite.png)

Item: Forever_Alone

Effects:
* Spawns a blue attack fly that orbits a long distance from Isaac and deals 2 damage per tick or 30 damage per second on contact.
* This item belongs to the Beelzebub set. Collecting three items from this set will transform Isaac into a giant humanoid fly.

# Future Work

In this example the second prediction was the correct item. While it's still not perfect, it seems that this methodology can give an indication of which item is in a picture without having to scan through hundreds of images manually to find a match. There is still some variability in results and in some situations the suggested items are not remotely close to the correct item. 

Future development may push towards an AI solution and use a convolutional neural network to determine which item is in an image. The issue here is with data generation. Using the debug console and a few Lua commands it should be possible to generate different floor layouts and take screenshots for all of the items in game so that there is enough variability that a model can properly generalize and correctly classify items.

