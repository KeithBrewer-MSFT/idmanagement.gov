---
layout: page
collection: implement
title: Configure Smart Card Login in Okta
pubdate: 2024--03
type: Markdown
permalink: /implement/cba-okta/
description: This guide is a walkthrough to configure PIV Smart Card Login for Okta
sidenav: implement
sticky_sidenav: true

subnav:
  - text: Prerequisites
    href: '#prerequisites'
  - text: Technology and terms
    href: '#technology-and-terms'
  - text: Prepare users for smart card login
    href: '#prepare-users-to-use-smart-cards'
  - text: Certificate chain configuration
    href: '#Certificate-chain-configuration'
  - text: Unique identifier configuration
    href: '#unique-identifier-configuration'
  - text: Okta user experience
    href: '#okta-user-experience'
  - text: Troubleshooting
    href: '#troubleshooting'
  - text: Resources
    href: '#resources'

---

This guide is a walkthrough to configure an Okta Identity Engine for multi-agency smart card login for Personal Identity Verification (PIV), Department of Defense (DoD) Common Access Card (CAC), or PIV Interoperable (PIV-I) 

## Prerequisites
1. Access Control: Ensure you have administrative access to Okta's admin console.
2. Determine Community of Users: Determine the types of federally-governed smart cards to accept. This may include PIV from multiple issuers, CAC, and PIV-I.
3. Certificate Chains: Obtain the Public Key Infrastructure (PKI) certificate chains related to community of users. 
4. Documentation: Familiarize yourself with Okta's official documentation on [smart card Identity Provider (IdP) connectors](https://help.okta.com/oie/en-us/content/topics/security/idp-smart-card-workflow.htm){:target="_blank"}{:rel="noopener noreferrer"}{:class="usa-link usa-link--external"}.

## Design Considerations
There are two design approaches to consider.
1. PKI certificate chain bundle.
2. Unique identifier.

### PKI certificate chain bundle
Okta requires a PEM formated certificate bundle and also recommends
1. A single certificate chain of all certificates.
2. One certificate chain per smart card issuer. For example:
- Chain 1 - US Access PIV (Entrust)
- Chain 2 - DoD CAC (DoD)
- Chain 3 - Treasury PIV & PIV-I (Treasury)

### Unique identifier


## Configuration steps
There are two primary steps to configure Okta smart card login.

### Step 1. Format a PKI certificate chain
1. Convert DER encoded root and intermediate certificates (with .cer, .crt extension) into PEM format using the following OpenSSL command.
```
openssl x509 -inform der -in $input-cert-file-name -out $out-cert-file-name-with-pem-extension
```

2. Concatenate all the PEM certificates into a single file with root certificate being the last one using the following command.
```
cat $intermediate-cert-file-1 ...$intermediate-cert-file-N $root-cert-file-with-pem-extension > smart-card-certs-chain.pem
```
**NOTE**: Ensure the root certificate is last in the sequence.

3. Upload smart-card-certs- chain.pem when creating the Smart Card Identity Provider.

### Step 2. Add a smart card IdP
1. Navigate to Security > Identity Providers in the Okta Admin Console.
2. Click on Add Identity Provider and select the Smart Card option. Create a name for this new IdP configuration.
3. Build a certificate chain by uploading the root and intermediate certificates. Ensure the Federal Common Policy trust anchor and sub CA certificates and all applicable intermediate and issuer certificates incorproated into a PEM encoded certificate chain.
    a. Click Browse to open a file explorer. Select the certificate file you want to add and click Open.
    b. To add another certificate, click Add Another, and repeat step 1.
    c. Click Build certificate chain. On success, the chain and its certificates are shown. If the build fails, correct any issues and try again.
    d. Click Reset certificate chain to replace the current chain with a new one.
4. Validate Certificate Chains:
    a. Enable Certificate Chain Validation in the smart card IdP settings.
    b. Arrange the certificates hierarchically (root > intermediate) to form the chain.
    c . Validate the certificate chain to ensure it is correctly formed and functional.
5. Select the attribute to locate the Okta user from the IdP username dropdown list. Verify the [current availabe attributes available from Okta](https://help.okta.com/en-us/content/topics/security/idp-enable-smart-card.htm){:target="_blank"}{:rel="noopener noreferrer"}{:class="usa-link usa-link--external"}
6. Choose the value Okta should Match against: Okta Username, Email, or Okta Username or Email.
    - For a user to sign in to Okta, they must have an existing Okta account, and that account's Okta username or email address must match the attribute or expression defined by IdP username.
    - The Okta Username or Email match option isn't available if the IDP Extensible Matching feature is enabled. Instead, Okta matches against a custom attribute that you choose from the dropdown list.
7. Click Finish. The system is configured to accept the smart card IdP as an alternate form of authentication.

Additional Steps
1. **Configure Matching Rules**
    - Within the Smart Card IdP Settings, go to Mapping Rules.
    - Configure the rules to map certificate attributes to Okta user attributes. This is crucial for authentication workflows.
2. **Attribute Extraction**
    - Specify which attributes (e.g., Common Name, Organization) will be extracted from the smart card for authentication.
4. **Enable Checks for Certificate Revocation**
    - Configure the Certificate Revocation List (CRL) or Online Certificate Status Protocol (OCSP) settings within Okta.
    - Ensure that Okta can reach the CRL or OCSP responders for each type of smart card.
5. **Apply and Test Policies**
    - In the Sign-On Policy, add a rule to enforce smart card authentication.
    - Conduct pilot testing with various smart cards (PIV, CAC, PIV-I) to ensure that the certificate chains are correctly recognized and validated.
6. **Setup Monitoring and Logging**
    - Make sure to enable logging for smart card authentication.
    - Regularly review logs to ensure that certificate chain validation occurs as expected.
7. **Testing and Validation**
    - Before the smart card configuration is put into production, conduct testing to validate entire certificate chains in a test and integration environment.

## Okta user experience

## Troubleshooting
There are four errors a user or administration may experience.
1. Unique identifier / subject name matching: Ensure that the Subject Alternate Name or expression result matches the Okta attribute you specified. **It must be either email or Okta username**.
2. Certificate Chain: Ensure that the entire certificate chain of issuers is uploaded in the correct format.
3. CRL or OCSP endpoint is not accessible / 401 error: Okta requires all CRL or OCSP endpoints to be publicly accessible. Typically, Certificate Revocation Lists are posted in a publicly reachable HTTP (non-HTTPS) location on the internet, but in some highly secure environments, the revocation endpoints aren't public. **All PIV, CAC, and PIV-I require publicly accessible revocation checking methods**.
4. User account state: Ensure that the user has an account in an active state. Password reset is considered active.
5. Browser session: Always start with a brand new browser session to avoid caching issues. Close out all browser windows before testing the feature.

## Resources
1. [Okta - Add a Smart Card IdP](https://help.okta.com/oie/en-us/content/topics/security/idp-smart-card-workflow.htm){:target="_blank"}{:rel="noopener noreferrer"}{:class="usa-link usa-link--external"}
2. [Okta - Troubleshooting smart card/PIV authentication](https://help.okta.com/oie/en-us/content/topics/security/idp-troubleshooting-smart-card.htm){:target="_blank"}{:rel="noopener noreferrer"}{:class="usa-link usa-link--external"}
