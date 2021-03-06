## Watching the Foreign Intelligence Surveillance Court

This project "watches" [the public docket of the FISC](http://www.fisc.uscourts.gov/), and upon any changes, alerts the public and the administrator through tweets, emails, and text messages.

It is a Ruby script that downloads the FISC's public docket and compares it against the last time it was run. If there are changes, any configured alert mechanisms (such as SMS or Twitter) will fire.

To use it, you should have a computer available that can automatically run the script every few minutes, all day throughout the day.

It's currently powering the [@FISACourt](https://twitter.com/fisacourt) Twitter account, maintained by [@konklone](https://twitter.com/konklone).

Reader: **do you work for the FISA Court?** If so, see [my notes at the bottom](#a-note-to-the-fisc).

### Background

The [Foreign Intelligence Surveillance Court](https://en.wikipedia.org/wiki/United_States_Foreign_Intelligence_Surveillance_Court) (FISC) is responsible, under the law known as [FISA](https://en.wikipedia.org/wiki/Foreign_Intelligence_Surveillance_Act), for overseeing the surveillance activity of the US executive branch.

It has operated since 1978, but had no public docket of orders or opinions until June of 2013. That month, amidst heightened public attention to national surveillance policies, the [EFF](https://www.eff.org) and the [ACLU](https://www.aclu.org) each filed motions aimed at unsealing FISC records, and successfully requested that these motions themselves become public.

To publish these, the FISC began operating a minimalist public docket that listed links to, mostly, scanned image PDFs. You can see [what that docket looked like in April of 2014](https://web.archive.org/web/20140416090332/http://www.uscourts.gov/uscourts/courts/fisc/index.html) courtesy of the Internet Archive.

On April 30th, 2014, the FISC launched a more full website at [fisc.uscourts.gov](http://www.fisc.uscourts.gov).


### Setup and Usage

You'll need a *nix-based system (not Windows) with `curl` and `wget` installed.

Install dependencies with bundler:

```bash
bundle install
```

Copy `config.yml.example` to `config.yml`, then uncomment and fill in any of the sections for `twitter`, `email`, and `twilio` (SMS) to enable those kinds of notifications. They're all optional. Details on enabling each of them are below.

Once configured, run the script to check the first page of FISC filings:

```bash
./check
```

If there are new filings, the new docket data will be committed to git, and any alert mechanisms you've configured will fire.

By default, `check` will check ETags for already downloaded PDFs, and only re-download PDFs if the ETags don't match. To force a re-download of PDFs, use `./check everything`.


### Testing alerts

To test out your alerts without requiring the FISA Court to actually update, run:

```bash
./check test
```

This will pretend a change was made and fire each of your alert mechanisms.

### Archiving everything

You can run the script with the `archive` command to re-scrape the entire site's metadata, *without* sending alerts and *without* commiting results.

```bash
./check archive
```

If changes are left in an uncommitted state, the next time a normal `./check` is run, it'll notice the changes and alert at that time.

By default, `check archive` will check ETags for already downloaded PDFs, and only re-download PDFs if the ETags don't match.

To force a re-download of all PDFs:

```bash
./check archive everything
```

If changes are made to how data is processed, running an `archive` command will safely re-process everything, and write to the `docket/` directory.

However, it is up to an admin to review, commit, and push the changes. It's recommended the cronjob be turned off during this process.

### Git

This project depends on its own git repository to detect and track changes. For git interaction to work correctly, you need to ensure:

* if a `git pull` is run, it will not generate a merge conflict (at HEAD, or can be fast-forwarded)
* there is a remote branch already, and the local branch is set to track it

This will generally already be the case for a clean checkout running on the master branch.

If the `git push` fails for some reason, it will continue on and alert the world (though it will not include a GitHub URL). It will also send an alert via Pushover, SMS, and email to the admin, if any of those are configured.

If the `git push` succeeds, but the remote branch is not configured correctly, it will post a GitHub URL to a non-existent commit (a 404). If the branch is then configured correctly and the commits pushed, the URL will then work as expected.

### GitHub integration

If you're using GitHub, then when FISC updates are detected you can have notification messages include a URL to view the change on GitHub.

To do this, set `config.yml`'s `github` value to `username/repo` (using your real username and repo name, e.g. `konklone/fisacourt`).

### Configuring alerts

Turn on different alert methods by uncommenting and filling out sections of `config.yml`.

**Email**

Currently, this project only supports sending emails through SMTP. In the future, it could be extended to support services like Postmark (send in a patch!).

```yaml
email:
  :to:
  :subject:
  :from:
  :via: :smtp
  :via_options:
    :address:
    :port:
    :user_name:
    :password:
    :enable_starttls_auto:
```

Put your email address in the `to` field, and the subject line you want in the `subject field`. The `from` field should be an email address you have permission to send from.

If you have a Gmail account, you have access to their SMTP server, but apparently Google doesn't like to talk about it. The best resources I found for figuring out the values are [here](http://email.about.com/od/accessinggmail/f/Gmail_SMTP_Settings.htm) and [here](http://support.qualityunit.com/107274-How-to-configure-Gmail-SMTP-settings-).

If you use [Pobox](http://pobox.com/) for forwarding (like I do: they're awesome), you can use [their SMTP details](https://www.pobox.com/helpspot/index.php?pg=kb.page&id=118).

**Text messages**

[Twilio](http://www.twilio.com/) is a cheap way to programmatically send text messages (SMS). This project uses Twilio, but there are also excellent alternatives, like [Tropo](https://www.tropo.com/). Both charge a penny, $0.01, for each text message sent (in the US).

```yaml
twilio:
  account_sid:
  auth_token:
  from:
  to:
```

Create a Twilio account, choose a phone number, and copy your Account SID and Auth Token to the `account_sid` and `auth_token` fields. Put your Twilio phone number in the `from` field, and the phone number you wish to receive text messages in the `to` field.


**Twitter**

The script can post to a Twitter account of your choice. You will need to tell Twitter about your application, but there is no delay, as they approve applications automatically (at this time).

```yaml
twitter:
  consumer_key:
  consumer_secret:
  oauth_token:
  oauth_token_secret:
```

To enable posting to Twitter, go to the [Twitter developer portal](https://dev.twitter.com/), and log in **as the account you wish to post from**.

Go to [My Applications](https://dev.twitter.com/apps) and create a new application. You will need to enter a name, description, and website. You do not need to supply a callback URL. Once created, go to the application's Settings tab and change the application's permissions from "Read only" to "Read and Write". Finally, create an access token using the form at the bottom. You may need to refresh the page after a minute to get the access token to show up.

Copy the "Consumer key" and "Consumer secret" to the `consumer_key` and `consumer_secret` fields. Copy the "Access token" and "Access token secret" to the `oauth_token` and `oauth_token_secret` fields.

**Pushover**

[Pushover](https://pushover.net/) provides basic push notifications for any device with the Pushover application installed ([Android](https://pushover.net/clients/android), [iOS](https://pushover.net/clients/ios)). The application costs $5, but messages are free (up to 7,500 per month), so Pushover may be a better choice than SMS if SMS is expensive or unavailable in your area.

You will need to tell Pushover about your application, but applications are automatically approved at this time.

```yaml
pushover:
  user_key:
  app_key:
```

To configure Pushover notifications:

* Go to the [Pushover Dashboard](https://pushover.net), create an account if needed, and log in. The dashboard will show your `user_key` underneath "Your User Key". Copy this into `config.yml`.
* Register a new application from the [New Application Page](https://pushover.net/apps/build) by entering the name and brief description of the application.
* This will bring you to your new application's detail screen. Copy the `app_key` field, underneath "API Token/Key", into `config.yml`.

And to enable them on your phone:

* Install the Pushover application. ([Android](https://pushover.net/clients/android), [iOS](https://pushover.net/clients/ios))
* Log into your Pushover account.
* Give your device a name and "add" it to Pushover.

### Atom Feed

You can use this URL to follow the FISA Court in your favorite feed reader:

> [https://github.com/konklone/fisacourt/commits/docket.atom](https://github.com/konklone/fisacourt/commits/docket.atom)

This works because the FISC's docket is versioned on the `docket` branch, and it is the **only** activity on that branch. So, GitHub's Atom feed for the `docket` branch is effectively a feed for FISA Court updates.

### A note to the FISC

If anyone from the FISC is reading this, and wondering why anyone would go to the trouble of scraping your brand new website (with an RSS feed and all):

* Your RSS feeds only show the last 10 items. For anyone to download the full metadata and contents of the docket of the FISC, the RSS feed is not useful.
* An RSS feed of 10 items also means that, should you ever upload more than 10 documents in one batch, some data will never appear on the RSS feed.
* Your RSS feed's `<description>` tags contain a raw HTML blob, which needs to be parsed in order to figure out anything useful (including the actual URL to download a file). So scraping your HTML is a necessity no matter what -- may as well just write a scraper for the whole thing.

There are some additional data problems:

* The publication dates for anything before 4/30 are mostly incorrect. Instead, the dates reflect some sort of batch upload process during the transition from the old to the new site.
* There are no real unique IDs for documents. URLs use slugs that are auto-generated based on order (e.g. `order-25` for the 25th document titled "Order" uploaded to your site). RSS feeds show a `438 at http://www.fisc.uscourts.gov`, which is clearly an auto-increment database ID (appearing nowhere visible on the site) that I would not want to stake my data interoperability on.

You could eliminate the need for anyone to scrape your website's HTML (and incur load on your systems, etc.) by providing **either** of:

* Improved, separate archival RSS feeds.
  * Paginate the RSS feed endpoints, so one can trace it back as far as needed.
  * Simplify the `<description>` tag to just a plaintext description.
  * (Ab)use `<category>` tags to include any other metadata.
* A web API to your content.
  * Your source code indicates you're using Drupal -- this ["services" plugin](https://drupal.org/project/services) seems to be reasonably popular and well-maintained.
  * Or, you may want to call up the FCC: [they used contentapi](http://www.fcc.gov/encyclopedia/content-api-drupal-module) on their site, though it looks like a less active project.
  * Or, invent your own. Feel free to take inspiration from other [APIs for public records](https://sunlightlabs.github.io/congress/).

Also, please assign unique IDs, and fix the publication dates for documents. Instead of reflecting the document's date of entry into the Drupal backend, the dates should reflect when they became public record.

One major issue (for you and me both, given the bandwidth) are all the PDFs. This system downloads every PDF once, and then uses [ETags](https://en.wikipedia.org/wiki/HTTP_ETag) to decide whether to ever re-download the PDF. However, ETags have no defined algorithm for calculating them, and without additional documentation, they can't be fully trusted. Because of this, the system has a mechanism to distrust ETags and force a re-download of everything. I'm not doing this very often, because of the expense. I could also use `Last-Modified` headers, but I'm honestly not sure I trust those either.

This expense could be mitigated by increasing that trust. That could be by publishing MD5SUMs or SHA hashes of each filing, in an index of some sort. Or, it could be by publicly documenting your algorithm for calculating ETags or `Last-Modified` headers. (You could do this last one by opening your website's source code. Open source is secure, I promise.)

Finally: please **provide a public email address**. That phone number to a voicemail machine is not an effective way to file bug reports, or, really, do anything at all.

## Public domain

This project is [dedicated to the public domain](LICENSE). As spelled out in [CONTRIBUTING](CONTRIBUTING.md):

> The project is in the public domain within the United States, and copyright and related rights in the work worldwide are waived through the [CC0 1.0 Universal public domain dedication](http://creativecommons.org/publicdomain/zero/1.0/).

> All contributions to this project will be released under the CC0 dedication. By submitting a pull request, you are agreeing to comply with this waiver of copyright interest.
