# Stem Image Dimensions

There are three possible stem and image configurations in LP. Max and min height and witdth for the image container (dependent on the user's screen or window size) are given for each configuration. The images are sized proportionally based on the width of their container - a smaller width will make the image height shrink proportionally. Images of each layout are attached

## 1. Alongside Stem, Large Screen

This configuration is used by all item types with stem images.

Max: 306w, 238h
Min: 222w, 238h

At small screens / browser sizes, the stem and image are reconfigured into one of the following:

## 2. Under Stem, Small Screen

Multiple choice item types use this configuration.

Single size: Technically this is not limited by our system, but for the layout to remain understandable, and for the image to be readable at larger screen sizes when it switches to configuration 1, the recommended size limits are 360w, 270h.

## 3. Alongside Stem, small screen

PSP and all next gen item types use this configuration. Because of the increased size and complexity of the answering area, the image cannot move below the stem; otherwise, it would prevent the stem and the answer area from being in the same screen at target browser / screen sizes.

Max: Transitions smoothly from configuration 1
Min: 192w, 160h

It's important to note that there is not a new image inserted during the transition from 1-2, or 1-3. The same image is resized. As such, the smallest dimensions for an image should be used as the basis for readability, but the image should be produced and uploaded at the largest size of use, preferably 1.5x the size of the planned maximum dimension to accommodate high ppi screens on modern laptops and devices. For instance, a Stem image for a multiple choice would be produced so when resized at 222px width any text or important graphics are readable, but would be uploaded at approx 500px width so that it displays well on high resolution devices. 

---

![Matching Table Reference Guide](../images/MatchingTableRef.png)

- Please note that the display height for all draggable objects is capped at 90px, with a variable width.
