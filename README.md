# googleapis-auth-adsense

### Google Adsense management API call with Authentication
This is references for how to call Google API's using google OAuth authorization.

The below example is to generate report from google Adsense, Prerequisite for Adsense:- you need to have Adsense enable account.

#### Enable API using google console and Generate OAuth secret keys
1. Login to Google Console (https://console.developers.google.com)
2. click on Create Project
3. click on Enable API
4. Select Management API / Any API you want to enable
5. Click on Enable link
6. Now, go to Credentials menu from left panel
7. Click on Create Credentials and select OAuth Client ID
8. If you have not configured Consent screen and configure it. click on Configure Consent, Give name and save.
9. Tf redirected again to on Credendtials screen then again choose OAuth Client ID and create.
10. Select Application Type as per your requirement, i hav eseleted as Web Application
11. Give Name to it and click on Create, it will redirect to credentials detail
12. Download JSON file from it and rename it to client_secret.json

You can get help from google support documentation if you find any difficulty on executing above steps.
https://developers.google.com/identity/sign-in/web/devconsole-project

Now, lets move ahead with Authorizationa and Adsense API calling.

```javascript
// Require node  modules
var fs = require('fs');
var readline = require('readline');
var google = require('googleapis');
var googleAuth = require('google-auth-library');
var adsense = google.adsense('v1.4'); // We can use any google's api here [https://github.com/google/google-api-nodejs-client/tree/master/apis]

// Delete your old credentials from  ~/.credentials/drive-nodejs-quickstart.json if modifying scope
var SCOPES = ['https://www.googleapis.com/auth/adsense'];
var TOKEN_DIR = '/.credentials/';// We can use, (process.env.HOME || process.env.HOMEPATH || process.env.USERPROFILE)
var TOKEN_PATH = TOKEN_DIR + 'drive-nodejs-quickstart.json';

// Load client secrets from a local file.
fs.readFile('client_secret.json', function processClientSecrets(err, content) {
  if (err) {
    console.log('Error loading client secret file: ' + err);
    return;
  }
  // Authorize a client with the loaded credentials
  authorize(JSON.parse(content), adsenseCallback);
});

/**
 * Create an OAuth2 client with the given credentials, and then execute the given callback function.
 * @param {Object} credentials The authorization client credentials.
 * @param {function} callback The callback to call with the authorized client.
 */
function authorize(credentials, callback) {
  console.log("inside authorize",credentials);
  var clientSecret = credentials.installed.client_secret;
  var clientId = credentials.installed.client_id;
  var redirectUrl = credentials.installed.redirect_uris[0];
  var auth = new googleAuth();
  var oauth2Client = new auth.OAuth2(clientId, clientSecret, redirectUrl);

  // Check if we have previously stored a token.
  fs.readFile(TOKEN_PATH, function(err, token) {
    if (err) {
      getNewToken(oauth2Client, callback);
    } else {
      oauth2Client.credentials = JSON.parse(token);
      callback(oauth2Client);
    }
  });
}

/**
 * Get and store new token after prompting for user authorization, and then
 * execute the given callback with the authorized OAuth2 client.
 *
 * @param {google.auth.OAuth2} oauth2Client The OAuth2 client to get token for.
 * @param {getEventsCallback} callback The callback to call with the authorized
 *     client.
 */
function getNewToken(oauth2Client, callback) {
  var authUrl = oauth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: SCOPES
  });
  console.log('Authorize this app by visiting this url: ');
  console.log(authUrl);
  var rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
  });
  rl.question('Enter the code from that page here: ', function(code) {
    rl.close();
    oauth2Client.getToken(code, function(err, token) {
      if (err) {
        console.log('Error while trying to retrieve access token', err);
        return;
      }
      oauth2Client.credentials = token;
      storeToken(token);
      callback(oauth2Client);
    });
  });
}

/**
 * Store token to disk be used in later program executions.
 *
 * @param {Object} token The token to store to disk.
 */
function storeToken(token) {
  try {
    fs.mkdirSync(TOKEN_DIR);
  } catch (err) {
    if (err.code != 'EEXIST') {
      throw err;
    }
  }
  fs.writeFile(TOKEN_PATH, JSON.stringify(token));
  console.log('Token stored to ' + TOKEN_PATH);
}

/**
 * Call Adsense generate api
 *
 * @param {google.auth.OAuth2} auth An authorized OAuth2 client.
 */
function adsenseCallback(auth) {
  console.log("inside adsenseCallback");
  var parameter = {
        accountId: 'Your Adsense ID, pub----',
        // currency : ,
        dimension : ["DATE","AD_FORMAT_CODE","AD_FORMAT_NAME"],
        // filter : ,
        // locale : ,
        // maxResults : ,
        metric  :["EARNINGS","AD_REQUESTS","AD_REQUESTS_RPM","CLICKS","COST_PER_CLICK","INDIVIDUAL_AD_IMPRESSIONS_RPM"] ,
        // sort : ,
        startDate: '2016-09-01',
        endDate: '2016-09-30',
        // startIndex : ,
        // useTimezoneReporting : ,
        auth: auth
    };
    adsense.reports.generate(parameter, function(err, response) {
     console.log("ERR :::: " + err);
     console.log("Response :::: " + JSON.stringify(response));
    });
```

* Save above code with testAdsense.js inside testgoogleapis directory
* Paste the client_secret.json to testgoogleapis directory
* create .credentials folder in testgoogleapis directory
* Install required node_modules using npm install and run it
```sh
cd testgoogleapis
npm install googleapis
node testAdsense
```
* When you run above code, it will ask for authorization in command line, go to link and authorize it using your login
* After approval, it will redirect to url which you have configured in client_secret.json, it will append access taken to rediret url in redirecturl?code=AASXXCXC21454 parameter. copy the code and paste it in command line and press Enter, if it validated, adsense api will be called and return json of respoense.
