# 🔌 Detect and monitor the state of an appliance based on its power consumption - V2.1.1 ✨
### Automation Blueprint for Home Assistant

<br>

<p align="center">
  <img width="240" height="auto" src="assets/automation-tree.jpg" alt="Monitor the state of an appliance - by leofabri"/>
</p>

This is the automation I made to control the state of the most power-hungry appliances in my house.

There aren’t numerous blueprints that do what I want, and I’m sure that I have the skills to automate my machines without wasting money to buy them new. Here’s my cheap and reliable solution that addresses the problem, once and for all.

I use this for my dishwasher and washing machine and the results have been great! The blueprint is very extensible and easy to use.
My family ❤️ this thing! Even if they don’t understand its complexity and get what all the fuss is about.

As I said, I’m sharing this with everyone. I’m sure that you’ll find it useful!

---
## **Features**

For the record: there is more inside but, here's a summary of what this thing allows you to do:
*   🎛️ **More triggers: more control,** even across Home Assistant reboots!
    *   **This automation is triggered on each power reading**. I'm doing the starting/finishing thresholds magic in the "actions" section, and you can also add some of your own.
The objective of this blueprint is to allow more granular control over what's happening and do that reliably.

*   ⏱️ **Faster machine start-to-finish detection**: one of the problems that I always face is **timing**. <i>What if you want to measure how long it took for your appliance to complete its job?</i> The automation has to be <strong>very reactive</strong>. This logic should solve that problem for you.
  
*   📉 **Unreliable power absorption filtering and inhibition**: This is useful! It prevents the blueprint from thinking that a job was completed as soon as the power falls below the finishing power threshold. You can set your timing here.
  
*   📅 **Set your custom actions** for each of the following conditions:
      * when an overload occurs
      * when the overload situation is solved, now paused
      * when the appliance is plugged back in, now paused
      * when the overload situation is solved, now resuming
      * when a new job cycle begins
      * when a job cycle resumes
      * when a job cycle is finished
      
      All of these are optional!

<br>

*   🥷 **The appliance has its own** [**State Machine**](https://en.wikipedia.org/wiki/Finite-state_machine)
    *   So that you can tell if it's in one of the following states:
          - <strong><u><ins>unplugged</ins></u></strong> - means that the appliance is no longer powered. 
            It happens when the user manually turns off the smart socket (from HA or the socket itself).
            
          - <strong><u><ins>idle</ins></u></strong> - There is no pending job, the machine is powered but idling.
            
          - <strong><u><ins>paused</ins></u></strong> - Indicates that a job is pending (incomplete cycle) but the appliance is not performing it. The inhibitors of these state are the <strong><i>detached_overload</i></strong> and <strong><i>unplugged</i></strong> states. 
            In this condition, the Power absorption is lower than the finishing power threshold. The appliance must be off (maybe the user turned it off manually, or maybe the job needs some time to recover).
            The blueprint is waiting for the appliance to resume.

            <strong>Pro Tip!</strong> You could use this state to diagnose and warn if a job is not resumed after x minutes. 
            
          - <strong><u><ins>detached_overload</ins></u></strong> - This happens when, during a cycle, the appliance used too much power and was therefore suspended. It is also technically unplugged.
            
          - <strong><u><ins>job_ongoing</ins></u></strong> - Triggered in two situations:
              - **when a new job cycle begins**: the previous one is completed and the Starting Power threshold is surpassed.
              - **when a job is resumed**.
              
          - <strong><u><ins>job_completed</ins></u></strong> - Triggered when the current incomplete job cycle is finished. The appliance consumes less than the Finishing Power threshold (with the possibility of selecting for how long).


        <strong>Note:</strong> Each state transition is handled automatically, you just have to provide the initial configuration.
<br>

Let's install it!

---
## 📋**Requirements:** What do I need to run this blueprint?

To achieve this level of control **the socket must have some basic features**: 
- Power monitoring (a polling rate of circa 60 sec. or less for best results)
- ON/OFF control over Home Assistant
- State reporting (if it's ON or OFF)

Mine is a <strong>Meross MSS310EU</strong>, but most smart sockets should be supported.

### One more thing:
- <u>OPTIONAL:</u> To get **the pause state** to work properly, **you also need another automation that can detect an overload**. I'm not the author of that, but the one I'm using is [here](https://github.com/andbad/HA_PowerControl) (disclaimer: it's just in Italian 🍝). <br>

---

## ⚙️ **Initial Setup:** Let's make some helper variables (<i style="text-color: red">mandatory!</i>)
There are **two possible ways** of doing this:

- **Setup via the UI [Discouraged but Beginner Friendly]:** newcomers that might be a little into shiny-but-dark UIs 😎 might want to set up this blueprint following [this documentation here](Alternative%20UI%20Setup.md).

  ...

- <u>**Setup via the old school YAML [Optimal & Suggested]:**</u> The best way of creating what we need is to create a package (suggestion provided by [@HollyFred](https://github.com/leofabri/hassio_appliance-status-monitor/issues/8#issue-1304478600)) because it allows us to create all the necessary helpers (timers, input_booleans, etc.) in one shot. <i>Continue reading for this one</i> ⬇️

Creating a package is super easy. Some of you may already have the right configuration in place, but I'll show you how to do that anyway.

> Note: you need to have the VS Code integration enabled. This will allow you to manually edit the necessary files. Need help with that? [Read this](https://www.home-assistant.io/docs/configuration/).

Back to us, here are the two steps for creating a package:

1. **Make sure that you have a directory called 'packages'**.
   
    If you are a beginner, or you are starting fresh with your HA installation, likely, the `packages` dir is not present.
   
   To add it, you can:
   > Open your HomeAssistant VS Code editor and create a directory called `packages` (at the same level as the configuration.yaml file).
  
   OR

   > Issue this command in the terminal section of your VS Code:
   >
   >  ```
   > cd /config && mkdir packages
   > ```
   > Easy-peasy.


   <u>**Result:**</u> You should see the packages' directory <u>**at the same level**</u> as the `configuration.yaml` file.


2. **Make sure that the packages support is enabled** inside the [`configuration.yaml`](home%20assistant/configuration.yaml) file as shown below:

    ```yaml
    homeassistant:
      ## Packages support enabled - Includes every .yaml file that is inside the packages/ directory
      packages: !include_dir_named packages/
      ## Warning! Please pay attention to the indentation!
    ```

**Now we are ready to create our first package**.

### Use the presets
><strong>PRESETS</strong> - If you want to simplify this setup even more, check out one of the already pre-made [packages](home%20assistant/packages/) I prepared for you. ([Washing Machine](home%20assistant/packages/washing_machine.yaml), [Dryer](home%20assistant/packages/dryer.yaml), [Dishwasher](home%20assistant/packages/dishwasher.yaml) and more are already supported)

### Alternative, custom package definition:

Open the `packages/` directory and create a file with the name of your appliance.
For example, I'm calling mine `washing_machine.yaml` but yours could be different. It's always better to call it something like `<your_appliance_name>.yaml`. [Here's a blank file for that](home%20assistant/packages/your_appliance_name.yaml).


Remember that in order to use the blueprint **you'll always need these four ingredients** inside a package:
### A. The State Machine

```yaml
input_select:
  <your_appliance_name>_state_machine:
    name: <Your Appliance Name> - State Machine
    options:
      - unplugged
      - idle
      - paused
      - detached_overload
      - job_ongoing
      - job_completed
    icon: mdi:<your_appliance_icon>
```


### B. The Job Cycle indicator

```yaml
input_boolean:
  <your_appliance_name>_job_cycle:
    name: <Your Appliance Name> - Job Cycle
    icon: mdi:<your_appliance_icon>
```

### C. The delay timer

```yaml
timer:
  <your_appliance_name>_delayed_job_completion_timer:
    name: <Your Appliance Name> - Delayed Job Completion Timer
    duration: "00:15:00" # <- Note that the time here. Leave it 15 min
    restore: true
    icon: mdi:<your_appliance_icon>
```

### D. The automation self-trigger
```yaml
input_boolean:
  <your_appliance_name>_automation_self_trigger:
    name: <Your Appliance Name> - Automation Self-trigger
    icon: mdi:<your_appliance_icon>
```

**Important!** Don't forget to edit all the fields `<your_appliance_name>`, `<Your Appliance Name>` and `<your_appliance_icon>` accordingly.

---

# ➕ Download & Import
>Are you **trying to update** to a newer version? [Let's do that in a snap! Follow this guide](#to-update).

If you have just created your package, please reboot Home Assistant ([at this page](https://my.home-assistant.io/redirect/server_controls/) → Verify & if okay, Reboot)

1.  Add this blueprint to your Home Assistant [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fleofabri%2Fhassio_appliance-status-monitor%2Fblob%2Fmain%2Fappliance-status-monitor.yaml)
2.  Create a New Automation from that blueprint
3.  Configure the variables as indicated inside the blueprint's UI.
   
That's it!

---
## Results 🎉
### You should see the 4 new entities:

> <strong>Warning |</strong> always use those entities as indicators, do not change their values manually! Leave that to your automation

<figure align="center">
  <img src="assets/washing_machine-entities-example.jpg" alt="my alt text"/>
  <figcaption><p align="center">Washing Machine entities</p></figcaption>
</figure>

### During a job cycle:

<figure align="center">
  <img src="assets/washing_machine-power-example.jpg" alt="power usage example"/>
  <figcaption><p align="center">This is my washing machine. Power usage example</p>
</figcaption>
</figure>

<figure align="center">
  <img src="assets/appliance-history.jpg" alt="appliance history image"/>
  <figcaption><p align="center">The State machine. The one in light blue is a simulated overload situation.</p>
</figcaption>
</figure>


## Congratulations! Your appliance is now smarter 🧠
<br>
<br>

___
<br>
<br>

# 🔔 Reminders (optional)

Tired of checking your phone to see if the washing machine is done? The **Appliance reminder** is a companion blueprint that sends you a notification when your appliance finishes a job cycle - and keeps re-notifying you until you take care of it.

*   📲 **You get a notification** with two buttons: **Taken care of** (dismisses the reminder) and **Snooze** (postpones the next reminder). The buttons also appear on your dashboard.
*   🔁 **It keeps nagging** - by default every 60 minutes (configurable) - until you press a button.
*   🧹 **No stale nag** - if a new job cycle begins, the reminder is cleared automatically (you must have emptied the machine, right?).
*   🔄 **Reboot safe** - if Home Assistant reboots while a reminder is pending, the re-notification loop picks up where it left off.
*   🏠 **Multi-appliance & multi-phone friendly** - one instance per appliance, and a notify group as target to reach all your devices.

> The buttons work with the official [Home Assistant mobile apps](https://companion.home-assistant.io/docs/notifications/actionable-notifications) (Android & iOS).

## Setting up the reminders

### 1. Create the 4 extra helpers

If you are using one of my [presets](home%20assistant/packages/), they already include these helpers - just reload Home Assistant.

Otherwise, add these to your appliance's package (edit the placeholders accordingly):

```yaml
input_boolean:
  <your_appliance_name>_needs_attention:
    name: <Your Appliance Name> - Needs Attention
    icon: mdi:<your_appliance_icon>
    restore_state: true

timer:
  <your_appliance_name>_reminder_timer:
    name: <Your Appliance Name> - Reminder Timer
    duration: "01:00:00" # <- Only a default: the actual re-nag interval is set in the reminder blueprint
    restore: true
    icon: mdi:<your_appliance_icon>

input_button:
  <your_appliance_name>_taken_care_of:
    name: <Your Appliance Name> - Taken Care Of
    icon: mdi:check
  <your_appliance_name>_snooze:
    name: <Your Appliance Name> - Snooze
    icon: mdi:alarm-plus
```

### 2. Import the second blueprint

1. Add the [Appliance reminder blueprint](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fleofabri%2Fhassio_appliance-status-monitor%2Fblob%2Fmain%2Fappliance-reminder.yaml) to your Home Assistant.
2. Create a **New Automation** from that blueprint - a *second* automation, alongside your "Monitor the state of an appliance" one.
3. Configure it:
    * **State Machine**: the same entity your first automation uses.
    * **Needs Attention flag**, **Reminder timer**, **Taken care of** and **Snooze** buttons: the new helpers you just created.
    * **Notify target**: the name of your mobile_app device (e.g. `mobile_app_my_phone`) or of a [notify group](https://www.home-assistant.io/integrations/group/#notify-groups) to reach several devices - without the `notify.` prefix.
    * **Re-nag interval** and **Snooze duration**: minutes (defaults: 60 and 180).
4. Reload Home Assistant.

That's it! When the appliance reaches the **job_completed** state, you'll be notified - and re-notified - until you press **Taken care of** or **Snooze** (from the phone or from the dashboard).

___
<br>
<br>


# Versions & Changelogs:
## 📯Important notice! 🚨
### 🆕 The code has been updated
1. [**Version 2.1.1 of 07/21/2022**](https://github.com/leofabri/hassio_appliance-status-monitor/tree/release/v2.1.1) - [**FIX!**]
   - Fixed [this bug](https://community.home-assistant.io/t/detect-and-monitor-the-state-of-an-appliance-based-on-its-power-consumption-v2-1-0/421670/37?u=leofabri):
    > ### Description:      
    > When you don’t indicate the Appliance suspended entity - which I
    > marked as optional - the automation becomes unresponsive, thus making the optional value a
    > required one.<br>

  - Added some new documentation. You can decide to make your helpers through the UI instead of VS Code.
  - We are now using packages instead of overloading the configuration.yaml file with the helpers.
2. The changes related to the **older versions** are shown <u>[here](older_versions_changelog.md)</u>.
---

## **How do I know if there is a new version?**
>
> Easy! [Look for the current version number in the title](https://github.com/leofabri/hassio_appliance-status-monitor#-detect-and-monitor-the-status-of-an-appliance-based-on-its-power-consumption). Now, open one of the automations that currently uses this blueprint and notice the version tag:
> <p align="center">
>  <img src="assets/need-to-update.jpg" alt="Do I need to update?"/>
</p>

##  <p id="update">**How to update:**</p>
>   * Open this page [![Open your Home Assistant instance and show your blueprints.](https://my.home-assistant.io/badges/blueprints.svg)](https://my.home-assistant.io/redirect/blueprints/) (just click the link, it's magical) and scroll till you find "*Monitor the status of an appliance - by leofabri*" and delete it by clicking on the trash bin 🗑. Don't worry, we'll add it right back in a few secs, you can leave the automation as it is for now.
>   * Create the missing helpers (entities) as shown in the instructions
>   * Add the blueprint again [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fleofabri%2Fhassio_appliance-status-monitor%2Fblob%2Fmain%2Fappliance-status-monitor.yaml) and reload Home Assistant.
>   * Open your automation and add the previously created helpers (entities)

# Q&A - Things you may want to know
[See the dedicated documentation](Frequent%20Questions%20and%20Answers.md)