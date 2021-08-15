---
title: Wearable technology for music performance - A hands-free talkbox
description: Gesture recognition controls and a multi-touch configuration interface to make you feel like a rockstar
date: 2021-04-11
tags:
  - music
  - random
layout: layouts/post.njk
eleventyExcludeFromCollections: true
---


## Motivation and Concept
I have recently been interested in the talkbox, a musical instrument which allows the player to shape the sound of a sawtooth tone using their mouth. The instrument consists of a tone generator (usually a compression driver) which is attached to a piece of rubber tubing. The player places this tube inside their mouth and grips it with their back teeth. The talkbox is surprisingly easy to master; one can mouth words as they normally do and instantly enjoy a cool robotic singing voice.

<div class="captionedimg">
<img src="/img/peter-frampton-show-me-the-way-1975--735x413.jpg">
Peter Frampton is often credited with popularizing the talkbox. His device used analogue tube amplifiers driven by an electric guitar to produce a distinctive and mesmerizing effect.
</div>

I wanted to build an extremely accessible talkbox that met the following conditions:
1. It should not require knowledge of any musical instruments, modern talkboxes require users to be proficient keyboardists or know how to play the guitar.
2. It must be controllable hands-free, leaving one's hands available for other things.
3. Ease of use is important, so the talkbox should be easy to configure and set up.

To meet these requirements, I am creating a mobile talkbox which uses a bespoke app and Nintendo Switch joycons to enable gesture controls.

## Materials
To make my wearable hands-free talkbox, I used the following materials:
- Two Nintendo Switch Joycons (left and right)
- 6 ft. 1/3rd inch plastic tubing
- Loctite Superglue (the best brand of super glue)
- Stretchy latex gloves
- Needle, thread, and fabric
- A good pair of scissors
- A smartphone

<div class="captionedimg">
<img src="/img/2021-04-12_03-19-18.jpg" style="max-height: 400px;"><br>
The basic materials
</div>

Originally, I attempted to use my Arduino Uno device (cutting out the need for a smartphone), but found the HC-05 Bluetooth module I ordered was incapable of interfacing with Joycons due to their usage of Bluetooth 4 features 😞. This was an unfortunate setback; however, most people have smartphones already!

You'll also need a computer running Linux or macOS to (easily) use the libraries for capturing Joycon input. This article will only document installation instructions for Ubuntu 20.04 LTS.


## Construction

1. Cut fingers off of latex glove.

2. Cut 10 inch tube from the spool, set aside.

3. Cut a small hole in the reverse end of the latex glove finger, insert plastic tubing.

4. Using Loctite super glue, securely attach the latex to the tube forming a weak seal.
<div class="captionedimg">
<img src="/img/2021-04-12_03-21-27.jpg" style="max-height: 300px"><br>
Your tube-glove finger attachment should look like this
</div>

5. Stretch product over smartphone


6. Sew or attach pouch to outside of shirt to hold smartphone.
<div class="captionedimg">
<img src="/img/2021-04-12_03-25-18.jpg" style="max-height: 350px"><br>
The tone generator and attached tube in card stock pocket.
</div>


7. Create wrist straps for the Joycons.
<div class="captionedimg">
<img src="/img/2021-04-19_03-13-57.jpg" style="max-height: 350px;">
</div>


Now we've got everything physical we need to move on!

## Programming implementation
This project requires a variety of tools in multiple programming languages. First, we'll be creating a tone generator in JavaScript as well as a user interface to control it. This will allow us to test our concept prior to adding motion controls and lay the groundwork for the rest of the project. After that, we'll be writing a Bluetooth receiver in C++ using the HIDAPI which captures data from the Joycons and forwards instructions to the tone generator running on the smartphone.

### Building our user interface
First, let's create a grid of notes using CSS and HTML. First, we'll need to define a flexbox to contain our notes and distribute them evenly. Next, we need to create 18 div containers, one for each note. After doing so, our HTML page will look like this.
```html
<!DOCTYPE html>
<html>
<head>
    <title>
        Multi-touch tone Generator
    </title>
    <meta charset="utf-8">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <style>
        body {
            -webkit-user-select: none;
            user-select: none;
            margin-right: 50px;
            touch-action: pan-y;
            background-color: black;
        }
        .notecontainer {
            display: flex;
            flex-direction: row;
            justify-content: space-evenly;
            flex-wrap: wrap;
        }
        .notebox {
            width: 15%;
            flex-grow: 1;
            height: 100px;
            background-color: black;
            border: 1px solid white;
            color: white;
            font-size: 25px;
        }
        #note-0 {
            background-color: #00FEFF;
        }

        #note-1 {
            background-color: #00C8FF;
        }

        #note-2 {
            background-color: #009BDF;
        }

        #note-3 {
            background-color: #005BFF;
        }

        #note-4 {
            background-color: #0024FF;
        }

        #note-5 {
            background-color: #001495;
        }


        #note-6 {
            background-color: #2a2d34;
        }

        #note-7 {
            background-color: #00dc0b;
        }

        #note-8 {
            background-color: #f26430;
        }

        #note-9 {
            background-color: #6761a8;
        }

        #note-10 {
            background-color: #009b72;
        }

        #note-11 {
            background-color: silver;
        }
    </style>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1,user-scalable=no" />
</head>
<body>
    <div class="notecontainer">
        <div class="notebox" id='note-0'></div>
        <div class="notebox" id="note-1"></div>
        <div class="notebox" id="note-2"></div>
        <div class="notebox" id="note-3"></div>
        <div class="notebox" id="note-4"></div>
        <div class="notebox" id="note-5"> </div>
        <div class="notebox" id="note-6"></div>
        <div class="notebox" id="note-7"> </div>
        <div class="notebox" id="note-8"></div>
        <div class="notebox" id="note-9"></div>
        <div class="notebox" id="note-10"></div>
        <div class="notebox" id="note-11"></div>
        <div class="notebox" id="note-12"></div>
        <div class="notebox" id="note-13"></div>
        <div class="notebox" id="note-14"></div>
        <div class="notebox" id="note-15"></div>
        <div class="notebox" id="note-16"></div>
        <div class="notebox" id="note-17"></div>
    </div>

</body>

</html>
```

Next, let's create a script tag and create a variable that stores the frequency values of every musical note. We'll need this later. Add this script tag inside your `<body>` tags.
```html
<script>
    var notes={C0:16.35,"C#0":17.32,Db0:17.32,D0:18.35,"D#0":19.45,Eb0:19.45,E0:20.6,F0:21.83,"F#0":23.12,Gb0:23.12,G0:24.5,"G#0":25.96,Ab0:25.96,A0:27.5,"A#0":29.14,Bb0:29.14,B0:30.87,C1:32.7,"C#1":34.65,Db1:34.65,D1:36.71,"D#1":38.89,Eb1:38.89,E1:41.2,F1:43.65,"F#1":46.25,Gb1:46.25,G1:49,"G#1":51.91,Ab1:51.91,A1:55,"A#1":58.27,Bb1:58.27,B1:61.74,C2:65.41,"C#2":69.3,Db2:69.3,D2:73.42,"D#2":77.78,Eb2:77.78,E2:82.41,F2:87.31,"F#2":92.5,Gb2:92.5,G2:98,"G#2":103.83,Ab2:103.83,A2:110,"A#2":116.54,Bb2:116.54,B2:123.47,C3:130.81,"C#3":138.59,Db3:138.59,D3:146.83,"D#3":155.56,Eb3:155.56,E3:164.81,F3:174.61,"F#3":185,Gb3:185,G3:196,"G#3":207.65,Ab3:207.65,A3:220,"A#3":233.08,Bb3:233.08,B3:246.94,C4:261.63,"C#4":277.18,Db4:277.18,D4:293.66,"D#4":311.13,Eb4:311.13,E4:329.63,F4:349.23,"F#4":369.99,Gb4:369.99,G4:392,"G#4":415.3,Ab4:415.3,A4:440,"A#4":466.16,Bb4:466.16,B4:493.88,C5:523.25,"C#5":554.37,Db5:554.37,D5:587.33,"D#5":622.25,Eb5:622.25,E5:659.26,F5:698.46,"F#5":739.99,Gb5:739.99,G5:783.99,"G#5":830.61,Ab5:830.61,A5:880,"A#5":932.33,Bb5:932.33,B5:987.77,C6:1046.5,"C#6":1108.73,Db6:1108.73,D6:1174.66,"D#6":1244.51,Eb6:1244.51,E6:1318.51,F6:1396.91,"F#6":1479.98,Gb6:1479.98,G6:1567.98,"G#6":1661.22,Ab6:1661.22,A6:1760,"A#6":1864.66,Bb6:1864.66,B6:1975.53,C7:2093,"C#7":2217.46,Db7:2217.46,D7:2349.32,"D#7":2489.02,Eb7:2489.02,E7:2637.02,F7:2793.83,"F#7":2959.96,Gb7:2959.96,G7:3135.96,"G#7":3322.44,Ab7:3322.44,A7:3520,"A#7":3729.31,Bb7:3729.31,B7:3951.07,C8:4186.01};
</script>
```

### Creating audio devices
Next, let's use the new HTML5 AudioContext API to create wave generators. We'll need to support multiple notes at once, so we need a system that creates multiple oscillator objects. Let's create a reference an AudioContext and an oscillators object to store our frequency generators:
```javascript
window.AudioContext = window.AudioContext || window.webkitAudioContext;
var ctx = new AudioContext();
let oscillators = {};
```

### Generating a tone
Now we need a method which can be passed a note ID and generate a tone. Each element of our `oscillators` object must contain two variables: the oscillator itself and a gain manager which controls the volume. The code below does both.
```javascript
function startNote(id) {
    var frq = notes[config[id]];
    var o = null;
    var g = null;
    if (id in oscillators) {
        // we already have an oscillator for this note id
        // store a reference to in in o and g
        o = oscillators[id][0];
        g = oscillators[id][1]
    } else {
        // we need to create a new oscillator
        let _o = ctx.createOscillator();
        _o.type = 'sawtooth';
        // and gain manager
        let _g = ctx.createGain();
        _g.gain.value = 0;
        _o.connect(_g);
        _g.connect(ctx.destination);
        // let's store our newly created oscillator in the oscillators object
        // the "false" means this oscillator has not been started.
        /* this is necessary because we cannot start our oscillators until the user touches
        something on the page. This restriction is imposed by mobile browers to prevent badly
        behaved pages from making noise without permission */
        oscillators[id] = [_o, _g, false];
        o = _o;
        g = _g;
    }

    o.type = 'sawtooth';
    if (frq) {
        if (!oscillators[id][2]) {
            o.start();
            oscillators[id][2] = true;
        }
        o.frequency.value = frq;
        g.gain.value = 1;
    }
}
```

The function to stop a note is much simpler:
```js
function stopNote(id) {
    oscillators[id][1].gain.value = 0;
}
```


### Creating and sorting the tone palette
Our application accepts configuration from an array of strings in the standard note-octave format. You could build a UI to make this configurable by the user, but that will not be covered in this tutorial. Here's an example config array:
```javascript
const config = ['F#2', 'G#2', 'F#3', 'A2', 'A3', 'C#4', 'B3', 'G#3', 'C#3', 'B2', 'E3', 'D3', 'D#3', 'F#4', 'D#4', 'B4', 'A4'];
```
We want our notes to be ordered correctly, but you probably noticed that **this array is not correctly sorted by tone.** So, let's write a [comparison function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) to sort these strings correctly by pitch.
```javascript
function sortNotes(a, b) {
    // the numeric value of the note in the octave, 0-6
    let a_note = null;
    let b_note = null;
    // the numeric value of the octave
    let a_octave = null;
    let b_octave = null;
    // whether the note is sharp or flat, -1 is flat, +1 is sharp
    let a_sharp_flat = 0;
    let b_sharp_flat = 0;

    // Check for a sharp/flat note and record.
    if (a.length == 3) {
        if (a[1] == '#') {
            a_sharp_flat = 1;
        } else if (_a[1] == 'b') {
            a_sharp_flat = -1;
        }
    }
    if (b.length == 3) {
        if (b[1] == '#') {
            b_sharp_flat = 1;
        } else if (_b[1] == 'b') {
            b_sharp_flat = -1;
        }
    }

    a_note = ['C', 'D', 'E', 'F', 'G', 'A', 'B'].indexOf(a[0]);
    b_note = ['C', 'D', 'E', 'F', 'G', 'A', 'B'].indexOf(b[0]);

    a_octave = a.slice(-1)
    b_octave = b.slice(-1);

    // if the octaves are different, we can immediately return
    if (a_octave > b_octave) {
        return 1;
    } else if (a_octave < b_octave) {
        return -1;
    }

    // we must be in the same octave, compare notes
    if (a_note > b_note) {
        return 1;
    } else if (a_note < b_note) {
        return -1;
    }

    // check sharp/flat
    if (a_sharp_flat > b_sharp_flat) {
        return 1;
    } else if (a_sharp_flat < b_sharp_flat) {
        return -1;
    }
    return 0;
}
```
That's a really big function, but at least it's readable and intuitive. Just for fun, let's [golf](https://codegolf.stackexchange.com/) this function:
```javascript
function sortNotes(n, t) {
    o = ["C", "D", "E", "F", "G", "A", "B"]; z = x => x.slice(-1);
    d = g => o.indexOf(g); l = d(n[0]); s = d(t[0]);
    return z(t) < z(n) ? 1 : z(n) < z(t) ? -1 : s < l ? 1 : l < s ? -1 :
        !(t[1] < n[1]) ? 1 : !(n[1] < t[1]) ? -1 : 0
}
```
That's better! This is the exact same approach; However, it takes advantage of string comparison (`!('#' > 'b') == true`) and the ternary operator. It's unfortunately much less readable. 😛 

### Hooking our interface up
Now, let's create click handlers for our UI. Since they all have the class `notebox` we can loop over them and add the event listeners:
```js
document.querySelectorAll('.notebox').forEach((f) => {
    f.addEventListener('touchstart', (e) => {
        e.preventDefault();
        const id = parseInt(e.currentTarget.id.split('-')[1]);
        const note = config[id];
        e.currentTarget.innerHTML = note;
        console.log(note);
        e.currentTarget.style.backgroundColor = 'white';
        startNote(id);
    }, false);
});
document.querySelectorAll('.notebox').forEach((f) => {
    f.addEventListener('touchend', (e) => {
        e.preventDefault();
        e.currentTarget.style.backgroundColor = null;
        stopNote(e.currentTarget.id.split('-')[1]);
    });
});
```


You can try the note test interface [here](https://jackson.sh/misc/2021-04-multitouch-keyboard), it only works on mobile devices due to its usage of JavaScript events `touchstart` and `touchend`.

To test our tone generation and interface, I recorded this video using the talkbox manually without motion controls. Here's me playing *Stronger* by Daft Punk very badly:

<video controls height="300">
    <source src="/img/2021-04-11.mp4" type="video/mp4">
</video>





### Capturing input from the Joycons
This was by far the most time consuming part of this project and involved using and compiling a variety of C++ libraries. Prior to moving on, you'll need to install [CMake](https://cmake.org/), [JoyShockLib](https://github.com/JibbSmart/JoyShockLibrary), and the [HIDAPI](https://github.com/signal11/hidapi) on your system.

Our implementation looks something like this:
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <chrono>
#include <fstream>
#include <iostream>
#include <mutex>
#include <string>

#include "JoyShockLibrary.h"
#include "SDL2/SDL.h"
#include "hidapi.h"

using std::cin;
using std::cout;
using std::endl;
using std::mutex;
using std::string;

#define LEFT_JOYCON 1
#define RIGHT_JOYCON 2

void connectDevices() { int numConnected = JslConnectDevices(); };

void cleanup() { JslDisconnectAndDisposeAll(); }

struct imu_frame_t {
    // local acceleration without gravity
    float accelX;
    float accelY;
    float accelZ;
    // gravity vector
    float gravX;
    float gravY;
    float gravZ;
};

void executeNoteChange(JOY_SHOCK_STATE state, int handId) {

}

// This function gets run every time we get a state update from the program.
void joyShockPollCallback(int jcHandle, JOY_SHOCK_STATE state, JOY_SHOCK_STATE lastState, IMU_STATE imuState,
                          IMU_STATE lastImuState, float deltaTime) {
    MOTION_STATE motion = JslGetMotionState(jcHandle);
    int hand = JslGetControllerType(jcHandle);
    if (hand == LEFT_JOYCON) {
        cout << "x: " << motion.gravX << endl;
        cout << "y: " << motion.gravY << endl;
        cout << "z: " << motion.gravZ << endl;
    }
    if (hand == RIGHT_JOYCON) {
        cout << "           x: " << motion.gravX << endl;
        cout << "           y: " << motion.gravY << endl;
        cout << "           z: " << motion.gravZ << endl;
    }

    if (deltaTime > 0.06) {
        // lag warning
        float lagAmount = (deltaTime / 0.015) * 100;
        cout << "WARNING: deltaTime of " << deltaTime * 1000 << " is greater than 30ms on controller "
             << (hand == LEFT_JOYCON ? "LEFT_JOYCON" : "RIGHT_JOYCON") << " (" << lagAmount << "% lag)" << endl;
    }
    if (motion.accelZ > .5 || motion.accelY > .5  || motion.accelX > .5) {
        // a large change in motion has occured, check the buffers to figure out what it is and forward the information.
        executeNoteChange(state, hand);
    }

};

int main(int argc, char *argv[]) {
    std::cout << "hi!" << std::endl;

    connectDevices();
    JslSetCallback(&joyShockPollCallback);

    bool quit = false;
    SDL_Event event;
    while (!quit) {
        while (SDL_PollEvent(&event) != 0) {
            // User requests quit
            if (event.type == SDL_QUIT) {
                quit = true;
            }
        }
    }
    cleanup();

    // Quit SDL subsystems
    SDL_Quit();
    return 0;
}

```

### Processing gestures for pitch control

In order to determine what gesture the user is making and send that information to the tone generator, we'll need to add some more code. First, you'll need to install and create the associated boilerplate code for [wsServer](https://github.com/Theldus/wsServer) and add something resembling this to your `executeNoteChange` function.
```cpp
if (handId == LEFT_JOYCON) {
    // go up a note
    ws_sendframe_txt(websocketClient, "down", false);
} else if (handId == RIGHT_JOYCON) {
    // go down a note
    ws_sendframe_txt(websocketClient, "up", false);
}
```


## Demo

This proof-of-concept demonstrates the ability for a user to control their pitch by gesturing with Joycons. The user can gesture to the side with either hand to turn the tone on and off. Gesturing upwards with the right hand raises the pitch to the next configured note, while gesturing upwards with the left hand raises the pitch two configured notes up.

<video controls height="300">
    <source src="/img/2021-04-12.mp4" type="video/mp4">
</video>

