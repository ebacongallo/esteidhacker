# EstEID hacker &nbsp; [![Build Status](https://travis-ci.org/martinpaljak/esteidhacker.svg?branch=master)](https://travis-ci.org/martinpaljak/esteidhacker) [![Coverity status](https://scan.coverity.com/projects/8551/badge.svg?flat=1)](https://scan.coverity.com/projects/martinpaljak-esteidhacker)  [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.martinpaljak/esteid/badge.svg)](https://mvnrepository.com/artifact/com.github.martinpaljak/esteid) [![Latest release](https://img.shields.io/github/release/martinpaljak/esteidhacker/all.svg)](https://github.com/martinpaljak/esteidhacker/releases/latest) [![LGPL-3.0 licensed](https://img.shields.io/badge/license-LGPL-blue.svg)](https://github.com/martinpaljak/esteidhacker/blob/master/LICENSE)

Java utility and source code for everything and anything related to [EstEID](https://esteid.org):

* [EstEID.java - host API](#esteidjava)
* [FakeEstEID.java](src/org/esteid/hacker/FakeEstEID.java) - utility for working with a [FakeEstEIDApplet](https://github.com/martinpaljak/AppletPlayground/wiki/FakeEstEID) instance. Supports emulation inside [vJCRE](https://github.com/martinpaljak/vJCRE#import-projavacardvre).
* [FakeEstEIDCA](#fakeesteidca)
* [CLI.java](src/org/esteid/hacker/CLI.java) - code of the command line utility that serves as usage documentation.

## Usage
* Fetch and build the software (requires Unix-like OS)

        git clone https://github.com/martinpaljak/esteidhacker.git
        cd esteidhacker
        ant

* In this README `esteid` is used as an alias for `java -jar esteid-app.jar`. `esteid.exe` can be used on Windows.

### Emulation
* Create a new FakeEstEID card

        esteid -install -ca fake.ca -new

* Run EstEID test-suite against a real card (via PC/SC):

        $ esteid -info -test-crypto -pin1 XXXX -pin2 YYYYY 
        ATR:  3BFE1800008031FE45803180664090A4162A00830F9000EF
        Type: JavaCard2011
        PIN tries remaining: PIN1: 3; PIN2: 3; PUK: 1;
        Doc#: AA0448165
        Cardholder: MARTIN PALJAK
        Certificate subject: C=EE,O=ESTEID,OU=authentication,CN=PALJAK\,MARTIN\,38207162722,SURNAME=PALJAK,GIVENNAME=MARTIN,SERIALNUMBER=38207162722
        Auth cert C=EE,O=ESTEID,OU=authentication,CN=PALJAK\,MARTIN\,38207162722,SURNAME=PALJAK,GIVENNAME=MARTIN,SERIALNUMBER=38207162722
        ENCRYPT: OK
        DECRYPT: OK
        Sign cert C=EE,O=ESTEID,OU=digital signature,CN=PALJAK\,MARTIN\,38207162722,SURNAME=PALJAK,GIVENNAME=MARTIN,SERIALNUMBER=38207162722
        ENCRYPT: OK


* Run EstEID test-suite against a real test Digi-ID card (via PC/SC):

        $ esteid -info -test
        ATR:  3BFE9400FF80B1FA451F034573744549442076657220312E3043
        Type: DigiID
        PIN tries remaining: PIN1: 3; PIN2: 3; PUK: 3;
        Doc#: N0000952
        Certificate subject: C=EE,O=ESTEID (DIGI-ID),OU=authentication,CN=ŽAIKOVSKI\,IGOR\,37101010021,SURNAME=ŽAIKOVSKI,GIVENNAME=IGOR,SERIALNUMBER=37101010021
        Auth cert C=EE,O=ESTEID (DIGI-ID),OU=authentication,CN=ŽAIKOVSKI\,IGOR\,37101010021,SURNAME=ŽAIKOVSKI,GIVENNAME=IGOR,SERIALNUMBER=37101010021
        ENCRYPT: OK
        DECRYPT: OK
        Sign cert C=EE,O=ESTEID (DIGI-ID),OU=digital signature,CN=ŽAIKOVSKI\,IGOR\,37101010021,SURNAME=ŽAIKOVSKI,GIVENNAME=IGOR,SERIALNUMBER=37101010021
        ENCRYPT: OK

* Run EstEID test-suite against an emulated card (read: test the FakeEstEIDApplet):
        
        $ esteid -emulate -info -test
        ATR:  3B80800101
        Type: AnyJavaCard
        PIN tries remaining: PIN1: 3; PIN2: 3; PUK: 3;
        Doc#: A0000001
        Cardholder: SIILIPOISS JÄNES-KARVANE
        Certificate subject: C=EE,O=ESTEID,OU=authentication,CN=UDUS\,SIILIPOISS\,10101010005,SURNAME=UDUS,GIVENNAME=SIILIPOISS,SERIALNUMBER=10101010005
        Auth cert C=EE,O=ESTEID,OU=authentication,CN=UDUS\,SIILIPOISS\,10101010005,SURNAME=UDUS,GIVENNAME=SIILIPOISS,SERIALNUMBER=10101010005
        ENCRYPT: OK
        DECRYPT: OK
        Sign cert C=EE,O=ESTEID,OU=digital signature,CN=UDUS\,SIILIPOISS\,10101010005,SURNAME=UDUS,GIVENNAME=SIILIPOISS,SERIALNUMBER=10101010005
        ENCRYPT: OK


* Clone a card

        $ esteid -clone


### Personalization
        $ esteid -perso testcard.conf -install # load the applet
        $ esteid -perso testcard.conf -data # store personal data file
        $ esteid -perso testcard.conf -genauth # generate authentication key ...
        $ esteid -perso testcard.conf -genauth -ca fake.ca # or generate key and load a certificate with the fake CA
        $ esteid -perso testcard.conf -gensign # generate signature key ...
        $ esteid -perso testcard.conf -gensign -ca fake.ca # or generate key and load a certificate with the fake CA
        # If certificates are generated elsewhere ...
        $ esteid -perso testcard.conf -authcert auth.pem # load authentication certificate from auth.pem
        $ esteid -perso testcard.conf -authcert sign.pem # load signature certificate from sign.pem
        $ esteid -perso testcard.conf -finalize # finalize personalization
        # All of the previous in one run
        $ esteid -perso testcard.conf -new -ca fake.ca
        # Be sure to specify the right CMK!
        $ esteid -cmk 1 -key XX..XX -loadpins -pin1 0090 -pin2 01497 -puk 17258403 # does not require PIN1
        $ esteid -cmk 2 -key XX..XX -genauth -pin1 0090 # generate new authentication key, requires PIN1
        $ esteid -cmk 2 -key XX..XX -gensign -pin1 0090 # generate new signature key, requires PIN1
        $ esteid -cmk 3 -key xx..XX -authcert auth.pem -pin1 0090 # load new authentication signature, requires PIN1
        $ esteid -cmk 3 -key xx..XX -authcert auth.pem -pin1 0090 # load new authentication signature, requires PIN1
        # Print the CA PEM files for the fake CA
        $ esteid -ca fake.ca -dump

## Dependencies
* [GlobalPlatformPro](https://github.com/martinpaljak/GlobalPlatformPro) (LGPL)
* [apdu4j](https://github.com/martinpaljak/apdu4j) (MIT)
* [FakeEstEIDApplet](https://github.com/martinpaljak/AppletPlayground/wiki/FakeEstEID) from [AppletPlayground](https://github.com/martinpaljak/AppletPlayground#applet-playground) (MIT)
* [vJCRE](https://github.com/martinpaljak/vJCRE#import-projavacardvre) (LGPL)
* [BouncyCastle](bouncycastle.org/java.html) JCE provider + PKIX (MIT)

## License
Mixed LGPL/MIT, please check individual files! Other options available upon request.

## Contact
* martin@martinpaljak.net

## Upcoming features
* Planned:
  * Pinpad support for PC/SC readers
* Wishlist:
  * A GUI maybe, not unlike qesteidutil?

## Components

### EstEID.java

Encapsulates all the APDU protocol knowledge and exposes high-level and meaningful API for making operations with the card (more precisely: on-card application). Can talk to any PC/SC terminal or somethig else exposed via `javax.smartcardio` CommandAPDU/ResponseAPDU pairs.

Exception handling:
* [CardException](https://docs.oracle.com/javase/8/docs/jre/api/security/smartcardio/spec/javax/smartcardio/CardException.html) when javax.smartcardio classes fail
* EstEIDException when card protocol (or data formats) have unexpected situations
* WrongPINException when a passed in PIN is incorrect

Source: [EstEID.java](src/org/esteid/EstEID.java)

#### Similar projects
* https://github.com/sleepless/jesteid
  * Very verbose but educating to read. Parses a lot of data.
* http://blog.codeborne.com/2010/10/javaxsmartcardio-and-esteid.html
  * Simple sample on how to read personal data file.
* https://eid.eesti.ee/index.php/Sample_applications#Claims_application
  * Complete "ecosystem" sample but not easily re-usable.
* [esteid.c](https://github.com/martinpaljak/esteid.c)
  * :( not yet in code  
* [MOCCA](https://www.egiz.gv.at/en/schwerpunkte/9-MOCCA) - [EstEIDCard.java](https://joinup.ec.europa.eu/svn/mocca/trunk/smcc/src/main/java/at/gv/egiz/smcc/EstEIDCard.java) http://git.egiz.gv.at/mocca/
  * :) Java
  * :| ... messy

### FakeEstEIDCA
Utility for maintaining a [SK](http://www.sk.ee) look-alike CA for [EstEID related certificates](https://www.sk.ee/repositoorium/sk-sertifikaadid/) (root, esteid, user auth/sign) based on BouncyCastle.

Source: [FakeEstEIDCA.java](src/esteidhacker/FakeEstEIDCA.java)

#### Similar and related projects
* [Metasploit SSL imersonation module](http://www.rapid7.com/db/modules/auxiliary/gather/impersonate_ssl) 
  * Instead of generic impersonator implements a specific profile. Possible to programmatically change the profile.
* `sk` utility in [python-esteid](https://github.com/martinpaljak/python-esteid)

----
All about the [EstEID](https://esteid.org)
