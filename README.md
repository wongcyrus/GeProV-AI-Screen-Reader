# GeProVis AI Screen Reader
Experience the magic of Gemini Pro Vision AI Screen Reader in this thrilling video. We've turbocharged the traditional Google ChromeVox Screen Reader with the mighty power of Google Gemini Pro Vision. Our mission? To tackle the challenge of web image information access for those with visual impairments.

The problem being addressed is the inability of visually impaired individuals to access image information due to the lack of adherence to W3C web accessibility initiatives by websites. Currently, about 60% of websites lack meaningful alternate text for their images. Moreover, it is unfeasible to retroactively add descriptive text to all existing websites manually.

[![GeProVis AI Screen Reader for visually impaired - GDSC Solution Challenge 2024](https://img.youtube.com/vi/SUkg_76mF6M/0.jpg)](https://www.youtube.com/watch?v=SUkg_76mF6M)


## Architecture 

1. **Frontend**: The utilization of ChromeVox Classic screen reader, Cloud Function, and Google Gemini Pro Vision is prevalent. ChromeVox Classic is favored due to its comprehensive functionality and widespread acclaim. Cloud Function is employed owing to its adaptability, ready-to-use environment, superior scalability, and cost-effectiveness. Lastly, Gemini Pro Vision is chosen due to its prowess as the most potent AI model capable of describing an image in a split second, API-based, and its low usage cost.This project enhances the open-source screen reader, Google ChromeVox Classic, which serves as an extension to the Google Chrome browser. The extension module employs Javascript.

2. **Backend**: The backend incorporates the Google Gemini Pro Vision API via the Google cloud function. This API generates descriptive text for images received in either URL or base64 format. The default language for this text is English, although this can be altered by supplying the 'lang' parameter. The text generated by the Gemini API is then sent to the Google Translation API for translation into the selected language and then returned to the front end.

3. **Cloud Functions**: Firstly, the user is retrieved through the API key. This process is protected by Google Cloud API Gateway, which also provides API key authentication and rate limiting. Following this, the image data is downloaded and if its size exceeds 3MB, it is adjusted. Then, gemini-1.0-pro-vision is invoked to acquire a caption for the image from all models’ available regions to maximize the rate limit, approximately 40 words long, based on the locale. Subsequently, the estimated cost for each AI call is computed. Finally, captions and usage are stored in the Google Cloud Datastore. To save cost and faster response, captions will be translated by Google translate on demand for duplicate image in different language.

4. **Google Cloud Datastore**: This contains three kinds: ApiKey, Caption, and Usage.
   - **ApiKey**: Matches an API key to a User ID.
   - **Caption**: Saves image hash to caption, which is a cache to skip AI calls for the same image. To protect piracy, we do not log down any URL.
   - **Usage**: Records usage for each user, including the cost and time. Each API key has a daily cost limit for budget control.


![Architecture of GeProVis AI Screen Reader](/images/architecture.jpg)

## Current limitation
ChromeVox, behind the scenes, captures URLs and sends this information, along with the browser locale, to the Cloud Function. We have experimented with two different approaches. The first involves ChromeVox downloading the image and sending it to the cloud function, but this has occasionally encountered CORS permission issues. The second approach has ChromeVox sending the URL to the cloud function, which then downloads the image. However, this doesn't work if the site requires a login to access the image.

After a testing period and gathering feedback from visually impaired users, we have chosen to proceed with the second approach.

We welcome any opinions or suggestions or pull request to help resolve these issues, as we recognize that we are not experts in web technology.


## Setup procedure

### Frontend:
#### Development setup
1. Follow [original instruction](https://source.chromium.org/chromium/chromium/src/+/main:docs/windows_build_instructions.md) to checkout and build ChromeVox Classic
2. Copy and replace all sources under [chromevoxclassic/](chromevoxclassic) to the chromium source under the path ui/accessibility/extensions/chromevoxclassic/
3. After building the ChromeVox Classic browser extension, install the extension under Chrome browser with developer mode turned on

#### Pre-built download
You may download the [pre-built application](chromevoxclassic/build/aichromevox.zip) for the testing

#### Usage and configuration
1. Install with "Load unpacked"; the language of generated image description follows the current selected voices of the TTS
2. The cloud function/API gateway URL can be configured via extension option page with the "Endpoint URL of Gemini API" under **Gemini API**
3. The API Key can be input with "Authentication Key" under **Gemini API** from the extension option page, just leave it empty if no API key will be used

### Backend: 
#### Manual (DEPRECIATED)
This is quick R&D without authenication and rate limit. 
1. Create a new Google Cloud Function, with settings: 2nd gen, memory >= 512 MB, and allow all traffic
2. Copy and replace all sources under [google_cloudfunction/](google_cloudfunction) to your Cloud Function 

#### Production Deplolyment
This creates a new project on the Google Cloud Platform that incorporates API Gateway, DataStore, and Cloud Function. Clone this repository and establish a Codespace.

##### Setup
Rename cdktf/.env.template to .env

Update the value. 

```
PROJECTID=gemini-screen-reader
BillING_ACCOUNT=<YOUR GCP Billing Account>
REGION=us-east1
MODEL_NAME=gemini-1.0-pro-vision-001
DAILY_BUDGET=0.1
LANGCHAIN_TRACING_V2=false
LANGCHAIN_API_KEY=
GOOGLE_APPLICATION_CREDENTIALS=/home/codespace/.config/gcloud/application_default_credentials.json
```
if you want to use LangSmith, set LANGCHAIN_TRACING_V2=true and LANGCHAIN_API_KEY=<API Key>.

DAILY_BUDGET is the cost limit within 24 hours per user.


##### Login your GCP account
```
gcloud auth application-default login
```

##### Create GCP resources
```
./deploy.sh 
```
Record down the output api-url, project-id, and service-name.

##### Enable the API

```
gcloud auth login
gcloud config set project <project-id>
gcloud auth application-default set-quota-project <project-id>
gcloud services enable <service-name>
```
If you omit to enable API, you will get "PERMISSION_DENIED: API xxxx.apigateway.gemini-screen-reader.cloud.goog is not enabled for the project.".

#### Admin Tools
A set of Python scripts for API key management in admin_tools folder.

###### Before using admin tools
```
gcloud auth login
gcloud config set project gemini-screen-reader
gcloud auth application-default set-quota-project gemini-screen-reader
```

1. Rename Namelist_template.xlsx to Namelist.xlsx, add maximum 500 users to the excel.
2. Update admin_tools/config.py.
4. run ```./setup.sh```
3. run ```source venv/bin/activate```


# Google Developer Student Clubs - GDSC Solution Challenge 2024 Top 100 by [GDCS-HKIIT (Formerly GDSC-IVE)](https://gdsc.community.dev/hong-kong-institute-of-vocational-education/).
