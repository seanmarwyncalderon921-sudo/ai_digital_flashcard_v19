# Firebase setup

1. Create a Firebase project and register a web app.
2. Enable Email/Password, Google, and Phone providers under Authentication.
3. Copy the web app values into `public/firebase-config.js`.
4. Create a service account under Project settings > Service accounts.
5. Put its project ID, client email, and private key into a local `.env` using `.env.example`.
6. In the project folder, sign in and connect this folder to your Firebase project:

```powershell
firebase login
firebase use --add
```

Choose the Firebase project you created. This creates a local `.firebaserc`
file containing the selected project ID.

7. Deploy `firestore.rules` from the Firebase CLI:

```powershell
firebase deploy --only firestore:rules
```

Chat attachments use Cloudinary. Create a Cloudinary account, copy the
**Cloud name** into `public/api-config.js`, and create an **unsigned upload
preset** under Settings → Upload presets. Copy that preset name into the same
file. Only the cloud name and unsigned preset belong in the browser; never add
the Cloudinary API secret there.

Cloudinary uploads are sent directly from the browser into the `sapc-chat`
folder. The app accepts images, GIFs, and PDFs up to 30 MB.

AI flashcard generation is limited to 150 prompts per local backend account,
and each request can produce at most 150 cards.

8. Start the server with `npm start`.

Friend search uses the `publicProfiles` collection, which contains only
display-name/profile fields. Friend requests are stored in `friendRequests`;
deploying `firestore.rules` is required before users can search or connect.

## Secure friend chat

Signed-in users can start a chat with any other account from search; only the
two conversation participants can read or write that conversation. The browser
creates an ECDH P-256 key pair for each signed-in account and stores the
private key locally in that browser. Messages are encrypted with AES-GCM
before they are written to Firestore; Firestore stores ciphertext and an IV,
not the message text.

This is device-local encryption: clearing browser storage, switching browsers,
or changing devices can make older messages unreadable. A future key-backup
feature should be added before treating chat history as recoverable. The
private key is also accessible to JavaScript running in the page, so standard
XSS prevention and a trusted deployment remain essential.

After changing chat rules, deploy them with:

```powershell
firebase deploy --only firestore:rules
```

The browser Firebase configuration is public by design. The service-account
private key and Groq keys must remain server-side and must never be added to
`public/` or committed to source control.

## Render backend

The Express AI backend is prepared for Render in `render.yaml`. Create a
Render Web Service from this project, set the secret environment variables
shown in the blueprint, and copy the resulting service URL into
`public/api-config.js`. Redeploy Firebase Hosting after changing that URL.

## Password-reset email branding

The password-reset email name and wording are controlled by Firebase, not by
the HTML app. In Firebase Console, open **Authentication → Templates →
Password reset**, then set the sender name and email subject to your website
name, such as **SAPC AI Academic Reviewer**. The reset link is configured to
return users to `https://ai-driven-academic-flashcards.web.app/`.
