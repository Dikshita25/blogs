---
title: "Upload files to drive in NodeJs with encryption"
template: "post"
category: "GoogleDrive"
tags:
  - "Nodejs"
  - "googledrive"
  - "Javascript"
published: true
draft: false
date: "2021-05-17T23:46:37.121Z"
slug: "/posts/upload-files-googleDrive"
description: "In this guide, we'll explore how to automatically upload files to a cloud storage service, like Google Drive, using Node.js while ensuring that the files are encrypted for security. "
---

This article is an extension to my previous article [generate front-end performance report](https://dikshitashirodkar.com/lighthouse_with_ga/), where we generated performance. In this article we are going to upload the reports to Google Drive.

## Why upload files to Google Drive?

So as testers, we always face challenges when it comes to reporting. Reporting can be in any form, but today we are going talk about reports generated when we run our e2e automation suites or reports which are generated when we run performance tests.

## Where do we store them?

If running the e2e tests via a CI/CD pipeline, let's take a example of `GitActions`. GitAction, spins a server when a workflow is triggered and it does the necessary jobs, provides us a report which are mostly preffered to in `html` format. But where to store them or how to access those reports?

There are various options available, like _posting into a database or uploading the reports to google drive_.

Yes you heard that correct!!. Google drive provides API which allows you to upload files in any format onto GoogleDrive.

So in today's blog, we are going to learn on how to upload files to google drive.

####`Before we start, here are some prerequisite:`####

Enable the google drive API, and generate a client credentials file and download the file on your local machine.

1. Enable the googleAPI
<figure class="float-center" style="width: 1000px">
	<img src="/media/googleDrive.png" alt="googleDrive">
</figure>

2. Generate the credentials File, that will have a clientID & clientSecret.
<figure class="float-center" style="width: 1000px">
	<img src="/media/credentials.png" alt="credentials">
</figure>

_*Note: This file should not be expose to others for security reasons.*_

Since the leak of such files are risky, let's us encrypt this files using a [json encryption](https://www.npmjs.com/package/json-file-encrypt) package.

Install the package using below command:

```
npm install json-file-encrypt

```

4. Create a `cryptography.js` that will help us to encrypt and decrypt any file which is in `json` format. Also the below code will automatically identify the whether the file needs to be encrypted or decrypted based on the extension of the file.<br>

_*Note: Encypted file will always have `.enc` extension in the filename.*_

```
const fs = require('fs');
const jfe = require('json-file-encrypt');

// path of the file to encrypt or decrypt
const filePath = process.argv[2];

// key to encrypt or decrypt any file
const key = process.argv[3];

//generating encryption key
const encryptionKey = new jfe.encryptor(key);

const content = fs.readFileSync(filePath);
if (filePath.includes('enc')) {
  const decryptedConfig = encryptionKey.decrypt(content.toString());
  const newFilePath = filePath.slice(0, filePath.length - 4);
  fs.writeFile(newFilePath, decryptedConfig, (err) => {
    if (err) {
      console.log('Error found', err);
      return;
    }
    console.log('Sucessfully decrypted file at', newFilePath);
  });
} else {
  const encryptedContent = encryptionKey.encrypt(content);
  const newFilePath = `${filePath}.enc`;
  fs.writeFile(newFilePath, encryptedContent, (err) => {
    if (err) {
      console.log('Error found', err);
      return;
    }
    console.log('Sucessfully encrypted file at', newFilePath);
  });
}

```

- Here's a command to encrypt or decrypt the file

```
node cryptography.js <Path of fileToEncrypt or fileToDecrypt> <keyToencrypt>
```

Example:

```
node cryptography.js ./client_secret.json xyZ@837YTVXW
```

The above command will encrypt or decrypt the file in a `.enc` extension

5. Now, lets install the Google API package, simply by using the command:

```
npm install googleapis --save
```

6. Here's the script `gdrive-auth-util.js` which exports an `authorize` function to access google drive.

```
const fs = require('fs');
const readline = require('readline');
const {google} = require('googleapis');
const jfe = require('json-file-encrypt');

const SCOPES = ['https://www.googleapis.com/auth/drive'];
const TOKEN_PATH = './token.json';

//generating encryption key
const encryptionKey = new jfe.encryptor(process.argv[2]);

// Reading the encrypted credentials.json.enc file and decrypting it
const authorize = (credentials, callback) => {
  const {client_secret, client_id, redirect_uris} = credentials.installed;
  const oAuth2Client = new google.auth.OAuth2(
    client_id,
    client_secret,
    redirect_uris[0],
  );
  fs.readFile(TOKEN_PATH, (err, token) => {
    if (err) {
      return getAccessToken(oAuth2Client, callback);
    }
    const decryptToken = encryptionKey.decrypt(token.toString());
    oAuth2Client.setCredentials(JSON.parse(decryptToken));
    callback(oAuth2Client);
  });
};

const getAccessToken = (oAuth2Client, callback) => {
  const authUrl = oAuth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: SCOPES,
  });
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  rl.question('Enter the code from that page here: ', (code) => {
    rl.close();
    oAuth2Client.getToken(code, (err, token) => {
      if (err) return console.error('Error retrieving access token', err);
      oAuth2Client.setCredentials(token);

      fs.writeFile(TOKEN_PATH, JSON.stringify(token), (err) => {
        if (err) return console.error(err);
      });
      callback(oAuth2Client);
    });
  });
};

module.exports = authorize;

```

7. Finally create a file `uploadToDrive.js` which uploads the `html` Report to google drive. Also you can pass the `parentID` (path of specific folder) you want to upload in.

```
/** @format */

const fs = require('fs');
const {google} = require('googleapis');
const jfe = require('json-file-encrypt');

const {authorize} = require('./gdrive-auth-util.js');

//generating encryption key
const encryptionKey = new jfe.encryptor(process.argv[2]);

// format the date for fileName
const date = new Date();
// eslint-disable-next-line max-len
const getTimeStamp = `${date.getFullYear()}-${date.getMonth()}-${date.getDate()}-${date.getHours()}-${date.getMinutes()}`;

// Creating the report fileMetaData
const fileMetaData = {
    fileName: 'example.html',
    name: `${getTimeStamp}-example.html`,
    // parentID helps to upload file to a particular folder
    parents: [asdjaskd2834728sdfs],
};

const uploadFile = (drive) => {
  var media = {
    mimeType: 'text/html',
    //Path of the report file
    body: fs.createReadStream(`./${fileMetaData.fileName}`),
  };
  drive.files.create(
    {
      resource: fileMetaData,
      media: media,
      fields: 'id',
    },
    (err, file) => {
      if (err) {
        // Handle error
        console.error(err);
      }

      console.log('Successfully uploded to drive!..')
    },
  );
};

const storeFiles = (auth) => {
  const drive = google.drive({version: 'v3', auth});
  uploadFile(drive);
};

const main = () => {
  fs.readFile('./client_secret.json.enc', (err, content) => {
    if (err) return console.log('Error loading client secret file:', err);
    const decryptCredentials = encryptionKey.decrypt(content.toString());
    authorize(JSON.parse(decryptCredentials), storeFiles);
  });
};

if (require.main === module) {
  try {
    main();
  } catch (e) {
    console.log(e);
  }
} else {
  module.exports = main;
}

```

A token is generated the very first time the authentication is made with Google drive (by executing the `uploadToDrive.js`). Copy the token and paste it into the `token.json` file. Encrypt the `token.json` file to `token.json.enc` for more security.

_Note: modifying the scopes or changing the permission to access your google drive will alter the user's access and refresh tokens, which is created automatically when the authorization flow completes for the first time._

Run the command to upload the file to google drive:

```
node uploadToDrive.js xyZ@837YTVXW
```

There you go.. Files will be uploaded to appropriate folder...

Happy reading 😄
