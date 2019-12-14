# Smart Xmas

![Sparkle Status](https://github.com/martinwoodward/smart-xmas/workflows/sparkle/badge.svg)

A simple project showing you how to easily control smart devices in your home from [GitHub Actions](https://github.com/features/actions) and put a bit of :sparkles: into your repos.

### Prerequisites

 - Smart power strip [[UK](https://amzn.to/2M9geWH)] [[US](https://amzn.to/2YOdDGC)]
 - [IFTTT.com](https://ifttt.com/) account
 - Thing to power - i.e. fairy lights, the gaudier and over the top the better
 
(_Optional_) If you want to trigger sound with you lights then:
 
 - [Adafruit Audio FX Mini Sound Board](https://www.adafruit.com/product/2341) [[UK](https://amzn.to/2EewFwk)] [[US](https://amzn.to/2RTpWQj)]
 - Computer speakers [[UK](https://amzn.to/35l6e4b)] [[US](https://amzn.to/2LRTbPG)]

### Configuration

First of all set up your smart power strip using instructions from your vendor. Lots of the inexpensive unbranded smart devices are OEM devices making using of the [Tuya](http://tuya.com/) cloud network. (The 'Smart Life' application used by many of these is a particularly common front-end to the Tuya API). Regardless of the provider, look for compatibility with [IFTTT](https://ifttt.com/) as that is how we are going to drive it.

Nearly all the applications will allow you to configure scenes or automation that happens on a trigger or when manually triggered (i.e. 'tap to run').  I created a scene called `TreeAnimate` which simply switches on all the power to the power strip, waits 30 seconds and then switches it all off again. However depending on your smart device you can do a lot more if you wanted, for example adjust the color of your RGB lighting, switch on a lava lamp or confetti filled leaf-blower etc.

![Smartlife Scene Configuration](/images/smartlife-scene.png)

If you want a sound to accompany your activity then the easiest way to do this is to hook up an [inexpensive sound-board from Adafruit](https://www.adafruit.com/product/2341). It's a simple device that shows up as a mass storage device when you connect it to your computer. You upload files using a particular naming convention and then they play when the trigger pin is connected to ground. 

![Smartlife Scene Configuration](/images/soundboard.jpg)

See the [Adafruit product tutorial for more information](https://learn.adafruit.com/adafruit-audio-fx-sound-board/triggering-audio). For our purposes we'll just wire tigger pin 0 so that it's permanently connected to ground and that way it will automatically start playing the sound as soon as there is power to the device.

If Kevin had access to these imagine what a different movie [Home Alone](https://amzn.to/2PJTkpD) would have been - wonder how Disney if going to address that in the reboot?.

![Animated Gif of Home Alone movie](https://media.giphy.com/media/L7r3oyzMECdnG/giphy.gif)

### IFTTT Configuration

Next up, we need to configure IFTTT to set off our `TreeAnimate` scene when it recieves a webhook.

The reason we're using IFTTT is that most smart device manufacturers already integrate with it. If you are wanting to configure a Tuya.com based smart device from code directly then take a look at the [tuyapi](https://github.com/codetheweb/tuyapi) project maintained by [Max Ison](https://github.com/codetheweb) - however that relies on being in the same network at the device itself and we want to be able to trigger your device from GitHub's Action servers running in the cloud.

  1. Create an account with IFTTT then follow your manufacturers instructions for configuring and authenticating your smart service with IFTTT. 

  2. Next, [create your IFTTT workflow](https://ifttt.com/create). For the `this` trigger select `Webhooks`, then 'recieve web request' and give your event a name. I rather unimaginative called mine `do_hook`

  ![IFTTT Trigger](/images/ifttt-webhook.png)

  3. For the `that` action, select your smart device vendor and then select the scene you wish to activate. In my case `TreeAnimate`

  ![IFTTT Action](/images/ifttt-action.png)

  4. Next you'll need the URL (with key) that we need to trigger the webhook. Go to the [Maker Webhooks service](https://ifttt.com/maker_webhooks/) and select 'Documentation' in the top right land side. IFTTT webhooks are triggered using the following URL pattern:

  ```
  https://maker.ifttt.com/trigger/{EVENT}/with/key/{KEY}
  ```
  
  In our example, the event is `do_hook`. We want copy the key value and store it as a repository secret in our repo.

  ![IFTTT Action](/images/ifttt-url.png)

### GitHub Action Configuration

 1. The in GitHub project you want to automate, go to 'Settings', then 'Secrets' and 'Add a new secret' For the key name use `IFTTT_KEY` and paste the key value from IFTTT.

 2. Then in the project select 'Actions' and select 'New worfflow'.

 3. In the top right hand side, select 'Set up a workflow yourself'. Name the file something meaningful (I called mine [`sparkle.yml`](https://github.com/martinwoodward/smart-xmas/blob/master/.github/workflows/sparkle.yml)). The workflow itself is very simple. It simply uses curl to trigger the webhook when someone stars the repo in GitHub.

 ```yaml
name: sparkle

on: [watch]

jobs:
  sparkle:

    runs-on: ubuntu-latest

    steps:
    - name: Call IFTTT to trigger lights
      run: curl -X POST https://maker.ifttt.com/trigger/do_hook/with/key/${{ secrets.IFTTT_KEY }}
 ```

Hit commit and you are ready to go.

### Extensions

While I did this one for fun just because I wanted a bit of holiday sparkle to cheer up a gloomy evening, you can use the same mechanism to trigger anything. With the use of [conditional workflow steps](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#jobsjob_idif) you coudld easily turn the lights red and sound an alert when the build fails. Or maybe something less stressful and turn down the lights and play some nice gentle music anytime someone leaves a sparkly heart in response to your pull request. You can easily get the scenes to trigger multiple devices all around your office.

