# GTM Template: Consent Management

This GTM template manages consent states based on cookie values and default settings. It sets default consent states, processes consent changes, and updates the consent state accordingly.

## Setup

To use this template, follow the steps below:

1. Copy the code below into your GTM Custom Template Editor.
2. Customize the `COOKIE_NAME` if necessary.
3. Define your default consent settings in the `defaultSettings` array within the `data` object.

## Code

```javascript
// The first two lines are optional, use if you want to enable logging
const log = require('logToConsole');
log('data =', data);
const setDefaultConsentState = require('setDefaultConsentState');
const updateConsentState = require('updateConsentState');
const getCookieValues = require('getCookieValues');
const JSON = require('JSON');
const COOKIE_NAME = 'cookies_consent';

/*
 *   Splits the input string using comma as a delimiter, returning an array of
 *   strings
 */
const splitInput = (input) => {
  return input.split(',')
      .map(entry => entry.trim())
      .filter(entry => entry.length !== 0);
};

/*
 *   Processes a row of input from the default settings table, returning an object
 *   which can be passed as an argument to setDefaultConsentState
 */
const parseCommandData = (settings) => {
  const regions = splitInput(settings['region']);
  const granted = splitInput(settings['granted']);
  const denied = splitInput(settings['denied']);
  const commandData = {};
  if (regions.length > 0) {
    commandData.region = regions;
  }
  granted.forEach(entry => {
    commandData[entry] = 'granted';
  });
  denied.forEach(entry => {
    commandData[entry] = 'denied';
  });
  return commandData;
};

/*
 *   Called when consent changes. Assumes that consent object contains keys which
 *   directly correspond to Google consent types.
 */
const onUserConsent = (consent) => {  
  const cookieValues = consent[0];
  const cookieObj = JSON.parse(cookieValues);
  const consentModeStates = {
    ad_storage: cookieObj.ad_storage,
    ad_user_data: cookieObj.ad_user_data,
    ad_personalization: cookieObj.ad_personalization,
    analytics_storage: cookieObj.analytics_storage,
    functionality_storage: cookieObj.functionality_storage,
    personalization_storage: cookieObj.personalization_storage,
    security_storage: cookieObj.security_storage,
  };    
  updateConsentState(consentModeStates);
};

/*
 *   Executes the default command, sets the developer ID, and sets up the consent
 *   update callback
 */
const main = (data) => {
  // Set default consent state(s)
  data.defaultSettings.forEach(settings => {
    const defaultData = parseCommandData(settings);
    // wait_for_update (ms) allows for time to receive visitor choices from the CMP
    defaultData.wait_for_update = 500;
    setDefaultConsentState(defaultData);
  });

  // Check if cookie is set and has values that correspond to Google consent
  // types. If it does, run onUserConsent().
  const settings = getCookieValues(COOKIE_NAME);
  if (typeof settings !== 'undefined') {
    onUserConsent(settings);
  }  
};

main(data);
data.gtmOnSuccess();

Functions
splitInput(input)
Splits a comma-separated string into an array of trimmed strings, excluding empty strings.

parseCommandData(settings)
Processes a row of input from the default settings table and returns an object suitable for setDefaultConsentState.

onUserConsent(consent)
Updates the consent state based on the cookie values.

main(data)
Sets the default consent state using data.defaultSettings.
Checks for existing consent cookies and updates the consent state if present.
Usage
Define Default Consent Settings:
data.defaultSettings should contain an array of objects with region, granted, and denied keys.

Set Cookie Name:
Ensure the COOKIE_NAME matches the name of your consent cookie.

Logging:
Enable logging by uncommenting the first two lines for debugging purposes.

Invoke Template:
Ensure the main(data) function is called with the appropriate data object to initialize consent management.
```
## Functions

### `splitInput(input)`
Splits a comma-separated string into an array of trimmed strings, excluding empty strings.

### `parseCommandData(settings)`
Processes a row of input from the default settings table and returns an object suitable for `setDefaultConsentState`.

### `onUserConsent(consent)`
Updates the consent state based on the cookie values.

### `main(data)`
Sets the default consent state using `data.defaultSettings`.  
Checks for existing consent cookies and updates the consent state if present.

## Usage

### Define Default Consent Settings:
`data.defaultSettings` should contain an array of objects with `region`, `granted`, and `denied` keys.

### Set Cookie Name:
Ensure the `COOKIE_NAME` matches the name of your consent cookie.

### Logging:
Enable logging by uncommenting the first two lines for debugging purposes.

### Invoke Template:
Ensure the `main(data)` function is called with the appropriate data object to initialize consent management.
