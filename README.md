# Email server setup script

This script installs an email server on Void Linux with all the features 
required in the modern web.

Special thanks to [Luke Smith](https://alexscarter.com) for the majority of
this script which I have only modified to work on Void instead of Debian.

To start, simply run

```sh
curl -LO alexscarter.com/emailwiz-void.sh
```

When prompted by a dialog menu at the beginning, select "Internet Site", then
give your full domain without any subdomain, e.g. `alexscarter.com`.

## This script installs

- **Postfix** to send and receive mail.
- **Dovecot** to get mail to your email client (mutt, Thunderbird, etc.).
- Config files that link the two above securely with native PAM log-ins.
- **Spamassassin** to prevent spam and allow you to make custom filters.
- **OpenDKIM** to validate you so you can send to Gmail and other big sites.
- **Certbot** SSL certificates, if not already present.
- **fail2ban** to increase server security, with enabled modules for the above
  programs.
- **Socklog** to log info and any errors on your mail server, as Void doesn't install with syslog daemon.
- **Snooze** to run commands at a specific time. Like cron but simpler for the runit init system.
- (optionally) **a self-signed certificate** instead of OpenDKIM and Certbot. This allows to quickly set up an isolated mail server that collects email notifications from devices in the same local network(s) or serves as secure/private messaging system over VPN.

## This script does _not_...

- use a SQL database or anything like that. We keep it simple and use normal
  Unix system users for accounts and passwords.
- set up a graphical web interface for mail like Roundcube or Squirrel Mail.
  You are expected to use a normal mail client like Thunderbird or K-9 for
  Android or good old mutt with
  [mutt-wizard](https://github.com/lukesmithxyz/mutt-wizard). Note that there
  is a guide for [Rainloop](https://landchad.net/rainloop/) on
  [LandChad.net](https://landchad.net) for those that want such a web
  interface.

## Prerequisites for Installation

1. Void Linux server.
2. DNS records that point at least your domain's `mail.` subdomain to your
   server's IP (IPv4 and IPv6). This is required on initial run for certbot to
   get an SSL certificate for your `mail.` subdomain.

## Mandatory Finishing Touches

### Unblock your ports

While the script enables your mail ports on your server, it is common practice
for all VPS providers to block mail ports on their end by default. Open a help
ticket with your VPS provider asking them to open your mail ports and they will
do it in short order.

(Optional) Open an [SMTP2Go](https://www.smtp2go.com/) account if you are running 
your server in a home network and can't get rDNS or PTR records changed. VPS servers 
don't need this.

### DNS records

At the end of the script, you will be given some DNS records to add to your DNS
server/registrar's website. These are mostly for authenticating your emails as
non-spam. The 4 records are:

1. An MX record directing to `mail.yourdomain.tld`.
2. A TXT record for SPF (to reduce mail spoofing).
3. A TXT record for DMARC policies.
4. A TXT record with your public DKIM key. This record is long and **uniquely
   generated** while running `emailwiz.sh` and thus must be added after
   installation.

They will look something like this:

```
@	MX	10	mail.example.org
mail._domainkey.example.org    TXT     v=DKIM1; k=rsa; p=anextremelylongsequenceoflettersandnumbersgeneratedbyopendkim
_dmarc.example.org     TXT     v=DMARC1; p=reject; rua=mailto:dmarc@example.org; fo=1
example.org    TXT     v=spf1 mx a: -all
```

The script will create a file, `~/dns_emailwiz` that will list our the records
for your convenience, and also prints them at the end of the script.

### Add a rDNS/PTR record as well! (unless using SMTP2Go)

Set a reverse DNS or PTR record to avoid getting spammed. You can do this at
your VPS provider, and should set it to `mail.yourdomain.tld`. Note that you
should set this for both IPv4 and IPv6.

## Making new users/mail accounts

Let's say we want to add a user Billy and let him receive mail, run this:

```
useradd -m -G mail billy
passwd billy
```

Any user added to the `mail` group will be able to receive mail. Suppose a user
Cassie already exists and we want to let her receive mail too. Just run:

```
usermod -a -G mail cassie
```

A user's mail will appear in `~/Mail/`. If you want to see your mail while ssh'd
in the server, you could just install mutt, add `set spoolfile="+Inbox"` to
your `~/.muttrc` and use mutt to view and reply to mail. You'll probably want
to log in remotely though:

## Installing with self-signed certificate, in "isolated" mode

This mode skips the setup of OpenDKIM and Certbot, and will instead create a self-signed cert that lasts 100 years. It also allows to customize the logic country name, state/province name and organization name to generate the certificate automatically. An example usecase is for an isolated server that collects notifications from devices in the same local network(s) or serves as secure/private messaging system over VPN (wireguard or whatever).
This server with self-signed certificate as configured will NOT be able to send anything to public mail servers (Gmail, Outlook and so on), at least not directly.

open the script and change the following line 
```
selfsigned="no" # yes no
```
to become 
```
selfsigned="yes" # yes no
```
it's also possible to customize and automate the self-signed certificate creation
by changing the following lines in the script 
```
use_cert_config="no"
```
to
```
use_cert_config="yes"
```

and then write country name, state/province name and organization name in the following lines
```
country_name="" # IT US UK IN etc etc
state_or_province_name=""
organization_name=""
```

## Logging in from email clients (Thunderbird/mutt/etc)

Let's say you want to access your mail with Thunderbird or mutt or another
email program. For my domain, the server information will be as follows:

- SMTP server: `mail.alexscarter.com`
- SMTP port: 465
- IMAP server: `mail.alexscarter.com`
- IMAP port: 993

## Benefited from this?

I'm thankful to have a script showing the necessary steps to set up
a server. I'm proud to now make it easily available on Void! If this
helped you, donate here:

- btc: `bc1q8zkytdfxh7hqpa0y0nel28hyga44wgg4dxma2r`
- xmr: `46zAjvGynLL9b8EVq7bmCdciyEZGjyGME2iU2aAxep14gVYtmsqMmiD4ogZkQgxiudVCL5ojdJFuB9qJ3hR5CoH9313uWoV`

Or visit [Luke's original script](https://github.com/LukeSmithxyz/emailwiz) to donate to him.

## Sites for Troubleshooting

Can't send or receive mail? Getting marked as spam? There are tools to double-check your DNS records and more:

- Always check `tail -f /var/log/socklog/mail/current` first for specific errors.
- This script creates a configuration file for Dovecot >= 2.4
- [Check your DNS](https://intodns.com/)
- [Test your TXT records via mail](https://appmaildev.com/en/dkim)
- [Is your IP blacklisted?](https://mxtoolbox.com/blacklists.aspx)
- [mxtoolbox](https://mxtoolbox.com/SuperTool.aspx)
