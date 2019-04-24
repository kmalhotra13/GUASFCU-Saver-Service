# GUASFCU Saver Service

[![Build Status](https://travis-ci.com/kmalhotra13/GUASFCU-Saver-Service.svg?branch=master)](https://travis-ci.com/kmalhotra13/GUASFCU-Saver-Service)

The purpose of this service is to automate savings for college students who might not have the ability to save by traditional means. The main service currently deployed is the Round Up Saver Tool. 

The tool functions via its remote server deployment, and every 10 minutes it gathers all new debit card transactions made out of the linked account. From there, the service aggregates the Penny value of the difference between each transaction amount and the next dollar, and deposits that value into savings. 

Through this methodology, for every cup of coffee you buy at $2.25, you will save $0.75. 

## Technical Prerequisites

- Git
- Anaconda 3.7
- Python 3.7
- Pip

## Banking Prerequisites

In order to utilize this service, you must have an account with GUASFCU. You can create this account by signing up at [www.guasfcu.com](www.guasfcu.com).

You must also have obtained an API key from GUASFCU. This key can be obtained by signing up with [Narmi](https://www.narmi.com/developers/guides/).

After you have received your token and secret, update the contents of the ".env" file to specify these values as environment variables called `token` and `secret`, respectively.


### SMS Prerequisities

For SMS capabilities, [sign up for a Twilio account](https://www.twilio.com/try-twilio), click the link in a confirmation email to verify your account, then confirm a code sent to your phone to enable 2FA.

Then [create a new project](https://www.twilio.com/console/projects/create) with "Programmable SMS" capabilities. And from the console, view that project's Account SID and Auth Token. Update the contents of the ".env" file to specify these values as environment variables called `TWILIO_ACCOUNT_SID` and `TWILIO_AUTH_TOKEN`, respectively.

You'll also need to [obtain a Twilio phone number](https://www.twilio.com/console/sms/getting-started/build) to send the messages from. After doing so, update the contents of the ".env" file to specify this value (including the plus sign at the beginning) as an environment variable called `SENDER_SMS`.

Finally, set an environment variable called `RECIPIENT_SMS` to specify the recipient's phone number (including the plus sign at the beginning).

## Installation

In order to set up this applet, please download install the source code:

```sh
git clone git@github.com:kmalhotra/GUASFCU-Saver-Service
cd GUASFCU-Saver-Service/
```

Install the package dependencies:

```sh
pip install -r requirements.txt
```

From there, you'll need to obtain your account IDs for the account you have connected to your debit card (traditionally the checking account) and the account you want the money to be saved into (traditionally the savings account). To do so, run the following shell script, substituting in your Narmi token and secret:

```sh
token='XXXX'
secret='YYYY'

date=`date -u +'%Y-%m-%dT%H:%M:%SZ'`

signature=`echo -n "date: $date" | openssl dgst -sha256 -binary -hmac "$secret" | base64`

curl -H "Authorization: Bearer $token" -H "Date: $date" -H "Signature: keyId=\"$token\",algorithm=\"hmac-sha256\",headers=\"date\",signature=\"$signature\"" 'https://api.guasfcu.com/v1/accounts/''
```

then from the JSON response, copy the account ID for the checking account into your .env file as the `FROM_ACCT_ID` and the account ID for your savings account as the `TO_ACCT_ID`. A sample response can be found at /Sample Accounts Response.txt. You're looking to take the "id" filed of the accounts portion of the response. 

Create the last_tranID.txt file:

1. Create a file under the storage folder entitled 'last_tranID.txt' — the file path should be as follows `storage/last_tranID.txt`
2. Run the following script, and obtain the transaction ID for the first transaction (the most recent) that is obtained:

```sh
token='XXXX'
secret='YYYY'

date=`date -u +'%Y-%m-%dT%H:%M:%SZ'`

signature=`echo -n "date: $date" | openssl dgst -sha256 -binary -hmac "$secret" | base64`

curl -H "Authorization: Bearer $token" -H "Date: $date" -H "Signature: keyId=\"$token\",algorithm=\"hmac-sha256\",headers=\"date\",signature=\"$signature\"" 'https://api.guasfcu.com/v1/accounts/FROM_ACCT_ID/transactions/''
```

3. Paste that transaction ID into the last_tranID.txt file, without any other characters. See the storage/last_tranID_example.txt for more information.