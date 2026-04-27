# VoiceIt API 3.0 Twilio Demo

[![Build](https://github.com/voiceittech/voiceit3-twilio-demo/actions/workflows/test.yml/badge.svg)](https://github.com/voiceittech/voiceit3-twilio-demo/actions/workflows/test.yml)
[![Dependabot](https://img.shields.io/github/issues-pr/voiceittech/voiceit3-twilio-demo/dependencies?label=dependabot&logo=dependabot&color=025e8c)](https://github.com/voiceittech/voiceit3-twilio-demo/pulls?q=is%3Apr+label%3Adependencies)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](https://github.com/voiceittech/voiceit3-twilio-demo/blob/master/LICENSE)
[![Node](https://img.shields.io/badge/node-18%2B-brightgreen)](https://nodejs.org/)
[![Twilio Serverless](https://img.shields.io/badge/twilio-serverless-red)](https://www.twilio.com/docs/serverless)
[![VoiceIt API](https://img.shields.io/badge/VoiceIt-API%203.0-blue)](https://voiceit.io)
[![Airtable](https://img.shields.io/badge/database-Airtable-yellow)](https://airtable.com)

## Documentation

* VoiceIt API Docs can be found [here](https://voiceit.io/documentation)
* The 120 supported languages and dialects can be found [here](https://voiceit.io/documentation#content-languages) 

## Telephony Audio Limitations

> **Security Notice:** Twilio phone call recordings are limited to **8 kHz, 8-bit, mono (mu-law)** due to the telephone network (PSTN/G.711). At this low audio quality, the biometric engines have significantly less spectral data to distinguish a real voice from a deepfake or synthetic voice clone. This makes telephone-based verification more vulnerable to voice spoofing attacks than the native SDKs.
>
> For production deployments, we strongly recommend integrating the **iOS, Android, or Web SDKs** directly into your mobile or web application rather than relying on phone calls. The native SDKs record at **48 kHz, 16-bit, mono**, which provides the full audio fidelity needed for both accurate speaker verification and robust deepfake rejection.
>
> | Source | Sample Rate | Deepfake Defense |
> |--------|------------|-----------------|
> | iOS/Android/Web SDK | 48 kHz 16-bit | Full protection |
> | Twilio phone call | 8 kHz 8-bit | Reduced — lower audio fidelity limits deepfake detection |

## Prerequisites  
* A VoiceIt account. VoiceIt offers an API for voice biometrics that we’ll be using during this blog. Follow this [link](https://voiceit.io/pricing) to sign up. Then log in to the [Dashboard](https://dashboard.voiceit.io) to manage your account.
* A Twilio account. Sign up [here](https://www.twilio.com/try-twilio) for a free trial
* An Airtable account. Sign up [here](https://airtable.com/#) for a free trial. You could use a different database or CRM here as well with some code changes. 
* A Linux or MacOS terminal with [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) and the [Twilio CLI](https://www.twilio.com/docs/twilio-cli/quickstart) installed

**Note:** Currently this project is deployed in Node 18+

## Setup 

### Environment variables

* Add .env file to the root folder

**Required variables**: 
* Twilio: ACCOUNT_SID, AUTH_TOKEN
* VoiceIt: VOICEIT_API_KEY, VOICEIT_API_TOKEN, CONTENT_LANGUAGE, VOICEPRINT_PHRASE
* Database: DATABASE_URL, AIRTABLE_API_KEY, AIRTABLE_BASE_ID

Example: 
```
VOICEIT_API_KEY='key_********************'
VOICEIT_API_TOKEN='tok_*********************'
CONTENT_LANGUAGE=en-US
VOICEPRINT_PHRASE='Never forget tomorrow is a new day'
DATABASE_URL=test
AIRTABLE_API_KEY='key************'
AIRTABLE_BASE_ID='app*********'
```

### Airtable
1. Log in to Airtable. Create a new workspace or use an existing one to create a new base. 
2. Name the table “Voice Biometric”
3. Rename the default primary field to “Phone Number”. Add another field for “Biometric UserId”. Your table should look like this:
 ![Air Table Setup Example](./airTableSetup.png?raw=true "Air Table Setup")
4. You’ll need this table’s Base ID to programmatically make changes. You can find that in the URL. For example, if your full URL is https://airtable.com/appXXXXXXXXXX/tblYYYYYYYYYYYY/viwZZZZZZZZZZZZblocks=hide, your Base ID is the section beginning with “app”: appXXXXXXXXXX. Update the AIRTABLE_BASE_ID value in your .env file. 
5. Copy your Airtable API Key from your Airtable Account page. Paste this value in your .env file after AIRTABLE_API_KEY=.


### Deploy to Twilio Serverless
* In your terminal, navigate to the root directory of this project.
* Run npm install to install the necessary packages.
* Authenticate into your Twilio account in the command line by running the following command, replacing the default values with your own. You can find these values in the Twilio Console.
```
export ACCOUNT_SID=ACXXXXXXXXXXXXXXXXXXXXXXXX
export AUTH_TOKEN=XXXXXXXXXXXXXXXXXXXXXX
```
* Run twilio serverless:deploy to deploy your functions and assets to Twilio’s serverless environment. 
### Connect your Phone Number 
* If you do not already have a Twilio Phone Number, follow [these instructions](https://www.twilio.com/docs/phone-numbers) to buy one. 
Access the Active Numbers page in the Console. Click the desired phone number to modify.
* Scroll to the Voice & Fax section. Under “Configure With”, select “Webhook, TwiML Bin, Function, Studio Flow, Proxy Service” from the drop down.
* Under “A Call Comes In”, select “Function”.
* Under “Service”, select “voiceit”
* Under “Environment”, select “dev-environment”. 
* Under “Function Path”, select “/voice/incoming_call”. 


## Flow of the Demo 

![Flow Chart of the Demo](./twilioVoiceItFlowDiagram.png?raw=true "Flow Chart of the Demo")

## Verification 

Flow: 
1. [Create a user](https://voiceit.io/documentation#create-a-user)
    - Made with VoiceIt.CreateUser API Call within incoming call
2. [Enroll voice](https://voiceit.io/documentation#create-voice-enrollment-by-url)
    - **Users must have at least 3 enrollments to verify**
3. [Verify](https://voiceit.io/documentation#voice-verification)
    - Users that do not have enough enrollments will be redirected to enrollments
    - If a user has bad enrollments(i.e. lots of background noise, poor connection at the time of the call, etc.), they can be deleted and redone. You can also check enrollments for a particular user in [dashboard](https://dashboard.voiceit.io)

## Identification

This can be useful if there are multiple users on the same phone number. 

Flow: 
1. [Create a group](https://voiceit.io/documentation#create-group)
2. [Add users to the group](https://voiceit.io/documentation#create-group)
    - **Users must have at least 3 enrollments to be identified**
    - For efficiency try to stay below 40 users in each group
3. [Make an identification call](https://voiceit.io/documentation#voice-identification)

Note the phrase and content language can be any of the 119 supported languages and can be any custom phrase you add to your account
## Support

If you find this demo useful, please consider giving it a star on GitHub — it helps others discover the project!

[![GitHub stars](https://img.shields.io/github/stars/voiceittech/voiceit3-twilio-demo?style=social)](https://github.com/voiceittech/voiceit3-twilio-demo/stargazers)

## License

voiceit3-twilio-demo is available under the MIT license. See the LICENSE file for more info.
