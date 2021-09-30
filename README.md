# Mail security checks

This repo contains a set of small scripts to identify how well domains
implement [email security controls](https://www.gov.uk/guidance/set-up-government-email-services-securely).

**These tools do not guarantee that the checked domains are compliant
with the standards in question.** It only provides an indication that
they _may_ support the standards.

Specifically, it provides tools to check for indications that the
domain supports:

- [TLS 1.2](https://www.gov.uk/government/publications/email-security-standards/transport-layer-security-tls)
  (checks live support, but doesn't check for absence of lower versions
  of TLS/SSL)
- [TLS Reporting (TLS-RPT)](https://www.gov.uk/government/publications/email-security-standards/using-tls-reporting-tls-rpt-in-your-organisation)
- [Mail Transfer Agent Strict Transport Security (MTA-STS)](https://www.digitalocean.com/community/tutorials/how-to-configure-mta-sts-and-tls-reporting-for-your-domain-using-apache-on-ubuntu-18-04)
- [Domain-based Message Authentication, Reporting and Conformance
  (DMARC)](https://www.gov.uk/government/publications/email-security-standards/domain-based-message-authentication-reporting-and-conformance-dmarc)
- [Sender Policy Framework](https://www.gov.uk/government/publications/email-security-standards/sender-policy-framework-spf)

It cannot check for [DomainKeys Identified Mail (DKIM)](https://www.gov.uk/government/publications/email-security-standards/domainkeys-identified-mail-dkim)
because [DKIM verification](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail#Verification)
depends on knowledge of message-specific parameters (notably, the
"selector") to identify the domain to check for DKIM compliance.

## Dependencies

The tools in this repository depend on having
[OpenSSL](https://www.openssl.org) and [GNU
Coreutils](https://www.gnu.org/software/coreutils) (specifically,
`timeout`) installed.

On a Mac with [Homebrew](https://brew.sh), you can install these by running:

```shell
brew install openssl coreutils
```

## `check-domain-security` command

The primary command is `check-domain-security`, which takes a single
domain name and returns, in CSV format, the primary `MX` hostname, TLS
connection response, and DNS values for TLS-RPT, SPF, DMARC, and MTA-STS:

```shell
$ check-domain-security digital.justice.gov.uk
"digital.justice.gov.uk","aspmx.l.google.com.","221 2.0.0 closing connection m17si5212918wmg.68 - gsmtp","""v=TLSRPTv1;rua=mailto:tls-rua@mailcheck.service.ncsc.gov.uk""","""v=spf1 include:_spf.google.com include:mail.zendesk.com include:sendgrid.net include:amazonses.com include:spf.protection.outlook.com include:spf.messagelabs.com ip4:54.194.156.23 ~all""","""v=DMARC1;p=none;rua=mailto:dmarc-rua@dmarc.service.gov.uk,mailto:dmarc@digital.justice.gov.uk;""",""
```

The CSV response has the following columns:

```csv
Requested domain, Primary MX host, TLS connection response, TLS-RPT, SPF, DMARC, MTA-STS
```

To check a large number of domains, we recommend creating a text file
containing one domain per line, and running the command using `xargs`
as follows:

```shell
$ cat input-domains.txt \
  | xargs -L1 -I{} check-domain-security {}
```

In this snippet, we read from `input-domains.txt`, then pipe (`|`) to
`xargs`, which calls `check-domain-security` once per line (`-L1`) with
the contents of the line as an argument (`-I{}` and `{}`).

## Other commands

`check-domain-security` uses a suite of other, smaller commands to
gather its data. Each command takes a single domain as their only
parameter. These are:

- `check-dmarc`: Gets the target domain's DMARC DNS record.
- `check-mta-sts`: Gets the target domain's MTA-STS DNS record. This
  **does not check for a valid MTA-STS policy** file.
- `check-spf`: Gets the target domain's SPF DNS record.
- `check-tls-1-2`: Takes a mail server domain (use `get-mx-hosts` to
  get this for your target domain) and attempts to open a TLS 1.2
  connection to it.
- `check-tls-rpt`: Gets the target domain's TLS-RPT record.
- `get-mx-hosts`: Gets the target domain's MX hosts in priority order.
