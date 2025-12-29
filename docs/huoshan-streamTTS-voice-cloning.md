# Console Volcano Engine Dual-Stream TTS + Voice Cloning Configuration Tutorial

This tutorial is divided into 4 stages: Preparation Stage, Configuration Stage, Cloning Stage, and Usage Stage. It mainly introduces the process of configuring Volcano Engine Dual-Stream TTS + Voice Cloning through the console.

## Stage 1: Preparation Stage
The super administrator should first activate the Volcano Engine service and obtain the App Id and Access Token. By default, Volcano Engine will provide one voice resource. This voice resource needs to be copied to this project.

If you want to clone multiple voices, you need to purchase and activate multiple voice resources. Just copy each voice resource's Voice ID (S_xxxxx) to this project. Then assign it to system accounts for use. Below are the detailed steps:

### 1. Activate Volcano Engine Service
Visit https://console.volcengine.com/speech/app to create an application in Application Management, and check Speech Synthesis Large Model and Voice Cloning Large Model.

### 2. Get Voice Resource ID
Visit https://console.volcengine.com/speech/service/9999 to copy three items: App Id, Access Token, and Voice ID (S_xxxxx). As shown in the image:

![Get Voice Resource](images/image-clone-integration-01.png)

## Stage 2: Configure Volcano Engine Service

### 1. Fill in Volcano Engine Configuration

Log in to the console with the super administrator account, click [Model Configuration] at the top, then click [Text-to-Speech] on the left side of the model configuration page, search for "Volcano Dual-Stream TTS", click modify, fill in your Volcano Engine `App Id` in the [Application ID] field, and fill in `Access Token` in the [Access Token] field. Then save.

### 2. Assign Voice Resource ID to System Account

Log in to the console with the super administrator account, click `Parameter Dictionary` at the top, and in the dropdown menu, click the `System Function Configuration` page. Check `Voice Cloning` on the page and click save configuration. You can then see the `Voice Cloning` button in the top menu.

Log in to the console with the super administrator account, click [Voice Cloning] at the top, then [Voice Resources].

Click the add button, select "Volcano Dual-Stream TTS" in [Platform Name];

Enter your Volcano Engine voice resource ID (S_xxxxx) in [Voice Resource ID], press Enter after entering;

Select the system account you want to assign to in [Owner Account], you can assign it to yourself. Then click save.

## Stage 3: Cloning Stage

If after logging in, clicking [Voice Cloning] > [Voice Cloning] at the top shows [Your account has no voice resources, please contact the administrator to assign voice resources], it means you haven't assigned the voice resource ID to this account in Stage 2. Go back to Stage 2 and assign voice resources to the corresponding account.

If after logging in, clicking [Voice Cloning] > [Voice Cloning] at the top shows the corresponding voice list. Please continue.

In the list, you will see the corresponding voice list. Select one voice resource and click the [Upload Audio] button. After uploading, you can preview the audio or trim a specific segment. After confirming, click the [Upload Audio] button.
![Upload Audio](images/image-clone-integration-02.png)

After uploading audio, you will see the corresponding voice in the list change to "Pending Clone" status. Click the [Clone Now] button. Results will return in 1-2 seconds.

If cloning fails, hover your mouse over the "Error Information" icon to see the failure reason.

If cloning succeeds, you will see the corresponding voice in the list change to "Training Successful" status. You can then click the modify button in the [Voice Name] column to change the voice resource name for easier selection later.

## Stage 4: Usage Stage

Click [Agent Management] at the top, select any agent, and click the [Configure Role] button.

For Text-to-Speech (TTS), select "Volcano Dual-Stream TTS". In the list, find the voice resource with "Cloned Voice" in its name (as shown), select it, and click save.
![Select Voice](images/image-clone-integration-03.png)

Next, you can wake up Xiaozhi and have a conversation with it.
