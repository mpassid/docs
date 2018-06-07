# MPASSid testipalveluun liittyminen

MPASSis palveluun liittyminen tarvitsee SAML2-pohjaisen asiakasohjelmiston. OIDC-tuki (OpenID Connect) on tulossa, mutta siitä enemmän tietoa syksyllä 2018. SAML2 toteutuksia on useita. Tässä ohjeessa käytetään Shiboleth Service Provider ohjelmistoa. Myöhemmin lyhennettynä SP. Myös MPASSid perustuu Shibboleth tuotteen päälle.

Tämä dokumentti on vain nopea asennusopas. Lisätietoja ja teknisempiä ohjeita löytyy täältä:
https://wiki.shibboleth.net/confluence/display/SHIB2/Home

Asennus
Tässä ohjeessa käydään läpi Shibboleth SP ohjelmiston asentaminen ja kytketään
yksinkertainen testisovellus osaksi MPASSid: testiympäristöä. 
Ohje toimii  Debian- tai Ubuntu -käyttöjärjestelmille ja käyttää valmiiksi paketoituja asennuspaketteja. 
Asenna Shibboleth SP seuraavalla komennolla.

```
apt-get install shibboleth-sp2-common libapache2-mod-shib2 apache2 shibboleth-sp2-utils
```

Seuraavaksi luodaan metadata ja tehdään shibbolethin peruskonfiguraatio. Metadatan avulla MPASSid proxy tunnistaa palvelun.

```
cd /etc/shibboleth 
shib-keygen
shib-metagen -2 -a FirsrName/LastName/admin.email@comppany.com -e https://vm1426.kaj.pouta.csc.fi/sptest -h vm1426.kaj.pouta.csc.fi > sptest-metadata.xml
```

Seuraavaksi käy hakemassa proxy palvelimen metadata, jotta asiakassovellus tunnistaa ja luottaa MPASSid-proxy palvelimeen.

```
wget -O mpass-proxy-test-metadata.xml https://mpass-proxy-test.csc.fi/idp/shibboleth
```

Loput konfiguraatiosta pitää tehdä käsin. Muokkaa shibboleth2.xml tiedostoa. 
Aluksi entityID tämä pitää olla sama kuin shib-metagen komennon -e vivulla annettu.

```
<ApplicationDefaults entityID="https://vm1426.kaj.pouta.csc.fi/sptest"
                     REMOTE_USER="eppn persistent-id targeted-id">
```

Seuraavaksi annetaan proxy palvelimen osoite, sekä siihen liittyviä tietoja

```
<SSO entityID="https://mpass-proxy-test.csc.fi/idp/shibboleth"
     discoveryProtocol="SAMLDS" discoveryURL="https://mpass-proxy-test.csc.fi/discovery/index.html">
     SAML2 SAML1
</SSO>
```

Lisää locally maintained metadata kohdan alle luomasi tunnistetiedot, sekä MPASSid proxyn metadata

```
<!-- Example of locally maintained metadata. -->
<MetadataProvider type="XML" validate="false" file=”sptest-metadata.xml"/>
<MetadataProvider type="XML" validate="false" file="mpass-proxy-test-metadata.xml"/>
```

Shibboleth2.xml on nyt valmis. Seuraavaksi  muokkaa tiedostoa attribute-map.xml. Tässä tiedostossa kerrotaan ne attribuutit joita sovelluksesi tarvitsee. Tarjolla olevat attribuutit löytyvät  löytyvät MPASSid tietomallista https://github.com/Digipalvelutehdas/MPASSid-proxy/wiki#data-model-10

```
<Attribute name="urn:mpass.id:municipality" id="MPASS-10-municipality"/>
<Attribute name="urn:mpass.id:municipalityCode" id="MPASS-10-municipalityCode"/>
<Attribute name="urn:mpass.id:legacyCryptId" id="MPASS-10-CID"/>
<Attribute name="urn:mpass.id:legacyCryptIde" id="MPASS-10-CIDe"/>
<Attribute name="urn:mpass.id:uid" id="MPASS-10-MPASSUID"/>
<Attribute name="urn:mpass.id:school" id="MPASS-10-school"/>
<Attribute name="urn:mpass.id:schoolCode" id="MPASS-10-schoolCode"/>
<Attribute name="urn:mpass.id:class" id="MPASS-10-class"/>
<Attribute name="urn:mpass.id:role" id="MPASS-10-role"/>
```

Sovelluksen testaaminen onnistuu seuraavasti. Teidän pitää aluksi konfiguroida Apache hakemisto, jonka haluatte suojata Shibbolethillla. 

```
<Location /sptest/>
AuthType shibboleth
ShibRequestSetting requireSession On
ShibRequestSetting exportAssertion On
ShibUseEnvironment Off
ShibUseHeaders On
Require valid-user
</Location>
```

Sovelluksen toimintaa ja proxyn välittämiä attribuutteja voi testata esimerkiksi seuraavalla yksinkertaisella PHP sovelluksella.

```
<html>
<head/>

<body>
<?php

foreach (getallheaders() as $name => $value) {
    echo "$name: $value<br>";
}

?>
</body>
</html>
```

Aina kun muutatte konfiguraatiota muistakaa käynnistää sovellukset välillä uudestaan 

```
systemctl restart apache2
systemctl status  apache2 -l

systemctl restart shibd
systemctl status  shibd -l
```

MPASSid proxy ei luota  sovellukseen ennen kun sen metadata on asetettu paikoilleen.  
Metadata on julkista tietoa ja se on saatavissa URL:n https://vm1426.kaj.pouta.csc.fi/Shibboleth.sso/Metadata kautta.

Kun olette valmiit liittymään MPASSid lähettäkää postia tuki@mpass.fi osoitteeseen. Liittäkää sähköpostiin

* Teknisen ja hallinnollisen henkilön yhteystiedot
* Lyhyt kuvaus palvelusta
* Yllä kuvattu URL josta ylläpito voi hakea palvelunne metadatan

Tervetuloa MPASS:iin !

