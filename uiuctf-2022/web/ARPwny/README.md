# 🎯 UIUCTF 2022: AR Pwny

![Contest Date: 30.07.2022](https://img.shields.io/badge/Contest%20Date-30.07.22--01.08.22-lightgrey.svg)
![Solved: During The Contest](https://img.shields.io/badge/Solved-During%20Contest-green.svg)
![Score: 50](https://img.shields.io/badge/Score-50-yellow.svg)
![Difficulty: Beginner](https://img.shields.io/badge/Difficulty-Beginner-brightyellow.svg)

## 🧾 Description
 
<h1 align="center"><img alt="arpwny" src="assets/arpwny.png" width="300px" /> </h1> 
<blockquote>
Welcome to the meataverse! 

http:// ar-pwny-web . chal . uiuc . tf

_author: ian5v_
</blockquote>

<!-- ## Attached Files

- None
-->

## 🍦 Solution TL;DR

* Perform forensics on the **GLB** binary file
* Convert it to **glTF** format and locate the flag segments
* Use a 3D rendering engine debugger to visualize the flag segments i.e QR codes
* Interpret the codes and combine them to obtain the flag


## 🔎 Detailed Solution

### **Step 1 : Web Inspection**
Navigating to the challenge  using a regular MS Edge browser, the webpage is basically a 3D [SIGpwny](https://sigpwny.com/) logo in interactive view.

<h1 align="center"><img alt="arpwny" src="assets/front-page.png" width="400px" /> </h1> 

Switch to mobile device? That's interesting! 

Open **view source** page with <kbd>CTRL + U</kbd> hotkey.

```html
<!-- redacted section -->
<meta name="twitter:card" content="player" />
<meta name="twitter:title" content="hey there ctf player >:o) why dont u solve my challenge :^)" />
<meta name="twitter:image" content="0" />

<!-- redacted section -->
<meta name="twitter:player:stream" content="https://video.twimg.com/ext_tw_video/1553060639530225664/pu/vid/920x720/qT8hHQq3KZUFi7uo.mp4?tag=12" />

<!-- redacted section -->
<script type="module" src="model-viewer.min.js"></script>

<model-viewer style="width: 100%; height: 80%" src="pwny.glb" ar ar-modes="webxr scene-viewer quick-look" seamless-poster shadow-intensity="1" camera-controls enable-pan></model-viewer>

<!-- redacted section -->
```

There are couple of useful info to start with:

* The 3D model is rendered by [model-viewer](https://modelviewer.dev/), an open source web component by Google for web browsers as well as **"Augmented Reality"** environments.

* A fishy video link is available on the meta tags put by author. Let's play it.<h1 align="center"><img alt="arpwny" src="assets/author-vid.png" width="400px" /> </h1> 
The author must be using an AR viewer app! (Challenge title is now making sense) <blockquote> In case you're wondering. an AR viewer app overlays the 3D immersive object on the top of your physical surroundings, which can be navigated through the lens of a smartphone camera.</blockquote> 
* A peculiar _pwny.glb_ file, which should contain the model data for rendering. A quick web search shows that the **.glb** file format carries binary information about 3D scenes, models, lighting, materials and node hierarchy.

 
### **Step 2 : What's in the Blackbox?**

Before proceeding into forensics of the **.glb** data, let's follow the challenge instructions first.

- First check whether acting like a mobile client using a PC would do the trick. Open _Edge Devtools_ (<kbd>CTRL+SHIFT+I</kbd>), toggle device emulator ON and refresh the page. <h1 align="center"><img alt="arpwny" src="assets/devtool-mobile.png" width="300px" /> </h1> There's no button on the bottom right for interaction, so this approach won't be useful.

- Let's pick up a real mobile device. Any recent smartphone model should have AR compatibility out of the box. <blockquote>For iOS devices, the minimum requirement is _iOS 11_ running on _A9 chip_ (that means any device model after _iPhone 5_ would do the job).</blockquote>Installing an AR viewer app (ex: [Augment](https://www.augment.com/)) and scanning the QR code takes us into the AR experience. Suddenly there's giant virtual pwny on my workstation floor! <h1 align="center"><img alt="arpwny" src="assets/pwny-floor.png" width="400px" /></h1> Searching every nook and corner around the surface don't reveal any clue (note that I'm doing the search assuming the object has **solid fill**). Guess it's the time for plan B - digging up the model data.


### **Step 3 : Whitebox Forensics** 

Running a quick string check with `strings` tool gives following results:
```json
bijoy@kali:~/Desktop/uiuctf_22/web/pwny$ strings pwny.glb
glTF
JSON{"asset":{"generator":"Khronos glTF Blender I/O v3.2.40","version":"2.0"},"scene":0,"scenes":[{"name":"Scene","nodes":[0,1,2,3,4,5,6,7,8,9,10,11,12]}],"nodes":[{"mesh":0,"name":"flag-firsthalf","rotation":[-0.7071068286895752,-0.7071068286895752,0,3.0908616110991716e-08],"scale":[2.5827651023864746,2.5827651023864746,2.582765579223633],"translation":[2.0828332901000977,-0.5367418527603149,-2.838243007659912]}

# REDACTED REMAINING OUTPUT
``` 
It's clear that the file contains some JSON data (will come in handy), and a web search reveals that it is generated via a [Blender](www.blender.org) add-on called **glTF Importer and Exporter** from _Khronos Group_. 
<br>Digging up a bit more, another format **glTF** (GL Transmission Format) turns out to be a clue. 

Basically, it's a stripped down JSON file that references the associated mesh, animation, and texture data in external files for quick transmission needs. In contrast, our **GLB** format contains the glTF asset (JSON, .bin and images) in a binary blob, just like the output suggested.

Let's open the file in **Visual Studio Code** editor. Almost immediately, it suggests me a marketplace extension called **glTF Tools** to handle the file (and that's why I love VS Code ❤)! <h1 align="center"><img alt="flag1" src="assets/ext-alert.png" width="400px" /> </h1>  <h1 align="center"><img alt="flag1" src="assets/gltf-tools.png" width="300px" /> </h1>

Using it, apply _Import from glb_ operation on the file to convert it to equivalent glTF file (with a .bin file as expected).  

Let's inspect the JSON structure. What do we have here? 
    <h1 align="center"><img alt="flag1" src="assets/flag1.png" width="300px" /> </h1> <br>
    <h1 align="center"><img alt="flag2" src="assets/flag2.png" width="300px" /> </h1> 
    
Looks like the flag is sliced in halves and placed into the model, but the associated data needs to be graphically interpreted for reconstruction. 

### **Step 4 : Lost in Babylon** 

Peeking through the extension [docs](https://github.com/AnalyticalGraphicsInc/gltf-vscode), the `Preview 3D Model` function opens the file up through a selectable rendering engine (namely _Babylon.js, Cesium_, _Filament_ or _Three.js_ ) for display.

> Adding a bit of context, BabylonJS is a powerful open source GUI system for displaying 3D gaming graphics in a web browser via HTML.
<h1 align="center"><img alt="flag1" src="assets/preview.png" width="400px" /> </h1> 

Switching into the glTF **outline view** reveals the tree structure of the scene nodes. 

<h1 align="center"><img alt="flag1" src="assets/tree.png" width="400px" /> </h1> 

Hierarchically, the data structure is arranged in the following order:

> **scene > node > mesh > primitive > vertices & triangles** 

So intuitively, being able to render only the _flag-firsthalf_ and _flag-secondhalf_ nodes minus the rest of the scene should bring the flag to light.

> BabylonJS comes with a dedicated visual debugging tool called **Inspector** to show model wireframes, normal vectors, texture channels and allows scene exploration. 

Cool! Just the thing we need. Toggle the _Inspector_ icon on top right.

<h1 align="center"><img alt="flag1" src="assets/inspector.png" width="500px" /> </h1> 

Expand the nodes until flag primitives are visible. Then use the _show/hide_ toggle to remove all other non-flag nodes one by one and voila!

<h1 align="center"><img alt="flag1" src="assets/partial-flag.png" width="400px" /> </h1> 

The model was not solid after all, rather a hollow container for QR codes! Those have been hiding inside the object all along!

<h1 align="center"><img alt="flag1" src="assets/flags.png" width="400px" /> </h1> 

Scan and decode the two QR codes, and  the flag slices are obtained. 

Slice 1: `uiuctf{welcome_2_the_meataverse_`

Slice 2: `erm_i_meant_pwnyverse}`

Combining both, the final flag is obtained. ✌

## ⛳ Flag

```
uiuctf{welcome_2_the_meataverse_erm_i_meant_pwnyverse}
```

## 🧲 Alternative Solutions

Since the model was hollow, looking carefully inside the object using an AR app in **STEP-2** would have given away the QR codes. (and save a lot of time ⌚)
<h1 align="center"><img alt="flag1" src="assets/ar_flag.png" width="250px" /> </h1> My teammate solved it via this blackbox technique. 

## 🙏 Acknowledgements
Regardless of approach, the challenge is pretty fun to solve! Credit goes to [ian5v](https://twitter.com/ian5v) from SIGpwny.