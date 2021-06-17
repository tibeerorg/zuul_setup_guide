# Zuul getting started - with GitHub 

This documentation is based on https://zuul-ci.org/docs/zuul/tutorials/quick-start.html
and NOT purposed for production usage.

### Get yourself a zuul instance

You might want to use local `docker` or some VM (with `docker`).
`docker-compose` is also required.

### Prepare GitHub app (on your machine)

For this demo, I will stick with the GitHub app method and NOT the webhook method.
You can either create one for your organization or for your account.
If can be public or private, though private would make more sense.
Otherwise other people could install them in their repository and might abuse
them for crypto-minig, etc.

1. Go to the settings of your account/organization
2. Select `Developer settings` on the left hand side
3. Select `GitHub Apps` on the left hand side
4. Select `New GitHub App` on the right hand side
5. Select a unique `GitHub App name`, e.g. ***tibeerorg-zuul***
6. Select a dummy `Homepage URL` (this URL is opened when installing this app to a repository). I used ***https://www.google.com***
7. Set the `Webhook URL` to
   ```xml
   http://<zuul-hostname>:<port>/api/connection/<connection-name>/payload
   ```
   where
   - `zuul-hostname` is the IP or DNS name of your server
   - `port` is the Zuul port (typically ***9000***)
   - `connection-name` is the name you will later set in your zuul configuration. In this case I use ***githubzuulapp***
8. Set the `Webhook secret` to a value of your choice and save it for later. I'll use ***wekhooksecretvalue***
9. Select the following `Repository persmissions`:
   | Permission        | Access           |
   |-------------------|------------------|
   | *Administration*  | **Read-only**    |
   | *Checks*          | **Read & write** |
   | *Contents*        | **Read & write** |
   | *Issues*          | **Read & write** |
   | *Pull requests*   | **Read & write** |
   | *Commit statuses* | **Read & write** |
10. Select the events you whish to subscribe to:
  - *Check run*
  - *Commit comment*
  - *Create*
  - *Issue comment*
  - *Issues*
  - *Label*
  - *Pull request*
  - *Pull request review*
  - *Pull request review comment*
  - *Push*
  - *Release*
  - *Status*
11. Select `Where can this GitHub App be installed?` to ***Only on this account***
12. Click on `Create GitHub App`
13. Save also the `App ID` for later (in our case ***121873***)
14. Scroll down to `Private keys` and click ***Generate a private key***
15. Save the received *pem* file for later
16. Install the GitHub app to your whished repositories / accounts / repositories by clicking on `Install App` on the left hand side.

Note: sometimes it might happen that GitHub does not save your `Webhook secret`. Just re-enter it and save it again.

### Prepare Zuul config (on your Zuul instance)

Clone the example repository
```sh
git clone https://opendev.org/zuul/zuul
```

Replace all content with the equivalant content below.
Note the name of the `source` stanza matches your connection name you put earlier in the `Webhook URL`.
Note that at least the `config-projects` GitHub respositories need to exists before
you start your Zuul services. You may use the one from this organization, but I would suggest
to clone it to your account/organization (then you will also be able to set the log server).
The *opendev.org* connection is highly recommended. The base jobs are based on this job collection.
By defining *include* you can restrict zuul to only load jobs and not something like pipelines.

*~/zuul/doc/source/examples/etc_zuul/main.yaml*
```yaml
- tenant:
    name: showcase
    source:
      githubzuulapp:
        config-projects:
          - tibeerorg/zuul_config:
              load-branch: main
        untrusted-projects:
          - tibeerorg/zuul_demo_repo
      opendev.org:
        untrusted-projects:
          - zuul/zuul-jobs:
              include:
                - job
```

Throw away all "connection" sections and replace all of them with these new two.
Note, that you need to insert your `webhook_token` and `app_id` values you saved
earlier. Also note that your connection name needs to match the name you put earlier
in the `Webhook URL`. Otherwise you won't get a connection.

*~/zuul/doc/source/examples/etc_zuul/zuul.conf*
```ini
[connection "githubzuulapp"]
driver=github
webhook_token=wekhooksecretvalue
app_id=121873
app_key=/etc/zuul/github.pem

[connection "opendev.org"]
name=opendev
driver=git
baseurl=https://opendev.org
```

The `github.pem` file is not there yet. Use the *pem* file you saved earlier and upload
it to your Zuul instance to this file: `~/zuul/doc/source/examples/etc_zuul/github.pem`

### Start zuul services (on your zuul instance)

```sh
cd zuul/doc/source/examples
sudo -E docker-compose -p zuul-tutorial up -d
```

This might take a few minutes.

### Check the connection (on your machine)

Go back to your GitHub app settings:
1. Open the `Settings`
2. Click on `Installed GitHub Apps` on the left hand side
3. Click `Configure` on the right hand side
4. Click `App settings` on the top
5. Click `Advanced` on the left hand side
6. Check if the top entry has a green tick (if not, click on ***Redeliver***)

### Access your Zuul (on your machine)

You should now be able to access your Zuul installation by directing your
browser to the IP address or DNS name of your server followed by the port (typically ***9000***)
