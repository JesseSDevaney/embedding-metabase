# Running Metabase in a Cross-Site Interactive Embedding Context with SSO

## Table of Contents

1. [Setup Custom Local Domain Name](#setup-custom-local-domain-name)
   - Linux and MacOS
   - Windows
2. [Setup Metabase and HTTPS](#setup-metabase-and-https)
   - Generate Java Keystore
   - Run Metabase with Environment Variables
   - Setup Metabase Pro/Enterprise Version
   - Enable Interactive Embedding
3. [Setup SSO](#setup-sso)
   - Enable SSO with JWT
4. [Setup Embedding App](#setup-embedding-app)
   - Set environment variables
   - Running the Embedded App

## Setup Custom Local Domain Name

https://github.com/JesseSDevaney/embedding-metabase/assets/22608765/b0d7f986-348d-44d1-b1d2-03972f784403

In order for cross-site embedding to work locally, we have to setup our localhost under a custom domain.

### Linux and MacOS

On Linux and MacOS, you can edit the file `/etc/hosts` adding an entry with your custom domain name.

```sh
sudo nano /etc/hosts
```

You should already see an entry like

```sh
127.0.0.1 localhost
```

in `/etc/hosts`.

We want to add an additional entry, so we can serve Metabase under localhost and our embedding app under a different domain.

Add an entry to `/etc/hosts` and make sure the second to last digit is different from your localhost. i.e. `127.0.0.1 localhost` has a different localhost ip then `127.0.1.1`.

```sh
127.0.1.1 embedding.local
```

Now when you run apps locally, you have two links you can access them through: `http://localhost` and `http://embedding.local` and they will be recognized as different sites by your browser.

**Note:** You may need to flush your DNS cache for these changes to take effect immediately. Test the changes first and if it does not work immediately, check your local system requirements for how to flush your DNS cache.

### Windows

Not tested on Windows but works in the same vain. `/etc/hosts` should be under `\Windows\System32\drivers\etc\hosts` instead.

## Setup Metabase and HTTPS

https://github.com/JesseSDevaney/embedding-metabase/assets/22608765/6db651ab-700a-4355-bdf4-d89109968ab1

### Generate Java Keystore to Run Metabase as HTTPS

1. Get the keytool tool (should be already installed with Java 11+)
2. Generate the keystore to store the cerficate files

```
keytool -genkey -keyalg RSA -alias localhost -keystore selfsigned.jks -validity 365 -keysize 2048
```

when asked about password use `storepass` and when asked about several inputs, always use `localhost`

when it says

```none
Is CN=localhost, OU=localhost, O=localhost, L=localhost, ST=localhost, C=localhost correct?
```

you should enter "yes"

3. when you finish creating the keystore, try checking if everything is cool with the following:

`keytool -list -keystore selfsigned.jks`

you should see something similar to

```none
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

localhost, Apr 4, 2023, PrivateKeyEntry,
Certificate fingerprint (SHA-256): xx:xx:xx:xx:...
```

### Running Metabase

In your Metabase app directory, start the developer instance with these SSL variables exported.

```bash
export MB_JETTY_SSL=true
export MB_JETTY_SSL_PORT=8443
export MB_JETTY_SSL_KEYSTORE=[/path/to/keystore.jks]
export MB_JETTY_SSL_KEYSTORE_PASSWORD=storepass
yarn dev-ee
```

Metabase should now be available at [https://localhost:8443](https://localhost:8443).

### Setup Metabase Pro/Enterprise Version

You'll need a Pro or Enterprise version of Metabase up and running use JWT SSO.

1. Setup Metabase as Pro/Enterprise Version
   - Add you license token in admin settings > license & billing

### Enable Interactive Embedding

https://github.com/JesseSDevaney/embedding-metabase/assets/22608765/706feb06-226b-483d-9619-4953cc654a25

1. Go to Admin Settings > Embedding
2. Click Enable
3. Go to Admin Settings > Embedding > Interactive Embedding
4. For your embedding app, add to authorized origins: `http://embedding.local:8081`
5. (if using SSO) Set SameSite cookie to `none`

## Setup SSO

https://github.com/JesseSDevaney/embedding-metabase/assets/22608765/4f225c54-b4a6-4e0f-98d2-3f366fbee310

### Enable SSO with JWT

Go to **Admin Settings** > **Settings** > **Authentication**.

On the card that says **JWT**, click the **Setup** button.

#### JWT Identity provider URI

In **JWT IDENTITY PROVIDER URI** field, paste `http://embedding.local:8081/login`.

#### String used by the JWT signing key

Click the **Generate key** button. Copy the key.

## Setup Embedding App

https://github.com/JesseSDevaney/embedding-metabase/assets/22608765/6a588f6d-c79b-455d-8b74-c66c7c56781a

### Set environment variables

You'll need to set some environment variables for your server.

- [METABASE_SITE_URL](#metabase_site_url)
- [METABASE_JWT_SHARED_SECRET](#metabase_jwt_shared_secret)
- [METABASE_DASHBOARD_PATH](#metabase_dashboard_path)

#### METABASE_SITE_URL

```sh
export METABASE_SITE_URL="https://myapp.metabaseapp.com"
```

Replacing `https://myapp.metabaseapp.com` with the root path of your Metabase. In the local case, `https://localhost:8443`

#### METABASE_JWT_SHARED_SECRET

```sh
export METABASE_JWT_SHARED_SECRET="COPY_SECRET_FROM_JWT_CONFIG"
```

You can get this key from your Metabase by clicking on the **gear** icon and going to **Admin Settings** > **Settings** > **Authentication** > **JWT**.

#### METABASE_DASHBOARD_PATH

Find the dashboard you want to redirect to on initial login and its id. Then export the variable below replacing `id` with the ID number of your dashboard.

```sh
export METABASE_DASHBOARD_PATH="/dashboard/id"
```

#### (Optional) Setting Up an Already Defined User in SSO

If you have a specific user you know you want to test in the embed, you can add that user's email to the JWT SSO and sign-in with that user's email.

```sh
export SSO_CONNECTED_EMAIL="read-only@metabase.com"
```

### Running the Embedded App

Start the server by running:

```sh
yarn dev
```

The app runs by default on port 8081.

Visit `http://embedding.local:8081/analytics` and sign in with your exported email and password as "password", or with the pre-defined user below

```
user: rene@example.com
password: password
```
