---
layout: base
title: Course Outlines
image: /images/mario_animation.png
hide: true
---

<!-- Liquid:  statements -->

<!-- Include submenu from _includes to top of pages -->
{% include nav_home.html %}
<!--- Concatenation of site URL to frontmatter image  --->
{% assign sprite_file = site.baseurl | append: page.image %}
<!--- Has is a list variable containing mario metadata for sprite --->
{% assign hash = site.data.mario_metadata %}  
<!--- Size width/height of Sprit images --->
{% assign pixels = 256 %} 

<!--- HTML for page contains <p> tag named "Mario" and class properties for a "sprite"  -->

<p id="mario" class="sprite"></p>
  
<!--- Embedded Cascading Style Sheet (CSS) rules, 
        define how HTML elements look 
--->
<style>

  /*CSS style rules for the id and class of the sprite...
  */
  .sprite {
    height: {{pixels}}px;
    width: {{pixels}}px;
    background-image: url('{{sprite_file}}');
    background-repeat: no-repeat;
  }

  /*background position of sprite element
  */
  #mario {
    background-position: calc({{animations[0].col}} * {{pixels}} * -1px) calc({{animations[0].row}} * {{pixels}}* -1px);
  }
</style>

<!--- Embedded executable code--->
<script>
  ////////// convert YML hash to javascript key:value objects /////////
  var mario_metadata = {}; //key, value object
  {% for key in hash %}
  var key = "{{key | first}}"  //key
  var values = {} //values object
  values["row"] = {{key.row}}
  values["col"] = {{key.col}}
  values["frames"] = {{key.frames}}
  mario_metadata[key] = values; //key with values added
  {% endfor %}
  ////////// game object for player /////////
  class Mario {
    constructor(meta_data) {
      this.tID = null;  //capture setInterval() task ID
      this.positionX = 0;  // current position of sprite in X direction
      this.currentSpeed = 0;
      this.marioElement = document.getElementById("mario"); //HTML element of sprite
      this.pixels = {{pixels}}; //pixel offset of images in the sprite, set by liquid constant
      this.interval = 100; //animation time interval
      this.obj = meta_data;
      this.marioElement.style.position = "absolute";
    }
    animate(obj, speed) {
      let frame = 0;
      const row = obj.row * this.pixels;
      this.currentSpeed = speed;
      this.tID = setInterval(() => {
        const col = (frame + obj.col) * this.pixels;
        this.marioElement.style.backgroundPosition = `-${col}px -${row}px`;
        this.marioElement.style.left = `${this.positionX}px`;
        this.positionX += speed;
        frame = (frame + 1) % obj.frames;
        const viewportWidth = window.innerWidth;
        if (this.positionX > viewportWidth - this.pixels) {
          document.documentElement.scrollLeft = this.positionX - viewportWidth + this.pixels;
        }
      }, this.interval);
    }

    animateFlipped(obj, speed,) {
      let frame = 0;
      const row = obj.row * this.pixels;
      this.currentSpeed = speed;
      this.tID = setInterval(() => {
        const col = (frame + obj.col) * this.pixels;
        this.marioElement.style.backgroundPosition = `-${col}px -${row}px`;

        // Flip the image for left movement
        this.marioElement.style.transform = 'scale(-0.2, 0.2)';
      


        this.marioElement.style.left = `${this.positionX}px`;
        this.positionX += speed;
        frame = (frame + 1) % obj.frames;
        const viewportWidth = window.innerWidth;

        if (this.positionX > viewportWidth - this.pixels) {
          document.documentElement.scrollLeft = this.positionX - viewportWidth + this.pixels;
        }
      }, this.interval);
    }
    startRight() {
      this.stopAnimate();
      this.animate(this.obj["Walk"], 5);
    }
    startRunning() {
      this.stopAnimate();
      this.animate(this.obj["Run1"], 10);
    }
    startRunningL() {
      this.stopAnimate();
      this.animateFlipped(this.obj["Run2"], -10);
    }
    startLeft() {
      this.stopAnimate();
      this.animateFlipped(this.obj["Walk"], -5);
    }
    startCheering() {
      this.stopAnimate();
      this.animate(this.obj["Cheer"], 0);
    }
    startFlipping() {
      this.stopAnimate();
      this.animate(this.obj["Flip"], 0);
    }
    startResting() {
      this.stopAnimate();
      this.animate(this.obj["Rest"], 0);
    }
    stopAnimate() {
      clearInterval(this.tID);
    }
    startPuffing() {
      this.stopAnimate();
      this.animate(this.obj["Puff"], 0);
    }
  }
  const mario = new Mario(mario_metadata);
  ////////// event control /////////
  window.addEventListener("keydown", (event) => {
    if (event.key === "ArrowRight") {
      event.preventDefault();
      if (event.repeat) {
        mario.startCheering();
      } else {
        if (mario.currentSpeed === 0) {
          mario.startRight();
        } else if (mario.currentSpeed === 5) {
          mario.startRunning();
        }
      }
    } 
    if (event.key === "ArrowLeft") {
      event.preventDefault();
      if (event.repeat) {
        mario.startCheering();
      } else {
        if (mario.currentSpeed === 0) {
          mario.startLeft();
        } else if (mario.currentSpeed === -5) {
          mario.startRunningL();
        }
      }
    } 

    if (event.key === "ArrowUp") {
        event.preventDefault();
        if (event.repeat) {
        mario.startFlipping();
      } else {
        mario.startFlipping()
      }
         // Call the function to set Mario to the resting state
      }

      if (event.key === "ArrowDown") {
        event.preventDefault();
        if (event.repeat) {
        mario.startPuffing();
      } else {
        mario.startPuffing()
      }
         // Call the function to set Mario to the resting state
      }
  });
  

  //touch events that enable animations
  window.addEventListener("touchstart", (event) => {
    event.preventDefault(); // prevent default browser action
    if (event.touches[0].clientX > window.innerWidth / 2) {
      // move right
      if (currentSpeed === 0) { // if at rest, go to walking
        mario.startWalking();
      } else if (currentSpeed === 3) { // if walking, go to running
        mario.startRunning();
      }
    } else {
      // move left
      mario.startPuffing();
    }
  });

  
  //stop animation on window blur
  window.addEventListener("blur", () => {
    mario.stopAnimate();
  });
  //start animation on window focus
  window.addEventListener("focus", () => {
     mario.startFlipping();
  });
  //start animation on page load or page refresh
  document.addEventListener("DOMContentLoaded", () => {
    // adjust sprite size for high pixel density devices
    const scale = window.devicePixelRatio;
    const sprite = document.querySelector(".sprite");
    sprite.style.transform = `scale(${0.2 * scale})`;
    mario.startResting();
  });
</script>




## SONI BLOG CSSE

### Welcome to My Blog! 
<br>
<img src = "https://images.unsplash.com/photo-1515879218367-8466d910aaa4?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8Y29tcHV0ZXIlMjBjb2RlfGVufDB8fDB8fHww&w=1000&q=80" >

<br>

## MY WONDERFUL PARTNERS

### The Coding Mode Gang: Soni, Nora & Varnika!!
 <img src= "The Code Mode Gang.png">


## About Me 

### Here are some interesting things about me! 

- I love to read!

<img width="400" height ="600" src = "">



