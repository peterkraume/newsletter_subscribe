# subscribe

* Subcribe for Newsletters for TYPO3 >= 9.5
* Depends on tt_address
* Scheduler Task to delete unconfirmed subscribers after a while (since v3.1.0)

## What does it do?

* Provides a plugin to double optin subscribe
* Provides a plugin to double optin unsubscribe
* Provides field in tt_address to generate unsubscribe link in direct_mail mailings

## Install
* Install via extension manager or
* Install via composer
* Include static template

## Configuration
<table>
<thead>
<tr>
<th>Parameter</th>
<th>Description</th>
<th>Type, Validation</th>
<th>Default</th>
</tr>
</thead>
  <tr>
    <th align="left">dataProtectionPage</th>
    <td align="left">Uid of page with information about the data protection policy</td>
    <td>integer</td>
    <td>1</td>
  </tr>
  <tr>
    <th align="left">adminEmail</th>
    <td align="left">Email address in admin notification mails</td>
    <td>string, email, mandatory</td>
    <td>admin.name@domain.tld</td>
  </tr>
  <tr>
    <th align="left">adminName</th>
    <td align="left">Name in admin notification mails</td>
    <td>string</td>
    <td>Your admin name</td>
  </tr>
  <tr>
    <th align="left">senderEmail</th>
    <td align="left">Email address in confirmation mails</td>
    <td>string, email, mandatory</td>
    <td>noreply@domain.tld</td>
  </tr>
  <tr>
    <th align="left">senderName</th>
    <td align="left">Name in confirmation mails</td>
    <td>string</td>
    <td>Your sender name</td>
  </tr>
  <tr>
    <th align="left">newsletterName</th>
    <td align="left">Title of the newsletter / subscription list</td>
    <td>string</td>
    <td>Newsletter</td>
  </tr>
  <tr>
    <th align="left">showFields</th>
    <td align="left">Additional fields to show in subscription form (gender,firstName,lastName,company).<br>email and dataProtection are always shown.</td>
    <td>string</td>
    <td>null</td>
  </tr>
  <tr>
    <th align="left">subscribePageUid</th>
    <td align="left">Uid of page with subscription form. Used for links in templates and mails.</td>
    <td>string</td>
    <td>null</td>
  </tr>
  <tr>
    <th align="left">useSimpleSpamPrevention</th>
    <td align="left">whether there should be a simple spam preventition using javascript and session (with <strong>session cookie</strong>)</td>
    <td>bool</td>
    <td>1</td>
  </tr>
  <tr>
    <th align="left">spamTimeout</th>
    <td align="left">time in seconds to wait before form gets rendered again if spam check fails</td>
    <td>int</td>
    <td>5</td>
  </tr>
  <tr>
    <th align="left">useHCaptcha</th>
    <td align="left">whether hCaptcha (https://www.hcaptcha.com/) should be used, needs further configuration</td>
    <td>bool</td>
    <td>0</td>
  </tr>
  <tr>
    <th align="left">hCaptchaSiteKey</th>
    <td align="left">hCaptcha site key, only if hCaptcha is used</td>
    <td>string</td>
    <td>10000000-ffff-ffff-ffff-000000000001</td>
  </tr>
  <tr>
    <th align="left">hCaptchaSecretKey</th>
    <td align="left">hCaptcha secret key, only if hCaptcha is used</td>
    <td>string</td>
    <td>0x0000000000000000000000000000000000000000</td>
  </tr>
  <tr>
    <th align="left">sendAdminInfo</th>
    <td align="left">whether the admin should get an info mail on every confirmation</td>
    <td>bool</td>
    <td>0</td>
  </tr>
  <tr>
    <th align="left">mailTemplateRootPath</th>
    <td align="left">path to the mail templates, root for different languages (e.g. en, de, dk)</td>
    <td>string</td>
    <td>EXT:newsletter_subscribe/Resources/Private/Templates/Mail/</td>
  </tr>
  <tr>
    <th align="left">mailLayoutRootPath</th>
    <td align="left">path to the mail layouts used from the templates</td>
    <td>string</td>
    <td>EXT:core/Resources/Private/Layouts/</td>
  </tr>
  <tr>
    <th align="left">overrideFlexformSettingsIfEmpty</th>
    <td align="left">Fields, which should be overridden from typosrcipt if left blank in the flexform (like in tx_news, thx to Georg Ringer!).</td>
    <td>string</td>
    <td>adminEmail, adminName, subscribePageUid, mailTemplateRootPath, dataProtectionPage, adminName, showFields, newsletterName</td>
  </tr>
</table>

## Site config (for nicer link in subscriber mails)

```yaml
routeEnhancers:
  UnSubscribe:
    type: Extbase
    extension: NewsletterSubscribe
    plugin: Unsubscribe
    routes:
      -
        routePath: '/unsubscribe/{unsubscribe}/{uid}'
        _controller: 'Subscribe::unsubscribe'
        _arguments:
          unsubscribe: subscriptionHash
          uid: uid
  Subscribe:
    type: Extbase
    extension: NewsletterSubscribe
    plugin: Subscribe
    routes:
      -
        routePath: '/confirm/create'
        _controller: 'Subscribe::createConfirmation'
      -
        routePath: '/confirm/{confirm}/{uid}'
        _controller: 'Subscribe::doConfirm'
        _arguments:
          confirm: subscriptionHash
          uid: uid
      -
        routePath: '/undosubscribe/{unsubscribe}/{uid}'
        _controller: 'Subscribe::undosubscribe'
        _arguments:
          unsubscribe: subscriptionHash
          uid: uid
```
## Unsubscribe link in direct_mail
1. First add the field subscription_hash to the fields of direct mail in the extension configuration of direct mail: 
![direct mail configuration](https://github.com/Gregor-Agnes/newsletter_subscribe/raw/master/Resources/Public/Gfx/ExtManDirectMail1.png)
2. Add the link in your mail template:\
`<a href="http://www.domain.tld/page/undosubscribe/###USER_subscription_hash###/###USER_uid###">unsubscribe</a>`
where this `undosubscribe/###USER_subscription_hash###/###USER_uid###"` is the important part.<br>Note: The subscribe plugin must be inserted on the page "page" in that url.

## Salutation in direct_mail
1. Add 'salutation' field (see above 'subscription_hash')
2. Add ###USER_salutation### on the page

## Scheduler Tasks / Console Commands

There are scheduler tasks / console commands available (TYPO3 v10 only) to fill empty database fields in `tt_address`:

- `newslettersubscribe:fillsalutation`  
  Updates the salutation field based on the sys_language_uid and gender fields of the tt_address records. The salutation can be configured via TypoScript.
- `newslettersubscribe:fillsubscriptionhash`  
  Updates the subscription_hash field. This is especially handy if there are subscriptions added manually in the TYPO3 backend or you have legacy data in tt_address. The subscription_hash is necessary for the unsubscribe link in direct_mail to work.

***

# To do
- creating ajax submit
- update documentation
