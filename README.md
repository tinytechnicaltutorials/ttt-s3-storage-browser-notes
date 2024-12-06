# AWS Project - S3 Storage Browser

This repo contains the code snippets used in this [YouTube video](https://youtu.be/UwnuPgNnmUg ).  The video is based loosely on this [AWS Blog](https://aws.amazon.com/blogs/aws/connect-users-to-data-through-your-apps-with-storage-browser-for-amazon-s3).

## TL;DR
Storage Browser for S3 is an AWS Amplify UI React component that allows external users to access your data stored in S3.  This video walks through how to set up a new Next.js App Router Amplify project (using the [Quickstart guide](https://docs.amplify.aws/nextjs/start/quickstart/nextjs-app-router-client-components/
)), add authentication, storage, and then add the Storage Browser UI component.

## Install the storage-specific package from the Amplify UI library
```
npm i @aws-amplify/ui-react-storage aws-amplify
```


## Authenticator code, in app\page.tsx
At the top of the file, add:
```
import { Authenticator } from '@aws-amplify/ui-react';
```

Update the ```return``` statement
```
 return (
    <Authenticator>
      {({ signOut, user }) => (
        <main>
          <h1>Hello {user?.username}</h1>
          <button onClick={signOut}>Sign out</button>
        </main>
      )}
    </Authenticator>
  );
```

## Instantiate storage and access rules, in amplify\storage\resource.ts
```
import { defineStorage } from '@aws-amplify/backend';

export const storage = defineStorage({
  name: 'myS3Bucket',
  access: (allow) => ({
    'public/*': [
      allow.guest.to(['read']),
      allow.authenticated.to(['read', 'write', 'delete']),
    ],
    'protected/{entity_id}/*': [
      allow.authenticated.to(['read']),
      allow.entity('identity').to(['read', 'write', 'delete'])
    ],
    'private/{entity_id}/*': [
      allow.entity('identity').to(['read', 'write', 'delete'])
    ]
  })
});
```

## Hook up backend to storage, in amplify\backend.ts:
At the top of the file, add:

```import { storage } from './storage/resource';```

In the ```defineBackend``` function, add ```storage```

```
defineBackend({
  auth,
  data,
  storage
});
```

## Component for StorageBrowser

At the root of the app (at the same level as ```app``` and ```public``` folders), create a folder called ```components```.

Inside the folder, create a file called ```StorageBrowser.tsx```.

Add the following code to ```StorageBrowser.tsx```:
```
import React from 'react';
import { createAmplifyAuthAdapter, createStorageBrowser } from '@aws-amplify/ui-react-storage/browser';
import '@aws-amplify/ui-react-storage/styles.css';
import { Amplify } from 'aws-amplify';
import config from '../amplify_outputs.json';

// Configure Amplify using the imported configuration
Amplify.configure(config);

// Create the StorageBrowser component with Amplify authentication
export const { StorageBrowser } = createStorageBrowser({
  config: createAmplifyAuthAdapter(),
});
```

## Add StorageBrowser to UI in app\page.tsx

At the top of the file, add:

```import { StorageBrowser } from '../components/StorageBrowser';```

Update the return statement to:
```
  return (
    <Authenticator>
      {({ signOut, user }) => (
        <main>
            <h1>Hello {user?.username}</h1>
            <button onClick={signOut}>Sign out</button>

          {/* StorageBrowser Component */}
          <h2>Your Files</h2>
          <StorageBrowser />

        </main>
      )}
    </Authenticator>
  );
}
```
