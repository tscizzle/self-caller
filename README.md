# Fake Mom Call
Make it easy to set up an automated call to your own phone (like if you ever need to get out of something)

Steps
=====

1) make a tiny HTML page
2) make our button call a Javascript function
3) sign up for a service to let us connect code to texting/calling (Twilio)
4) make the Javascript function use Twilio to call yourself

It ends up only about 44 lines of code, plus some steps on Twilio's website.

HTML Page
---------

In Sublime Text, make a new HTML file, called `main.html`, with some boilerplate (i.e. common code to start with, not unique to your project).

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Fake Mom Call</title>
  </head>

  <body>

  </body>
</html>
```

In the body, put an `input` tag so you can specify your phone number,

```html
<input id="phone-number-to-call" placeholder="Phone Number" />
```

and then a `button` tag so you can request a phone call.

```html
<button>
  Call Me
</button>
```

(Note that the `input` tag has an `id`. This is so that in our Javascript later, we can reference that input element.)

Open `main.html` in Chrome or Firefox and make sure you can see an `input` and a `button` on the page.

Connect Button To Javascript
----------------------------

At the bottom of the body, add a `script` tag so you can define a click handler (which will be a Javascript function) for your `button`.

```html
<script>
  function requestCall() {

  }
</script>
```

In the function, we will utilize that `id` we put on the `input` element, by using `document.getElementById`.

```javascript
function requestCall() {
  var phoneNumberElement = document.getElementById("phone-number-to-call");
}
```

For now, let's just grab whatever is typed into the `input`, by using `.value`. Then pop up that value, by using `alert`.

```javascript
function requestCall() {
  var phoneNumberElement = document.getElementById("phone-number-to-call");
  var phoneNumber = phoneNumberElement.value;

  alert('Phone number typed in: ' + phoneNumber);
}
```

(Later we will make this function actually trigger a call to your phone.)

This function `requestCall` is defined, but it won't run yet because we aren't calling it anywhere.

To make it run whenever your `button` is clicked, add an `onclick` handler to your `button`.

```html
<button onclick="requestCall()">
  Call Me
</button>
```

Status check: so far, your `main.html` file should look like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Fake Mom Call</title>
  </head>

  <body>
    <input id="phone-number-to-call" placeholder="Phone Number" />

    <button>
      Call Me
    </button>

    <script>
      function requestCall() {
        var phoneNumberElement = document.getElementById("phone-number-to-call");
        var phoneNumber = phoneNumberElement.value;

        alert('Phone number typed in: ' + phoneNumber);
      }
    </script>
  </body>
</html>
```

And when you type something into your `input` and then click your `button`, the thing you typed should pop up.

Sign Up For Twilio
------------------

We will come back to that HTML page in a bit, once we have a Twilio account.

Twilio is a service that lets you make texts and phone calls right from your code.

Follow the below steps to set up a Twilio account.
- Go to https://www.twilio.com and click "Sign Up".
- Fill out the form. For the questions, the answers don't matter. If you want, you can put:
  - Which product do you plan to use first? "Voice"
  - What are you building? "Other"
  - Choose your language "Javascript"
  - Potential monthly interactions "Less than 100,000"
- Verify your phone number.

From this dashboard, see Project Info. See these 2 things:
1) Account SID
2) Auth Token (for this to be visible, you have to click the dots)

**Copy and paste them into Sublime Text, so you have them available later.**

Next, set up a random new phone number on Twilio. (When Twilio makes calls for you, it will be from this number.)
- Under "Phone Numbers", click "Manage Numbers", click "Get Started", click "Get your first Twilio phone number".
- A pop up should appear.
  - If next to "Voice:" it says "This number can receive incoming calls and make outgoing calls.", click "Choose this Number".
  - If it doesn't say that next to "Voice:", I guess click "Search for a different number". I haven't come across this situation yet though.
- **Copy and paste this phone number to wherever you saved your Account SID and Auth Token. You will need it later as well.**

(You can add this number to your Contacts as some funny name. Something you can show the person you're with and be like "Ooh, sorry, gotta take this.")

Soon we will trigger a phone call from running code. But first, let's test our setup, since Twilio lets you test that straight from their website.

To test your setup by triggering a Twilio phone call to your own phone:
- In the left menu, click the circle with the dots, click "Runtime", click "API Explorer", click "Calls", click "Make a Call".
- Under "Parameters", the 2 fields you need to fill in are "TO" (the number to call), and "URL" (says what to do when the call connects).
  - Fill in the "TO" field with your own real phone number.
  - Fill in the "URL" field with https://demo.twilio.com/welcome/voice/ (when the call connects, it will use this demo auto-response).
  - (The "FROM" field is already filled in for you as the randomly generated phone number from earlier.)
- Note that they show you something called a `curl` command in the panel on the right.
  - You can actually run the `curl` command from your Terminal if you wanted to, but we don't need to.
- Scroll down and click "Make Request".
- You should get a call on your real phone! It should say it's from the phone number that Twilio randomly generated earlier.
- Feel free to answer it to hear the demo auto-response.

Call Yourself From Code
-----------------------

Go back to `main.html` in Sublime Text. It's time to trigger that same phone call from your own web page.

We are going to make what's called an HTTP request to ask Twilio to trigger the phone call.

You have triggered HTTP requests before, in 2 different ways:

1) When you type www.google.com into your browser and press Enter, it sends an HTTP request, asking for Google's website to be sent to you.
2) When you clicked "Make Request" in Twilio's dashboard, it sent an HTTP request, asking for Twilio to make a phone call.
3) Making Twilio's suggested `curl` command from your Terminal would have also sent the same HTTP request.

In your web page, we will use yet another way to send that HTTP request: Javascript's built-in `fetch` function.

First, remove the `alert` line. Our new code will go where it was, underneath `var phoneNumber = phoneNumberInput.value;`.

Now, take the Account SID, Auth Token, and random phone number from earlier, and save them as variables so we can use them in the code.

```javascript
// These are from your Twilio account. Yours won't be the same as mine.
var TWILIO_ACCOUNT_SID = 'Ab98fa5386ae87611Ab98fa5386ae87611';
var TWILIO_AUTH_TOKEN = '43ea04641f5e525f43ea04641f5e525f';
var TWILIO_PHONE_NUMBER = '+17742069275';
```

Then, we need the target URL. HTTP requests are sent to a target URL (like www.google.com).

In this case, it's one I had to look up in Twilio's documentation:

```javascript
var callRequestURL = 'https://api.twilio.com/2010-04-01/Accounts/' + TWILIO_ACCOUNT_SID + '/Calls.json';
```

(Note that it depends on your Account SID. Different people's URL's will be slightly different because their Account SID's are different.)

Next, we define the parameters we send with this request. This is kind of like specifying arguments to a function.

```javascript
var myParameters = new URLSearchParams();
myParameters.set('To', phoneNumber);
myParameters.set('From', TWILIO_PHONE_NUMBER);
myParameters.set('Url', 'https://demo.twilio.com/welcome/voice/');
```

If you remember from when we made the test phone call from Twilio's website, these are the same parameters we had to fill in then ("To", "From", "Url").

One last thing before writing our `fetch` call, is Authorization.

If this part seems a little weird, that's ok. Just know it's kind of like "logging in" in order to make a request.

```javascript
var myHeaders = new Headers();
var username = TWILIO_ACCOUNT_SID;
var password = TWILIO_AUTH_TOKEN;
myHeaders.append('Authorization', 'Basic ' + btoa(username + ":" + password));
```

Finally, now that we have all the parts we need, we can make the HTTP request by calling `fetch`.

```javascript
fetch(callRequestURL, {
  method: 'POST',
  body: myParameters,
  headers: myHeaders,
});
```

At this point, your `main.html` file should look like this (but with some of your own account's values filled in):

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Fake Mom Call</title>
  </head>

  <body>
    <input id="phone-number-to-call" placeholder="Phone Number" />

    <button onclick="requestCall()">
      Call Me
    </button>

    <script>

      function requestCall() {
        var phoneNumberInput = document.getElementById("phone-number-to-call");
        var phoneNumber = phoneNumberInput.value;

        var TWILIO_ACCOUNT_SID = 'Ab98fa5386ae87611Ab98fa5386ae87611';
        var TWILIO_AUTH_TOKEN = '43ea04641f5e525f43ea04641f5e525f';
        var TWILIO_PHONE_NUMBER = '+17742069275';

        var callRequestURL = 'https://api.twilio.com/2010-04-01/Accounts/' + TWILIO_ACCOUNT_SID + '/Calls.json';

        var myParameters = new URLSearchParams();
        myParameters.set('To', phoneNumber);
        myParameters.set('From', TWILIO_PHONE_NUMBER);
        myParameters.set('Url', 'https://demo.twilio.com/welcome/voice/');

        var myHeaders = new Headers();
        var username = TWILIO_ACCOUNT_SID;
        var password = TWILIO_AUTH_TOKEN;
        myHeaders.append('Authorization', 'Basic ' + btoa(username + ":" + password));

        fetch(callRequestURL, {
          method: 'POST',
          body: myParameters,
          headers: myHeaders,
        });
      }
    </script>
  </body>
</html>
```

And when you type your phone number into your `input` and then click your `button`, you should receive a phone call!

Future Steps
------------

Possible next steps:
- delay the call by X seconds so it's not right after you click the button
- make the button a giant red circle button in the middle of the page
- host this website online, so you can use it for real from any computer or phone

Or anything else you all come up with!
