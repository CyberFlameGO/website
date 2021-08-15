---
title: SYSK Random Episode Generator
description: An episode shuffler for Stuff You Should Know enthusiasts
date: 2021-04-13
tags:
    - tools
    - podcasts
layout: layouts/post.njk
---

<div
    style="border: 1px solid black; border-radius: 10px; width: 100%; padding: 10px; text-align: center; -webkit-box-shadow: 5px 5px 20px 5px rgba(0,0,0,0.67); box-shadow: 5px 5px 20px 5px rgba(0,0,0,0.67); line-height: 30px; margin-bottom: 30px;">
    <div style="font-size: 18px; font-weight: bold;"><span id="episode-name"></span> <button
            onclick="copyToClipboard()">Copy title</button></div>
    <div style="font-size: 16px; margin-top: 10px;">
        <div style="display: inline-block;"><b>Length</b>: <span id="episode-length"></span></div>
        <div style="display: inline-block"><b>Release date</b>: <span id="episode-release-date"></span></div>
    </div>
    <div style="margin-top: 10px;">
        <button style="background-color: green; color: white; padding: 10px; font-size: 18px;" onclick="pick()">Pick another
            episode</button>
    </div>
    <div style="font-size: 12px; text-align: left;">
        <a target="_blank" href="https://en.wikipedia.org/wiki/List_of_Stuff_You_Should_Know_episodes">Data sourced from
            Wikipedia</a>
    </div>
</div>


Stuff You Should Know is my favorite podcast of all time. The show has an absolutely massive (1500+) back catalog of episodes. Unfortunately, neither Spotify nor Apple Podcasts has a shuffle feature for podcasts. As a result, I often find myself scrolling for a *reaaaaallllyy* long time when I'm trying to find an old episode I haven't listened to.

To solve this, I made a little tool that picks a random SYSK episode so you can copy the title into your podcasts app. Hope you find it useful!

[Leave a comment on reddit](https://www.reddit.com/r/stuffyoushouldknow/comments/mq8fsu/i_love_listening_to_old_episodes_i_havent_heard/)

<script>
const getJSON = async url => {
  try {
    const response = await fetch(url);
    if(!response.ok) // check if response worked (no 404 errors etc...)
      throw new Error(response.statusText);

    const data = await response.json(); // get JSON from the response
    return data; // returns a promise, which resolves to this data value
  } catch(error) {
    return error;
  }
}

function getRndInteger(min, max) {
  return Math.floor(Math.random() * (max - min) ) + min;
}

let episodes;

const randomEp = () => {
  return episodes[getRndInteger(0, episodes.length - 1)];
}

let current_random;

const pick = () => {
  current_random = randomEp();
  document.querySelector('#episode-name').innerHTML = current_random.title;
  document.querySelector('#episode-length').innerHTML = current_random.length;
  document.querySelector('#episode-release-date').innerHTML = current_random.date.split('(')[0];
}

const copyToClipboard = () => {
  const el = document.createElement('textarea');
  el.value = current_random.title;
  document.body.appendChild(el);
  el.select();
  document.execCommand('copy');
  document.body.removeChild(el);
};

(async () => {
  episodes = await getJSON('/misc/sysk_episodes.json');
  pick();
})();

</script>




